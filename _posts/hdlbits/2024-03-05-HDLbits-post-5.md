---
layout:       post
title:        'HDLbits刷题中文完整版-5'
date:         2024-03-05
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

#### 3.1.2 Multiplexers

#### 3.1.2.1 2-to-1 multiplexer 

问题描述

>  
 创建一位宽的 2 对 1 多路复用器。当 sel=0 时，选择 a。当 sel=1 时，选择 b。 


Verilog代码

```verilog
module top_module( 
    input a, b, sel,
    output out ); 
    assign out = sel?b:a;
endmodule
```

####  3.1.2.2 2-to-1 bus multiplexer

问题描述

>  
 创建一个 100 位宽的 2 对 1 多路复用器。当 sel=0 时，选择 a。当 sel=1 时，选择 b。 


verilog代码

```verilog
module top_module( 
    input [99:0] a, b,
    input sel,
    output [99:0] out );
    assign out = sel?b:a;
endmodule

```

#### 3.1.2.3 9-to-1 multiplexer

问题描述

>  
 创建一个 16 位宽的 9 对 1 多路复用器。sel=0 选择 a，sel=1 选择 b，等等。对于未使用的情况（sel=9 到 15），将所有输出位设置为“1”。 


verilog代码 

```verilog
module top_module( 
    input [15:0] a, b, c, d, e, f, g, h, i,
    input [3:0] sel,
    output [15:0] out );
    always @(*)
        begin
            case(sel)
                0: out = a;
                1: out = b;
                2: out = c;
                3: out = d;
                4: out = e;
                5: out = f;
                6: out = g;
                7: out = h;
                8: out = i;
                default out = {16{1'b1}};
            endcase
        end
endmodule

```

#### 3.1.2.4 256-to-1 multiplexer

问题描述

>  
 创建一个 1 位宽、256 对 1 的多路复用器。256 个输入全部打包成一个 256 位输入向量。sel=0 应该选择in[0]， sel=1 选择[1]中的位， sel=2 选择[2]中的位，等等。 


verilog代码

```verilog
module top_module( 
    input [255:0] in,
    input [7:0] sel,
    output out );
    assign out = in[sel];
endmodule

```

#### 3.1.2.5 256-to-1 4-bit multiplexer

问题描述

>  
 创建一个 4 位宽、256 对 1 的多路复用器。256 个 4 位输入全部打包成一个 1024 位输入向量。sel=0 应该选择[3:0]中的位， sel=1 选择[7:4]中的位， sel=2 选择[11:8]中的位等。 


verilog代码 

```verilog
module top_module( 
    input [1023:0] in,
    input [7:0] sel,
    output [3:0] out );
    assign out = {in[sel*4+3],in[sel*4+2],in[sel*4+1],in[sel*4]};
endmodule

```

知识点

>  
 此题与上一题对比，很容易想到这样一种实现方法 assign out = in[2*sel+3:2*sel]；verilog中这样的写法是错误的 ，verilog中要求向量下标可以是变量，但是位宽必须是常量，而上式无法证明位宽是常量。 


#### 3.1.3 Airthmetic Circuits

#### 3.1.3.1 Half adder

问题描述

>  
 创建一个半加法器。半加器将两位相加（没有进位）得到加和和进位。 


verilog代码

```verilog
module top_module( 
    input a, b,
    output cout, sum );
	assign sum = a^b;
    assign cout = a & b;
endmodule
```

#### 3.1.3.2 Full adder

问题描述

>  
 创建一个全加器。全加器将三位相加（包括进位）并产生和和进位。 


verilog代码 

```verilog
module top_module( 
    input a, b, cin,
    output cout, sum );
    assign sum = a^b^cin;
    assign cout = a & b | a & cin | b & cin;
endmodule
```

#### 3.1.3.3 3-bit binary adder

问题描述

>  
 创建 3 个实例来创建一个 3 位二进制波纹进位加法器。加法器将两个 3 位数字和一个进位相加产生一个 3 位加和和进位。为了鼓励实例化全加器，还要输出纹波进位加法器中每个全加器的进位。cout[2] 是最后一个全加器的最终进位，也是您通常看到的进位。 


verilog代码

```verilog
//第一种方法
module top_module( 
    input [2:0] a, b,
    input cin,
    output [2:0] cout,
    output [2:0] sum );
    add1 u1 (.a(a[0]),.b(b[0]),.cin(cin),.sum(sum[0]),.cout(cout[0]));
    add1 u2 (.a(a[1]),.b(b[1]),.cin(cout[0]),.sum(sum[1]),.cout(cout[1]));
    add1 u3 (.a(a[2]),.b(b[2]),.cin(cout[1]),.sum(sum[2]),.cout(cout[2]));
endmodule
 
module add1 ( input a, input b, input cin,   output sum, output cout );
	assign sum = a^b^cin;
    assign cout = a & b|a & cin|b & cin;
endmodule

```

```verilog
//第二种方法
module top_module( 
    input [2:0] a, b,
    input cin,
    output [2:0] cout,
    output [2:0] sum );
	genvar i;
    generate
        for(i=0;i<3;i=i+1) begin:adder
            if(i==0) 
                begin
                    assign sum[i] = a[i]^b[i]^cin;
                    assign cout[i] = a[i] & b[i] | a[i] & cin | b[i] & cin;
                end
            else 
                begin
                    assign sum[i]=a[i]^b[i]^cout[i-1];
                    assign cout[i]=a[i] & b[i] | a[i] & cout[i-1] | b[i] & cout[i-1];
                end     
        end   
    endgenerate
endmodule
```

#### 3.1.3.4 Adder

问题描述

>  
 实现下面电路，FA是全加器： 

<img alt="" height="133" src="https://s2.loli.net/2024/02/21/jyZbDNLkswt18S4.png" width="360">

verilog代码 

```verilog
//第一种写法
module top_module (
    input [3:0] x,
    input [3:0] y, 
    output [4:0] sum);
    wire [3:0] cout;
    genvar i;
    generate
        for(i=0;i<4;i=i+1) begin:adder
            if(i==0)
            	FA u1 (.a(x[0]),.b(y[0]),.sum(sum[0]),.cout(cout[0]));
            else
                FA u1 (.a(x[i]),.b(y[i]),.cin(cout[i-1]),.sum(sum[i]),.cout(cout[i])); 
        end
        assign sum[4] = cout[3];
    endgenerate
endmodule
module FA ( input a, input b, input cin,   output sum, output cout );
	assign sum = a^b^cin;
    assign cout = a & b|a & cin|b & cin;
endmodule

```

```verilog
//第二种写法
module top_module (
	input [3:0] x,
	input [3:0] y,
	output [4:0] sum
);
	assign sum = x+y;	
endmodule

```

#### 3.1.3.5 signed addition overflow

问题描述

>  
 假设您有两个 8 位 的补码，a[7:0] 和 b[7:0]。这些数字相加产生 s[7:0]。还要计算是否发生了（有符号的）溢出。 


verilog代码

```verilog
module top_module (
    input [7:0] a,
    input [7:0] b,
    output [7:0] s,
    output overflow
); 
    assign s = a+b;
    assign overflow = (a[7] & b[7] & ~s[7])|(~a[7] & ~b[7] & s[7]);
endmodule
```

知识点

>  
 当两个正数相加产生负结果或两个负数相加产生正结果时，会发生有符号溢出。有几种检测溢出的方法：可以通过比较输入和输出数的符号来计算，或者从位 n 和 n-1 的进位推导出。  


#### 3.1.3.6 100-bit binary adder

问题描述

>  
 创建一个 100 位二进制加法器。加法器将两个 100 位数字和一个进位相加，产生一个 100 位加和和进位。 


verilog代码 

```verilog
module top_module( 
    input [99:0] a, b,
    input cin,
    output cout,
    output [99:0] sum );
    assign {cout,sum} = a+b+cin;
endmodule
```

#### 3.1.3.7 4-digit BCD adder

问题描述

>   为您提供了一个名为bcd_fadd的 BCD（二进制编码的十进制）一位加法器，它将两个 BCD 数字和进位相加，并产生一个加和和进位。 
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
>  实例化 4 个bcd_fadd副本以创建一个 4 位 BCD 波纹进位加法器。您的加法器应该将两个 4 位 BCD 数字（打包成 16 位向量）和一个进位相加，以产生一个 4 位加和和进位。 


verilog代码

```verilog
module top_module ( 
    input [15:0] a, b,
    input cin,
    output cout,
    output [15:0] sum );
    wire cout1;wire cout2;wire cout3;
    bcd_fadd u0(
        .a(a[3:0]),
        .b(b[3:0]),
        .cin(cin),
        .cout(cout1),
        .sum(sum[3:0]));
     bcd_fadd u1(
        .a(a[7:4]),
        .b(b[7:4]),
        .cin(cout1),
        .cout(cout2),
        .sum(sum[7:4]));
    bcd_fadd u2(
        .a(a[11:8]),
        .b(b[11:8]),
        .cin(cout2),
        .cout(cout3),
        .sum(sum[11:8]));
    bcd_fadd u3(
        .a(a[15:12]),
        .b(b[15:12]),
        .cin(cout3),
        .cout(cout),
        .sum(sum[15:12]));
endmodule
```

#### 3.1.4 Karnaugh Map to Circuit

#### 3.1.4.1 3-variable 

问题描述

>  
 实现下面卡诺图描述的电路，在编码之前尝试简化 k-map。 尝试和积和积和形式。 我们无法检查您是否有 k-map 的最佳简化。 但是我们可以检查你的归约是否等价，我们可以检查你是否可以将 k-map 转换为电路。 



<img alt="" height="241" src="https://s2.loli.net/2024/02/21/eorKRNIMP8ClWmp.png" width="176">

verilog代码

```verilog
module top_module(
    input a,
    input b,
    input c,
    output out  ); 
    assign out = a|b|c;
endmodule
```

#### 3.1.4.2 4-variable

问题描述 

>  
 实现下面卡诺图描述的电路。  

<img alt="" height="238" src="https://s2.loli.net/2024/02/21/QFVitzjfkWAG4v1.png" width="262">

verilog代码

```verilog
module top_module(
    input a,
    input b,
    input c,
    input d,
    output out  ); 
    assign out = (~b & ~c)|(~a & ~d)|(b & c & d)|(a & c & d);
endmodule
```

#### 3.1.4.3 4-variable

问题描述

>  
 实现下面卡诺图描述的电路。  

<img alt="" height="203" src="https://s2.loli.net/2024/02/21/uWd81iExSjh9NkX.png" width="210">

verilog代码

```verilog
module top_module(
    input a,
    input b,
    input c,
    input d,
    output out  ); 
    assign out = (~b & c)|a;
endmodule
```

#### 3.1.4.4 4-variable

>  
  实现下面卡诺图描述的电路。 

<img alt="" height="195" src="https://s2.loli.net/2024/02/21/chQj6DJz8rpYUZu.png" width="200">

verilog代码 

```verilog
module top_module(
    input a,
    input b,
    input c,
    input d,
    output out  ); 
    assign out = (~a & ~b & ~c & d)|(~a & b & ~c & ~d)|(a & b & ~c & d)|(a & ~b & ~c & ~d)|(~a & b & c & d)|(a & ~b & c & d)|(~a & ~b & c & ~d)|(a & b & c & ~d);
endmodule
```

#### 3.1.4.5 Minimum SOP and POS

问题描述

>   具有四个输入（a、b、c、d）的单输出数字系统在输入上出现 2、7 或 15 时生成逻辑 1，当输入上出现 0、1、4、5、6 、9、10、13 或 14 出现时生成逻辑 0。数字 3、8、11 和 12 的输入条件在此系统中永远不会出现。例如，7 对应于 a、b、c、d 分别设置为 0、1、1、1。 
 确定最小SOP形式(积之和 最小项)的输出out_sop，以及最小POS形式(和之积 最大项)的输出out_pos。 


verilog代码

```verilog
module top_module (
    input a,
    input b,
    input c,
    input d,
    output out_sop,
    output out_pos
); 
    assign out_sop = (~a & ~b & c)|(c & d);
    assign out_pos = c & (~b|d) & (~a|d);
endmodule

```

#### 3.1.4.6 Karnaugh map

问题描述

>  
 考虑下面卡诺图中所示 的函数**f ，**实现这个功能。**d**是不关心，这意味着您可以选择输出任何方便的值。 

<img alt="" src="https://img-blog.csdnimg.cn/2147bad92c474cea8b84b8e09b65e5f4.png">

verilog代码

```verilog
module top_module (
    input [4:1] x, 
    output f );
    assign f = (~x[1] & x[3])|(x[2] & x[4]);
endmodule

```

#### 3.1.4.7 Karnaugh map

问题描述

>  
 考虑下面卡诺图中所示的函数**f，**实现这个功能。（原始考试问题要求简化 SOP 和 POS 形式的函数）注意卡诺图中 x[4:1] 输入位的顺序。 

<img alt="" height="156" src="https://s2.loli.net/2024/02/21/XOf1JokNRQGzbiW.png" width="191">

verilog代码 

```verilog
module top_module (
    input [4:1] x,
    output f
); 
    assign f = (~x[1] & x[3])|(x[2] & x[3] & x[4])|(~x[2] & ~x[4]);
endmodule
```

#### 3.1.4.8 K-map implemented with a multiplexer

问题描述 

>  
 对于下面的卡诺图，给出使用一个 4 对 1 多路复用器和尽可能多的 2 对 1 多路复用器的电路实现，但使用尽可能少。不允许使用任何其他逻辑门，并且必须使用a和b作为多路复用器选择器输入，如下面的 4 对 1 多路复用器所示。您只实现了标记为top_module的部分，以便整个电路（包括 4 对 1 多路复用器）实现 K-map。 

<img alt="" height="264" src="https://s2.loli.net/2024/02/21/tfn1ZcwvE2kzDme.png" width="283">

 verilog代码

```verilog
module top_module (
    input c,
    input d,
    output [3:0] mux_in
); 
    assign mux_in[0] = c|d;
    assign mux_in[1] = 0;
    assign mux_in[2] = ~d;
    assign mux_in[3] = c & d;
endmodule
```