---
layout:       post
title:        'HDLbits刷题中文完整版-11'
date:         2024-03-11
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

#### 3.2.5.14 one-hot FSM

 问题描述

>  
 给定以下具有 1 个输入和 2 个输出的状态机： 


<img alt="" height="162" src="https://s2.loli.net/2024/02/26/VlbsHGNXvJL34e1.png" width="539">

>  
 假设此状态机使用 one-hot 编码，其中state[0]到state[9]分别对应于状态 S0 到 S9。除非另有说明，否则输出为零。 
 实现状态机的状态转换逻辑和输出逻辑部分（但不是状态触发器）。您在state[9:0]中获得当前状态，并且必须生成next_state[9:0]和两个输出。假设 one-hot 编码，通过检查推导逻辑方程。（测试台将使用非一个热输入进行测试，以确保您不会尝试做更复杂的事情）。 
 单热状态转换逻辑的逻辑方程可以通过查看状态转换图的边缘来导出。   


verilog代码

```verilog
module top_module(
    input in,
    input [9:0] state,
    output [9:0] next_state,
    output out1,
    output out2);
    parameter s0=4'd0,s1=4'd1,s2=4'd2,s3=4'd3,s4=4'd4,s5=4'd5,s6=4'd6,s7=4'd7,s8=4'd8,s9=4'd9;
    assign next_state[s0] = ~in & (state[s0]|state[s1]|state[s2]|state[s3]|state[s4]|state[s7]|state[s8]|state[s9]);
    assign next_state[s1] = in & (state[s0]|state[s8]|state[s9]);
    assign next_state[s2] = in & (state[s1]);
    assign next_state[s3] = in & (state[s2]);
    assign next_state[s4] = in & (state[s3]);
    assign next_state[s5] = in & (state[s4]);
    assign next_state[s6] = in & (state[s5]);
    assign next_state[s7] = in & (state[s6]|state[s7]);
    assign next_state[s8] = ~in & (state[s5]);
    assign next_state[s9] = ~in & (state[s6]);
    
    assign out1 = state[s8]|state[s9];
    assign out2 = state[s7]|state[s9];
endmodule
```

#### 3.2.5.15 PS/2 packet parser

问题描述

>  
 PS/2 鼠标协议发送三个字节长的消息。然而，在连续的字节流中，消息的开始和结束位置并不明显。唯一的指示是每个三字节消息的第一个字节总是有bit[3]=1（但其他两个字节的bit[3]可能是1或0，具体取决于数据）。 
 我们想要一个有限状态机，当给定输入字节流时，它将搜索消息边界。我们将使用的算法是丢弃字节，直到我们看到一个带有bit[3]=1的字节。然后我们假设这是消息的第 1 个字节，并在收到所有 3 个字节后发出消息的接收完成信号（完成）。 
 FSM 应该在成功接收到每个消息的第三个字节后立即在循环中 发出完成信号。 
 一些时序图来解释所需的行为。 
 在无错误的情况下，每三个字节形成一条消息： 

<img alt="" height="126" src="https://s2.loli.net/2024/02/26/8JK3mzWEl1o5VQ2.png" width="406">

>  
  发生错误时，搜索字节 1： 

<img alt="" height="123" src="https://s2.loli.net/2024/02/26/MfkqsJpmt8c19K3.png" width="438">

>  
 请注意，这与1xx序列识别器不同。此处不允许重叠序列： 

<img alt="" height="110" src="https://s2.loli.net/2024/02/26/9pPZUTJ4qKdu3Vg.png" width="415">

>  
 尽管 in[7:0] 是一个字节，但 FSM 只有一个输入：in[3]。 你需要4个状态。三个状态可能不起作用，因为其中一个需要断言done，并且对于每个接收到的消息， done只断言一个周期。 


verilog代码

```verilog
module top_module(
    input clk,
    input [7:0] in,
    input reset,    // Synchronous reset
    output done);
    parameter BYTE1=2'b00,BYTE2=2'b01,BYTE3=2'b10,DONE=2'b11;
    reg [1:0] state,next_state;
    always @(*)
        begin
            case(state)
                BYTE1:begin
                    if(in[3])
                        next_state = BYTE2;
                    else
                        next_state = BYTE1;
                end
                BYTE2:next_state = BYTE3;
                BYTE3:next_state = DONE;
                DONE:begin
                    if(in[3])
                        next_state = BYTE2;
                    else
                        next_state = BYTE1;
                end
            endcase
        end
    always @(posedge clk)
        begin
            if(reset)
                state <= BYTE1;
            else
                state <= next_state;
        end
    assign done = (state == DONE);
endmodule

```

#### 3.2.5.13 PS/2 packet parser and datapath

问题描述

>   现在您有了一个状态机，可以识别 PS/2 字节流中的三字节消息，添加一个数据路径，该路径也将在收到数据包时输出 24 位（3 字节）消息（out_bytes[23:16]是第一个字节，out_bytes[15:8]是第二个字节，依此类推）。 
 每当断言完成信号时， out_bytes 都需要有效。您可以在其他时间输出任何内容（即，不关心）。例如： 

<img alt="" height="126" src="https://s2.loli.net/2024/02/26/FAKUj8WC2RMXDh3.png" width="537">

verilog代码 

```verilog
module top_module(
    input clk,
    input [7:0] in,
    input reset,    // Synchronous reset
    output [23:0] out_bytes,
    output done);
    parameter BYTE1=2'b00,BYTE2=2'b01,BYTE3=2'b10,DONE=2'b11;
    reg [1:0] state,next_state;
    always @(*)
        begin
            case(state)
                BYTE1:begin
                    if(in[3])
                        next_state = BYTE2;
                    else
                        next_state = BYTE1;
                end
                BYTE2:next_state = BYTE3;
                BYTE3:next_state = DONE;
                DONE:begin
                    if(in[3])
                        next_state = BYTE2;
                    else
                        next_state = BYTE1;
                end
            endcase
        end
    always @(posedge clk)
        begin
            if(reset)
                state <= BYTE1;
            else
                state <= next_state;
        end
    assign done = (state == DONE);
    always @(posedge clk)
        begin
            if((state==BYTE1)||(state==DONE) &  & (next_state==BYTE2)) begin
                out_bytes[23:16] <= in;
            end
            else if(state==BYTE2) begin
                out_bytes[15:8] <= in;
            end
            else if(state==BYTE3)
                out_bytes[7:0] <= in;
        end
endmodule
```

#### 3.2.5.14 Serial receiver

问题描述

>   在许多（较旧的）串行通信协议中，每个数据字节都与一个起始位和一个停止位一起发送，以帮助接收器从位流中划定字节。一种常见的方案是使用 1 个起始位 (0)、8 个数据位和 1 个停止位 (1)。当没有传输任何内容（空闲）时，该线路也处于逻辑 1。 
 设计一个有限状态机，当给定比特流时，它将识别何时正确接收到字节。它需要识别起始位，等待所有 8 个数据位，然后验证停止位是否正确。如果停止位未按预期出现，则 FSM 必须等到找到停止位后再尝试接收下一个字节。 
 一些时序图 
 无错误： 

<img alt="" height="89" src="https://s2.loli.net/2024/02/26/ZmNWFE5xXa7Ifk1.png" width="380">

>  
 未找到停止位。第一个字节被丢弃：  

<img alt="" height="100" src="https://s2.loli.net/2024/02/26/iFTEc5yugmYWCt8.png" width="417">

verilog代码

```verilog
module top_module(
    input clk,
    input in,
    input reset,    // Synchronous reset
    output done
); 
    parameter start=4'd0,D1=4'd1,D2=4'd2,D3=4'd3,D4=4'd4,D5=4'd5,D6=4'd6,D7=4'd7,D8=4'd8,stop=4'd9,idle=4'd10,WAIT=4'd11;
    reg [3:0] state,next_state;
    always @(*)
        begin
            case(state)
                start:next_state = D1;
                D1:next_state = D2;
                D2:next_state = D3;
                D3:next_state = D4;
                D4:next_state = D5;
                D5:next_state = D6;
                D6:next_state = D7;
                D7:next_state = D8;
                D8:begin
                    if(in)
                        next_state = stop;
                    else
                        next_state = WAIT;
                end
                stop:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = start;
                end
                idle:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = start;
                end
                WAIT:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = WAIT;
                end
                default:next_state = idle;
            endcase
        end
    always @(posedge clk)
        begin
            if(reset)
                state <= idle;
            else
                state <= next_state;
        end
    assign done = (state == stop);
endmodule
```

#### 3.2.5.15 Serial receiver and datapath

问题描述

>  
 现在您有了一个有限状态机，可以识别何时在串行比特流中正确接收到字节，添加一个数据路径来输出正确接收到的数据字节。out_byte需要在done为1时有效，否则不在乎。 
 请注意，串行协议首先发送最低有效位。 
 无错误： 

<img alt="" height="135" src="https://s2.loli.net/2024/02/26/ADRCV9NMpncbsqx.png" width="473">

 verilog代码

```verilog
module top_module(
    input clk,
    input in,
    input reset,    // Synchronous reset
    output [7:0] out_byte,
    output done
); 
    parameter start=4'd0,D1=4'd1,D2=4'd2,D3=4'd3,D4=4'd4,D5=4'd5,D6=4'd6,D7=4'd7,D8=4'd8,stop=4'd9,idle=4'd10,WAIT=4'd11;
    reg [3:0] state,next_state;
    reg [7:0] temp_in;
    always @(*)
        begin
            case(state)
                start: begin next_state = D1; temp_in[0] = in; end
                D1: begin next_state = D2; temp_in[1] = in; end
                D2: begin next_state = D3; temp_in[2] = in; end
                D3: begin next_state = D4; temp_in[3] = in; end
                D4: begin next_state = D5; temp_in[4] = in; end
                D5: begin next_state = D6; temp_in[5] = in; end
                D6: begin next_state = D7; temp_in[6] = in; end
                D7: begin next_state = D8; temp_in[7] = in; end
                D8:begin
                    if(in)
                        next_state = stop;
                    else
                        next_state = WAIT;
                end
                stop:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = start;
                end
                idle:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = start;
                end
                WAIT:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = WAIT;
                end
                default:next_state = idle;
            endcase
        end
    always @(posedge clk)
        begin
            if(reset)
                state <= idle;
            else
                state <= next_state;
        end
    assign done = (state == stop);
    assign out_byte = done?temp_in:8'b0;
endmodule
```

#### 3.2.5.16 Serial receiver with pariting checking

问题描述

>   我们想为串行接收器添加奇偶校验。奇偶校验在每个数据字节后增加一位。我们将使用奇校验，其中接收到的 9 位中1的数量必须是奇数。例如，101001011满足奇校验（有 5 个1 s），但001001011不满足。 
>   更改您的 FSM 和数据路径以执行奇校验检查。只有当一个字节被正确接收并且它的奇偶校验通过时，才断言完成信号。像串行接收器FSM，这个 FSM 需要识别起始位，等待所有 9 个（数据和奇偶校验）位，然后验证停止位是否正确。如果停止位未按预期出现，则 FSM 必须等到找到停止位后再尝试接收下一个字节。 
>   为您提供了以下模块，可用于计算输入流的奇偶校验（这是一个带复位的 TFF）。预期用途是应该给它输入比特流，并在适当的时间重置，以便计算每个字节 中1的比特数。 
>
>  ```verilog
>  module parity (
>      input clk,
>      input reset,
>      input in,
>      output reg odd);
>  
>      always @(posedge clk)
>          if (reset) odd <= 0;
>          else if (in) odd <= ~odd;
>  
>  endmodule
>  ```
>
>   请注意，串行协议先发送最低有效位，然后再发送 8 个数据位之后的奇偶校验位。 
>   没有构图错误。第一个字节奇校验通过，第二个字节失败。 

<img alt="" height="134" src="https://s2.loli.net/2024/02/26/xK1Jyp6aG7YPQTW.png" width="494">

 verilog代码

```verilog
module top_module(
    input clk,
    input in,
    input reset,    // Synchronous reset
    output [7:0] out_byte,
    output done
); 
    parameter idle = 3'd0,start = 3'd1,data = 3'd2,Parity = 3'd3,stop = 3'd4,WAIT = 3'd5;
    reg [2:0] state,next_state;
    reg [3:0] counter;
    reg [7:0] temp_in;
    wire odd;
    wire en;
    always @(posedge clk)
        begin
            if(reset)
                state <= idle;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                idle:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = start;
                end
                start:next_state = data;
                data:begin
                    if(counter == 4'd8)
                        next_state = Parity;
                    else
                        next_state = data;
                end
                Parity:begin
                    if(in)
                        next_state = stop;
                    else
                        next_state = WAIT;
                end
                stop:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = start;
                end
                WAIT:begin
                    if(in)
                        next_state = idle;
                    else
                        next_state = WAIT;
                end
            endcase
        end
    always @(posedge clk)
        begin
            if(reset) begin
                done <= 1'b0;
                out_byte <= 8'd0;
                counter <= 4'd0;
            end
            else begin
                case(next_state)
                    idle:begin
                        done <= 1'b0;
                        out_byte <= 8'd0;
                        counter <= 4'd0;
                    end
                    start:begin
                        done <= 1'b0;
                        out_byte <= 8'd0;
                        counter <= 4'd0;
                    end
                    data:begin
                        counter <= counter + 1'b1;
                        done <= 1'b0;
                        temp_in[counter] <= in;
                        out_byte <= 8'd0;
                    end
                    Parity:begin
                        done <= 1'b0;
                        out_byte <= 8'd0;
                        counter <= 4'd0;
                    end
                    stop:begin
                        if(odd == 1'b1) begin
                            done <= 1'b1;
                            out_byte <= temp_in;
                        end
                        else begin
                            done <= 1'b0;
                            out_byte <= 8'd0;
                        end
                    end
                    WAIT:begin
                        done <= 1'b0;
                        out_byte <= 8'd0;
                    end
                    default:begin
                        done <= 1'b0;
                        out_byte <= 8'd0;
                        counter <= 4'd0;
                    end
                endcase
            end
        end
    assign en = (reset == 1'b1 || next_state == idle || next_state == start);
    parity u1 (.clk(clk),.reset(en),.in(in),.odd(odd));            
endmodule
```

#### 3.2.5.17 Sequence recognition

问题描述

>  
 同步HDLC成帧涉及对数据的连续比特流进行解码，以寻找指示帧（数据包）开始和结束的比特模式。恰好看到 6 个连续的 1（即01111110）是指示帧边界的“标志”。为避免数据流意外包含“标志”，发送方在每 5 个连续的 1 后插入一个零，接收方必须检测并丢弃该 0。如果有 7 个或更多连续的 1，我们还需要发出错误信号。 
 创建一个有限状态机来识别这三个序列： 
 - 0111110 : 需要丢弃信号位（光盘）。
 - 01111110：标记帧的开始/结束（标志）。
 - 01111111...：错误（7 个或更多 1s）（err）。 

当 FSM 被重置时，它应该处于一个状态，就像之前的输入为 0 一样。 
以下是一些说明所需操作的示例序列。 
丢弃0111110： 

<img alt="" height="167" src="https://s2.loli.net/2024/02/26/Pdm3ie5Ayo2xJRH.png" width="485">

 标志01111110：

<img alt="" height="144" src="https://s2.loli.net/2024/02/26/oO7QIXTYyhE4Fnp.png" width="475">

 重置行为和错误01111111...：

<img alt="" height="135" src="https://s2.loli.net/2024/02/26/gQ3E7ws4ibK8SaN.png" width="475">

verilog代码

```verilog
module top_module(
    input clk,
    input reset,    // Synchronous reset
    input in,
    output disc,
    output flag,
    output err);
    parameter None=0,one=1,two=2,three=3,four=4,five=5,six=6,errstate=7,discstate=8,flagstate=9;
    reg [3:0] state;
    reg [3:0] next_state;
    always @(*) begin
        case(state)
            None:begin
                if(in==1'b1)
                    next_state=one;
                else
                    next_state=None;
            end
            one:begin
                if(in==1'b1)
                    next_state=two;
                else
                    next_state=None;
            end
            two:begin
                if(in==1'b1)
                    next_state=three;
                else
                    next_state=None;
            end
            three:begin
                if(in==1'b1)
                    next_state=four;
                else
                    next_state=None;
            end
            four:begin
                if(in==1'b1)
                    next_state=five;
                else
                    next_state=None;
            end
            five:begin
                if(in==1'b1)
                    next_state=six;
                else
                    next_state=discstate;
            end
            six:begin
                if(in==1'b1)
                    next_state=errstate;
                else
                    next_state=flagstate;
            end
            errstate:begin
                if(in==1'b1)
                    next_state=errstate;
                else
                    next_state=None;
            end
            discstate:begin
                if(in==1'b1)
                    next_state=one;
                else
                    next_state=None;
            end
            flagstate:begin
                if(in==1'b1)
                    next_state=one;
                else
                    next_state=None;
            end
        endcase
    end
    always @(posedge clk) begin
        if(reset)
            state<=None;
        else
            state<=next_state;
    end
    assign disc=(state==discstate);
    assign flag=(state==flagstate);
    assign err=(state==errstate);
endmodule
```

#### 3.2.5.18 Design a Mealy FSM

问题描述

>  
 实现一个Mealy类型的有限状态机，它可以识别名为x的输入信号上的序列“101” 。您的 FSM 应该有一个输出信号z，当检测到“101”序列时，它被断言为逻辑 1。您的 FSM 还应该有一个低电平有效的异步复位。您的状态机中可能只有 3 个状态。您的 FSM 应该能够识别重叠序列。 


verilog代码

```verilog
module top_module (
    input clk,
    input aresetn,    // Asynchronous active-low reset
    input x,
    output z );
    parameter wait1=0,one=1,two=2;
    reg [1:0] state;
    reg [1:0] next_state;
    always @(*) begin
        case(state)
            wait1:next_state = x ? one:wait1;
            one:next_state = x ? one:two;
            two:next_state=x?one:wait1;
        endcase
    end
    always @(posedge clk or negedge aresetn) begin
        if(!aresetn)
            state<=wait1;
        else
            state<=next_state;
    end
    always @(*) begin
        case(state)
            wait1:z =1'b0;
            one:z = 1'b0;
            two:z=x;
        endcase
    end
endmodule
```

知识点

>   1：输出只和当前状态有关而与输入无关，则称为摩尔（Moore）状态机； 
  2：输出不仅和当前状态有关而且和输入有关，则称为米利（Mealy）状态 