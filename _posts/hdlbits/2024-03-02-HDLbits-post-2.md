---
layout:       post
title:        'HDLbits刷题中文完整版-2'
date:         2024-03-02
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

### 2.3  Modules: Hierarchy（**模块：层次结构**）

####  2.3.1 Modules（**模块**）

问题描述

>   **将信号连接到模块端口** 
 有两种常用的方法将电线连接到端口：按位置或按名称。 
 **按职位** 
 按位置将电线连接到端口的语法应该很熟悉，因为它使用类似C的语法。实例化模块时，端口根据模块的声明从左到右连接。例如：`mod_a instance1 ( wa, wb, wc );` 
 这将实例化类型的模块并为其提供**实例名称**"instance1"，然后将信号（新模块外部）连接到新模块**的第一个**端口 （）、**第二**个端口 （） 和第**三**个端口 （）。此语法的一个缺点是，如果模块的端口列表发生更改，则还需要查找和更改模块的所有实例化以匹配新模块。 
 **按名称** 
 **通过名称**将信号连接到模块的端口，即使端口列表发生变化，电线也能保持正确连接。但是，此语法更为冗长。例如：`mod_a instance2 ( .out(wc), .in1(wa), .in2(wb) );` 
 上面的行实例化了一个名为"instance2"的模块类型，然后将信号（模块外部）连接到**名为**`wc` 的端口 ，连接到**名为**`wa` 的端口，以及**名为** `wb`的端口。请注意，端口的排序在这里是如何无关紧要的，因为无论其在子模块的端口列表中的位置如何，都将与正确的名称建立连接。另请注意此语法中紧挨着端口名称前面的句点。 

<img alt="" src="https://s2.loli.net/2024/02/07/gNvOlDq4ViIY9yb.png">

 verilog代码 

```verilog
module top_module (
	input a,
	input b,
	output out
);

	// Create an instance of "mod_a" named "inst1", and connect ports by name:
	mod_a inst1 ( 
		.in1(a), 	// Port"in1"connects to wire "a"
		.in2(b),	// Port "in2" connects to wire "b"
		.out(out)	// Port "out" connects to wire "out" 
				// (Note: mod_a's port "out" is not related to top_module's wire "out". 
				// It is simply coincidence that they have the same name)
	);

/*
	// Create an instance of "mod_a" named "inst2", and connect ports by position:
	mod_a inst2 ( a, b, out );	// The three wires are connected to ports in1, in2, and out, respectively.
*/
	
endmodule
```

#### 2.3.2 Connecting ports by position（按位置连接端口）

问题描述

>  
 此问题与上一个问题类似 （).您将获得一个名为该模块的模块，该模块具有 2 个输出和 4 个输入（按该顺序排列）。必须按位置将 6 个端口**按该**顺序连接到顶级模块的端口 
 您将获得以下模块： 
 `module mod_a ( output, output, input, input, input, input );` 

<img alt="" src="https://s2.loli.net/2024/02/08/IQONF1drtS9Xsvz.png">

verilog代码

```verilog
module top_module ( 
    input a, 
    input b, 
    input c,
    input d,
    output out1,
    output out2
);
    mod_a u_mod_a(out1,out2,a,b,c,d);
endmodule
```

#### 2.3.3 Connecting ports by name（按名称连接端口）

问题描述

>  
此问题类似于.您将获得一个名为该模块的模块，该模块按某种顺序具有 2 个输出和 4 个输入。您必须**按名称**将 6 个端口连接到顶级模块的端口 

<img alt="" src="https://s2.loli.net/2024/02/08/hQAPIr61OVvEmey.png">

verilog代码

```verilog
module top_module ( 
    input a, 
    input b, 
    input c,
    input d,
    output out1,
    output out2
);
    mod_a u_mod_a (  
        .out1(out1),
        .out2(out2),
        .in1 (a   ),
        .in2 (b   ),
        .in3 (c   ),
        .in4 (d   )
    );
endmodule
```

#### 2.3.4 Three modules（三个模块）

问题描述

>  
 您将获得一个具有两个输入和一个输出的模块（实现D触发器）。实例化其中三个，然后将它们链接在一起以形成长度为 3 的移位寄存器。端口需要连接到所有实例。`my_dff``clk` 
 提供给您的模块是：`module my_dff ( input clk, input d, output q );` 
 请注意，要进行内部连接，您需要声明一些电线。在命名电线和模块实例时要小心：名称必须是唯一的。 

<img alt="" src="https://s2.loli.net/2024/02/08/pOFW9CT3txuE8w6.png">

verilog代码

```verilog
module top_module ( input clk, input d, output q );
    wire q1,q2;
    my_dff u1_my_dff(
        .clk(clk),
        .d(d),
        .q(q1)
    );
    my_dff u2_my_dff(
        .clk(clk),
        .d(q1),
        .q(q2)
    );
    my_dff u3_my_dff(
        .clk(clk),
        .d(q2),
        .q(q)
    );
endmodule
```

#### 2.3.5 Module and vectors（模块和矢量）

问题描述

>   本练习是 module_shift 的扩展。模块端口不再只是单个引脚，我们现在有带有向量作为端口的模块，您将连接线向量而不是普通线。与 Verilog 中的其他任何地方一样，端口的向量长度不必与连接到它的导线相匹配，但这会导致向量的零填充或截断。本练习不使用向量长度不匹配的连接。 
 您将获得一个带有两个输入和一个输出的模块 my_dff8（实现一组 8 个 D 触发器）。实例化其中的三个，然后将它们链接在一起以形成一个长度为 3 的 8 位宽移位寄存器。此外，创建一个 4 对 1 多路复用器（未提供），它根据 sel[1:0] 选择输出什么：输入 d 处的值，在第一个、第二个或第三个 D 触发器之后。 （本质上，sel 选择延迟输入的周期数，从零到三个时钟周期。） 
 提供给你的模块是：module my_dff8(input clk, input [7:0] d, output [7:0] q); 
 未提供多路复用器。一种可能的编写方法是在一个带有 case 语句的 always 块中。 

<img alt="" src="https://s2.loli.net/2024/02/08/2clo9MHC8sgnmkA.png">

verilog代码

```verilog
module top_module (
	input clk,
	input [7:0] d,
	input [1:0] sel,
	output reg [7:0] q
);

	wire [7:0] o1, o2, o3;		// output of each my_dff8
	
	// Instantiate three my_dff8s
	my_dff8 d1 ( clk, d, o1 );
	my_dff8 d2 ( clk, o1, o2 );
	my_dff8 d3 ( clk, o2, o3 );

	// This is one way to make a 4-to-1 multiplexer
	always @(*)		// Combinational always block
		case(sel)
			2'h0: q = d;
			2'h1: q = o1;
			2'h2: q = o2;
			2'h3: q = o3;
		endcase

endmodule
```

#### 2.3.6 Adder1（加法器 1）

问题描述

>   您将获得一个执行 16 位加法的模块 add16。 实例化其中两个以创建一个 32 位加法器。 一个 add16 模块计算加法结果的低 16 位，而第二个 add16 模块在接收到第一个加法器的进位后计算结果的高 16 位。 您的 32 位加法器不需要处理进位（假设为 0）或进位（忽略），但内部模块需要才能正常工作。 （换句话说，add16 模块执行 16 位 a + b + cin，而您的模块执行 32 位 a + b）。 
 如下图所示将模块连接在一起。 提供的模块 add16 具有以下声明： 
 module add16 ( input[15:0] **a**, input[15:0] **b**, input **cin**, output[15:0] **sum**, output **cout** ); 

<img alt="" height="281" src="https://s2.loli.net/2024/02/08/IsH7ltxc3Bq6dUw.png" width="454">

  verilog代码

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire cout,cout1;
    add16 u1_add16(
        .a(a[15:0]),
        .b(b[15:0]),
        .cin(1'b0),
        .sum(sum[15:0]),
        .cout(cout)
    );
   add16 u2_add16(
       .a(a[31:16]),
       .b(b[31:16]),
       .cin(cout),
       .sum(sum[31:16]),
       .cout(cout1)
    );
endmodule
```

#### 2.3.7 Adder2（加法器 2）

问题描述

>   在本练习中，您将创建具有两个层次结构的电路。您的 top_module 将实例化 add16 的两个副本（已提供），每个副本将实例化 add1 的 16 个副本（您必须编写）。因此，您必须编写两个模块：top_module 和 add1。 
 与 module_add 一样，您将获得一个执行 16 位加法的模块 add16。您必须实例化其中的两个以创建 32 位加法器。一个 add16 模块计算加法结果的低 16 位，而第二个 add16 模块计算结果的高 16 位。您的 32 位加法器不需要处理进位（假设为 0）或进位（忽略）。 
 如下图所示将 add16 模块连接在一起。提供的模块 add16 具有以下声明： 
 模块add16（输入[15:0] a，输入[15:0] b，输入cin，输出[15:0] sum，输出cout）； 
 在每个 add16 中，实例化了 16 个全加器（模块 add1，未提供）以实际执行加法。您必须编写具有以下声明的完整加法器模块： 
 模块add1（输入a，输入b，输入cin，输出总和，输出cout）； 
 回想一下，全加器计算 a+b+cin 的和和进位。 
 综上所述，本设计共有三个模块： 
 top_module — 您的顶级模块，其中包含两个... add16, provided — 一个 16 位加法器模块，由 16 个... add1 — 1 位全加器模块。 
 如果您的提交缺少模块 add1，您将收到一条错误消息，显示错误 (12006)：节点实例“user_fadd[0].a1”实例化未定义实体“add1”。 

<img alt="" src="https://s2.loli.net/2024/02/08/kDT5OiAsr9SzqZY.png">

 verilog代码

```verilog
module top_module (
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);//
    wire cout1,cout2;
    add16 u1_add16(
        .a(a[15:0]),
        .b(b[15:0]),
        .cin(1'b0),
        .sum(sum[15:0]),
        .cout(cout1)
    );
    add16 u2_add16(
        .a(a[31:16]),
        .b(b[31:16]),
        .cin(cout1),
        .sum(sum[31:16]),
        .cout(cout2)
    );
endmodule

module add1 ( input a, input b, input cin,   output sum, output cout );

// Full adder module here
    assign {cout,sum} = a+b+cin;
endmodule
```

#### 2.3.8 Carry-select adder（携带选择加法器）

问题描述

>  
 纹波进位加法器的一个缺点（参见前面的练习）是加法器计算进位的延迟（从进位，在最坏的情况下）相当慢，并且第二级加法器无法开始计算其执行直到第一级加法器完成。这使加法器变慢。一种改进是进位选择加法器，如下所示。第一级加法器和以前一样，但是我们复制第二级加法器，一个假设进位=0，一个假设进位=1，然后使用快速2对1多路复用器选择哪个结果碰巧是正确的。 在本练习中，为您提供与前一练习相同的模块 add16，它将两个 16 位数字与进位相加，并产生一个进位和 16 位和。您必须使用您自己的 16 位 2 对 1 多路复用器来实例化其中的三个以构建进位选择加法器。 如下图所示将模块连接在一起。提供的模块 add16 具有以下声明： 
 module add16 ( input[15:0] a, input[15:0] b, input cin, output[15:0] sum, output cout ); 

<img alt="" src="https://s2.loli.net/2024/02/08/xAtp5Uf4Z3vBWid.png">

verilog代码

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire cout1,cout2,cout3;
    wire [15:0] sum0;
    wire [15:0] sum1;
    add16 u1_add16(
        .a(a[15:0]),
        .b(b[15:0]),
        .cin(1'b0),
        .sum(sum[15:0]),
        .cout(cout1)
    );
    add16 u2_add16(
        .a(a[31:16]),
        .b(b[31:16]),
        .cin(1'b0),
        .sum(sum0[15:0]),
        .cout(cout2)
    );
    add16 u3_add16(
        .a(a[31:16]),
        .b(b[31:16]),
        .cin(1'b1),
        .sum(sum1[15:0]),
        .cout(cout3)
    );
    assign sum[31:16]=cout1 ? sum1 : sum0;
endmodule

```

#### 2.3.9 Adder-subtractor（加法器减法器）

问题描述

>   加法器-减法器可以通过选择性地取反一个输入来从加法器构建，这相当于将输入反相然后加 1。最终结果是一个可以执行两种操作的电路：(a + b + 0) 和 ( a + ~b + 1)。 如果您想更详细地了解该电路的工作原理，请参阅 Wikipedia。 构建下面的加减法器。 为您提供了一个 16 位加法器模块，您需要对其进行两次实例化： module add16 ( input[15:0] **a**, input[15:0] **b**, input **cin**, output[15:0] **sum**, output **cout** ); 
 每当 sub 为 1 时，使用 32 位宽的 XOR 门来反转 b 输入。（这也可以被视为 b[31:0] 与 sub 复制 32 次进行异或。请参阅复制运算符。）。 还将子输入连接到加法器的进位。 

<img alt="d862da4658605feafca7ef72a851c609.png" src="https://s2.loli.net/2024/02/08/AEPyHzqKwj4hQCT.png">

verilog代码 

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    input sub,
    output [31:0] sum
);
    wire[31:0]	b_com;
    wire		cout;
    
    assign b_com = sub ?~b:b;
    //assign b_com = {32{sub}} ^ b;
    add16 u1_add16(
        .a      (a[15:0]        ),
        .b      (b_com[15:0]	),
        .cin    (sub            ),
        .sum    (sum[15:0]   ),
        .cout   (cout           )
    );
    
    add16 u2_add16(
        .a      (a[31:16]       ),
        .b      (b_com[31:16]	),
        .cin    (cout	        ),
        .sum    (sum[31:16]	),
        .cout   (               )
    );
endmodule
```



