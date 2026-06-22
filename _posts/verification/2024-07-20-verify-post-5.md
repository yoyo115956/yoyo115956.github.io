---
layout:       post
title:        '覆盖率驱动验证：功能覆盖率与回归测试收敛策略'
date:         2024-07-20
header-style: text
catalog:      true
mathjax:      true
tags:
  - 芯片验证
  - UVM
  - 覆盖率
  - 核心框架
---

> 覆盖率是衡量验证进度的量化指标。本文系统介绍代码覆盖率、功能覆盖率的原理，以及如何通过分析覆盖率漏洞提升验证质量。

# 一、覆盖率体系

验证覆盖率分为两大类：

```
覆盖率
├── 代码覆盖率（Code Coverage）—— 自动收集，由工具生成
│   ├── 行覆盖率（Line）
│   ├── 语句覆盖率（Statement）
│   ├── 分支覆盖率（Branch）
│   ├── 条件覆盖率（Condition）
│   ├── 翻转覆盖率（Toggle）
│   └── FSM 覆盖率（State / Transition）
│
└── 功能覆盖率（Functional Coverage）—— 手动设计，反映业务语义
    ├── Covergroup / Coverpoint
    ├── Cross Coverage（交叉覆盖率）
    └── Sequence Coverage（时序场景覆盖）
```

**代码覆盖率高 ≠ 验证完整**：可能执行了所有代码行，但从未触发特定的业务场景组合。功能覆盖率才能真正衡量验证的完整性。

---

# 二、代码覆盖率详解

## 2.1 分支覆盖率示例

```systemverilog
// RTL 代码
always @(posedge clk) begin
    if (req && grant)          // 分支 1: T/F
        state <= ACTIVE;
    else if (!req && timeout)  // 分支 2: T/F
        state <= TIMEOUT;
    else
        state <= IDLE;
end

// 代码覆盖率需要覆盖：
// 分支 1 为真 (req=1, grant=1)
// 分支 1 为假且分支 2 为真 (req=0, timeout=1)
// 两者都为假
```

## 2.2 FSM 覆盖率

```systemverilog
// 仿真器自动识别 FSM，检查：
// 1. 所有状态被访问过
// 2. 所有状态转移被触发过
// 3. 所有弧（arc）上的条件都被覆盖

typedef enum {IDLE, REQ, GRANT, DONE, ERR} state_t;
state_t state, next_state;
```

---

# 三、功能覆盖率设计

## 3.1 Covergroup 基础

```systemverilog
class apb_coverage extends uvm_subscriber #(apb_seq_item);
    `uvm_component_utils(apb_coverage)
    
    apb_seq_item txn;
    
    // 在类内部定义 covergroup
    covergroup cg_apb_traffic @(posedge vif.clk iff vif.psel);
        // 地址范围分区
        cp_addr: coverpoint txn.addr {
            bins reg_ctrl  = {[32'h0000 : 32'h00FF]};
            bins reg_data  = {[32'h0100 : 32'h01FF]};
            bins reg_status= {[32'h0200 : 32'h02FF]};
            bins others    = default;
        }
        
        // 读写类型
        cp_rw: coverpoint txn.rw {
            bins read  = {0};
            bins write = {1};
        }
        
        // 数据值边界
        cp_data_boundary: coverpoint txn.data {
            bins zero    = {32'h0};
            bins all_one = {32'hFFFF_FFFF};
            bins normal  = {[32'h1 : 32'hFFFF_FFFE]};
        }
        
        // 交叉覆盖率：地址 × 读写
        cx_addr_rw: cross cp_addr, cp_rw;
        
    endgroup
    
    // 基于事务采样的 covergroup
    covergroup cg_apb_seq;
        // 连续读写序列覆盖
        cp_consec_rw: coverpoint txn.rw {
            bins read_after_write  = (1 => 0);  // 写后读
            bins write_after_read  = (0 => 1);  // 读后写
            bins back_to_back_write= (1 => 1);  // 连续写
            bins back_to_back_read = (0 => 0);  // 连续读
        }
    endgroup
    
    function new(string name, uvm_component parent);
        super.new(name, parent);
        cg_apb_traffic = new();
        cg_apb_seq = new();
    endfunction
    
    function void write(apb_seq_item t);
        txn = t;
        cg_apb_traffic.sample();
        cg_apb_seq.sample();
    endfunction
    
    function void report_phase(uvm_phase phase);
        `uvm_info("COV", $sformatf(
            "Traffic Coverage: %0.2f%%  Seq Coverage: %0.2f%%",
            cg_apb_traffic.get_coverage(),
            cg_apb_seq.get_coverage()), UVM_NONE)
    endfunction
endclass
```

## 3.2 覆盖率 Hole（漏洞）分析

```systemverilog
// 排除不可能到达的 bin（避免误报）
cp_resp: coverpoint resp {
    bins ok   = {2'b00};
    bins err  = {2'b10};
    ignore_bins rsvd = {2'b01, 2'b11};  // 协议保留值，不需要覆盖
}

// 非法 bin（触发则报错）
cp_state: coverpoint state {
    bins valid_states = {IDLE, BUSY, DONE};
    illegal_bins invalid = default;  // 其他状态为非法
}
```

---

# 四、回归测试策略

## 4.1 测试分类

| 测试类型 | 目的 | 比例 |
|----------|------|------|
| 定向测试（Directed Test） | 覆盖特定功能点、边界条件 | ~20% |
| 随机测试（Random Test） | 广泛探索状态空间 | ~60% |
| 场景测试（Scenario Test） | 复杂协议交互、压力测试 | ~20% |

## 4.2 种子管理（Seed Management）

随机测试每次运行时使用不同的随机种子：

```bash
# VCS 运行多个种子
for seed in 1 2 3 42 100 999; do
    ./simv +UVM_TESTNAME=rand_test \
           +ntb_random_seed=$seed \
           -l sim_seed${seed}.log &
done
wait

# 合并覆盖率数据库
urg -dir simv.vdb -format text -report cov_report/
```

## 4.3 覆盖率收敛流程

```
初始随机测试（随机种子批次）
         ↓
收集并分析覆盖率报告
         ↓
    ┌────────────────────────────────┐
    │ 覆盖率 < 目标？                 │
    │ ↓YES                          │
    │ 分析未覆盖的 bin               │
    │ ↓                             │
    │ 添加定向测试 OR 增加约束偏置    │
    │ ↓                             │
    └────────────────────────────────┘
         ↓NO（覆盖率达标）
    Gate-Level Simulation
         ↓
      Tape-Out
```

## 4.4 约束偏置（Constraint Biasing）

当某些场景很少被随机命中时，用约束偏置提高命中概率：

```systemverilog
// 普通随机：地址全范围均匀分布，边界命中率极低
rand bit [31:0] addr;

// 偏置约束：20% 概率选地址 0 或最大地址
constraint c_addr_bias {
    addr dist {
        32'h0            := 10,
        32'hFFFF_FFFC    := 10,
        [32'h4 : 32'hFFFF_FFF8] := 80
    };
}
```

---

# 五、覆盖率报告解读

VCS/Questa 生成的覆盖率报告示例：

```
COVERGROUP COVERAGE:
  Covergroup              Hits   Goal   Status
  ──────────────────────────────────────────────
  cg_apb_traffic          87.5%  100%   UNCOVERED
    cp_addr               100%
    cp_rw                 100%
    cp_data_boundary       75%   ← 未覆盖 all_one bin
    cx_addr_rw             75%   ← reg_status + read 未触发
  
  cg_apb_seq             100%   100%   COVERED

CODE COVERAGE SUMMARY:
  Line         94.3%
  Branch       91.2%   ← 关注这里
  Condition    88.7%   ← 关注这里
  Toggle       96.1%
  FSM State   100%
  FSM Trans    83.3%   ← 有未覆盖的状态转移
```

**分析策略**：
1. 优先关注**功能覆盖率**漏洞，对应真实业务场景
2. Branch/Condition 覆盖率低 → 增加特定场景的定向测试
3. FSM Transition 未覆盖 → 找到触发该转移的条件，写专项测试
4. Toggle 覆盖率低 → 可能信号从未翻转，检查是否存在设计问题

---

# 六、验证 Sign-Off 检查清单

```
□ 代码覆盖率
  □ 行/语句 > 98%
  □ 分支 > 95%
  □ FSM 状态 100%，FSM 转移 > 95%

□ 功能覆盖率 100%（无 ignore_bins 的 bin 全部命中）

□ 回归测试
  □ 所有定向测试 PASS
  □ ≥100 个随机种子 PASS

□ 形式验证（如适用）
  □ 所有 SVA 属性 PROVEN 或 PASS

□ Bug 状态
  □ 无未关闭的 P1/P2 Bug
  □ 所有已知 workaround 已文档化
```
