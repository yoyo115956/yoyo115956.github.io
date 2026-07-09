---
layout:       post
title:        'IC验证面试题（三）：UVM 激励机制与 TLM 通信'
date:         2025-03-10
header-style: text
catalog:      true
mathjax:      false
tags:
  - 芯片验证
  - UVM
  - 面试
---

> 本篇覆盖 Sequence/Sequencer/Driver 通信机制、Virtual Sequencer、TLM 端口类型、Sequencer 仲裁等高频考点。

---

## 一、Sequence / Sequencer / Driver 三者关系

### Q1：为什么要把 Sequence、Sequencer、Driver 分开？好处是什么？

```
Sequence（蓄水池）─→ Sequencer（调度站）─→ Driver（工厂）─→ DUT
```

- **Sequence**：只负责产生不同的激励（sequence item），与接口无关
- **Sequencer**：负责调度，协调多个 sequence 的优先级和时序
- **Driver**：只负责把 item 转化为时序信号驱动到 DUT 接口，不关心激励从哪来

**好处**：三者相互独立，可以在不改变 driver 和 sequencer 的情况下随意切换 sequence，极大提升可重用性和灵活性。传统的 generator + driver 方案代码臃肿、耦合度高，难以复用。

---

### Q2：Sequence 和 Sequencer 的通信流程？

```
sequence.body() 调用 start_item(req) + finish_item(req)
         ↕ TLM put/get
sequencer 调度队列
         ↕ seq_item_port（TLM FIFO）
driver 调用 seq_item_port.get_next_item(req)
              ... 驱动 DUT ...
         seq_item_port.item_done()
```

**Driver 侧代码框架**：
```systemverilog
task run_phase(uvm_phase phase);
    forever begin
        seq_item_port.get_next_item(req);  // 阻塞获取 item
        drive_item(req);                    // 驱动 DUT
        seq_item_port.item_done();          // 通知 sequencer 完成
        // 如果是读操作，可以先 get_next_item 后用 item_done(rsp) 返回响应
    end
endtask
```

**Sequence 侧代码框架**：
```systemverilog
task body();
    req = MyItem::type_id::create("req");
    start_item(req);           // 申请 sequencer，阻塞直到 sequencer 就绪
    assert(req.randomize());   // 随机化
    finish_item(req);          // 发送给 driver，阻塞直到 item_done
endtask
```

---

### Q3：`get_next_item()` 和 `try_next_item()` 的区别？

| 方法 | 特性 | 返回值 |
|------|------|--------|
| `get_next_item()` | **阻塞**调用，没有 item 时持续等待 | 始终返回有效 item 指针 |
| `try_next_item()` | **非阻塞**调用，立即返回 | 有 item 时返回指针，没有时返回 `null` |

`try_next_item()` 常用于 driver 需要在无激励时执行默认行为（如驱动 idle 状态）的场景。

---

## 二、启动 Sequence 的方法

### Q4：有哪些方式可以启动 Sequence？

**方式一：`sequence.start()` 显式启动**
```systemverilog
task main_phase(uvm_phase phase);
    MySeq seq = MySeq::type_id::create("seq");
    phase.raise_objection(this);
    seq.start(env.agent.sequencer);  // 绑定到目标 sequencer
    phase.drop_objection(this);
endtask
```

**方式二：Default Sequence 隐式启动（通过 config_db）**
```systemverilog
// 在 test 的 build_phase 中配置
uvm_config_db #(uvm_object_wrapper)::set(
    this, "env.agent.sequencer.main_phase",
    "default_sequence", MySeq::get_type()
);
```

**方式三：`uvm_do` 系列宏**
```systemverilog
// 在父 sequence 的 body 中启动子 sequence 或 item
`uvm_do(sub_seq)                          // 创建并随机化并执行
`uvm_do_with(item, {addr==8'h10;})        // 带 inline 约束
`uvm_do_on(seq, p_sequencer.other_sqr)   // 指定 sequencer
```

---

## 三、Virtual Sequence 与 Virtual Sequencer

### Q5：Virtual Sequencer 和普通 Sequencer 的区别？

| 对比 | Sequencer | Virtual Sequencer |
|------|-----------|-------------------|
| 与 Driver 连接 | 是 | **否**（不传递 item，不连 driver）|
| 职责 | 调度面向单一接口的 sequence | 协调多个不同类型的 sequencer |
| 传递 item | 是 | 否（只持有底层 sequencer 的句柄）|

**Virtual Sequencer 内部结构**：
```systemverilog
class my_virtual_sequencer extends uvm_sequencer;
    // 只持有各子 sequencer 的句柄，不和 driver 连接
    my_apb_sequencer apb_sqr;
    my_axi_sequencer axi_sqr;
endclass
```

**Virtual Sequence 用法**：
```systemverilog
class my_virtual_seq extends uvm_sequence;
    // 使用 virtual sequencer 的句柄访问各子 sequencer
    `uvm_declare_p_sequencer(my_virtual_sequencer)
    
    task body();
        MyApbSeq apb_seq = MyApbSeq::type_id::create("apb_seq");
        MyAxiSeq axi_seq = MyAxiSeq::type_id::create("axi_seq");
        
        // 并行启动两个不同接口的 sequence
        fork
            apb_seq.start(p_sequencer.apb_sqr);
            axi_seq.start(p_sequencer.axi_sqr);
        join
    endtask
endclass
```

**virtual 含义**：virtual sequencer 不传递 item，不与 driver 连接，只是协调各 agent 中 sequencer 的中央路由器。

---

## 四、Sequence 分类

### Q6：Sequence 有哪几种类型？

| 类型 | 说明 |
|------|------|
| **扁平序列（Flat Sequence）** | 只组织 item 实例，产生最细粒度的激励 |
| **层次序列（Hierarchical Sequence）** | 由高层 sequence 组织底层 sequence，实现顺序或并行执行 |
| **虚拟序列（Virtual Sequence）** | 顶层控制整个测试场景，协调多个不同接口的 sequencer |

---

## 五、Sequencer 仲裁

### Q7：Sequencer 的仲裁策略有哪些？

当多个 sequence 同时向 sequencer 发送 item 时，由仲裁策略决定优先级：

| 策略 | 说明 |
|------|------|
| `SEQ_ARB_FIFO`（默认）| 先进先出 |
| `SEQ_ARB_WEIGHTED` | 按权重随机选择 |
| `SEQ_ARB_RANDOM` | 完全随机 |
| `SEQ_ARB_STRICT_FIFO` | 严格 FIFO，考虑优先级 |
| `SEQ_ARB_STRICT_RANDOM` | 按优先级严格随机 |
| `SEQ_ARB_USER` | 用户自定义仲裁函数 |

```systemverilog
sequencer.set_arbitration(SEQ_ARB_WEIGHTED);
seq.set_priority(200); // 设置较高优先级（默认 100）
```

**锁定机制**：
- `lock(sqr)`：排他性锁定，等待所有当前 item 完成后独占 sequencer
- `grab(sqr)`：抢占式锁定，立即独占，当前 item 之后即生效（更高优先级）
- `unlock()` / `ungrab()`：释放锁定

---

## 六、TLM 通信

### Q8：TLM 端口的三种类型和特点？

| 端口类型 | 角色 | 特点 |
|----------|------|------|
| `uvm_*_port` | Initiator（发起方）| 连接的起点 |
| `uvm_*_export` | 中间层（可转发）| 连接 port 和 imp 之间的桥梁 |
| `uvm_*_imp` | Target（终点）| 只能作为终点，需要实现对应方法 |

**连接规则**：`port.connect(export_or_imp)` —— 只能 port 连 export 或 imp，不能反向。

---

### Q9：`put`、`get`、`peek` 的区别？

| 方法 | 方向 | 对数据的影响 |
|------|------|------------|
| `put(item)` | Initiator → Target | 将 item 传入 target，target 消化数据 |
| `get(item)` | Target → Initiator | 从 target 取出 item，**target 中该数据被移除** |
| `peek(item)` | Target → Initiator | 从 target 查看 item，**target 中数据保留** |

常用组合：先 `peek` 获取数据（不删除，防止 put 端立即又发送新数据），处理完后再 `get` 删除。

---

### Q10：Analysis Port 和普通 TLM Port 的区别？

| 对比 | 普通 TLM Port | Analysis Port |
|------|---------------|---------------|
| 连接关系 | 一对一（一个 port 连一个 imp）| **一对多**（一个 analysis_port 连多个 analysis_imp）|
| 通信模式 | 请求/响应（握手）| 广播（发布/订阅）|
| 方法 | `put`/`get`/`peek` | `write(item)`（所有连接的 imp 都会被调用）|
| 是否阻塞 | blocking/nonblocking 版本均有 | 非阻塞（`write` 立即返回）|

**Analysis Port 使用示例**：
```systemverilog
// Monitor 中
uvm_analysis_port #(MyItem) ap;
ap = new("ap", this);

// 采集到事务后广播
ap.write(item);  // 所有连接到此 port 的 scoreboard、coverage 都会收到

// Scoreboard 中实现 write 方法
uvm_analysis_imp #(MyItem, MyScoreboard) ap;
function void write(MyItem item);
    // 处理接收到的 item
endfunction
```

**Analysis Port 可以不连接**：若没有任何 imp 连接，`write()` 调用不会报错（广播给空集合）。

---

### Q11：`uvm_tlm_fifo` 和 `analysis_fifo` 的区别？

- **`uvm_tlm_fifo`**：有缓冲的通用 TLM 管道，提供 put/get/peek 端口，用于两个组件之间的带缓冲通信
- **`uvm_tlm_analysis_fifo`**：继承自 `uvm_tlm_fifo`，同时提供 `analysis_export`（实现了 `write()` 方法），可以直接连接 analysis port，作为 monitor 和 scoreboard 之间的缓冲桥接器
