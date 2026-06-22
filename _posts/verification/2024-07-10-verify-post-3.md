---
layout:       post
title:        'UVM 验证方法学：架构解析与完整环境搭建'
date:         2024-07-10
header-style: text
catalog:      true
mathjax:      true
tags:
  - 芯片验证
  - UVM
  - SystemVerilog
  - 核心框架
---

> UVM（Universal Verification Methodology）是业界标准的芯片验证框架，基于 SystemVerilog 构建，提供了完整的可重用验证基础设施。

# 一、UVM 整体架构

```
┌────────────────────────────────────────────────────┐
│                      Test                          │
├──────────────┬─────────────────────────────────────┤
│              │           Environment               │
│   Scoreboard │   ┌─────────────┐  ┌─────────────┐ │
│              │   │   Agent 0   │  │   Agent 1   │ │
│   Coverage   │   │ ┌─────────┐ │  │ ┌─────────┐ │ │
│   Collector  │   │ │Sequencer│ │  │ │Sequencer│ │ │
│              │   │ ├─────────┤ │  │ ├─────────┤ │ │
│              │   │ │ Driver  │ │  │ │ Driver  │ │ │
│              │   │ ├─────────┤ │  │ ├─────────┤ │ │
│              │   │ │Monitor  │ │  │ │Monitor  │ │ │
│              │   └─────────────┘  └─────────────┘ │
└──────────────┴─────────────────────────────────────┘
                          ↕ virtual interface
                   ┌────────────────────┐
                   │      DUT (RTL)     │
                   └────────────────────┘
```

---

# 二、UVM 核心组件详解

## 2.1 uvm_sequence_item（事务对象）

表示一次硬件操作的数据载体：

```systemverilog
class apb_seq_item extends uvm_sequence_item;
    `uvm_object_utils(apb_seq_item)
    
    rand bit [31:0] addr;
    rand bit [31:0] data;
    rand bit        rw;      // 0=read, 1=write
    rand bit [3:0]  strb;    // 字节使能
    
    constraint c_align { addr[1:0] == 2'b00; }
    
    function new(string name = "apb_seq_item");
        super.new(name);
    endfunction
    
    // 必须实现：用于打印调试信息
    function string convert2string();
        return $sformatf("addr=%0h data=%0h rw=%0b", addr, data, rw);
    endfunction
endclass
```

## 2.2 uvm_driver（驱动器）

从 Sequencer 获取 sequence_item，通过虚接口驱动 DUT：

```systemverilog
class apb_driver extends uvm_driver #(apb_seq_item);
    `uvm_component_utils(apb_driver)
    
    virtual apb_if.driver_mp vif;
    
    function new(string name, uvm_component parent);
        super.new(name, parent);
    endfunction
    
    function void build_phase(uvm_phase phase);
        super.build_phase(phase);
        // 从配置数据库获取虚接口
        if (!uvm_config_db #(virtual apb_if)::get(
                this, "", "vif", vif))
            `uvm_fatal("NO_VIF", "Virtual interface not found")
    endfunction
    
    task run_phase(uvm_phase phase);
        apb_seq_item req;
        forever begin
            seq_item_port.get_next_item(req);  // 从 Sequencer 取 item
            drive_item(req);                    // 驱动到 DUT
            seq_item_port.item_done();          // 通知完成
        end
    endtask
    
    task drive_item(apb_seq_item req);
        @(vif.driver_cb);
        vif.driver_cb.paddr  <= req.addr;
        vif.driver_cb.pwdata <= req.data;
        vif.driver_cb.pwrite <= req.rw;
        vif.driver_cb.psel   <= 1;
        @(vif.driver_cb);
        vif.driver_cb.penable <= 1;
        @(vif.driver_cb iff vif.driver_cb.pready);
        if (!req.rw)
            req.data = vif.driver_cb.prdata;  // 读操作取回数据
        vif.driver_cb.psel    <= 0;
        vif.driver_cb.penable <= 0;
    endtask
endclass
```

## 2.3 uvm_monitor（监视器）

被动监控接口信号，采集实际行为并广播：

```systemverilog
class apb_monitor extends uvm_monitor;
    `uvm_component_utils(apb_monitor)
    
    virtual apb_if.monitor_mp vif;
    uvm_analysis_port #(apb_seq_item) ap;  // 广播端口
    
    function void build_phase(uvm_phase phase);
        super.build_phase(phase);
        ap = new("ap", this);
        uvm_config_db #(virtual apb_if)::get(this, "", "vif", vif);
    endfunction
    
    task run_phase(uvm_phase phase);
        apb_seq_item txn;
        forever begin
            // 等待一次完整的 APB 传输
            @(vif.monitor_cb iff (vif.monitor_cb.psel && 
                                   vif.monitor_cb.penable && 
                                   vif.monitor_cb.pready));
            txn = apb_seq_item::type_id::create("txn");
            txn.addr = vif.monitor_cb.paddr;
            txn.rw   = vif.monitor_cb.pwrite;
            txn.data = (vif.monitor_cb.pwrite) ?
                        vif.monitor_cb.pwdata : vif.monitor_cb.prdata;
            ap.write(txn);  // 广播到 Scoreboard 和 Coverage
        end
    endtask
endclass
```

## 2.4 uvm_sequence（激励序列）

生成并发送 sequence_item 的逻辑容器：

```systemverilog
// 基础序列：随机读写
class apb_rand_seq extends uvm_sequence #(apb_seq_item);
    `uvm_object_utils(apb_rand_seq)
    
    int num_txns = 20;
    
    function new(string name = "apb_rand_seq");
        super.new(name);
    endfunction
    
    task body();
        apb_seq_item txn;
        repeat(num_txns) begin
            txn = apb_seq_item::type_id::create("txn");
            start_item(txn);
            assert(txn.randomize());
            finish_item(txn);
        end
    endtask
endclass

// 定向序列：覆盖边界条件
class apb_boundary_seq extends uvm_sequence #(apb_seq_item);
    `uvm_object_utils(apb_boundary_seq)
    
    task body();
        apb_seq_item txn;
        // 写地址 0
        `uvm_do_with(txn, {addr == 32'h0; rw == 1;})
        // 写最大地址
        `uvm_do_with(txn, {addr == 32'hFFFF_FFFC; rw == 1;})
        // 读回验证
        `uvm_do_with(txn, {addr == 32'h0; rw == 0;})
        `uvm_do_with(txn, {addr == 32'hFFFF_FFFC; rw == 0;})
    endtask
endclass
```

## 2.5 uvm_scoreboard（记分板）

比较 DUT 实际输出与期望值：

```systemverilog
class apb_scoreboard extends uvm_scoreboard;
    `uvm_component_utils(apb_scoreboard)
    
    uvm_analysis_imp #(apb_seq_item, apb_scoreboard) ap;
    
    // 参考模型（内存模型）
    bit [31:0] ref_mem[bit [31:0]];
    int pass_cnt, fail_cnt;
    
    function void build_phase(uvm_phase phase);
        super.build_phase(phase);
        ap = new("ap", this);
    endfunction
    
    // Monitor 广播时调用
    function void write(apb_seq_item txn);
        if (txn.rw) begin
            // 写操作：更新参考模型
            ref_mem[txn.addr] = txn.data;
        end else begin
            // 读操作：比较实际值与期望值
            if (ref_mem.exists(txn.addr)) begin
                if (txn.data === ref_mem[txn.addr]) begin
                    pass_cnt++;
                    `uvm_info("SCB", $sformatf("PASS: addr=%0h", txn.addr), UVM_HIGH)
                end else begin
                    fail_cnt++;
                    `uvm_error("SCB", $sformatf(
                        "FAIL: addr=%0h exp=%0h got=%0h",
                        txn.addr, ref_mem[txn.addr], txn.data))
                end
            end
        end
    endfunction
    
    function void report_phase(uvm_phase phase);
        `uvm_info("SCB", $sformatf("PASS=%0d FAIL=%0d", pass_cnt, fail_cnt), UVM_NONE)
    endfunction
endclass
```

---

# 三、UVM Phase 机制

UVM 通过 **Phase** 管理组件的生命周期：

| Phase | 类型 | 说明 |
|-------|------|------|
| build_phase | function | 创建子组件，配置参数 |
| connect_phase | function | 连接端口（TLM port/export）|
| start_of_simulation_phase | function | 打印拓扑信息 |
| run_phase | task | 主仿真逻辑（可并行）|
| extract_phase | function | 提取结果 |
| check_phase | function | 检查错误 |
| report_phase | function | 打印报告 |
| final_phase | function | 最终清理 |

---

# 四、uvm_config_db 配置数据库

```systemverilog
// 顶层 testbench 设置虚接口
initial begin
    uvm_config_db #(virtual apb_if)::set(
        null,           // context（null = 全局）
        "uvm_test_top.env.agent.*",  // 路径通配符
        "vif",          // 参数名
        apb_if_inst     // 值
    );
    run_test();
end

// 组件内部获取
uvm_config_db #(virtual apb_if)::get(this, "", "vif", vif);
```

---

# 五、典型目录结构

```
tb/
├── top/
│   └── tb_top.sv          # 顶层 module（例化 DUT 和接口）
├── tests/
│   ├── base_test.sv
│   ├── rand_test.sv
│   └── directed_test.sv
├── env/
│   └── apb_env.sv
├── agent/
│   ├── apb_agent.sv
│   ├── apb_driver.sv
│   ├── apb_monitor.sv
│   └── apb_sequencer.sv
├── seq_lib/
│   ├── apb_base_seq.sv
│   ├── apb_rand_seq.sv
│   └── apb_boundary_seq.sv
├── ref_model/
│   └── apb_scoreboard.sv
├── coverage/
│   └── apb_coverage.sv
└── seq_items/
    └── apb_seq_item.sv
```

---

# 六、运行命令（VCS）

```bash
# 编译
vcs -sverilog -ntb_opts uvm-1.2 \
    -f filelist.f \
    -o simv

# 运行（指定测试类）
./simv +UVM_TESTNAME=rand_test \
       +UVM_VERBOSITY=UVM_MEDIUM \
       -l sim.log

# 回归测试脚本
for test in rand_test directed_test boundary_test; do
    ./simv +UVM_TESTNAME=$test -l ${test}.log
done
```
