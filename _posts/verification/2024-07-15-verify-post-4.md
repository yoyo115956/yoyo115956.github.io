---
layout:       post
title:        'SVA 断言验证：从属性定义到形式验证'
date:         2024-07-15
header-style: text
catalog:      true
mathjax:      true
tags:
  - 芯片验证
  - SVA
  - 形式验证
  - SystemVerilog
---

> SVA（SystemVerilog Assertions）是描述硬件时序属性的语言，既可用于仿真检查，也是形式验证的输入。

# 一、断言的两种类型

## 1.1 立即断言（Immediate Assertion）

在程序执行到某点时立即检查条件（与 C 语言的 `assert` 类似）：

```systemverilog
// 在过程块中使用
always @(posedge clk) begin
    assert (valid -> data_ready)
        else `uvm_error("ASSERT", "data_ready must be high when valid");
    
    // 带 pass/fail action block
    assert (count < MAX_COUNT) begin
        $display("Count OK: %0d", count);
    end else begin
        $error("Count overflow: %0d", count);
    end
end
```

## 1.2 并发断言（Concurrent Assertion）

基于时钟采样，在整个仿真过程中持续监控时序行为：

```systemverilog
// 基本形式
assert property (prop_name);

// 带标签
assert_valid_req: assert property (
    @(posedge clk) disable iff (!rst_n)
    valid |-> ##1 ack
) else $error("ACK must follow VALID after 1 cycle");
```

---

# 二、序列（Sequence）

序列描述信号随时间变化的模式：

```systemverilog
// 基本时钟延迟
##n          // 恰好 n 个时钟周期
##[m:n]      // m 到 n 个时钟周期
##[*]        // 任意个周期（##[0:$]）
##[+]        // 至少 1 个周期（##[1:$]）

// 序列操作符
// 连接：s1 ##1 s2    （s1 结束后 1 周期 s2 开始）
// 并发：s1 and s2    （同时开始和结束）
// 交叉：s1 intersect s2
// 第一匹配：first_match(s)
// 重复：sig [*n]      恰好重复 n 次
//        sig [*m:n]   重复 m 到 n 次
//        sig [->n]    goto 重复：非连续 n 次

// 示例
sequence s_req_ack;
    req ##[1:4] ack;  // req 后 1 到 4 周期内 ack 拉高
endsequence

sequence s_burst_write;
    valid && write ##1 (valid [*3]) ##1 !valid;  // 连续 4 拍写
endsequence
```

---

# 三、属性（Property）

属性是对序列施加的时序约束：

```systemverilog
// implication（蕴含）操作符
// p |-> q ：如果 p 成立，则 q 必须立即成立（overlapping）
// p |=> q ：如果 p 成立，则下一拍 q 必须成立（non-overlapping）

// 示例集合
property p_handshake;
    @(posedge clk) disable iff (!rst_n)
    valid |-> ##[1:8] ready;  // valid 后 1-8 拍内 ready 必须拉高
endproperty

property p_no_x_on_data;
    @(posedge clk)
    valid |-> !$isunknown(data);  // valid 时 data 不能有 X
endproperty

property p_stable_during_burst;
    @(posedge clk)
    (valid && !last) |=> $stable(addr);  // 非最后一拍时 addr 不变
endproperty

property p_onehot;
    @(posedge clk)
    valid |-> $onehot(grant);  // grant 必须独热码
endproperty
```

---

# 四、常用系统函数

| 函数 | 说明 |
|------|------|
| `$rose(sig)` | 信号上升沿（0→1）|
| `$fell(sig)` | 信号下降沿（1→0）|
| `$stable(sig)` | 信号在当前采样点与上一采样点相同 |
| `$changed(sig)` | 信号值发生变化 |
| `$past(sig, n)` | n 个周期前的信号值 |
| `$isunknown(sig)` | 信号包含 X 或 Z |
| `$onehot(sig)` | 信号只有 1 位为 1 |
| `$onehot0(sig)` | 信号最多只有 1 位为 1 |
| `$countones(sig)` | 统计 1 的个数 |

---

# 五、AXI4 协议断言实例

AXI4 协议有严格的握手时序要求，非常适合用 SVA 验证：

```systemverilog
module axi4_assertions (
    input clk, rst_n,
    // Write Address Channel
    input awvalid, awready,
    input [31:0] awaddr,
    input [7:0] awlen,
    // Write Data Channel
    input wvalid, wready, wlast,
    // Write Response Channel
    input bvalid, bready,
    input [1:0] bresp
);

// 规则1：VALID 一旦拉高，必须保持直到 READY
property p_awvalid_stable;
    @(posedge clk) disable iff (!rst_n)
    $rose(awvalid) |=> (awvalid || awready) [*0:$] ##0 awvalid;
endproperty
// 等价于：awvalid 拉高后，在 awready 确认前不能拉低
property p_awvalid_hold;
    @(posedge clk) disable iff (!rst_n)
    (awvalid && !awready) |=> awvalid;
endproperty
ap_awvalid_hold: assert property (p_awvalid_hold);

// 规则2：WLAST 必须在 awlen+1 拍数据后出现
property p_wlast_timing;
    @(posedge clk) disable iff (!rst_n)
    (awvalid && awready) |->
        ##1 wvalid [->awlen+1] ##0 wlast;
endproperty

// 规则3：bresp 不能为 DECERR (2'b11)
property p_no_decerr;
    @(posedge clk) disable iff (!rst_n)
    bvalid |-> bresp != 2'b11;
endproperty
ap_no_decerr: assert property (p_no_decerr);

// 规则4：地址必须对齐
property p_addr_align;
    @(posedge clk)
    awvalid |-> awaddr[1:0] == 2'b00;  // 4 字节对齐
endproperty

endmodule
```

---

# 六、在 UVM 中集成断言

```systemverilog
class apb_monitor extends uvm_monitor;
    
    task run_phase(uvm_phase phase);
        // 在 monitor 中动态打开/关闭断言
        $assertoff(0, tb_top.dut.assert_handshake);
        
        // 等待复位结束后打开
        @(posedge vif.rst_n);
        $asserton(0, tb_top.dut);
        
        // 主监控循环
        forever begin
            // ...
        end
    endtask
endclass
```

---

# 七、形式验证（Formal Verification）

SVA 断言可以直接作为形式验证工具的输入：

```bash
# JasperGold 基本使用
# 1. 分析设计
analyze -sv dut.sv axi4_assertions.sv

# 2. 精化（指定顶层和时钟）
elaborate -top axi4_assertions
clock clk -period 10
reset -expression {!rst_n}

# 3. 运行属性检查
prove -property {ap_awvalid_hold ap_no_decerr}

# 结果：PROVEN（数学证明正确）或 CEX（找到反例）
```

形式验证的优势：
- **完备性**：不依赖特定激励，穷举所有可能输入
- **自动发现边界 Bug**：仿真难以覆盖的角落场景
- **快速收敛**：对小模块（<100K 门）非常高效

形式验证的局限：
- **状态爆炸**：对大型复杂设计，状态空间过大难以收敛
- **建模成本**：需要精确的环境约束（`assume`）
