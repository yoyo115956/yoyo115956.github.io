---
layout:       post
title:        'HDLbits刷题中文完整版-10'
date:         2024-03-10
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

#### 3.2.5.10 Lemmings 1

问题描述

>   Lemmings游戏涉及大脑相当简单的小动物。如此简单，我们将使用有限状态机对其进行建模。 
 在旅鼠的 2D 世界中，旅鼠可以处于以下两种状态之一：向左行走或向右行走。如果遇到障碍物，它会切换方向。特别是，如果 Lemming 撞到左边，它会向右走。如果它撞到右边，它会向左走。如果它同时在两侧碰撞，它仍然会切换方向。 
 实现一个具有两个状态、两个输入和一个输出的摩尔状态机来模拟这种行为。 

<img alt="" height="109" src="https://s2.loli.net/2024/02/24/BmUqt12eCilMGQT.png" width="495">

verilog代码

```verilog
module top_module(
    input clk,
    input areset,    // Freshly brainwashed Lemmings walk left.
    input bump_left,
    input bump_right,
    output walk_left,
    output walk_right);   
    parameter LEFT=0, RIGHT=1;
    reg state, next_state;
    always @(*)
        begin
            case(state)
                LEFT:begin
                    if(bump_left)
                        next_state = RIGHT;
                    else
                        next_state = LEFT;
                end
                RIGHT:begin
                    if(bump_right)
                        next_state = LEFT;
                    else
                        next_state = RIGHT;
                end
            endcase
        end
    always @(posedge clk, posedge areset) begin
        if(areset)
            state <= LEFT;
        else
            state <= next_state;
    end   
    assign walk_left = (state == LEFT);
    assign walk_right = (state == RIGHT);
```

#### 3.2.5.11 Lemmings 2

问题描述

>  
 除了左右行走之外，如果地面消失在旅鼠脚下，旅鼠还会摔倒（并且可能会“啊啊！”）。 
 除了左右走动和碰撞时改变方向外，当ground=0时，旅鼠会摔倒并说“啊啊！”。当地面重新出现 ( ground=1 ) 时，旅鼠将继续沿与坠落前相同的方向行走。跌倒时被撞不影响行走方向，与地面消失（但尚未跌倒）同一个周期被撞，或仍在跌倒时再次出现地面时，也不影响行走方向。 
 构建一个模拟这种行为的有限状态机。 

<img alt="" height="128" src="https://s2.loli.net/2024/02/24/UrBO2HTceWoIun1.png" width="409">

verilog代码

```verilog
module top_module(
    input clk,
    input areset,    // Freshly brainwashed Lemmings walk left.
    input bump_left,
    input bump_right,
    input ground,
    output walk_left,
    output walk_right,
    output aaah ); 
    parameter LEFT=2'b00,RIGHT=2'b01,FALL_L=2'b10,FALL_R=2'b11;
    reg [1:0] state,next_state;
    always @(*)
        begin
            case(state)
            LEFT:begin
                if(ground==1'b0) begin
                    next_state = FALL_L;
                end
                else if(bump_left)
                    next_state = RIGHT;
                else
                    next_state = LEFT;
            end
            RIGHT:begin
                if(ground==1'b0) begin
                    next_state = FALL_R;
                end
                else if(bump_right)
                    next_state = LEFT;
                else
                    next_state = RIGHT;
            end
            FALL_L:begin
                if(ground==1'b0)
                    next_state = FALL_L;
                else
                    next_state = LEFT;
            end
            FALL_R:begin
                if(ground==1'b0)
                    next_state = FALL_R;
                else
                    next_state = RIGHT;
            end
            endcase
        end
    always @(posedge clk,posedge areset)
        begin
            if(areset)
                state <= LEFT;
            else
                state <= next_state;
        end
    assign walk_left = (state == LEFT);
    assign walk_right = (state == RIGHT);
    assign aaah = ((state == FALL_L)||(state == FALL_R));
endmodule
```

#### 3.2.5.12 Lemmings 3

问题描述

>  
 除了走路和摔倒之外，旅鼠有时会被告知做一些有用的事情，比如挖掘（当dig=1时它开始挖掘）。如果旅鼠当前在地面上行走（ground=1并且没有下落），它可以挖掘，并且会继续挖掘直到它到达另一边（ground=0）。到那时，由于没有地面，它会掉下来（啊啊！），然后一旦再次撞到地面，就继续沿原来的方向行走。与坠落一样，挖掘时被撞到没有效果，并且在坠落或没有地面时被告知要挖掘被忽略。 
 （换句话说，一只行走的旅鼠可以跌倒、挖掘或切换方向。如果满足这些条件中的一个以上，则跌倒的优先级高于挖掘，挖掘的优先级高于切换方向。） 
 扩展您的有限状态机来模拟这种行为。   

<img alt="" height="177" src="https://s2.loli.net/2024/02/24/5ZpqCnPrhTwRU6b.png" width="413">

 verilog代码

```verilog
module top_module(
    input clk,
    input areset,    // Freshly brainwashed Lemmings walk left.
    input bump_left,
    input bump_right,
    input ground,
    input dig,
    output walk_left,
    output walk_right,
    output aaah,
    output digging ); 
    parameter LEFT=3'b000,RIGHT=3'b001,DIG_L=3'b010,DIG_R=3'b011,FALL_L=3'b100,FALL_R=3'b101;
    reg [2:0] state,next_state;
    always @(*)
        begin
            case(state)
                LEFT:begin
                    if(!ground)
                        next_state = FALL_L;
                    else if(dig)
                        next_state = DIG_L;
                    else if(bump_left)
                        next_state = RIGHT;
                    else
                        next_state = LEFT;
                end
                RIGHT:begin
                    if(!ground)
                        next_state = FALL_R;
                    else if(dig)
                        next_state = DIG_R;
                    else if(bump_right)
                        next_state = LEFT;
                    else
                        next_state = RIGHT;
                end
                DIG_L:begin
                    if(!ground)
                        next_state = FALL_L;
                    else
                        next_state = DIG_L;
                end
                DIG_R:begin
                    if(!ground)
                        next_state = FALL_R;
                    else
                        next_state = DIG_R;
                end
                FALL_L:begin
                    if(!ground)
                        next_state = FALL_L;
                    else
                        next_state = LEFT;
                end
                FALL_R:begin
                    if(!ground)
                        next_state = FALL_R;
                    else
                        next_state = RIGHT;
                end
            endcase
        end
    always @(posedge clk,posedge areset)
        begin
            if(areset)
                state <= LEFT;
            else
                state <= next_state;
        end
    assign walk_left = (state == LEFT);
    assign walk_right = (state == RIGHT);
    assign aaah = ((state == FALL_L)||(state == FALL_R));
    assign digging = ((state == DIG_L)||(state == DIG_R));
endmodule
```

#### 3.2.5.13  Lemmings 4

问题描述

>  
 虽然旅鼠可以行走、跌倒和挖掘，但旅鼠并非无懈可击。如果旅鼠跌落太久然后撞到地面，它可能会飞溅。特别是，如果 Lemming 跌落超过 20 个时钟周期然后撞到地面，它会飞溅并停止行走、跌落或挖掘（所有 4 个输出变为 0），永远（或直到 FSM 重置）。旅鼠在落地前可以坠落的距离没有上限。旅鼠只有在落地时才会飞溅；它们不会在半空中飞溅。 
 扩展您的有限状态机来模拟这种行为。 


>  
 跌倒20个周期是可以生存的： 

<img alt="" height="120" src="https://s2.loli.net/2024/02/24/nPN5utEH67d3pqx.png" width="502">

>  
 跌倒 21 个周期会导致飞溅： 

<img alt="" height="103" src="https://s2.loli.net/2024/02/24/VGKBy9duNA2wcrO.png" width="436">

>  
 使用 FSM 控制跟踪旅鼠下落时间的计数器。 


verilog代码

```verilog
module top_module(
    input clk,
    input areset,    // Freshly brainwashed Lemmings walk left.
    input bump_left,
    input bump_right,
    input ground,
    input dig,
    output walk_left,
    output walk_right,
    output aaah,
    output digging ); 
    parameter LEFT=3'b000,RIGHT=3'b001,DIG_L=3'b010,DIG_R=3'b011,FALL_L=3'b100,FALL_R=3'b101,SPLAT=3'b110,DEAD=3'b111;
    reg [2:0] state,next_state;
    reg [4:0] count;
    initial
        count = 5'b1;
    always @(posedge clk,posedge areset)
        begin
            if(areset)
                count <= 5'b1;
            else if((next_state == FALL_L)||(next_state == FALL_R))
                count <= count + 1'b1;
            else
                count <= 5'd1;
        end
    always @(*)
        begin
            case(state)
                LEFT:begin
                    if(!ground)
                        next_state = FALL_L;
                    else if(dig)
                        next_state = DIG_L;
                    else if(bump_left)
                        next_state = RIGHT;
                    else
                        next_state = LEFT;
                end
                RIGHT:begin
                    if(!ground)
                        next_state = FALL_R;
                    else if(dig)
                        next_state = DIG_R;
                    else if(bump_right)
                        next_state = LEFT;
                    else
                        next_state = RIGHT;
                end
                DIG_L:begin
                    if(!ground)
                        next_state = FALL_L;
                    else
                        next_state = DIG_L;
                end
                DIG_R:begin
                    if(!ground)
                        next_state = FALL_R;
                    else
                        next_state = DIG_R;
                end
                FALL_L:begin
                    if((!ground) &  & (count<=5'd20))
                        next_state = FALL_L;
                    else if((!ground) &  & (count>5'd20))
                        next_state = SPLAT;
                    else
                        next_state = LEFT;
                end
                FALL_R:begin
                    if((!ground) &  & (count<=5'd20))
                        next_state = FALL_R;
                    else if((!ground) &  & (count>5'd20))
                        next_state = SPLAT;
                    else
                        next_state = RIGHT;
                end
                SPLAT:begin
                    if(ground)
                        next_state = DEAD;
                    else
                        next_state = SPLAT;
                end
                DEAD:begin
                    next_state = DEAD;
                end
            endcase
        end
    always @(posedge clk,posedge areset)
        begin
            if(areset)
                state <= LEFT;
            else
                state <= next_state;
        end
    assign walk_left = (state == LEFT);
    assign walk_right = (state == RIGHT);
    assign aaah = ((state == FALL_L)||(state == FALL_R)||(state == SPLAT));
    assign digging = ((state == DIG_L)||(state == DIG_R));
endmodule
```