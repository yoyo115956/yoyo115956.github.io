---
layout:       post
title:        'HDLbits刷题中文完整版-1'
date:         2024-03-01
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

# HDLbits刷题中文完整版，按照刷题网站顺序每日更新一道

## 1 GettingStarted（开始）

### 1.1 GettingStarted（开始）

问题描述 

>  
 构建一个没有输入和一个输出的电路。该输出应始终驱动 1（或逻辑高电平）。  


verilog代码

```verilog
module top_module( output one );

// Insert your code here
    assign one = 1'b1;

endmodule

```

### 1.2 OutputZero（输出零点 ）

问题描述d

>  
 构建一个没有输入的电路和一个输出的电路，输出一个常数0 


verilog代码

```verilog
module top_module(
    output zero
);// Module body starts after semicolon
    assign zero = 1'b0;
endmodule
```

##  2 Verilog Language（开始）

### 2.1 Basics 

#### 2.1.1 Simple wire （简单电线）

 问题描述

>   创建一个具有一个输入和一个输出的模块，其行为类似于电线。 
 与物理导线不同，Verilog 中的导线（和其他信号）是**定向的**。这意味着信息只在一个方向上流动，从（通常是一个）**源**到**接收器**（该源通常也称为**将值驱动**到导线上的**驱动程序**）。在Verilog"连续赋值"（assign left_side = right_side;）中，右侧信号的值被驱动到左侧的导线上。分配是"连续的"，因为即使右侧的值发生变化，分配也会一直继续。连续分配不是一次性事件。`assign left_side = right_side;` 
 模块上的端口也有一个方向（通常是输入或输出）。输入端口由模块外部的东西**驱动**，而输出端口**则驱动**外部的东西。从模块内部查看时，输入端口是驱动器或源，而输出端口是接收器。 
 下图说明了电路的每个部分如何对应于Verilog代码的每个位。模块和端口声明创建电路的黑色部分。您的任务是通过添加要连接到 的语句来创建一条线路（绿色）。开箱即用的部件不是您关心的问题，但您应该知道，通过将测试线束的信号连接到顶部_模块的端口，可以测试您的电路。 

<img alt="" src="https://s2.loli.net/2024/02/07/WxSn8FG5zshMPbo.png">

verilog代码

```verilog
module top_module( input in, output out );
    assign out = in;
endmodule
```

#### 2.1.2 Four wires （四线）

 问题描述

>  
 创建一个具有 3 个输入和 4 个输出的模块，其行为类似于建立这些连接的电线： 
 <pre style="margin-left:0;">a -&gt; w
b -&gt; x
b -&gt; y
c -&gt; z</pre> 
 下图说明了电路的每个部分如何对应于Verilog代码的每个位。从模块外部，有三个输入端口和四个输出端口。当您有多个赋值语句时，它们在代码中的显示**顺序****无关紧要**。与编程语言不同，赋值语句（"连续赋值"）描述事物之间的**连接**，而不是将值从一个事物复制到另一个事物的**操作**。现在也许应该澄清的一个潜在的混淆来源是：这里的绿色箭头代表电线**之间的**连接，但本身不是电线。模块本身**已经**声明了 7 条导线（命名为 a、b、c、w、x、y 和 z）。这是因为，除非另有说明，否则声明实际上声明了一条线。写 `input wire a` 写 `input a 是一样的`.。因此，这些语句不是在创建导线，而是在已经存在的 7 根导线之间创建连接。 input wire a</code> 写 `input a 是一样的`.。因此，这些语句不是在创建导线，而是在已经存在的 7 根导线之间创建连接。</p> 
</blockquote> 


  <img alt="" src="https://s2.loli.net/2024/02/07/eoqDRjAkL7Gi21f.png"> 

   verilog代码  


```verilog
module top_module( 
    input a,b,c,
    output w,x,y,z );
    assign w = a;
    assign x = b,y = b;
    assign z = c;
    //如果我们确定每个信号的宽度，使用
    //串联运算符等效且更短：
    //assign {w，x，y，z}={a，b，b，c}；
endmodule
```

#### 2.1.3 Inverter  （逆变器）

 问题描述

>  
 创建一个实现 NOT 门的模块。该电路类似于，但略有不同。当从电线连接到电线时，我们将实现逆变器（或"非门"）而不是普通电线。使用assign语句。assign语句将持续将in的非转换为out。 



<img alt="" src="https://s2.loli.net/2024/02/07/s7hbGBHf1PRcMLv.png">

verilog代码

```verilog
module top_module( input in, output out );
    assign out = ~in;
endmodule
```

#### 2.1.4 AND gate  （和门）

 问题描述

>   创建实现 AND 门的模块。 
 该电路现在有三条导线（a、b和out）。导线a和b已经具有由输入端口驱动的值。但wire out目前并不是由任何因素驱动的。写一个assign语句，用a和b的AND信号输出。 
 请注意，该电路与NOT门非常相似，只是多了一个输入。如果听起来不一样，那是因为我已经开始描述信号是被驱动的（已知值由附加到它的某个东西决定）还是不是被某个东西驱动的。输入线由模块外部的东西驱动。assign语句将把一个逻辑电平驱动到一条线上。正如您所料，一条导线不能有多个驱动器（如果有，其逻辑级别是多少？），没有驱动程序的导线将有一个未定义的值（在合成硬件时通常被视为0）。 



<img alt="" src="https://s2.loli.net/2024/02/07/ALXr2eRBYbQK3If.png">

verilog代码

```verilog
module top_module( 
    input a, 
    input b, 
    output out );
    assign out = a & b;
endmodule
```

####  2.1.5 NOR gate  （或非门）

 问题描述

>   创建一个实现或非门的模块。或非门是输出反转的或门。在Verilog中编写NOR函数时需要两个运算符。 
 assign语句用一个值驱动一条线（或者更正式地称为“网”）。该值可以是任意复杂的函数，只要它是组合函数（即无内存、无隐藏状态）。assign语句是一种连续赋值，因为每当其任何输入发生变化时，都会“重新计算”输出，就像一个简单的逻辑门一样。 

<img alt="" src="https://s2.loli.net/2024/02/07/2wTMlbQsy6UjEtg.png">

verilog代码

```verilog
module top_module( 
    input a, 
    input b, 
    output out );
    assign out = ~(a | b);
endmodule
```

####  2.1.6 XNOR gate  （异或非门）

 问题描述

>  
 创建一个实现 XNOR 门的模块。 

<img alt="" src="https://s2.loli.net/2024/02/07/9ohpOg8TMNCaDzs.png">

 verilog代码

```verilog
module top_module( 
    input a, 
    input b, 
    output out );
    //assign out = a ~^ b;a和b相同输出0，不同输出1
    //assign out = a ^~ b;
    assign out = ~(a ^ b);
endmodule

```

#### 2.1.7 Declaring wires（声明导线）

问题描述 

>  
 执行以下电路。创建两条中间导线（可以任意命名）将与门连接在一起。请注意，为NOT gate馈电的导线实际上是wire out，因此不一定需要在此处声明第三条导线。请注意，导线是如何由一个源（门的输出）驱动的，但可以提供多个输入。 
 如果你遵循图表中的电路结构，你应该得到四个assign语句，因为有四个信号需要赋值。 



<img alt="" src="https://s2.loli.net/2024/02/07/vSmBgfriyUsh5uw.png">

verilog代码

```verilog
module top_module (
	input a,
	input b,
	input c,
	input d,
	output out,
	output out_n );
	
	wire w1, w2;		// Declare two wires (named w1 and w2)
	assign w1 = a & b;	// First AND gate
	assign w2 = c & d;	// Second AND gate
	assign out = w1|w2;	// OR gate: Feeds both 'out' and the NOT gate

	assign out_n = ~out;	// NOT gate
	
endmodule
```

#### 2.1.8 7458 chip（ 7458芯片）  

问题描述 

>   7458是一款带有四个“与”门和两个“或”门的芯片。这个问题比7420稍微复杂一些。 
 创建一个功能与7458芯片相同的模块。它有10个输入和2个输出。您可以选择使用assign语句来驱动每条输出线，也可以选择声明（四）条线用作中间信号，其中每条内部线由一个AND门的输出驱动。如果想进行额外的练习，可以尝试两种方法。 



<img alt="" src="https://s2.loli.net/2024/02/07/Llc7SmY1IyJMGvR.png">

verilog代码

```verilog
module top_module ( 
    input p1a, p1b, p1c, p1d, p1e, p1f,
    output p1y,
    input p2a, p2b, p2c, p2d,
    output p2y );
    //第一种方法
    assign p2y = (p2a & p2b) | (p2c & p2d);
    assign p1y = (p1a & p1c & p1b) | (p1f & p1e & p1d);
    //第二种方法
    wire a,b,c,d;
    assign a = p2a & p2b;
    assign b = p2c & p2d;
    assign p2y = a | b;
    assign c = p1a & p1c & p1b;
    assign d = p1f & p1e & p1d;
    assign p1y = c | d;
endmodule
```

###  2.2 Vectors（矢量）

####  2.2.1 Vectors（矢量） 

 问题描述

>   向量用于使用一个名称对相关信号进行分组，以便于操作。例如，导线[7:0]w；声明一个名为w的8位向量，该向量在功能上相当于有8条独立的导线。 
 请注意，向量的声明将维度放在向量名称之前，这与C语法相比是不寻常的。但是，零件选择的尺寸标注位于向量名称之后，正如您所期望的那样。 
 构建一个具有一个3位输入的电路，然后输出相同的矢量，并将其拆分为三个单独的1位输出。将输出o0连接到输入向量的位置0，将o1连接到位置1，等等。 
 在图表中，旁边带有数字的记号表示向量（或“总线”）的宽度，而不是为向量中的每一位画一条单独的线。 

<img alt="" src="https://s2.loli.net/2024/02/07/y9PFwMJhkxOSY6o.png">

 verilog代码

```verilog
module top_module(
	input [2:0] vec, 
	output [2:0] outv,
	output o2,
	output o1,
	output o0
);
	
	assign outv = vec;

	// This is ok too: assign {o2, o1, o0} = vec;
	assign o0 = vec[0];
	assign o1 = vec[1];
	assign o2 = vec[2];
	
endmodule
```

#### 2.2.2 Vectors in more detail（更详细的矢量） 

 问题描述

>  
 构建一个组合电路，将输入半字（16位[15:0]）拆分为低位[7:0]和高位[15:8]字节。 


verilog代码

```verilog
module top_module (
	input [15:0] in,
	output [7:0] out_hi,
	output [7:0] out_lo
);
	
	assign out_hi = in[15:8];
	assign out_lo = in[7:0];
	
	// Concatenation operator also works: assign {out_hi, out_lo} = in;
	
endmodule
```

#### 2.2.3 Vector part select（矢量部分选择） 

 问题描述

>  
 32 位矢量可以被视为包含 4 个字节（位 [31：24]、[23：16] 等）。构建一个**电路，该**电路将反转 4 字节字的字节顺序。 
 <pre>AaaaaaaaBbbbbbbbCcccccccDddddddd =&gt; DdddddddCcccccccBbbbbbbbAaaaaaaa</pre> 
 当需要交换一段数据的时，例如在小端 x86 系统和许多 Internet 协议中使用的大端格式之间，通常使用此操作。 


```verilog
module top_module (
	input [31:0] in,
	output [31:0] out
);

	assign out[31:24] = in[ 7: 0];	
	assign out[23:16] = in[15: 8];	
	assign out[15: 8] = in[23:16];	
	assign out[ 7: 0] = in[31:24];	
	
endmodule
```

#### 2.2.4 Bitwise operators（按位运算符） 

 问题描述

>  
 构建一个具有两个3位输入的电路，用于计算两个向量的位或、两个向量的逻辑或，以及两个向量的逆（非）。将b的非数放在out_not的上半部分（即位[5:3]），将a的非放在下半部分。 



<img alt="" src="https://s2.loli.net/2024/02/07/jKysY9MreiHgzZ8.png">

 verilog代码

```verilog
module top_module(
	input [2:0] a, 
	input [2:0] b, 
	output [2:0] out_or_bitwise,
	output out_or_logical,
	output [5:0] out_not
);
	
	assign out_or_bitwise = a | b;
	assign out_or_logical = a || b;

	assign out_not[2:0] = ~a;	// Part-select on left side is o.
	assign out_not[5:3] = ~b;	//Assigning to [5:3] does not conflict with [2:0]
	
endmodule
```

#### 2.2.5 Four-input gates（四输入门） 

 问题描述

>   在[3:0]中构建一个具有四个输入的组合电路。 
 有3个输出： 
 out_和：4输入与门的输出。 
 out_或：4输入或门的输出。 
 输出异或：4输入异或门的输出。 
 要查看AND、OR和XOR运算符，请参阅andgate,norgate,xnorgate 


verilog代码

```verilog
module top_module( 
    input [3:0] in,
    output out_and,
    output out_or,
    output out_xor
);
   assign out_and = & in;
   assign out_or = | in;
   assign out_xor = ^in;
endmodule
```

#### 2.2.6 Vector concatenation operator（矢量串联运算符） 

 问题描述

>  
 给定多个输入向量，将它们连接在一起，然后将它们拆分为多个输出向量。有六个 5 位输入向量：a、b、c、d、e 和 f，总共 30 位输入。有四个 8 位输出向量：w、x、y 和 z，用于 32 位输出。输出应是输入向量的串联，后跟两个 1 位： 


verilog代码

```verilog
module top_module (
    input [4:0] a, b, c, d, e, f,
    output [7:0] w, x, y, z );//

    // assign { ... } = { ... };
    assign {w,x,y,z} = {a,b,c,d,e,f,2'b11};
endmodule
```

#### 2.2.7 Vector reversal1（矢量反转） 

 问题描述

>  
 给定一个 8 位输入向量 [7：0]，反转其位顺序。 


verilog代码

```verilog
module top_module (
	input [7:0] in,
	output [7:0] out
);
//第一种方法
	assign {out[0],out[1],out[2],out[3],out[4],out[5],out[6],out[7]} = in;
	
    /*
//第二种方法
// 创建组合逻辑块。for-loop 描述的是电路的*行为*，而不是*结构*，
// 因此只能在程序块（如 always 块）中使用。创建的电路（导线和门）不进行任何迭代： 
// 它只会产生与迭代相同的结果。实际上，逻辑合成器会在编译时进行迭代，
// 以确定要生成什么样的电路。(相反，Verilog 仿真器会在仿真过程中按顺序执行循环）。
	always @(*) begin	
		for (int i=0; i<8; i++)	// int is a SystemVerilog type. Use integer for pure Verilog.
			out[i] = in[8-i-1];
	end

//第三种方法
// 也可以使用generate-for 循环来实现这一功能。
// 生成循环看起来像过程 for 循环，但在概念上有很大不同，而且不容易理解。
// 生成循环用于对 "事物 "进行实例化（与过程循环不同，它不描述操作）。
// 这些 "事物 "包括赋值语句、模块实例、净值/变量声明和过程块（不在过
// 程内部时可以创建的事物）。生成循环（和 genvars）完全在编译时进行评估。
// 你可以将生成块视为一种预处理形式，用于生成更多代码，然后在逻辑合成器中运行。
// 在下面的示例中，generate-for 循环首先在编译时创建了 8 个赋值语句，
// 然后进行合成。需要注意的是，由于其预期用途（在编译时生成代码），
// 使用方法会受到一些限制。例如 1. Quartus 要求generate-for 循环
// 附加一个命名的 begin-end 块（在本例中，命名为 "my_block_name"）。
// 2. 在循环体中，genvars 只可读。

	generate
		genvar i;
		for (i=0; i<8; i = i+1) begin: my_block_name
			assign out[i] = in[8-i-1];
		end
	endgenerate
	*/
	
endmodule

```

#### 2.2.8 Repliction operator（复制操作员） 

 问题描述

>  
 复制操作员常见的一个地方是，将一个较小的数字扩展为一个较大的数字，同时保留其有符号值。这是通过复制左边较小数字的符号位（最高有效位）来实现的。例如，将4'b0101扩展到8位的符号导致8'b00000101，而将4'b1101扩展到8位的符号导致8'b11111101。 
 构建一个将8位数字扩展到32位的电路。这需要将符号位的24个副本（即复制位[7]24次）与8位数字本身串联起来。 


verilog代码

```verilog
module top_module (
	input [7:0] in,
	output [31:0] out
);

	// Concatenate two things together:
	// 1: {in[7]} repeated 24 times (24 bits)
	// 2: in[7:0] (8 bits)
	assign out = { {24{in[7]}}, in };
	
endmodule
```

#### 2.2.9 More replication（更多复制） 

 问题描述

>  
 给定五个 1 位信号（a、b、c、d 和 e），计算 25 位输出矢量中的所有 25 个成对单位比较。如果要比较的两位相等，则输出应为 1。 
 <pre>out[24] = ~a ^ a;   // a == a, so out[24] is always 1.
out[23] = ~a ^ b;
out[22] = ~a ^ c;
...
out[ 1] = ~e ^ d;
out[ 0] = ~e ^ e;
</pre> 


```verilog
module top_module (
	input a, b, c, d, e,
	output [24:0] out
);

	wire [24:0] top, bottom;
	assign top    = { {5{a}}, {5{b}}, {5{c}}, {5{d}}, {5{e}} };
	assign bottom = {5{a,b,c,d,e}};
	assign out = ~top ^ bottom;	// Bitwise XNOR

	// This could be done on one line:
	// assign out = ~{ {5{a}}, {5{b}}, {5{c}}, {5{d}}, {5{e}} } ^ {5{a,b,c,d,e}};
	
endmodule
```