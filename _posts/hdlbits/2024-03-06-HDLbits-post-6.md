---
layout:       post
title:        'HDLbits刷题中文完整版-6'
date:         2024-03-06
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

### 3.2 Sequential Logic

#### 3.2.1 Latches and Flip-Flops

#### 3.2.1.1 D flip-flop

问题描述

>  
 D触发器是一种存储位并定期更新的电路，通常在时钟信号的上升沿触发。 
 当使用时钟控制的always 块时，逻辑合成器会创建 D 触发器。D触发器是“组合逻辑块后接触发器”的最简单形式，其中组合逻辑部分只是一条线。创建一个 D 触发器。 

<img alt="" src="https://s2.loli.net/2024/02/21/sXZ8oflnR2IwQJa.png">

 verilog代码

```verilog
module top_module (
    input clk,    // Clocks are used in sequential circuits
    input d,
    output reg q );//

    // Use a clocked always block
    //   copy d to q at every positive edge of clk
    //   Clocked always blocks should use non-blocking assignments
    always @(posedge clk) begin
        q<=d;
    end
endmodule

```

#### 3.2.1.2 D flip-flops

问题描述

>  
 创建 8 位 D 触发器。所有 DFF 都应由clk的上升沿触发。 


verilog代码

```verilog
module top_module (
    input clk,
    input [7:0] d,
    output [7:0] q
);
    always @(posedge clk) begin
        q<=d;
    end
endmodule
```

#### 3.2.1.3 DFF with reset

问题描述 

>  
  创建 8 位具有高电平有效同步复位的 D 触发器。所有 DFF 都应由clk的上升沿触发。 


verilog代码

```verilog
module top_module (
    input clk,
    input reset,            // Synchronous reset
    input [7:0] d,
    output [7:0] q
);
    always@(posedge clk) begin
       if(reset)
           q<=8'd0;
       else
           q<=d; 
    end
endmodule

```

#### 3.2.1.4 DFF with reset value

问题描述

>  
 创建 8 位具有高电平有效同步复位的 D 触发器。触发器必须重置为 0x34 而不是零。所有 DFF 都应由**clk**的下降沿触发。将寄存器重置为“1”有时称为“预设” 


verilog代码

```verilog
module top_module (
    input clk,
    input reset,
    input [7:0] d,
    output [7:0] q
);
    always @(negedge clk)
        begin
            if(reset)
                q <= 8'h0x34;
            else
                q <= d;
        end
endmodule
```

#### 3.2.1.5 DFF with asynchronous reset

问题描述 

>  
 创建 8 位具有高电平有效异步复位的 D 触发器。所有 DFF 都应由clk的上升沿触发。 
 同步和异步复位触发器之间代码的唯一区别在于灵敏度列表。 


 verilog代码

```verilog
module top_module (
    input clk,
    input areset,   // active high asynchronous reset
    input [7:0] d,
    output [7:0] q
);
    always @(posedge clk or posedge areset) begin
        if(areset)
            q<=8'd0;
        else
            q<=d;
    end
endmodule

```

#### 3.2.1.6 DFF with byte enable

问题描述

>  
 创建 16 位D 触发器。有时只修改一组触发器的一部分很有用。字节使能输入控制 16 个寄存器中的每个字节是否应在该周期写入。byteena[1]控制高字节d[15:8]，而byteena[0]控制低字节d[7:0]。resetn是一个同步的低电平有效复位。所有 DFF 都应由clk的上升沿触发。 


verilog代码

```verilog
module top_module (
    input clk,
    input resetn,
    input [1:0] byteena,
    input [15:0] d,
    output [15:0] q
);
    always @(posedge clk) begin
        if(!resetn)
            q<=16'd0;
        else 
            begin
                q[7:0]<=byteena[0]?d[7:0]:q[7:0];
                q[15:8]<=byteena[1]?d[15:8]:q[15:8];
            end
    end
endmodule
```

#### 3.2.1.7 D Latch

问题描述

>   实现以下电路： 
 请注意，这是一个锁存器，因此预计会出现关于已推断锁存器的 Quartus 警告。锁存器是电平敏感（非边沿敏感）电路，因此在一个始终块中，它们使用电平敏感灵敏度列表。但是，它们仍然是顺序元素，因此应该使用非阻塞赋值，D 锁存器在启用时就像一条线，在禁用时保留当前值。 

<img alt="" height="69" src="https://s2.loli.net/2024/02/21/SLQ4rKVMcqku6pa.png" width="106">

verilog代码 

```verilog
module top_module (
    input d, 
    input ena,
    output q);
    always @(*)
        begin
            if(ena)
                q <= d;
        end
endmodule
```

#### 3.2.1.8 DFF

 问题描述

>  
 实现以下电路： 

<img alt="" height="84" src="https://s2.loli.net/2024/02/21/quA17K6soyMJUIb.png" width="96">

verilog代码

```verilog
module top_module (
    input clk,
    input d, 
    input ar,  //异步复位
    output q);
    always @(posedge clk or posedge ar)
        begin
            if(ar)
                q <= 1'b0;
            else
                q <= d;
        end
endmodule
```

#### 3.2.1.9 DFF

 问题描述

>  
 实现以下电路： 



<img alt="" height="71" src="https://s2.loli.net/2024/02/21/5qMt9nubNKsPJ2G.png" width="82">

verilog代码

```verilog
module top_module (
    input clk,
    input d, 
    input r,   // synchronous reset 同步复位
    output q);
    always @(posedge clk)
        begin
            if(!r)
                q <= d;
            else
                q <= 1'b0;
        end
endmodule
```

#### 3.2.1.10 DFF+gate

问题描述

>   实现以下电路 

![Exams m2014q4d.png](https://s2.loli.net/2024/02/21/lmQ2Uf5gVB6JHtN.png)


verilog代码

```verilog
module top_module (
    input clk,
    input in, 
    output out);
    always @(posedge clk) begin
        out<=in^out;
    end
endmodule
```

#### 3.2.1.11 Mux and DFF

问题描述

>  
 考虑下面的时序电路，假设您要为此电路实现分层 Verilog 代码，使用其中具有触发器和多路复用器的子模块的三个实例化。为此子模块编写一个名为top_module的 Verilog 模块（包含一个触发器和多路复用器）。 

<img alt="" height="174" src="https://s2.loli.net/2024/02/21/IE6RLyClNmhbM1s.png" width="450">

verilog代码

```verilog
module top_module (
	input clk,
	input L,
	input r_in,
	input q_in,
	output reg Q);
    always @(posedge clk)
        begin
            Q <= (L)?r_in:q_in;
        end
endmodule
```

#### 3.2.1.12 Mux and DFF

问题描述

>  
 考虑如下所示的n位移位寄存器电路，为该电路的一个阶段编写一个名为 top_module 的 Verilog 模块，包括触发器和多路复用器。  

<img alt="" height="143" src="https://s2.loli.net/2024/02/21/9ZHKEMJVgaoT4Ip.png" width="414">

verilog代码

```verilog
module top_module (
    input clk,
    input w, R, E, L,
    output Q
);
    always @(posedge clk) begin
        Q<=L?R:(E?w:Q);
    end
endmodule
```

#### 3.2.1.13 DFFs and gates

问题描述

>  
 给定如图所示的有限状态机电路，假设 D 触发器在机器开始之前初始复位为零，建立这个电路。小心复位状态。确保每个 D 触发器的Q非·输出确实是其 Q 输出的倒数，即使在模拟的第一个时钟沿之前也是如此。  

<img alt="" height="255" src="https://s2.loli.net/2024/02/21/4QdNt3KCATlVpm1.png" width="345">

verilog代码

```verilog
module top_module (
    input clk,
    input x,
    output z
); 
    reg q1,q2,q3;
    always @(posedge clk) begin
            q1 <= x^q1;
            q2 <= x & (~q2);
            q3 <= x|(~q3);
    end
    assign z = ~(q1|q2|q3);
endmodule
```

#### 3.2.1.14 Create circuit from truth table

问题描述

>  
 JK 触发器具有以下真值表。实现一个只有 D 型触发器和门的 JK 触发器。注意：Qold 是正时钟沿之前 D 触发器的输出。 

<img alt="" height="147" src="https://s2.loli.net/2024/02/21/jH82xgbZ1BVoktE.png" width="109">

verilog代码

```verilog
module top_module (
    input clk,
    input j,
    input k,
    output Q); 
    always @(posedge clk) begin
        case({j,k})
            2'b00: Q <= Q;
            2'b01: Q <= 0;
            2'b10: Q <= 1;
            2'b11: Q <= ~Q;
        endcase
    end
endmodule
```

#### 3.2.1.15 Detect an edge

问题描述

>  
 对于 8 位向量中的每一位，检测输入信号何时从一个时钟周期的 0 变为下一个时钟周期的 1（类似于上升沿检测）。输出位应在发生 0 到 1 转换后的周期设置。这里有些例子，为清楚起见，in[1] 和 pedge[1] 分别显示如下。 

<img alt="" height="119" src="https://s2.loli.net/2024/02/21/mBjOz3tax6hfoHZ.png" width="461">

 verilog代码

```verilog
module top_module (
    input clk,
    input [7:0] in,
    output [7:0] pedge
);
    reg [7:0] temp_in;
    always @(posedge clk) begin
        temp_in <= in;
        pedge <= ~temp_in & in;
    end
endmodule
```

#### 3.2.1.16 Detect both edges

问题描述

>  
 对于 8 位向量中的每一位，检测输入信号从一个时钟周期变为下一个时钟周期时发生变化（检测任何边沿）。输出位应在发生 0 到 1 转换后的周期设置。这里有些例子。为清楚起见，in[1] 和 anyedge[1] 分别显示如下： 


<img alt="" height="109" src="https://s2.loli.net/2024/02/21/cFAmxXC9KaNnuRo.png" width="448">

verilog代码

```verilog
module top_module (
    input clk,
    input [7:0] in,
    output [7:0] anyedge
);
    reg [7:0] temp_in;
    always @(posedge clk) begin
        temp_in <= in;
        anyedge <= temp_in^in;
    end
endmodule
```

#### 3.2.1.17 Edge capture register

问题描述

>  对于 32 位向量中的每一位，在输入信号从一个时钟周期的 1 变为下一个时钟周期的 0 时进行捕捉。“捕获”表示输出将保持为 1，直到寄存器复位（同步复位）。
>
>  每个输出位的行为类似于 SR 触发器：输出位应在 1 到 0 转换发生后的周期设置（为 1）。当复位为高电平时，输出位应在正时钟沿复位（为 0）。如果上述两个事件同时发生，则重置优先。在下面示例波形的最后 4 个周期中，“reset”事件比“set”事件早一个周期发生，因此这里不存在冲突。
>
>  在下面的示例波形中，为清楚起见，reset、in[1] 和 out[1] 再次分别显示如下。 
![image-20240221184208174](https://s2.loli.net/2024/02/21/u3IC2wlmYAKtjnM.png)


verilog代码

```verilog
module top_module (
    input clk,
    input reset,
    input [31:0] in,
    output [31:0] out
);
    reg [31:0] temp_in;
    always @(posedge clk) begin
        temp_in <= in;
        if(reset)
            out <= 32'b0;
        else
            out <= temp_in & ~in|out;
    end
endmodule
```

#### 3.2.1.18 Dual-edge triggered flip-flop

问题描述

>   您熟悉在时钟上升沿或时钟下降沿触发的触发器。在时钟的两个边沿触发双边触发触发器。但是，FPGA 没有双边触发触发器，并且始终不接受 @(posedge clk 或 negedge clk)作为合法的敏感度列表。 
 构建一个功能类似于双边触发触发器的电路： 
 （注意：它不一定完全等效：触发器的输出没有毛刺，但模拟这种行为的更大组合电路可能会。但我们将在这里忽略这个细节。） 
![image-20240221185407391](https://s2.loli.net/2024/02/21/hfoAQ6jDINsgdS3.png)

 - 您无法在 FPGA 上创建双边触发触发器。但是您可以同时创建正沿触发和负沿触发触发器。
 - 这个问题是一个中等难度的电路设计问题，但只需要基本的 Verilog 语言特性。（这是电路设计问题，而不是编码问题。）在尝试编码之前先用手画出电路图可能会有所帮助。 


verilog代码

```verilog
module top_module (
    input clk,
    input d,
    output q
);
    reg q1,q2;
    always @(posedge clk) begin
            q1 <= d;
    end
    always @(negedge clk) begin
            q2 <= d;
    end
    assign q = clk?q1:q2;
endmodule
```