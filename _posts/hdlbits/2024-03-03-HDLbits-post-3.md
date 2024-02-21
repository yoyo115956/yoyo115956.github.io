---
layout:       post
title:        'HDLbits刷题中文完整版-3'
date:         2024-03-03
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

### 2.4 Procedures

#### 2.4.1 Always blocks(combinational)

问题描述

>  
 使用 assign 语句和组合 always 块构建 AND 门。 



<img alt="" height="284" src="https://s2.loli.net/2024/02/08/G4R6YWfCgq95FdK.png" width="503">

verilog代码 

```verilog
module top_module(
    input a, 
    input b,
    output wire out_assign,
    output reg out_alwaysblock
);
    assign out_assign = a & b;
    always@(*)begin
        out_alwaysblock=a & b;   
    end
endmodule
```

知识点

>   **assign 组合逻辑和always@(*)组合逻辑** 
 描述组合逻辑一般常用的有两种：assign赋值语句和always@(*)语句。两者之间的差别有： 
        1. 被assign赋值的信号定义为wire型，被always@(*)结构块下的信号定义为reg型，值得注意的是，这里的reg并不是一个真正的触发器，只有敏感列表为上升沿触发的写法才会综合为触发器，在仿真时才具有触发器的特性。 
              . 另外一个区别则是更细微的差别：举个例子, 
                   wire a; 
                     reg b; 
                  assign a = 1'b0; 
                  always@(*) 
                      b = 1'b0; 
                   在这种情况下，做仿真时a将会正常为0， 但是b却是不定态。这是为什么？verilog规定，always@(*)中的*是指该always块内的所有输入信号的变化为敏感列表，也就是仿真时只有当always@(*)块内的输入信号产生变化，该块内描述的信号才会产生变化，而像always@(*) b = 1'b0; 
                   这种写法由于1'b0一直没有变化，所以b的信号状态一直没有改变，由于b是组合逻辑输出，所以复位时没有明确的值（不定态），而又因为always@(*)块内没有敏感信号变化，因此b的信号状态一直保持为不定态。 



#### 2.4.2 Always blocks(blocked)

问题描述 

>   以三种方式构建异或门，使用分配语句、组合块和时钟块。请注意，时钟块产生的电路与其他两个不同：有一个触发器，因此输出被延迟。 

阻塞与非阻塞赋值
Verilog 中有三种类型的赋值：

连续赋值（赋值 x = y;）。只能在不在过程内部时使用（"始终阻塞"）。
过程阻塞赋值：（x = y;）。只能在过程内部使用。
程序非阻塞赋值：（x <= y;）。只能在存储过程内部使用。

在组合式始终块中，使用阻塞式赋值。在时钟始终块中，使用非阻塞赋值。要完全理解原因，对硬件设计并没有特别大的帮助，而且需要很好地理解 Verilog 模拟器是如何跟踪事件的。不遵循这一规则会导致极难发现的错误，这些错误既是非确定性的，又在仿真和综合硬件之间存在差异。

<img alt="" height="274" src="https://s2.loli.net/2024/02/08/wESNiQcljnGAZhR.png" width="393">



 verilog代码

```verilog
module top_module(
    input clk,
    input a,
    input b,
    output wire out_assign,
    output reg out_always_comb,
    output reg out_always_ff   );
    assign out_assign = a^b;
    always @(*)
        begin
            out_always_comb = a^b;
        end
    always @(posedge clk)
        begin
           out_always_ff <= a^b; 
        end
endmodule

```

#### 2.4.3 If statement

 问题描述

>  
 构建一个在a和b之间进行选择的 2 对 1 多路复用器。如果两个 sel_b1和sel_b2是真，选择b。否则，选择a。做同样的事情两次，一次使用分配语句，一次使用过程 if 语句。 


verilog代码

```verilog
// synthesis verilog_input_version verilog_2001
module top_module(
    input a,
    input b,
    input sel_b1,
    input sel_b2,
    output wire out_assign,
    output reg out_always   ); 
    
    always @(*)
        begin
            if(sel_b1 == 1'b0) begin
                out_always = a;
            end
            else if(sel_b2 == 1'b0) begin
                out_always = a;
            end
            else begin
                out_always = b;
            end
        end
    assign out_assign = (sel_b1)?((sel_b2)?b:a):a;
endmodule

```

#### 2.4.4 If statement latches

问题描述

>  
 以下代码包含创建闩锁的不正确行为。修复错误。  

![img](https://s2.loli.net/2024/02/08/4xSgDkYhN8GKuya.png)

```verilog
always @(*) begin
    if (cpu_overheated)
       shut_off_computer = 1;
end

always @(*) begin
    if (~arrived)
       keep_driving = ~gas_tank_empty;
end
```

 verilog代码

```verilog
module top_module (
    input      cpu_overheated,
    output reg shut_off_computer,
    input      arrived,
    input      gas_tank_empty,
    output reg keep_driving  ); //
 
    always @(*) begin
        if (cpu_overheated)
           shut_off_computer = 1;
        else 
           shut_off_computer = 0;
    end
 
    always @(*) begin
        if (~arrived)
           keep_driving = ~gas_tank_empty;
        else
           keep_driving = 0; 
    end
endmodule

```

#### 2.4.5 Case statement

 问题描述

>  
 如果有大量 case，case 语句比 if 语句更方便。因此，在本练习中，创建一个 6 对 1 多路复用器。当sel在 0 到 5 之间时，选择对应的数据输入。否则，输出 0。数据输入和输出均为 4 位宽。小心推断锁存器。 


 verilog代码

```verilog
module top_module ( 
    input [2:0] sel, 
    input [3:0] data0,
    input [3:0] data1,
    input [3:0] data2,
    input [3:0] data3,
    input [3:0] data4,
    input [3:0] data5,
    output reg [3:0] out   );
 
    always@(*) begin
        case(sel)
                3'b000: out = data0;
                3'b001: out = data1;
                3'b010: out = data2;
                3'b011: out = data3;
                3'b100: out = data4;
                3'b101: out = data5;
                default: out = 0;
        endcase
    end
 
endmodule

```

#### 2.4.6 Priority encoder

问题描述

>  
 优先级编码器是一个组合电路，给定一个输入位向量时，输出第一的位置1中的向量位。例如，给定输入8'b100 1 0000的 8 位优先级编码器将输出3'd4，因为 bit[4] 是第一个高位。 
 构建一个 4 位优先级编码器。对于这个问题，如果没有一个输入位为高（即输入为零），则输出零。请注意，一个 4 位数字有 16 种可能的组合。 


Verilog代码

```verilog
module top_module (
    input [3:0] in,
    output reg [1:0] pos  );
    always@(*)begin
        case(1)
            in[0]:pos=2'd0;
            in[1]:pos=2'd1;
            in[2]:pos=2'd2;
            in[3]:pos=2'd3;
            default:pos=2'd0;
        endcase    
    end
endmodule
```

#### 2.4.7 Priority encoder with casez

问题描述

>   为 8 位输入构建优先级编码器。给定一个 8 位向量，输出应报告向量中的第一位1。如果输入向量没有高位，则报告零。例如，输入8'b100 1 0000应该输出3'd4，因为 bit[4] 是第一个高位。如果 case 语句中的 case 项支持 don't-care 位，我们可以减少这个（减少到 9 个 case）。这就是z 的情况：它在比较中将具有值z 的位视为不关心。 
 case 语句的行为就好像每个项目都是按顺序检查的（实际上，它更像是生成一个巨大的真值表然后制作门）。请注意某些输入（例如4'b1111）如何匹配多个案例项目。选择第一个匹配项（因此4'b1111匹配第一项，out = 0，但不匹配任何后面的项）。 
 - 还有一个类似的casex将x和z 都视为无关。我认为在casez上使用它没有多大意义。- 数字？是z的同义词。所以2'bz0和2'b?0 一样 


verilog代码 

```verilog

module top_module (
    input [7:0] in,
    output reg [2:0] pos  );
    always @(*)
        casez(in[7:0])
            8'bzzzzzzz1: pos = 3'b000;
            8'bzzzzzz1z: pos = 3'b001;
            8'bzzzzz1zz: pos = 3'b010;
            8'bzzzz1zzz: pos = 3'b011;
            8'bzzz1zzzz: pos = 3'b100;
            8'bzz1zzzzz: pos = 3'b101;
            8'bz1zzzzzz: pos = 3'b110;
            8'b1zzzzzzz: pos = 3'b111;
            default: pos = 3'b000;
        endcase
endmodule

```

#### 2.4.8 Avoiding latches

问题描述

>   假设您正在构建一个电路来处理来自游戏的 PS/2 键盘的扫描码。鉴于收到的扫描码的最后两个字节，您需要指出是否已按下键盘上的箭头键之一。这涉及到一个相当简单的映射，它可以实现为具有四个案例的案例语句（或 if-elseif）。 
>   您的电路有一个 16 位输入和四个输出。构建识别这四个扫描码并断言正确输出的电路。 
>   为避免创建锁存器，必须在所有可能的条件下为所有输出分配一个值。仅仅有一个默认情况是不够的。您必须为所有四种情况和默认情况下的所有四个输出分配一个值。这可能涉及大量不必要的输入。解决此问题的一种简单方法是在 case 语句之前为输出分配一个“默认值” ： 
>
>  ```verilog
>  always @(*) begin
>      up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0;
>      case (scancode)
>          ... // Set to 1 as necessary.
>      endcase
>  end
>  ```
>
>  这种代码风格确保在所有可能的情况下为输出分配一个值（0），除非 case 语句覆盖了分配。这也意味着default: case 项变得不必要了。 


verilog代码

```verilog
module top_module (
    input [15:0] scancode,
    output reg left,
    output reg down,
    output reg right,
    output reg up  ); 
    always @(*) begin
        up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0;
        case (scancode)
            16'he06b: left = 1;
            16'he072: down = 1;
            16'he074: right = 1;
            16'he075: up = 1;
        endcase
    end
endmodule

```

### 2.5 More Verilog Features

#### 2.5.1 Conditional temary operator

问题描述

>  
 给定四个无符号数，求最小值。无符号数可以与标准比较运算符（a < b）进行比较。使用条件运算符制作两路最小电路，然后将其中的一些组成一个四路最小电路。您可能需要一些线向量作为中间结果。  


verilog代码

```verilog
module top_module (
    input [7:0] a, b, c, d,
    output [7:0] min);
    wire [7:0] minab;
    wire [7:0] mincd;
    assign minab = (a<b)?a:b;
    assign mincd = (c<d)?c:d;
    assign min = (minab<mincd)?minab:mincd;
endmodule

```

#### 2.5.2 Reduction operators

问题描述

>  
 您已经熟悉两个值之间的按位运算，例如a  &  b或a ^ b。有时，您想创建一个对一个向量的所有位进行操作的宽门，例如(a[0]  &  a[1]  &  a[2]  &  a[3] ... )，如果向量很长。 
 归约运算符可以对向量的位进行 AND、OR 和 XOR，产生一位输出： 
  &  a[3:0] // 与：a[3] & a[2] & a[1] & a[0]。相当于 (a[3:0] == 4'hf) | b[3:0] // 或：b[3]|b[2]|b[1]|b[0]。相当于 (b[3:0] != 4'h0) ^ c[2:0] // 异或：c[2]^c[1]^c[0] 这些是只有一个操作数的一元运算符（类似于 NOT 运算符 ! 和 ~）。您还可以反转这些输出以创建 NAND、NOR 和 XNOR 门，例如(~ &  d[7:0])。 
 练习：当通过不完善的通道传输数据时，奇偶校验通常用作检测错误的简单方法。创建一个电路，计算 8 位字节的奇偶校验位（这将向字节添加第 9 位）。我们将使用“偶数”奇偶校验，其中奇偶校验位只是所有 8 个数据位的 XOR。 


verilog代码

```verilog
module top_module (
    input [7:0] in,
    output parity); 
    assign parity= ^ in[7:0];
endmodule
```

#### 2.5.3 Reduction : Even wider gates

问题描述

>  
 在 [99:0] 中构建一个具有 100 个输入的组合电路。 
 有3个输出： 
 - out_and：100 输入与门的输出。
 - out_or：100 输入或门的输出。
 - out_xor：100 输入异或门的输出。 


veriog代码

```verilog
module top_module( 
    input [99:0] in,
    output out_and,
    output out_or,
    output out_xor 
);
    assign out_and =  & in[99:0];
    assign out_or = |in[99:0];
    assign out_xor = ^in[99:0];
endmodule
```

#### 2.5.4 Combinational for-loop: Vector reversal 2

问题描述

>  
 给定一个 100 位输入向量 [99:0]，反转其位顺序。 
 for 循环（在组合的 always 块或 generate 块中）在这里很有用。在这种情况下，我更喜欢组合的 always 块，因为不需要模块实例化（需要生成块）。 


Verilog代码 

```verilog
module top_module( 
    input [99:0] in,
    output [99:0] out
);
    integer i;
    always @(*)begin
        for(i=0;i<100;i=i+1)begin
            out[i]=in[99-i];  
        end
    end
endmodule
```

#### 2.5.5 Combinational for-loop: 255-bit population count

问题描述

>  
 “人口计数”电路计算输入向量中“1”的数量。为 255 位输入向量构建人口计数电路。 
 这么多东西要添加... for循环怎么样？ 


verilog代码

```verilog
module top_module( 
    input [254:0] in,
    output [7:0] out );
	integer i;
    always @(*)begin
        out = 8'd0;
        for(i=0;i<255;i=i+1) 
            out=in[i]?out+8'd1:out;
    end
endmodule
```

#### 2.5.6 Generate for-loop:100-bit binary adder 2

问题描述

>  
 练习：通过实例化100个全加器来创建一个 100 位二进制波纹进位加法器。加法器将两个 100 位数字和一个进位相加，产生一个 100 位和并执行。为了鼓励您实际实例化全加器，还要输出纹波进位加法器中每个全加器的进位。cout[99] 是最后一个全加器的最终进位，也是您通常看到的进位。 
 有许多全加器要实例化。实例数组或生成语句将在这里有所帮助 


 verilog代码

```verilog

module top_module( 
    input [99:0] a, b,
    input cin,
    output [99:0] cout,
    output [99:0] sum );
    genvar i;
    generate
        for(i=0;i<100;i++) begin:adder
            if(i==0)
                assign {cout[0],sum[0]} = a[0]+b[0]+cin;
            else
                assign {cout[i],sum[i]} = a[i]+b[i]+cout[i-1];
        end           
    endgenerate
endmodule

```

#### 2.5.7  Generate for-loop:100-digit BCD adder

问题描述

>  为您提供了一个名为 bcd_fadd 的 BCD 一位数加法器，它将两个 BCD 位数相加并带入，然后产生一个和并带出。
>
>  ```verilog
>  module bcd_fadd (
>      input [3:0] a,
>      input [3:0] b,
>      input     cin,
>      output   cout,
>      output [3:0] sum );
>  ```
>
>  实例化 100 份 bcd_fadd，创建一个 100 位 BCD 波纹载波加法器。您的加法器应将两个 100 位 BCD 数（打包成 400 位向量）相加，再加上一个进位，生成一个 100 位数的和，然后输出。 


 verilog代码

```verilog
module top_module( 
    input [399:0] a, b,
    input cin,
    output cout,
    output [399:0] sum );
    wire [99:0] cout_temp;
    genvar i;
    generate
        for(i=0;i<100;i++) begin:bcd_fadd
            if(i == 0)
                bcd_fadd bcd_inst(a[3:0],b[3:0],cin,cout_temp[0],sum[3:0]);
            else
                bcd_fadd bcd_inst(a[4*i+3:4*i],b[4*i+3:4*i],cout_temp[i-1],cout_temp[i],sum[4*i+3:4*i]);
        end
        assign cout = cout_temp[99];
    endgenerate
endmodule
```
