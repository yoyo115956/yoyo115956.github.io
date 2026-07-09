---
layout:       post
title:        'IC验证面试题（四）：覆盖率与断言'
date:         2025-03-15
header-style: text
catalog:      true
mathjax:      false
tags:
  - 芯片验证
  - SVA
  - 覆盖率
  - 面试
---

> 本篇覆盖代码/功能/断言覆盖率类型、SVA 并发断言语法、sequence 操作符详解，以及覆盖率不达标时的处理策略。

---

## 一、覆盖率分类

### Q1：代码覆盖率、功能覆盖率、断言覆盖率的区别？

| 类型 | 来源 | 衡量什么 |
|------|------|----------|
| **代码覆盖率（Code Coverage）** | EDA 工具自动生成 | 代码被执行的比例（行/分支/条件/翻转/状态机）|
| **功能覆盖率（Functional Coverage）** | 验证工程师手写 covergroup | 设计规格中的功能特性是否被激励到 |
| **断言覆盖率（Assertion Coverage）** | 断言中的 cover property | 指定的时序场景是否被触发过 |

**最重要区别**：
- 代码覆盖率 100% 不代表功能被验证充分——代码跑到了，但关键特性组合未必出现
- 功能覆盖率由人驱动，反映的是**验证意图**是否达成
- 两者必须综合使用：功能覆盖率指导激励，代码覆盖率查漏补缺

---

### Q2：如果覆盖率未达到 100% 怎么处理？

1. **分析未覆盖点**：定位哪个 coverpoint 或 cross 未命中
2. **检查约束**：是否因约束太严导致某种组合从未出现
3. **增加 directed test**：针对性地写定向测试补全边角场景
4. **调整权重/分布**：用 `dist` 约束增大目标场景被随机到的概率
5. **回归仿真**：延长仿真时间，增加测试次数
6. **排除无效覆盖点**：与设计负责人确认该点是否 DUT 根本无法到达（如硬件约束），若确认则使用 `ignore_bins` 排除

---

## 二、SystemVerilog 断言（SVA）

### Q3：立即断言和并发断言有什么区别？

| 类型 | 触发时机 | 语法 | 适用场景 |
|------|----------|------|----------|
| **立即断言（Immediate Assertion）** | 像普通语句，遇到时立即判断（组合逻辑时刻）| `assert (expr);` | 简单值检查，用在 initial/always 块中 |
| **并发断言（Concurrent Assertion）** | 基于采样时钟，在时钟边沿采样并判断 | `assert property (@clk) ...;` | 时序行为检验，跨多个时钟周期的协议检查 |

```systemverilog
// 立即断言：判断一个变量当前值
always @(posedge clk) begin
    assert (data != 'x) else $error("Data is X!");
end

// 并发断言：验证时序协议（req 后 1~3 周期内 ack 必须有效）
property req_ack_p;
    @(posedge clk) disable iff (!rst_n)
    req |-> ##[1:3] ack;
endproperty

assert property (req_ack_p) else $error("ACK timeout!");
```

---

### Q4：SVA 中并发断言的三种 sequence 操作符详解？

| 操作符 | 语义 | 示例 |
|--------|------|------|
| `a[*n]` | 连续重复 n 次（a 必须在连续 n 个时钟周期内全部为真）| `req[*3]`：req 连续高 3 周期 |
| `a[->n]` | 非连续跳转重复 n 次（a 在任意位置为真 n 次，每次之间可以插入任意个 a 为假的周期，以 a 为真的周期结尾）| `ack[->2]`：ack 在任意周期出现 2 次 |
| `a[=n]` | 非连续等于重复 n 次（a 在任意位置为真 n 次，最后一次不必以 a 结尾，序列在 a 的最后一次后继续）| `valid[=3]`：valid 在序列内出现恰好 3 次 |

**区别总结**：
- `[->n]`：序列**以第 n 次 a 为真结束**
- `[=n]`：序列**在 a 出现 n 次后，可继续延伸**（不以 a 结束）

```systemverilog
// 示例：APB 协议 PSEL 拉高后，PENABLE 在下一拍拉高，并在 1 周期内完成传输
property apb_setup;
    @(posedge clk)
    (psel && !penable) |=> (psel && penable);
endproperty
```

---

### Q5：`$past` 的用法？

`$past(expr, n)` 返回 **n 个时钟周期之前** `expr` 的采样值（默认 n=1）。

```systemverilog
// 验证：当 ack 有效时，req 在 2 个周期前应为高
property req_before_ack;
    @(posedge clk)
    ack |-> ($past(req, 2) == 1'b1);
endproperty

// 常见用途：检查信号稳定性（write 有效时，addr 在前一拍已稳定）
property addr_stable;
    @(posedge clk)
    pwrite |-> ($past(paddr) == paddr);
endproperty
```

---

### Q6：`assert property` 和 `cover property` 有什么区别？

| 关键字 | 作用 | 触发条件 |
|--------|------|----------|
| `assert property (p)` | 验证属性 p 必须成立，违反则报错 | 属性为假时（错误检测）|
| `cover property (p)` | 记录属性 p 是否被触发过 | 属性为真时（覆盖率收集）|
| `assume property (p)` | 约束形式验证输入（在形式验证中使用）| 约束输入空间 |

---

## 三、Covergroup 详解

### Q7：Covergroup 的基本结构和常用语法？

```systemverilog
covergroup cg_transaction @(posedge clk);
    // coverpoint：对单个变量的覆盖
    cp_addr: coverpoint addr {
        bins low_addr  = {[0:7]};
        bins mid_addr  = {[8:15]};
        bins high_addr = {[16:31]};
        illegal_bins x_val = {8'hxx}; // 标记为非法值（仿真中出现即报错）
        ignore_bins  reserved = {8'hF0, 8'hFF}; // 忽略的值（不计入覆盖率）
    }

    // cross：对多个变量的交叉覆盖（笛卡尔积）
    cx_addr_rw: cross cp_addr, rw;
    
    // 带条件触发的 coverpoint
    cp_data: coverpoint data iff (valid);
endgroup

// 实例化
cg_transaction cg = new();

// 查询覆盖率
$display("Coverage = %0.2f%%", cg.get_inst_coverage());
```

---

### Q8：Bins 的几种类型？

| Bins 类型 | 关键字 | 说明 |
|-----------|--------|------|
| 命名 bins | `bins name = {vals}` | 显式命名，多个值映射到一个 bin |
| 自动 bins | （不声明任何 bins）| 自动为每个枚举值创建一个 bin |
| 数组 bins | `bins arr[4] = {[0:15]}` | 将范围平均分为 4 个 bin |
| 非法 bins | `illegal_bins` | 出现即仿真错误 |
| 忽略 bins | `ignore_bins` | 排除出覆盖率计算 |
| 转换 bins | `bins trans = (0=>1)` | 捕获信号从 0 变 1 的转换 |

---

## 四、分析端口与多连接

### Q9：Analysis Port 能不连接或连接多个 imp 吗？

- **可以不连接**：`analysis_port.write()` 在没有连接时不会报错，也不会有任何副作用（写给空集合）
- **可以连接多个 imp**：这正是 analysis port 的核心优势——**广播语义**，一对多连接
  - 一个 monitor 的 analysis port 可以同时连接 scoreboard、coverage collector、logger 等多个接收器

```systemverilog
// connect_phase 中
mon.ap.connect(scoreboard.ap_imp);    // 连 scoreboard
mon.ap.connect(cov_collector.ap_imp); // 同时连 coverage
mon.ap.connect(logger.ap_imp);        // 同时连 logger
// 三者都会在 monitor 调用 write() 时收到同一个 item
```

---

### Q10：如果一个 component 需要连接多个 analysis port（接收来自多个 monitor 的数据），如何区分？

使用参数化宏 `uvm_analysis_imp_decl` 为每个 imp 声明独立的 `write_*` 方法：

```systemverilog
`uvm_analysis_imp_decl(_req)  // 定义 write_req()
`uvm_analysis_imp_decl(_rsp)  // 定义 write_rsp()

class my_scoreboard extends uvm_scoreboard;
    uvm_analysis_imp_req #(MyReqItem, my_scoreboard) ap_req;
    uvm_analysis_imp_rsp #(MyRspItem, my_scoreboard) ap_rsp;
    
    function void write_req(MyReqItem item); /* ... */ endfunction
    function void write_rsp(MyRspItem item); /* ... */ endfunction
endclass
```
