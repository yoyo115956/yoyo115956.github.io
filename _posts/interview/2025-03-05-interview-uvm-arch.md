---
layout:       post
title:        'IC验证面试题（二）：UVM 架构与核心机制'
date:         2025-03-05
header-style: text
catalog:      true
mathjax:      false
tags:
  - 芯片验证
  - UVM
  - 面试
  - 核心框架
---

> 本篇覆盖 UVM 架构、Phase 机制、Factory 工厂、config_db、objection、field_automation、callback 等面试高频考点。

---

## 一、UVM 整体架构

### Q1：UVM 验证环境由哪些组件构成？各组件的职责是什么？

```
Test
 └── Env
      ├── Agent
      │    ├── Sequencer  —— 接收 sequence item，调度并转发给 Driver
      │    ├── Driver     —— 驱动 DUT 接口，消费 sequence item
      │    └── Monitor    —— 被动采集 DUT 接口信号，广播事务
      ├── Scoreboard      —— 比对 Monitor 采集结果与参考模型输出
      └── Coverage        —— 收集功能覆盖率
```

| 组件 | 继承自 | 核心职责 |
|------|--------|----------|
| `uvm_test` | `uvm_component` | 顶层测试，实例化 env，启动 sequence |
| `uvm_env` | `uvm_component` | 容纳并连接所有 agent、scoreboard |
| `uvm_agent` | `uvm_component` | 封装 driver + sequencer + monitor |
| `uvm_driver` | `uvm_component` | 通过虚接口驱动 DUT |
| `uvm_monitor` | `uvm_component` | 被动采集，通过 analysis port 广播 |
| `uvm_scoreboard` | `uvm_component` | 接收 monitor 数据，对比验证 |
| `uvm_sequence_item` | `uvm_object` | 一次事务的数据载体 |
| `uvm_sequence` | `uvm_object` | 产生并组织 sequence item |

**component 与 object 的核心区别**：
- `uvm_component`：有 phase 机制和树形层次，在仿真全程存在
- `uvm_object`：无 phase、无父子结构，是轻量级数据容器（item、sequence、config 均是 object）

---

### Q2：UVM 从哪里启动？启动流程是什么？

```systemverilog
// 在 top module 的 initial 块中调用
initial begin
    uvm_config_db #(virtual apb_if)::set(null, "uvm_test_top.*", "vif", apb_if_inst);
    run_test("my_test"); // 启动 UVM
end
```

**启动流程**：
1. `import uvm_pkg::*` 时，UVM 自动创建 `uvm_root` 单例（`uvm_top`）
2. `run_test("my_test")` 根据字符串从 factory 中创建 `my_test` 实例
3. 依次执行各 phase（自顶向下/自底向上）

---

## 二、Phase 机制

### Q3：UVM 的 9 大 Phase 顺序是什么？哪些是 function，哪些是 task？

| Phase | 类型 | 执行顺序 |
|-------|------|----------|
| `build_phase` | function | **自顶向下**（先创建父，再创建子）|
| `connect_phase` | function | 自底向上 |
| `end_of_elaboration_phase` | function | 自底向上 |
| `start_of_simulation_phase` | function | 自底向上 |
| **`run_phase`** | **task** | **并行**（可消耗仿真时间）|
| `extract_phase` | function | 自底向上 |
| `check_phase` | function | 自底向上 |
| `report_phase` | function | 自底向上 |
| `final_phase` | function | 自顶向下 |

**关键点**：
- 只有 `run_phase` 和 12 个 run-time sub-phase 是 task，其余全是 function
- `build_phase` 和 `final_phase` 自顶向下，其余 function phase 自底向上
- `run_phase` 与 12 个子 phase（`reset_phase → shutdown_phase`）**并行运行**，最好不要混用

### Q4：`run_phase` 和 `main_phase` 之间的关系？

- 两者都是 task phase，**并行运行**
- `run_phase` 是最常用的方式；12 个 run-time sub-phase 是对 `run_phase` 的进一步细分
- 若在 sub-phase 中提起了 objection，`run_phase` 无需自己 raise 就会自动运行
- **不建议同时使用** `run_phase` 和 sub-phase，否则行为复杂难控制

### Q5：如何从 `main_phase` 跳转到 `reset_phase`？

```systemverilog
task main_phase(uvm_phase phase);
    // 检测到复位信号
    if (reset_detected) begin
        phase.jump(uvm_reset_phase::get()); // 跳转到 reset_phase
    end
endtask
```

### Q6：Phase Domain 是什么？

Domain 用于将不同组件划分成独立运行的时钟域：
- 默认情况下所有组件属于 `common_domain`（9 大 phase）和 `uvm_domain`（12 个 sub-phase）
- 同一 domain 的同名 phase 必须同步运行（所有组件完成后才进入下一 phase）
- 将两个 driver 放入不同 domain，它们的 sub-phase 就可以异步独立运行

---

## 三、Factory 机制

### Q7：UVM Factory 机制的作用和使用步骤？

**作用**：在不修改 TB 代码的情况下，通过**类型覆盖**或**实例覆盖**替换组件/对象，提升可重用性和灵活性。

**三个步骤**：
```systemverilog
// Step 1：注册类到 factory（在类定义时）
class MyDriver extends uvm_driver;
    `uvm_component_utils(MyDriver)  // component 用这个宏
endclass

class MyItem extends uvm_sequence_item;
    `uvm_object_utils(MyItem)       // object 用这个宏
endclass

// Step 2：使用 factory 方式创建实例（不能用 new()）
MyDriver drv = MyDriver::type_id::create("drv", this);

// Step 3：在需要覆盖时（通常在 test 的 build_phase 中）
MyExtDriver::type_id::set_type_override(MyDriver::get_type());
// 或实例级覆盖
MyExtDriver::type_id::set_inst_override(MyDriver::get_type(), "uvm_test_top.env.agent.drv");
```

**使用工厂覆盖的要求**：
1. 被覆盖类和替换类都必须注册到 factory
2. 被覆盖类必须用 `type_id::create()` 实例化（而非 `new()`）
3. 替换类**必须**派生自被覆盖类

---

## 四、config_db 配置数据库

### Q8：`uvm_config_db` 的作用和用法？四个参数分别是什么？

```systemverilog
// Set：在高层次（通常是 top module 或 test）设置
uvm_config_db #(virtual apb_if)::set(
    null,                        // context：null 表示全局
    "uvm_test_top.env.agent.*",  // 路径（支持通配符）
    "vif",                       // 参数名（set 和 get 必须一致）
    apb_if_inst                  // 值（virtual interface）
);

// Get：在低层次组件的 build_phase 中获取
function void build_phase(uvm_phase phase);
    if (!uvm_config_db #(virtual apb_if)::get(
            this, "", "vif", vif))
        `uvm_fatal("NO_VIF", "Virtual interface not found!")
endfunction
```

**三种常见用途**：
1. **传递虚接口**：将 DUT 的物理接口传递给 driver/monitor（最常用）
2. **传递单一变量**：int、string、enum 等配置参数
3. **传递配置对象**：将多个参数封装成 `uvm_object`，整体传递，便于维护

**优先级规则**：层次越高的 `set` 优先级越高；同层次时时间靠后的覆盖时间靠前的。

**接口传递注意事项**：
- `uvm_config_db::set` 必须在 `run_test()` 之前调用
- 传递类型必须是 `virtual interface`（句柄），不能是实际 interface（物理实例）

---

## 五、Objection 机制

### Q9：UVM Objection 机制的作用？

Objection 用于**控制 run phase（及各子 phase）的结束时机**。

- UVM 进入 run phase 时开始监控所有 objection
- 所有参与 objection 机制的组件都 `drop_objection` 后，计数器归零，run phase 结束
- 若进入 phase 时没有任何组件 raise objection，**该 phase 立即结束**

```systemverilog
task run_phase(uvm_phase phase);
    phase.raise_objection(this);   // 提起，阻止退出
    // ... 执行测试逻辑 ...
    phase.drop_objection(this);    // 放下，允许退出
endtask
```

**重要原则**：
- `raise_objection` 必须在第一个消耗仿真时间的语句之前
- UVM 推荐在 **sequence** 中控制 objection（在 `body()` 里），而不是在 component 中

---

## 六、field_automation 机制

### Q10：`field_automation` 机制有什么作用？

使用 `uvm_field_*` 系列宏注册字段后，可以**自动实现** `copy()`、`compare()`、`print()`、`pack()`、`unpack()` 等方法，无需手动编写：

```systemverilog
class MyItem extends uvm_sequence_item;
    `uvm_object_utils_begin(MyItem)
        `uvm_field_int(addr,   UVM_ALL_ON)
        `uvm_field_int(data,   UVM_ALL_ON)
        `uvm_field_enum(rw_t, rw, UVM_ALL_ON)
    `uvm_object_utils_end
    
    rand bit [31:0] addr, data;
    rand rw_t rw;
endclass
```

调用示例：
```systemverilog
item1.copy(item2);           // 自动复制所有注册字段
if (item1.compare(item2)) …  // 自动比较
item1.print();               // 自动格式化打印
```

---

## 七、Callback 机制

### Q11：Callback 机制的作用，与 Factory 的区别？

**Callback 机制**：在不修改原组件代码的前提下，通过在特定时机插入用户自定义的回调函数，动态改变组件行为。

| 对比项 | Factory | Callback |
|--------|---------|----------|
| 替换方式 | 创建一个新类替换原类 | 在原类指定位置插入回调函数 |
| 结果 | 整个类被替换 | 类本身不变，行为被扩充 |
| 适用场景 | 大规模行为改变 | 局部、灵活的行为注入（如注入错误）|

```systemverilog
// Step 1：定义 callback 空壳类
class my_driver_cbs extends uvm_callback;
    virtual task pre_drive(my_driver drv, my_item item); endtask
endclass

// Step 2：在 component 中嵌入 callback 调用点
class my_driver extends uvm_driver;
    `uvm_register_cb(my_driver, my_driver_cbs)
    task run_phase(uvm_phase phase);
        `uvm_do_callbacks(my_driver, my_driver_cbs, pre_drive(this, req))
        // ... 正常驱动逻辑
    endtask
endclass

// Step 3：扩展 callback，注入错误
class error_inject_cb extends my_driver_cbs;
    task pre_drive(my_driver drv, my_item item);
        item.data ^= 32'hDEAD_BEEF; // 翻转数据，注入错误
    endtask
endclass
```
