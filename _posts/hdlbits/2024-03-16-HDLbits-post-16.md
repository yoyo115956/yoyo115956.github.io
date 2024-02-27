---
layout:       post
title:        'HDLbits刷题中文完整版-16'
date:         2024-03-16
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

# 【verilog学习26】HDLBits：Cs450

## I .（Cs450/timer）

### 1.代码编写

```verilog
module top_module(
	input clk, 
	input load, 
	input [9:0] data, 
	output tc
);
    reg [9:0] counter;
    always @(posedge clk ) begin
        if (load) begin
            counter<=data;
        end
        else if (counter=='0) begin
            counter<='0;//当递减到0，counter保持0
        end
        else if (!load) begin
            counter<=counter-1'b1;
        end
        else begin
            counter<=counter;
        end
    end

    assign tc=(counter=='0);
endmodule
```

### 2.提交结果

Success

### 3.题目分析

Implement a timer that counts down for a given number of clock cycles, then asserts a signal to indicate that the given duration has elapsed. A good way to implement this is with a **down-counter** that asserts an output signal when the count becomes 0.

At each clock cycle:
- If load = 1, load the internal counter with the 10-bit data, the number of clock cycles the timer should count before timing out. **The counter can be loaded at any time**, including when it is still counting and has not yet reached 0.
- If load = 0, the internal counter should decrement by 1.

The output signal tc (“terminal count”) indicates whether the internal counter has reached 0. **Once the internal counter has reached 0, it should stay 0** (stop counting) until the counter is loaded again.

Below is an example of what happens when asking the timer to count for 3 cycles: <img src="https://s2.loli.net/2024/02/27/Eo9PxbA3fIem8j1.png" alt="在这里插入图片描述">

## II . (Cs450/counter 2bc)

>  
 分支预测器介绍： https://blog.csdn.net/edonlii/article/details/8754724 


### 1.代码编写

```verilog
module top_module(
    input clk,
    input areset,
    input train_valid,
    input train_taken,
    output [1:0] state
);
    always@(posedge clk,posedge areset) begin
        if(areset)
            state <= 2'b01;
        else begin
            casex({train_valid,train_taken})
                2'b11: begin
                    state <= (state == 2'd3)? 2'd3:state + 2'd1;
                end
                2'b10: begin
                    state <= (state == 2'd0)? 2'd0:state - 2'd1;
                end
                2'b0?: begin
                    state <= state;
                end
                default: state <= 2'd1; // weakly not taken 2'b01
            endcase
        end
    end
endmodule
```

### 2.提交结果

Success

### 3.题目分析

Branch direction predictors are often structured as tables of counters indexed by the program counter and branch history. Each table entry usually uses two-bits of state because one-bit of state (remember last outcome) does not have enough hysteresis and flips states too easily.

<img src="https://s2.loli.net/2024/02/27/Ow3QaTgb4eWdz2l.png" alt="在这里插入图片描述">

A two-bit state machine that works fairly well is a saturating counter[1], which counts up to 3 (or 2’b11) or down to 0 (or 2’b00) but does not wrap around. A “taken” result increments the counter, while a “not-taken” result decrements the counter. A branch is predicted to be taken when the count is 2 or 3 (or 2’b1x). Adding some hysteresis prevents a flipping of the prediction when a strongly-biased branch occasionally takes a different direction, requiring two increments in the opposite direction before the prediction is flipped.

**References**

>  
 R. Nair, “Optimal 2-bit branch predictors”, IEEE Trans. Computers,vol. 44 no. 5, May, 1995 


**Description** Build a two-bit saturating counter.

The counter increments (up to a maximum of 3) when train_valid = 1 and train_taken = 1. It decrements (down to a minimum of 0) when train_valid = 1 and train_taken = 0. When not training (train_valid = 0), the counter keeps its value unchanged.

areset is an asynchronous reset that resets the counter to weakly not-taken (2’b01). Output state[1:0] is the two-bit counter value.

一开始没看懂题。 这是一个二位饱和计数器形式的分支预测器，这种方法的优点是，该条件分支指令必须连续选择某条分支两次，才能从强状态翻转，从而改变了预测的分支。 实现它有用FSM的方法和counter的方法。 其实从题目意图来看，这个FSM（SNT,WNT,WT,ST）状态转移有规律可循，可以用更简单地实现。当{train_valid,train_taken}=2’b11，taken可能性增强，相当于state+1计数，当{train_valid,train_taken}=2’b10，taken可能性减弱，相当于state-1计数，当train_valid=0，state保持其值。 <img src="https://s2.loli.net/2024/02/27/klKZnaj38WbgudQ.jpg" alt="在这里插入图片描述">

## III . (Cs450/history shift)

### 1.代码编写

```verilog
module top_module(
    input clk,
    input areset,

    input predict_valid,
    input predict_taken,
    output [31:0] predict_history,

    input train_mispredicted,
    input train_taken,
    input [31:0] train_history
);
    always@(posedge clk,posedge areset) begin
        if(areset)
            predict_history <= 32'd0;
        else begin
            if(train_mispredicted) begin
                predict_history <= {train_history[30:0],train_taken};
            end
            else begin
                predict_history <= (predict_valid)? {predict_history[30:0],predict_taken}:predict_history;
            end
        end     
    end
endmodule
```

### 2.提交结果

Success

### 3.题目分析

分支预测器，依旧可以参考上一道题的文章链接。 Branch direction predictors are often structured as tables of counters indexed by the program counter and branch history. The branch history is a sequence of “taken” or “not taken” results from recent branches.

In hardware, the branch history register can be implemented as a N-bit shift register. After each conditional branch direction is predicted, its predicted direction is shifted into the shift register. The shift register thus holds the most recent N branch results. <img src="https://s2.loli.net/2024/02/27/8UjNklhM172G4Kx.png" alt="在这里插入图片描述"> **Branch history register with surrounding hardware. This exercise builds the branch history register, inside the blue dashed rectangle. This diagram shows branch mispredictions being signalled by the branch execution unit, but depending on the processor design, this could also be at retirement or some other point.**

Additional complexity arises due to pipeline flushes because branch predictions are done speculatively. When a branch misprediction occurs, the processor state needs to be rolled back to the state immediately after the mispredicted branch. This includes rolling back the global history register, which may contain predicted branch results that were shifted-in by branches younger than the mispredicted branch, but now need to be discarded.

We assume here that there is hardware outside of the branch predictor that remembers the state of the branch history register that was used to predict each branch, which is saved for later use for branch predictor training and pipeline flushes. When a branch misprediction occurs, this hardware informs the branch predictor that a branch has mispredicted, the direction the branch should have taken, and the state of the branch history register corresponding to the point in the program immediately before the mispredicted branch.

Of course, since the processor restarts to the point after the mispredicted branch, the branch history register after the pipeline flush needs to have the actual direction of the mispredicted branch appended.

**Description** Build a 32-bit global history shift register, including support for rolling back state in response to a pipeline flush caused by a branch misprediction.

When a branch prediction is made (predict_valid = 1), shift in predict_taken from the LSB side to update the branch history for the predicted branch. (predict_history[0] is the direction of the youngest branch.)

When a branch misprediction occurs (train_mispredicted = 1), load the branch history register with the history after the completion of the mispredicted branch. This is the history before the mispredicted branch (train_history) concatenated with the actual result of the branch (train_taken).

If both a prediction and misprediction occur at the same time, the misprediction takes precedence, because the pipeline flush will also flush out the branch that is currently making a prediction.

predict_history is the value of the branch history register.

areset is an asynchronous reset that resets the history counter to zero.

看要求，无非是：
- 当predict_valid=1，将predict_taken移位进入predict_history（从LSB）。
- 当发生预测错误（train_mispredicted = 1），将发生错误之前的历史（train_history）移入预测历史（predict_history）覆盖原值，并将这一步正确的结果移位进入predict_history（从LSB）。
- train_mispredicted具有比predict_valid更高的优先级。
- 异步高电平置位areset将predict_history置为0。
## IV . (Cs450/gshare)

### 1.代码编写

```verilog
module top_module(
    input clk,
    input areset,

    input  predict_valid,
    input  [6:0] predict_pc,
    output predict_taken,
    output [6:0] predict_history,

    input train_valid,
    input train_taken,
    input train_mispredicted,
    input [6:0] train_history,
    input [6:0] train_pc
);
    reg pht1[127:0],pht0[127:0];//定义128个1位寄存器pht1

    wire [6:0]ad,ad2;
    assign predict_taken=pht1[ad2];
    assign ad=train_history^train_pc;
    assign ad2=predict_history^predict_pc;
    always@(posedge clk or posedge areset)begin
        if(areset)
            for(int i=0;i<128;i++)begin
                pht1[i]=0;pht0[i]=1;end
        else if(train_valid&train_taken)begin
            if({pht1[ad],pht0[ad]}==2'b11)
                {pht1[ad],pht0[ad]}<=2'b11;
            else
                {pht1[ad],pht0[ad]}<={pht1[ad],pht0[ad]}+1;
        end
        else if(train_valid&~train_taken)begin
            if({pht1[ad],pht0[ad]}==2'b00)
                {pht1[ad],pht0[ad]}<=2'b00;
            else
                {pht1[ad],pht0[ad]}<={pht1[ad],pht0[ad]}-1;
        end//之前的饱和计数器
    end
    always@(posedge clk or posedge areset)begin
        if(areset)
            predict_history<=0;
        else if(train_mispredicted & train_valid)
            predict_history <= {train_history[5:0],train_taken};
        else if(predict_valid )
            predict_history <= {predict_history[5:0],predict_taken}; 
    end
endmodule
```

### 2.提交结果

Success

### 3.题目分析


![image-20240227154046981](https://s2.loli.net/2024/02/27/nEUXkwaAThg7Wv4.png)

当train_valid=1时，PHT的位置由train_pc与train_history异或得到，而PHT的值则是由状态机或者说由train_taken来决定。状态机是之前题目中的饱和计数器，当状态是11或10时，会决定predict_taken为1. predict_taken的值有PHT的状态决定。

### Branch direction predictor

A branch direction predictor generates taken/not-taken predictions of the direction of conditional branch instructions. It sits near the front of the processor pipeline, and is responsible for directing instruction fetch down the (hopefully) correct program execution path. A branch direction predictor is usually used with a branch target buffer (BTB), where the BTB predicts the target addresses and the direction predictor chooses whether to branch to the target or keep fetching along the fall-through path.

Sometime later in the pipeline (typically at branch execution or retire), the results of executed branch instructions are sent back to the branch predictor to train it to predict more accurately in the future by observing past branch behaviour. There can also be pipeline flushes when there is a mispredicted branch.

[![img](https://s2.loli.net/2024/02/27/wtY46sBGbJ5Ddoe.png)](https://hdlbits.01xz.net/wiki/File:Branch_predictor.png)

Branch direction predictor located in the Fetch stage. The branch predictor makes a prediction using the current pc and history register, with the result of the prediction affecting the next pc value. Training and misprediction requests come from later in the pipeline.

For this exercise, the branch direction predictor is assumed to sit in the fetch stage of a hypothetical processor pipeline shown in the diagram on the right. This exercise builds only the branch direction predictor, indicated by the blue dashed rectangle in the diagram.

The branch direction prediction is a combinational path: The `pc` register is used to compute the taken/not-taken prediction, which affects the next-pc multiplexer to determine the value of `pc` in the next cycle.

Conversely, updates to the pattern history table (PHT) and branch history register take effect at the next positive clock edge, as would be expected for state stored in flip-flops.

### Gshare predictor

Branch direction predictors are often structured as tables of counters indexed by the program counter and branch history. The table index is a hash of the branch address and history, and tries to give each branch and history combination its own table entry (or at least, reduce the number of collisions). Each table entry contains a two-bit saturating counter to remember the branch direction when the same branch and history pattern executed in the past.

One example of this style of predictor is the gshare predictor[[1\]](https://hdlbits.01xz.net/wiki/Cs450/gshare#cite_note-1). In the gshare algorithm, the branch address (`pc`) and history bits "share" the table index bits. The basic gshare algorithm computes an N-bit PHT table index by xoring N branch address bits and N global branch history bits together.

The N-bit index is then used to access one entry of a 2N-entry table of two-bit saturating counters. The value of this counter provides the prediction (0 or 1 = not taken, 2 or 3 = taken).

Training indexes the table in a similar way. The training `pc` and history are used to compute the table index. Then, the two-bit counter at that index is incremented or decremented depending on the actual outcome of the branch.

#### References

1. [Jump up↑](https://hdlbits.01xz.net/wiki/Cs450/gshare#cite_ref-1) S. McFarling, "[Combining Branch Predictors](https://www.hpl.hp.com/techreports/Compaq-DEC/WRL-TN-36.pdf)", *WRL Technical Note TN-36*, Jun. 1993

### Description

Build a gshare branch predictor with 7-bit `pc` and 7-bit global history, hashed (using xor) into a 7-bit index. This index accesses a 128-entry table of two-bit saturating counters (similar to [cs450/counter_2bc](https://hdlbits.01xz.net/wiki/cs450/counter_2bc)). The branch predictor should contain a 7-bit global branch history register (similar to [cs450/history_shift](https://hdlbits.01xz.net/wiki/cs450/history_shift)).

The branch predictor has two sets of interfaces: One for doing predictions and one for doing training. The prediction interface is used in the processor's Fetch stage to ask the branch predictor for branch direction predictions for the instructions being fetched. Once these branches proceed down the pipeline and are executed, the true outcomes of the branches become known. The branch predictor is then trained using the actual branch direction outcomes.

When a branch prediction is requested (`predict_valid` = 1) for a given `pc`, the branch predictor produces the predicted branch direction and state of the branch history register used to make the prediction. The branch history register is then updated (at the next positive clock edge) for the predicted branch.

When training for a branch is requested (`train_valid` = 1), the branch predictor is told the `pc` and branch history register value for the branch that is being trained, as well as the actual branch outcome and whether the branch was a misprediction (needing a pipeline flush). Update the pattern history table (PHT) to train the branch predictor to predict this branch more accurately next time. In addition, if the branch being trained is mispredicted, also recover the branch history register to the state immediately after the mispredicting branch completes execution.

If training for a misprediction and a prediction (for a different, younger instruction) occurs in the same cycle, both operations will want to modify the branch history register. When this happens, training takes precedence, because the branch being predicted will be discarded anyway. If training and prediction of the same PHT entry happen at the same time, the prediction sees the PHT state before training because training only modifies the PHT at the next positive clock edge. The following timing diagram shows the timing when training and predicting PHT entry 0 at the same time. The training request at cycle 4 changes the PHT entry state in cycle 5, but the prediction request in cycle 4 outputs the PHT state at cycle 4, without considering the effect of the training request in cycle 4.

![image-20240227154206766](https://s2.loli.net/2024/02/27/gt2snYhfSzdZAFT.png)

`areset` is an asynchronous reset that clears the entire PHT to 2b'01 (weakly not-taken). It also clears the global history register to 0.
