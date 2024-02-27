---
layout:       post
title:        'HDLbits刷题中文完整版-13'
date:         2024-03-13
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

#### 3.3 Building Large Circuits

#### 3.3.1 couter with 1000 

 问题描述

>  
 建立一个从 0 到 999 的计数器，包括 0 到 999，周期为 1000 个周期。复位输入是同步的，应将计数器复位为 0。 

<img alt="" src="https://s2.loli.net/2024/02/26/vezs9xnfI7Mc26g.png">

verilog代码

```verilog
module top_module (
    input clk,
    input reset,
    output [9:0] q);
    always @(posedge clk)begin
        if(reset)
            q<=10'd0;
        else if(q==10'd999)
            q<=10'd0;
        else 
            q<=q+10'd1;            
    end
endmodule
```

#### 3.3.2 4-bit shfit register and down counter

 问题描述

>  
 构建一个四位移位寄存器，该寄存器也用作递减计数器。当shift_ena为 1时，数据首先移入最高有效位。当count_ena为 1时，当前在移位寄存器中的数字递减。由于整个系统不会同时使用shift_ena和count_ena，因此您的电路无关紧要如果两个控制输入都为 1（这主要意味着哪种情况获得更高优先级并不重要）。 

<img alt="" src="https://s2.loli.net/2024/02/26/L6zE2fVMNXauUB8.png">

verilog代码

```verilog
module top_module (
    input clk,
    input shift_ena,
    input count_ena,
    input data,
    output [3:0] q);
    always @(posedge clk)
        begin
            if(shift_ena) begin
                q[0] <= data;
                q[1] <= q[0];
                q[2] <= q[1];
                q[3] <= q[2];
            end
            else if(count_ena)
                q <= q - 1'b1;
            else
                q <= q;
        end
endmodule

```

#### 3.3.3 FSM:Sequence 1101 recognizer

问题描述

>  
 构建一个有限状态机，在输入比特流中搜索序列 1101。找到序列后，应将start_shifting设置为 1，直到重置。陷入最终状态旨在模拟在尚未实现的更大 FSM 中进入其他状态。我们将在接下来的几个练习中扩展这个 FSM。 

<img alt="" src="https://s2.loli.net/2024/02/26/rxOuijKWB7PVtAf.png">

verilog代码

```verilog
module top_module (
    input clk,
    input reset,      // Synchronous reset
    input data,
    output start_shifting);
    parameter start=0,A=1,B=2,C=3,success=4;
    reg [2:0] state;
    reg [2:0] next_state;
    always @(*)begin
        case(state)
            start:
                begin
                    if(data)
                        next_state=A;
                    else
                        next_state=start;
                end
            A:
                begin
                    if(data)
                        next_state=B;
                    else
                        next_state=start;
                end
            B:
                begin
                    if(data)
                        next_state=B;
                    else
                        next_state=C;
                end
            C:
                begin
                    if(data)
                        next_state=success;
                    else
                        next_state=start;
                end
            success:
                next_state<=success;
        endcase
    end
    
    always @(posedge clk)begin
        if(reset)
            state<=start;
        else 
            state<=next_state;
    end
    
    assign start_shifting=(state==success);
endmodule
```

#### 3.3.4 FSM:Enable shift register

问题描述

>  
 作为用于控制移位寄存器的 FSM 的一部分，我们希望能够在检测到正确的位模式时启用移位寄存器恰好 4 个时钟周期。SM 的这一部分仅处理启用 4 个周期的移位寄存器。 

<img alt="" src="https://s2.loli.net/2024/02/26/FUhP4zLg3QbIXMa.png">

verilog代码

```verilog
module top_module (
    input clk,
    input reset,      // Synchronous reset
    output shift_ena);
    reg [3:0] counter;
    always @(posedge clk)
        begin
            if(reset)
                counter <= 4'b0;
            else if(shift_ena) begin
                if(counter >= 4'd3)
                    counter <= 4'b0;
                else
                    counter <= counter + 1'b1;
            end
        end
    always @(posedge clk)
        begin
            if(reset)
                shift_ena <= 1'b1;
            else if((shift_ena == 1'b1) & (counter >= 4'd3))
                shift_ena <= 1'b0;
        end
endmodule
```

#### 3.3.5 FSM:The complete FSM

问题描述

>  
 我们想创建一个计时器： 
 - 当检测到特定模式 (1101) 时开始，
 - 再移 4 位以确定延迟的持续时间，
 - 等待计数器完成计数，并且
 - 通知用户并等待用户确认计时器。 

在这个问题中，只实现控制定时器的有限状态机。此处不包括数据路径（计数器和一些比较器）。 
串行数据在数据输入引脚上可用。当接收到模式 1101 时，状态机必须断言输出shift_ena正好 4 个时钟周期。 
之后，状态机断言其计数输出以指示它正在等待计数器，并一直等到输入done_counting为高。 
此时，状态机必须断言完成以通知用户定时器已超时，并等待直到输入ack为 1，然后才被重置以寻找下一次出现的启动序列 (1101)。 
状态机应重置为开始搜索输入序列 1101 的状态。 
这是预期输入和输出的示例。'x' 状态读起来可能有点混乱。它们表明 FSM 不应该关心该周期中的特定输入信号。例如，一旦检测到 1101 模式，FSM 将不再查看数据输入，直到完成其他所有操作后恢复搜索。    

<img alt="" height="167" src="https://s2.loli.net/2024/02/26/DHlxtJNe2Ygs5nh.png" width="475">

 verilog代码

```verilog
module top_module (
    input clk,
    input reset,      // Synchronous reset
    input data,
    output shift_ena,
    output counting,
    input done_counting,
    output done,
    input ack );
    parameter find=0,A=1,B=2,C=3,success=4,count_state=5,done_state=6;
    reg [2:0] state;
    reg [2:0] next_state;
    reg [1:0] count;
    always @(posedge clk) begin
        if(state==success)
            count<=count+2'd1;
        else if(count==2'd3)
            count<=2'd0;
        else
            count<=2'd0;
    end
    
    always @(*) begin
        case(state)
            find:
                begin
                    if(data)
                        next_state=A;
                    else
                        next_state=find;
                end
            A:
                begin
                    if(data)
                        next_state=B;
                    else
                        next_state=find;
                end
            B:
                begin
                    if(data)
                        next_state=B;
                    else
                        next_state=C;
                end
            C:
                begin
                    if(data)
                        next_state=success;
                    else
                        next_state=find;
                end
            success:
                begin
                    if(count==2'd3)
                        next_state=count_state;
                    else
                        next_state=success;
                end
            count_state:
                begin
                    if(done_counting)
                        next_state=done_state;
                    else
                        next_state=count_state;
                end
            done_state:
                begin
                    if(ack)
                        next_state=find;
                    else
                        next_state=done_state;
                end          
        endcase
    end
    
    always @(posedge clk)begin
        if(reset)
            state<=find;
        else
            state<=next_state;
    end
    assign shift_ena=(state==success);
    assign counting=(state==count_state);
    assign done=(state==done_state);
endmodule

```

#### 3.3.6 The complete timer

问题描述

>  
 我们想创建一个带有一个输入的计时器： 
 - 当检测到特定输入模式 (1101) 时启动，
 - 再移 4 位以确定延迟的持续时间，
 - 等待计数器完成计数，并且
 - 通知用户并等待用户确认计时器。 

串行数据在数据输入引脚上可用。当接收到模式 1101 时，电路必须移入接下来的 4 位，首先是最高有效位。这 4 位决定了定时器延迟的持续时间。我将其称为delay[3:0]。 
之后，状态机断言其计数输出以指示它正在计数。状态机必须精确计数(delay[3:0] + 1) * 1000 个时钟周期。例如，delay=0 表示计数 1000 个周期，delay=5 表示计数 6000 个周期。同时输出当前剩余时间。这应该等于delay 1000 个周期，然后delay-1 1000 个周期，依此类推，直到 0 1000 个周期。当电路不计数时，count[3:0] 输出是无关紧要的（任何值方便您实现）。 
此时，电路必须断言done以通知用户计时器已超时，并等待输入ack为 1，然后再复位以查找下一次出现的启动序列 (1101)。 
电路应重置为开始搜索输入序列 1101 的状态。 
这是预期输入和输出的示例。'x' 状态读起来可能有点混乱。它们表明 FSM 不应该关心该周期中的特定输入信号。例如，一旦读取了 1101 和 delay[3:0]，电路就不再查看数据输入，直到在其他所有操作完成后恢复搜索。在本例中，电路计数 2000 个时钟周期，因为 delay[3:0] 值为 4'b0001。最后几个周期以 delay[3:0] = 4'b1110 开始另一个计数，它将计数 15000 个周期。 

<img alt="" height="128" src="https://s2.loli.net/2024/02/26/SQ3gq4UrOj6sDuX.png" width="433">

verilog代码

```verilog
module top_module (
    input clk,
    input reset,      // Synchronous reset
    input data,
    output [3:0] count,
    output counting,
    output done,
    input ack );
    parameter s=4'd0,s1=4'd1,s11=4'd2,s110=4'd3,b0=4'd4,b1=4'd5,b2=4'd6,b3=4'd7,Count=4'd8,WAIT=4'd9;
    reg [3:0] state,next_state;
    reg [3:0] temp_in;
    reg [15:0] counter;
    reg [3:0] a;
    wire done_counting;
    always @(posedge clk)
        begin
            if(reset)
                state <= s;
            else
                state <= next_state;
        end
    always @(posedge clk)
        begin
            if(reset)
                counter <= 16'd0;
            else if(next_state == WAIT)
                counter <= 16'd0;
            else if(next_state == Count)
                counter <= counter + 1'b1;
        end
    always @(*)
        begin
            if(counter <= 16'd1000)
                a = 4'd0;
            else if(counter <= 16'd2000)
                a = 4'd1;
            else if(counter <= 16'd3000)
                a = 4'd2;
            else if(counter <= 16'd4000)
                a = 4'd3;
            else if(counter <= 16'd5000)
                a = 4'd4;
            else if(counter <= 16'd6000)
                a = 4'd5;
            else if(counter <= 16'd7000)
                a = 4'd6;
            else if(counter <= 16'd8000)
                a = 4'd7;
            else if(counter <= 16'd9000)
                a = 4'd8;
            else if(counter <= 16'd10000)
                a = 4'd9;
            else if(counter <= 16'd11000)
                a = 4'd10;
            else if(counter <= 16'd12000)
                a = 4'd11;
            else if(counter <= 16'd13000)
                a = 4'd12;
            else if(counter <= 16'd14000)
                a = 4'd13;
            else if(counter <= 16'd15000)
                a = 4'd14;
            else
                a = 4'd15;
        end
    assign done_counting  = ((state == Count) & (counter == (temp_in + 1)*1000))?1'b1:1'b0;
    always @(*)
        begin
            case(state)
                s:next_state = data?s1:s;
                s1:next_state = data?s11:s;
                s11:next_state = data?s11:s110;
                s110:next_state = data?b0:s;
                b0:begin next_state = b1;temp_in[3] = data; end
                b1:begin next_state = b2;temp_in[2] = data; end
                b2:begin next_state = b3;temp_in[1] = data; end
                b3:begin next_state = Count;temp_in[0] = data; end
                Count:next_state = done_counting?WAIT:Count;
                WAIT:next_state = ack?s:WAIT;
            endcase
        end
    assign count = (state == Count)?(temp_in - a):4'd0;
    assign counting = (state == Count);
    assign done = (state == WAIT);
endmodule
```

#### 3.3.7 FSM:One-hot logic equations

问题描述

>  
 给定以下具有 3 个输入、3 个输出和 10 个状态的状态机： 

<img alt="" height="124" src="https://s2.loli.net/2024/02/26/kafAo5vTEhzICmN.png" width="549">

>  
 假设使用以下 one-hot 编码， 通过检查导出下一状态逻辑方程和输出逻辑方程： (S, S1, S11, S110, B0, B1, B2, B3, Count, Wait) = (10'b0000000001, 10 'b0000000010, 10'b0000000100, ... , 10'b1000000000) 
 假设 one-hot 编码，通过检查导出状态转换和输出逻辑方程。仅实现此状态机的状态转换逻辑和输出逻辑（组合逻辑部分）。（测试台将使用非一个热输入进行测试，以确保您不会尝试做更复杂的事情）。 
 编写生成以下等式的代码： 
 - B3_next -- next-state logic for state B1- S_next- S1_next- Count_next- Wait_next- done -- output logic- counting- shift_ena 
 单热状态转换逻辑的逻辑方程可以通过查看状态转换图的边缘来导出。  

<img alt="" height="121" src="https://s2.loli.net/2024/02/26/RbnO9sdXxDcy6UY.png" width="553">

verilog代码

```verilog
module top_module(
    input d,
    input done_counting,
    input ack,
    input [9:0] state,    // 10-bit one-hot current state
    output B3_next,
    output S_next,
    output S1_next,
    output Count_next,
    output Wait_next,
    output done,
    output counting,
    output shift_ena
); 
    parameter S=0, S1=1, S11=2, S110=3, B0=4, B1=5, B2=6, B3=7, Count=8, Wait=9;
 
    // assign B3_next = state[6];
    assign B3_next    = state[B2];
    assign S_next     = (~d & state[S]) | (~d & state[S1]) | (~d & state[S110]) | (ack & state[Wait]);
    assign S1_next    = d & state[S];
    assign Count_next = state[B3] | (~done_counting & state[Count]);
    assign Wait_next  = (done_counting & state[Count]) | (~ack & state[Wait]);
    assign done 	  = state[Wait];
    assign counting   = state[Count];
    assign shift_ena  = state[B0] | state[B1] | state[B2] | state[B3];


endmodule
```
