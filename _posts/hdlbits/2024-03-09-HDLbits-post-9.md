---
layout:       post
title:        'HDLbits刷题中文完整版-9'
date:         2024-03-09
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

#### 3.2.5 Finite State Machines 

#### 3.2.5.1 Simple FSM 1 (asynchronous reset)

问题描述

>  
 这是一个具有两种状态的摩尔状态机，一种输入和一种输出。实现这个状态机。请注意，重置状态为 B。本练习与fsm1相同，但使用异步复位。 

<img alt="" height="104" src="https://s2.loli.net/2024/02/24/IRQf3gk4nHUrEqz.png" width="286">

 verilog代码

```verilog
module top_module(clk, reset, in, out);
    
    input clk;
    input reset;    
    input in;
    output out; 
    reg out;
 
	parameter A=0,B=1;
 
    reg present_state, next_state;
    
    always@(posedge clk)begin
        if(reset)
            present_state <= B;
        else
            present_state <= next_state;   
    end
    
    always @(*) begin
        if (reset) begin  
            next_state = B;
        end 
        else begin
            case (present_state)
                A :
                    if(in)
                        next_state <= A;
                	else
                        next_state <= B;
                B :
                    if(in)
                        next_state <= B;
                	else
                        next_state <= A; 
                default: ;
            endcase
        end
    end
        
    assign out = (present_state) ? 1'b1 : 1'b0;
    
endmodule
```

知识点

>  
 两段式状态机：最后使用组合逻辑对结果进行判断，但组合逻辑容易产生毛刺等不稳定因素。  




#### 3.2.5.2 Simple FSM 1 (synchronous reset)

问题描述

>  
 这是一个具有两种状态的摩尔状态机，一种输入和一种输出。实现这个状态机。请注意，重置状态为 B。本练习与fsm1相同，但使用同步复位。 

<img alt="" height="94" src="https://s2.loli.net/2024/02/24/IRQf3gk4nHUrEqz.png" width="258">

verilog代码

```verilog

module top_module(clk, reset, in, out);
    input clk;
    input reset;    // Synchronous reset to state B
    input in;
    output out;//  
    reg out;
    parameter A = 0, B = 1;
    reg present_state, next_state;
    always @(posedge clk) begin
        if (reset) begin  
            present_state <= B;
        end
        else
            present_state <= next_state;
    end
    always @(*)
        begin
            case(present_state)
                A:begin
                    if(in == 1'b1)
                        next_state = A;
                    else
                        next_state = B;
                end
                B:begin
                    if(in == 1'b1)
                        next_state = B;
                    else
                        next_state = A;
                end
            endcase
        end
    always @(*)
        begin
            if(present_state == B)
                out = 1'b1;
            else
                out = 1'b0;
        end
endmodule
```

知识点

>  
 三段式状态机：采用寄存器输出，可以避免毛刺，改善时序条件，但是三段式状态机分割了两部分组合逻辑（状态转移条件组合逻辑和输出组合逻辑），因此这条路径的时序相对紧张。  


#### 3.2.5.3 Simple FSM 2 (asynchronous reset)

问题描述

>  
 这是一个具有两个状态、两个输入和一个输出的摩尔状态机。实现这个状态机。使用异步复位。这是一个 JK 触发器。  

<img alt="" height="94" src="https://s2.loli.net/2024/02/24/sda6XiMTowt5zmI.png" width="278">

verilog代码

```verilog
module top_module(
    input clk,
    input areset,    // Asynchronous reset to OFF
    input j,
    input k,
    output out); //  

    parameter OFF=0, ON=1; 
    reg state, next_state;

    always @(*) begin
        // State transition logic
        case(state)
            OFF:begin
                if(j==1'b1)
                    next_state<=ON;
                else
                    next_state<=OFF; 
            end
            ON:begin
                if(k==1'b1)
                    next_state<=OFF;
                else
                    next_state<=ON; 
            end
        endcase
    end

    always @(posedge clk, posedge areset) begin
        // State flip-flops with asynchronous reset
        if(areset)
            state<=OFF;
        else
            state<=next_state;
    end

    // Output logic
    assign out = (state == ON);

endmodule
```

#### 3.2.5.4 Simple FSM 2 (synchronous reset)

问题描述

>  
 这是一个具有两个状态、两个输入和一个输出的摩尔状态机。实现这个状态机。 
 本练习与fsm2相同，但使用同步复位。 

<img alt="" height="84" src="https://s2.loli.net/2024/02/24/DNuX5IQGU4AkWtr.png" width="248">

verilog代码

```verilog
module top_module(
    input clk,
    input reset,    // Synchronous reset to OFF
    input j,
    input k,
    output out); //  

    parameter OFF=0, ON=1; 
    reg state, next_state;

    always @(*) begin
        // State transition logic
        case(state)
            OFF:begin
                if(j==1'b1)
                    next_state<=ON;
                else
                    next_state<=OFF;
            end
            ON:begin
                if(k==1'b1)
                    next_state<=OFF;
                else
                    next_state<=ON;
            end
        endcase
    end

    always @(posedge clk) begin
        // State flip-flops with synchronous reset
        if(reset)
            state<=OFF;
        else
            state<=next_state;
    end

    // Output logic
    // assign out = (state == ...);
    always @(*) begin
        if(state==ON)
            out<=1'b1;
        else
            out<=1'b0;
    end
endmodule

```

#### 3.2.5.5 Simple state transitions 3

问题描述

>   下面是一输入一输出四状态的摩尔状态机的状态转移表。使用以下状态编码：A=2'b00, B=2'b01, C=2'b10, D=2'b11。 
 仅实现此状态机的状态转换逻辑和输出逻辑（组合逻辑部分）。给定当前状态 ( state)，根据状态转换表 计算next_state和输出 (out )。 

<img alt="" height="175" src="https://s2.loli.net/2024/02/24/I8jdeEbwYfraFHV.png" width="168">

verilog代码 

```verilog
module top_module(
    input in,
    input [1:0] state,
    output [1:0] next_state,
    output out); //

    parameter A=0, B=1, C=2, D=3;

    // State transition logic: next_state = f(state, in)
    always @(*) begin
        case(state)
            A:begin
                if(in==1'b0)
                    next_state<=A;
                else 
                    next_state<=B;
            end
            B:begin
                if(in==1'b0)
                    next_state<=C;
                else 
                    next_state<=B;
            end
            C:begin
                if(in==1'b0)
                    next_state<=A;
                else 
                    next_state<=D;
            end
            D:begin
                if(in==1'b0)
                    next_state<=C;
                else 
                    next_state<=B;
            end           
        endcase
    end
    // Output logic:  out = f(state) for a Moore state machine
    always @(*) begin
        if(state==D)
            out<=1'b1;
        else
            out<=1'b0;  
    end
endmodule
```

#### 3.2.5.6 Simple one-hot state transitions 3

问题描述

>   下面是一输入一输出四状态的摩尔状态机的状态转移表。使用以下单热状态编码：A=4'b0001, B=4'b0010, C=4'b0100, D=4'b1000。 
 假设 one-hot 编码，通过检查导出状态转换和输出逻辑方程。仅实现此状态机的状态转换逻辑和输出逻辑（组合逻辑部分）。（测试台将使用非一个热输入进行测试，以确保您不会尝试做更复杂的事情）。 
 单热状态转换逻辑的逻辑方程可以通过查看状态转换图的边缘来导出。 

<img alt="" height="168" src="https://s2.loli.net/2024/02/24/ieIsXj6Ff1JWkyT.png" width="153">

verilog代码

```verilog
module top_module(
    input in,
    input [3:0] state,
    output [3:0] next_state,
    output out); 
    parameter A=0, B=1, C=2, D=3;
    assign next_state[A] = (state[A] & ~in)|(state[C] & ~in);
    assign next_state[B] = (state[A] & in)|(state[B] & in)|(state[D] & in);
    assign next_state[C] = (state[B] & ~in)|(state[D] & ~in);
    assign next_state[D] = (state[C] & in);
    assign out = (state[D]);
endmodule

```

知识点

>  
 独热码，在英文文献中称做 one-hot code, 直观来说就是有多少个状态就有多少比特，而且只有一个比特为1，其他全为0的一种码制。通常，在通信网络协议栈中，使用八位或者十六位状态的独热码，且系统占用其中一个状态码，余下的可以供用户使用。  
 例如，有6个状态的独热码状态编码为：000001，000010，000100，001000，010000，100000。 


#### 2.2.5.7 Simple FSM 3 (asynchronous reset)

问题描述

>  
 下面是一输入一输出四状态的摩尔状态机的状态转移表。实现这个状态机。包括将 FSM 重置为状态 A 的异步重置。 

<img alt="" height="173" src="https://s2.loli.net/2024/02/24/iwY1uQFbf2Ttolv.png" width="163">

verilog代码

```verilog
module top_module(
    input clk,
    input in,
    input areset,
    output out); 
    parameter A=2'b00,B=2'b01,C=2'b10,D=2'b11;
    reg [1:0] state,next_state;
    always @(posedge clk,posedge areset)
        begin
            if(areset)
                state <= A;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                A:begin
                    if(in)
                        next_state <= B;
                    else
                        next_state <= A;
                end
                B:begin
                    if(in)
                        next_state <= B;
                    else
                        next_state <= C;
                end
                C:begin
                    if(in)
                        next_state <= D;
                    else
                        next_state <= A;
                end
                D:begin
                    if(in)
                        next_state <= B;
                    else
                        next_state <= C;
                end
            endcase
        end
    assign out = (state == D);
endmodule
```

#### 2.2.5.8 Simple FSM 3 (synchronous reset)

问题描述

>  
 以下是一输入一输出四状态的摩尔状态机的状态转移表。实现这个状态机。包括将 FSM 重置为状态 A 的同步重置。（这与Fsm3的问题相同，但有同步重置。） 

<img alt="" height="167" src="https://s2.loli.net/2024/02/24/PBjJQbVGqIzy4mY.png" width="157">

verilog代码

```verilog
module top_module(
    input clk,
    input in,
    input reset,
    output out); 
    parameter A=2'b00,B=2'b01,C=2'b10,D=2'b11;
    reg [1:0] state,next_state;
    always @(posedge clk)
        begin
            if(reset)
                state <= A;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                A:begin
                    if(in)
                        next_state <= B;
                    else
                        next_state <= A;
                end
                B:begin
                    if(in)
                        next_state <= B;
                    else
                        next_state <= C;
                end
                C:begin
                    if(in)
                        next_state <= D;
                    else
                        next_state <= A;
                end
                D:begin
                    if(in)
                        next_state <= B;
                    else
                        next_state <= C;
                end
            endcase
        end
    assign out = (state == D);
endmodule
```

#### 2.2.5.9 Design a more FSM

问题描述

>  
 还包括一个高电平有效同步复位，它将状态机复位到相当于水位长时间处于低位的状态（没有传感器断言，并且所有四个输出都断言）。 

<img alt="" height="369" src="https://s2.loli.net/2024/02/24/fcrdUYo7es4BOMi.png" width="480">

verilog代码

```verilog
module top_module (
    input clk,
    input reset,
    input [3:1] s,
    output fr3,
    output fr2,
    output fr1,
    output dfr
); 
    parameter A=0,B=1,C=2,D=3,E=4,F=5;
    reg [2:0] state,next_state;
    always @(*)
        begin
            case(state)
                A:next_state = s[1]?B:A;
                B:next_state = s[2]?D:(s[1]?B:A);
                C:next_state = s[2]?D:(s[1]?C:A);
                D:next_state = s[3]?F:(s[2]?D:C);
                E:next_state = s[3]?F:(s[2]?E:C);
                F:next_state = s[3]?F:E;
                default:next_state = 'x;
            endcase
        end
    always @(posedge clk)
        begin
            if(reset)
                state <= A;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                A:{fr3,fr2,fr1,dfr} = 4'b1111;
                B:{fr3,fr2,fr1,dfr} = 4'b0110;
                C:{fr3,fr2,fr1,dfr} = 4'b0111;
                D:{fr3,fr2,fr1,dfr} = 4'b0010;
                E:{fr3,fr2,fr1,dfr} = 4'b0011;
                F:{fr3,fr2,fr1,dfr} = 4'b0000;
                default: {fr3,fr2,fr1,dfr} = 'x;
            endcase
        end
endmodule
```