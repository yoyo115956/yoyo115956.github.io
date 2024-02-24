---
layout:       post
title:        'HDLbits刷题中文完整版-8'
date:         2024-03-08
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

#### 3.2.3 Shift Registers

#### 3.2.3.1 4-bit shift register

问题描述

>  
 构建一个 4 位移位寄存器（右移），具有异步复位、同步加载和启用。 
 - areset：将移位寄存器重置为零。
 - load ：用data[3:0]加载移位寄存器而不是移位。
 - ena：右移（q[3]变为零，q[0]移出并消失）。
 - q：移位寄存器的内容。 

如果load和ena输入都被置位 (1)，则load输入具有更高的优先级。 


 verilog代码

```verilog
module top_module(
    input clk,
    input areset,  // async active-high reset to zero
    input load,
    input ena,
    input [3:0] data,
    output reg [3:0] q); 
    always @(posedge clk or posedge areset) begin
        if(areset)
        	q<=4'd0;
        else if(load)
            q<=data;
        else
            if(ena)
                q<=q>>1;
            else
                q<=q;         
    end
endmodule

```

#### 3.2.3.2 Left/right rotator

问题描述

>  
 构建一个 100 位左/右旋转器，具有同步加载和左/右使能。与丢弃移出位并移入零的移位器不同，旋转器从寄存器的另一端移入移出的位。如果启用，旋转器会旋转位并且不会修改/丢弃它们。 
- load ：用data[99:0]加载移位寄存器而不是旋转。
- ena[1:0]：选择是否旋转以及旋转的方向。
  - 2'b01向右旋转一位
  - 2'b10向左旋转一位
  - 2'b00和2'b11不旋转。

- q : 旋转器的内容。 


verilog代码

```verilog
module top_module(
    input clk,
    input load,
    input [1:0] ena,
    input [99:0] data,
    output reg [99:0] q); 
    always @(posedge clk) begin
        if(load)
            q<=data;
        else if(ena==2'b01)
            q<={q[0],q[99:1]};
        else if(ena==2'b10)
            q<={q[98:0],q[99]};
        else
            q<=q;
    end
endmodule
```

#### 3.2.3.3 Left/right arithmetic shift by 1 or 8

问题描述

>  
 构建一个 64 位算术移位寄存器，同步加载。移位器可以向左和向右移动 1 位或 8 位位置，由数量选择。 
 算术右移补符号位（在这种情况下为q[63] ），而不是逻辑右移补零。考虑算术右移的另一种方法是，它假设被移动的数字是有符号的并保留符号，因此算术右移将带符号的数字除以 2 的幂。 
 逻辑左移和算术左移没有区别。 
- load ：用data[63:0]加载移位寄存器而不是移位。
- ena : 选择是否换档。
- amount：选择要移动的方向和量。
    - 2'b00：左移 1 位。
    - 2'b01：左移 8 位。
    - 2'b10：右移 1 位。
    - 2'b11：右移 8 位。

- q：移位器的内容。 

5 位数11000算术右移 1 是11100，而逻辑右移将产生01100。 
类似地，5 位数字01000算术右移 1 是00100，逻辑右移会产生相同的结果，因为原始数字是非负数。 


verilog代码

```verilog

module top_module(
    input clk,
    input load,
    input ena,
    input [1:0] amount,
    input [63:0] data,
    output reg [63:0] q); 
    always @(posedge clk)
        begin
            if(load)begin
                q <= data;
            end
            else begin
                if(ena)
                    begin
                        case(amount)
                            2'b00: q <= {q[62:0],1'b0};
                            2'b01: q <= {q[55:0],8'b0};
                            2'b10: q <= {q[63],q[63:1]};
                            2'b11: q <= {{8{q[63]}},q[63:8]};
                        endcase
                    end
                else
                    q <= q;
            end
        end                   
endmodule
```

知识点

 - **有**符号数，符号位为1，使用>>>，高位补1；
 - **有**符号数，符号位为0，使用>>>，高位补0（和>>相同）；
 - **无**符号数，无论最高位是什么，使用>>>，高位补0； 


#### 3.2.3.4 5-bit LFSR

问题描述

>  
 线性反馈移位寄存器是一种移位寄存器，通常带有几个异或门来产生移位寄存器的下一个状态。伽罗瓦 LFSR 是一种特殊的安排，其中带有“抽头”的位位置与输出位进行异或运算以产生其下一个值，而没有抽头位置进行移位。如果仔细选择抽头位置，则可以将 LFSR 设置为“最大长度”。n 位的最大长度 LFSR 在重复之前循环通过 2 的n次方-1 个状态（永远不会达到全零状态）。 
 下图显示了一个 5 位最大长度的 Galois LFSR，在位位置 5 和 3 处具有抽头。（抽头位置通常从 1 开始编号）。请注意，为了保持一致性，我在位置 5 处绘制了 XOR 门，但 XOR 门输入之一是 0。 
 构建这个 LFSR。复位应将 LFSR 复位为 1 。 

<img alt="" height="100" src="https://s2.loli.net/2024/02/23/cSbp9RMdXE6nq2x.png" width="476">

>  
 从 1 开始的前几个状态是00001 , 10100 , 01010 , 00101 , ... LFSR 应该在返回00001之前循环通过 31 个状态。 


verilog代码

```verilog
module top_module(
    input clk,
    input reset,    // Active-high synchronous reset to 5'h1
    output [4:0] q
); 
    always @(posedge clk) begin
        if(reset)
            q<=5'h1;
        else
        	begin
               q<=q>>1;
               q[2]<=q[0]^q[3];
               q[4]<=0^q[0];
            end       
    end
endmodule
```

#### 3.2.3.5 3-bit LFSR

问题描述

>   为此时序电路编写 Verilog 代码（子模块可以，但顶层必须命名为top_module）。假设您要在 DE1-SoC 板上实现电路。将R输入连接到SW开关，将 Clock 连接到KEY[0]，并将L连接到KEY[1]。将Q输出连接到红灯LEDR。 
 该电路是线性反馈移位寄存器(LFSR) 的一个示例。最大周期 LFSR 可用于生成伪随机数，因为它在重复之前循环通过 2 的n次方-1 个组合。全零组合不会出现在此序列中。 

<img alt="" height="168" src="https://s2.loli.net/2024/02/24/5W6pCZflQcUiVaT.png" width="450">

 verilog代码

```verilog

module top_module (
	input [2:0] SW,      // R
	input [1:0] KEY,     // L and clk
	output [2:0] LEDR);  // Q
    always @(posedge KEY[0])
        begin
            if(KEY[1])begin
                LEDR[2] <= SW[2];
                LEDR[1] <= SW[1];
                LEDR[0] <= SW[0];
            end
            else begin
                LEDR[2] <= LEDR[2]^LEDR[1];
                LEDR[1] <= LEDR[0];
                LEDR[0] <= LEDR[2];
            end
        end
endmodule
```

#### 3.2.3.6 32-bit LFSR

问题描述

>   在位位置 32、22、2 和 1 处构建具有抽头的 32 位 Galois LFSR。 
 这足够长，以至于您想要使用向量，而不是 32 个 DFF 实例。 


verilog代码 

```verilog
module top_module(
    input clk,
    input reset,    // Active-high synchronous reset to 32'h1
    output [31:0] q
);
    always @(posedge clk)
        begin
            if(!reset)begin
                q[31] <= q[0]^1'b0;
                q[21] <= q[22]^q[0];
                q[1] <= q[2]^q[0];
                q[0] <= q[1]^q[0];
                q[20:2] <= q[21:3];
                q[30:22] <= q[31:23];
            end
            else
                q <= 32'h1;
        end
endmodule
```

#### 3.2.3.7 Shift register

问题描述

>  
 实现以下电路： 

<img alt="" height="138" src="https://s2.loli.net/2024/02/24/tpqxi6mGSMDIUN2.png" width="418">

verilog代码

```verilog
module top_module (
    input clk,
    input resetn,   // synchronous reset
    input in,
    output out);
    reg in1,in2,in3;
    always @(posedge clk) begin
        if(!resetn)
            begin
            	out<=0;
                in1<=0;
                in2<=0;
                in3<=0;
            end
        else
            begin
                in1<=in;
                in2<=in1;
                in3<=in2;
                out<=in3;
            end
    end
endmodule
```

#### 3.2.3.8 Shift register

问题描述

>   考虑如下所示 的n位移位寄存器电路： 
 假设n = 4 ，为移位寄存器编写顶层 Verilog 模块（名为 top_module）。在顶层模块中实例化 MUXDFF 子电路的四个副本。假设您要在 DE2 板上实现电路。 
 - 将R输入连接到SW开关，
 - clk到KEY[0] ,
 - E到KEY[1] ,
 - L到KEY[2]，并且
 - w到KEY[3]。
 - 将输出连接到红灯LEDR[3:0]。 

<img alt="" height="170" src="https://s2.loli.net/2024/02/24/VTkbyY9RCz6KAxw.png" width="496">

  verilog代码

```verilog
module top_module (
    input [3:0] SW,
    input [3:0] KEY,
    output [3:0] LEDR
);
    MUXDFF u1 (.R(SW[0]),.clk(KEY[0]),.E(KEY[1]),.L(KEY[2]),.w(LEDR[1]),.Q(LEDR[0]));
    MUXDFF u2 (.R(SW[1]),.clk(KEY[0]),.E(KEY[1]),.L(KEY[2]),.w(LEDR[2]),.Q(LEDR[1]));
    MUXDFF u3 (.R(SW[2]),.clk(KEY[0]),.E(KEY[1]),.L(KEY[2]),.w(LEDR[3]),.Q(LEDR[2]));
    MUXDFF u4 (.R(SW[3]),.clk(KEY[0]),.E(KEY[1]),.L(KEY[2]),.w(KEY[3]),.Q(LEDR[3]));
endmodule
 
module MUXDFF (
    input R,
    input clk,
    input E,
    input L,
    input w,
    output Q
);
    always @(posedge clk)
        begin
            Q <= (L)?R:((E)?w:Q);
        end
endmodule
```

#### 3.2.3.9 3-input LUT

问题描述

>   在这个问题中，您将为 8x1 存储器设计一个电路，其中写入存储器是通过移入位来完成的，而读取是“随机访问”，就像在典型的 RAM 中一样。然后，您将使用该电路实现 3 输入逻辑功能。 
 首先，创建一个带有 8 个 D 型触发器的 8 位移位寄存器。标记来自 Q[0]...Q[7] 的触发器输出。移位寄存器输入应称为S，它馈入 Q[0] 的输入（首先移入 MSB）。使能输入控制是否移位。然后，将电路扩展为具有 3 个附加输入A、B、C和一个输出Z。电路的行为应该如下：当 ABC 为 000 时，Z=Q[0]，当 ABC 为 001 时，Z=Q[1]，依此类推。您的电路应该只包含 8 位移位寄存器和多路复用器。（旁白：该电路称为 3 输入查找表 (LUT)）。 


verilog代码

```verilog
module top_module (
    input clk,
    input enable,
    input S,
    input A, B, C,
    output Z ); 
    reg [7:0] q;
    always @(posedge clk)
        begin
            if(enable)begin
                q <= {q[6:0],S};
            end
            else begin
                q <= q;
            end
        end
    assign Z = q[{A,B,C}];
endmodule
```


#### 3.2.4 More Circuits

#### 3.2.4.1 Rule 90

问题描述

>   规则90是一个具有有趣特性的一维元胞自动机。 
 规则很简单。有一个一维的单元格数组（开或关）。在每个时间步，每个单元的下一个状态是单元的两个当前邻居的 XOR。下表是表达此规则的更详细的方式，其中单元格的下一个状态是其自身及其两个邻居的函数： 

<img alt="" height="222" src="https://s2.loli.net/2024/02/24/p2bP9FUjukTtWCL.png" width="198">

>  
 （“规则 90”的名称来自阅读“下一个状态”列：01011010 是十进制的 90。） 在此电路中，创建一个 512 单元系统 ( q[511:0] )，并在每个时钟周期前进一个时间步长。加载输入指示系统的状态应加载data[511:0]。假设边界（q[-1]和q[512]）都为零（关闭）。   


verilog代码

```verilog

module top_module(
    input clk,
    input load,
    input [511:0] data,
    output [511:0] q ); 
    always @(posedge clk)
        begin
            if(load)begin
                q <= data;
            end
            else begin
                q <= {1'b0,q[511:1]}^{q[510:0],1'b0};
            end
        end

endmodule
```

#### 3.2.4.2 Rule 110

问题描述

>  
 是一个具有有趣特性（例如）的一维元胞自动机。有一个一维的单元格数组（开或关）。在每个时间步，每个单元格的状态都会发生变化。在规则 110 中，每个单元格的下一个状态仅取决于它自己和它的两个邻居，如下表所示： 

<img alt="" height="230" src="https://img-blog.csdnimg.cn/30972232d75a49b98bef631d786f290e.png" >

>  
 （“规则 110”的名称来自阅读“下一个状态”列：01101110 是十进制的 110。） 
 在此电路中，创建一个 512 单元系统 ( q[511:0] )，并在每个时钟周期前进一个时间步长。加载输入指示系统的状态应加载data[511:0]。假设边界（q[-1]和q[512]）都为零（关闭）。 


verilog代码

```verilog
module top_module(
    input clk,
    input load,
    input [511:0] data,
    output [511:0] q
); 
    wire [511:0] q_left, q_right;
    assign q_left = {1'b0,q[511:1]};
    assign q_right = {q[510:0],1'b0};
    
    
    always@(posedge clk) begin
        if(load)
            q <= data;
        else begin
            q <= (q  &  ~q_right) | (~q_left  &  q_right) | (~q  &  q_right);
        end
    end
endmodule
```

#### 3.2.4.3 Conway's game of life 16*16

问题描述

>  
 康威的生命游戏是一个二维元胞自动机。 
 “游戏”是在一个二维单元格上进行的，其中每个单元格要么是 1（活着），要么是 0（死去）。在每个时间步，每个单元格都会根据它拥有的邻居数量来改变状态： 
 - 0-1 邻居：单元格变为 0。
 - 2个邻居：小区状态不变。
 - 3 个邻居：单元格变为 1。
 - 4 个以上的邻居：单元格变为 0。 

该游戏是为无限网格制定的。在这个电路中，我们将使用 16x16 网格。为了让事情更有趣，我们将使用一个 16x16 的环形，其中边环绕到网格的另一边。例如，角单元 (0,0) 有 8 个邻居：(15,1) , (15,0) , (15,15) , (0,1) , (0,15) , (1,1)、(1,0)和(1,15)。16x16 的网格由一个长度为 256 的向量表示，其中每行 16 个单元格由一个子向量表示：q[15:0] 为第 0 行，q[31:16] 为第 1 行，以此类推（此工具接受 SystemVerilog，因此您可以根据需要使用 2D 向量。） 

 - load：在下一个时钟沿将数据加载到q中，用于加载初始状态。
 - q：游戏的 16x16 当前状态，每个时钟周期更新。 


verilog代码 

```verilog

module top_module(
    input clk,
    input load,
    input [255:0] data,
    output [255:0] q ); 
    
    reg [3:0] counter;
    reg [323:0] data_padding;
    reg [255:0] q_next;
    
    //change state
    always @(posedge clk) begin
        if(load)
        	q <= data;
        else 
            q <= q_next;
    end
    
    //calculate q_next
    always @(*) begin
        data_padding[17:0] = {q[240],q[255:240],q[255]};
        data_padding[323:306] = {q[0],q[15:0],q[15]};
        for(int i=1;i<17;i++) begin
            data_padding[i*18 +:18] = {q[(i-1)*16],q[(i-1)*16 +: 16],q[i*16-1]};
        end
        for(int i=1;i<17;i++) begin
            for(int j=1;j<17;j++) begin
                counter = data_padding[18*i+j-1]+data_padding[18*i+j+1]+data_padding[18*(i-1)+j-1]
                +data_padding[18*(i-1)+j]+data_padding[18*(i-1)+j+1]
                +data_padding[18*(i+1)+j-1]+data_padding[18*(i+1)+j]+data_padding[18*(i+1)+j+1];
                case(counter)
                    2: q_next[16*(i-1)+j-1] <= q[16*(i-1)+j-1];
                    3: q_next[16*(i-1)+j-1] <= 1;
                    default: q_next[16*(i-1)+j-1] <= 0;
                endcase
            end
        end
    end
endmodule
```