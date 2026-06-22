---
layout:       post
title:        'SystemVerilog 验证核心特性：从语法到约束随机'
date:         2024-07-05
header-style: text
catalog:      true
mathjax:      true
tags:
  - 芯片验证
  - SystemVerilog
  - UVM
---

> SystemVerilog 是硬件验证的主流语言，在 Verilog 基础上引入了面向对象编程、约束随机、覆盖率收集等特性。

# 一、SystemVerilog vs Verilog

| 特性 | Verilog | SystemVerilog |
|------|---------|---------------|
| 用途 | RTL 设计 | 设计 + 验证 |
| 数据类型 | 有限 | 丰富（logic, int, bit...）|
| 面向对象 | 无 | 支持（class, extends...）|
| 约束随机 | 无 | 支持（rand, randc, constraint）|
| 覆盖率 | 无 | 支持（covergroup, coverpoint）|
| 接口 | 无 | 支持（interface, clocking block）|
| 断言 | 无 | 支持（assert, property, sequence）|

---

# 二、数据类型

## 2.1 双状态 vs 四状态

```systemverilog
// 四状态（0, 1, X, Z）— 来自 Verilog，用于 RTL
wire, reg, logic

// 双状态（0, 1）— 验证专用，速度快
bit       // 1 位无符号
byte      // 8 位有符号
shortint  // 16 位有符号
int       // 32 位有符号
longint   // 64 位有符号
```

## 2.2 常用数据结构

```systemverilog
// 动态数组
int dyn_arr[];
dyn_arr = new[10];        // 分配 10 个元素
dyn_arr = new[20](dyn_arr); // 扩展到 20 个，保留原内容

// 队列（最常用！）
int q[$];
q.push_back(1);
q.push_front(0);
int val = q.pop_front();
$display("size = %0d", q.size());

// 关联数组（哈希表）
int assoc_arr[string];
assoc_arr["key1"] = 100;
if (assoc_arr.exists("key1")) ...

// 枚举
typedef enum {IDLE, BUSY, DONE} state_t;
state_t state;
```

---

# 三、面向对象编程（OOP）

## 3.1 类的基本结构

```systemverilog
class Packet;
    // 属性
    rand bit [7:0]  addr;
    rand bit [31:0] data;
    rand bit        rw;    // 0=read, 1=write
    
    // 约束
    constraint valid_addr {
        addr inside {[8'h00 : 8'hFF]};
        addr % 4 == 0;  // 4 字节对齐
    }
    
    // 构造函数
    function new(bit [7:0] a = 0);
        addr = a;
    endfunction
    
    // 方法
    function void display();
        $display("Packet: addr=%0h data=%0h rw=%0b", addr, data, rw);
    endfunction
    
    // 深拷贝
    function Packet copy();
        Packet p = new();
        p.addr = this.addr;
        p.data = this.data;
        p.rw   = this.rw;
        return p;
    endfunction
endclass
```

## 3.2 继承与多态

```systemverilog
class ExtPacket extends Packet;
    rand bit [7:0] burst_len;
    
    constraint burst_limit {
        burst_len inside {1, 2, 4, 8, 16};
    }
    
    // 重写父类方法
    function void display();
        super.display();
        $display("  burst_len=%0d", burst_len);
    endfunction
endclass

// 多态
Packet pkt_h;           // 基类句柄
ExtPacket ext_pkt = new();
pkt_h = ext_pkt;        // 上转型
pkt_h.display();        // 调用 ExtPacket::display()（如果是 virtual）
```

## 3.3 虚方法与抽象类

```systemverilog
class BaseDriver;
    virtual task drive(Packet pkt);  // 虚方法，允许子类覆盖
        // 默认实现
    endtask
endclass

// 纯虚方法（强制子类实现）
virtual class AbstractBase;
    pure virtual function void build();
endclass
```

---

# 四、约束随机（Constrained Random）

## 4.1 随机变量

```systemverilog
class Transaction;
    rand  bit [7:0] addr;   // 每次 randomize() 都重新随机化
    randc bit [1:0] mode;   // 循环随机：轮流出现所有值后再重复
    bit   [31:0]    data;   // 非随机，randomize() 不影响
    
    constraint c_addr {
        addr >= 8'h10;
        addr <= 8'hEF;
    }
    
    constraint c_mode {
        mode != 2'b11;  // 排除某个值
    }
    
    // 权重约束（dist）
    constraint c_data_dist {
        data dist {
            32'h0000_0000 := 10,    // 权重 10
            [32'h0001:32'hFFFE] := 80,
            32'hFFFF_FFFF := 10
        };
    }
endclass
```

## 4.2 调用约束随机

```systemverilog
Transaction txn = new();

// 基本随机化
assert(txn.randomize()) else $fatal("Randomize failed!");

// 内联约束（override）
assert(txn.randomize() with {
    addr == 8'h20;
    data < 32'h100;
}) else $fatal;

// 关闭特定约束
txn.c_addr.constraint_mode(0);  // 关闭 c_addr
assert(txn.randomize());
txn.c_addr.constraint_mode(1);  // 重新打开
```

---

# 五、接口（Interface）与虚接口

```systemverilog
// 定义接口
interface apb_if (input clk, rst_n);
    logic [31:0] paddr, pwdata, prdata;
    logic        psel, penable, pwrite, pready;
    
    // Clocking block（采样和驱动时序）
    clocking driver_cb @(posedge clk);
        output paddr, pwdata, psel, penable, pwrite;
        input  prdata, pready;
    endclocking
    
    clocking monitor_cb @(posedge clk);
        input paddr, pwdata, prdata, psel, penable, pwrite, pready;
    endclocking
    
    // modport（限制访问方向）
    modport driver_mp  (clocking driver_cb, input rst_n);
    modport monitor_mp (clocking monitor_cb, input rst_n);
endinterface

// 虚接口（在类中使用接口）
class ApbDriver;
    virtual apb_if.driver_mp vif;  // 虚接口句柄
    
    task drive_write(input [31:0] addr, data);
        @(vif.driver_cb);
        vif.driver_cb.paddr   <= addr;
        vif.driver_cb.pwdata  <= data;
        vif.driver_cb.pwrite  <= 1;
        vif.driver_cb.psel    <= 1;
        @(vif.driver_cb);
        vif.driver_cb.penable <= 1;
        @(vif.driver_cb iff vif.driver_cb.pready);
        vif.driver_cb.psel    <= 0;
        vif.driver_cb.penable <= 0;
    endtask
endclass
```

---

# 六、覆盖率收集

```systemverilog
class Transaction;
    rand bit [7:0] addr;
    rand bit       rw;
    rand bit [1:0] size;
    
    covergroup cg_transaction;
        cp_addr: coverpoint addr {
            bins low    = {[8'h00 : 8'h3F]};
            bins mid    = {[8'h40 : 8'hBF]};
            bins high   = {[8'hC0 : 8'hFF]};
        }
        cp_rw:   coverpoint rw;
        cp_size: coverpoint size;
        
        // 交叉覆盖率
        cx_rw_size: cross cp_rw, cp_size;
    endgroup
    
    function new();
        cg_transaction = new();
    endfunction
    
    function void sample_cov();
        cg_transaction.sample();
    endfunction
endclass

// 查询覆盖率
$display("Coverage = %0.2f%%", cg_transaction.get_coverage());
```

---

# 七、小结：SV 验证代码规范

1. **类名**：首字母大写，如 `ApbDriver`
2. **变量名**：小写+下划线，如 `burst_len`
3. **句柄命名**：加 `_h` 后缀，如 `pkt_h`
4. **接口传入类**：统一用虚接口（`virtual interface`）
5. **总是检查 randomize() 返回值**：`assert(obj.randomize())`
6. **避免 `initial`/`always` 块**：在类方法和 `program` 块中使用 `task/function`
