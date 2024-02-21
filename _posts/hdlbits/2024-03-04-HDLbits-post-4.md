---
layout:       post
title:        'HDLbits刷题中文完整版-4'
date:         2024-03-04
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

## 3. Circuits

### 3.1 Combinational logic

#### 3.1.1 Basic gates

#### 3.1.1.1 Wire

问题描述

>  
 实现以下电路 

<img alt="" height="36" src="https://s2.loli.net/2024/02/20/OflbPnpi9BmtwK1.png" width="238">

verilog代码

```verilog
module top_module (
    input in,
    output out);
    assign out = in;
endmodule
```

#### 3.1.1.2 GND

 问题描述

>  
 实现以下电路 

<img alt="" height="111" src="https://s2.loli.net/2024/02/20/OaBzS3rWEyk8hdm.png" width="227">

verilog代码

```verilog
module top_module (
    output out);
    assign out = 1'b0;
endmodule
```

#### 3.1.1.3 NOR

  问题描述

>  
 实现以下电路 

<img alt="" height="108" src="https://s2.loli.net/2024/02/20/amA7db9BuZUjzls.png" width="292">  

verilog代码

```verilog
module top_module (
    input in1,
    input in2,
    output out);
    assign out = ~(in1|in2);
endmodule

```

#### 3.1.1.4 Another gate

  问题描述

>  
 实现以下电路 

<img alt="" height="116" src="https://s2.loli.net/2024/02/20/UrH5fJa1poun8I6.png" width="295">

verilog代码

```verilog
module top_module (
    input in1,
    input in2,
    output out);
    assign out = in1  &  (~in2);
endmodule

```

#### 3.1.1.5 Two gates

  问题描述

>  
 实现以下电路 

<img alt="" height="159" src="https://s2.loli.net/2024/02/20/96HnbFDLJcB3ipm.png" width="508">

verilog代码

```verilog
module top_module (
    input in1,
    input in2,
    input in3,
    output out);
    assign out = (in1 ~^ in2) ^ in3;
endmodule
```

#### 3.1.1.6 More logic gates

问题描述 

>   好的，让我们尝试同时构建几个逻辑门。构建具有两个输入a和b的组合电路。 
 有 7 个输出，每个都有一个逻辑门驱动它： 
 out_and: a 和 b out_or：a 或 b out_xor: a xor b out_nand: 一个 nand b out_nor: a 或 b out_xnor: a xnor b out_anotb：a 与 b非 


verilog代码

```verilog
module top_module( 
    input a, b,
    output out_and,
    output out_or,
    output out_xor,
    output out_nand,
    output out_nor,
    output out_xnor,
    output out_anotb
);
    assign out_and = a  &  b;
    assign out_or = a | b;
    assign out_xor = a ^ b;
    assign out_nand = ~(a  &  b);
    assign out_nor = ~(a | b);
    assign out_xnor = ~(a ^ b);
    assign out_anotb = a  &  (~b);
endmodule

```

#### 3.1.1.7 7420 chip

问题描述

>  
 7400 系列集成电路是一系列数字芯片，每个芯片都有几个门。7420 是具有两个 4 输入与非门的芯片。 
 创建一个与 7420 芯片功能相同的模块。它有 8 个输入和 2 个输出。 

<img alt="" height="443" src="https://s2.loli.net/2024/02/20/5V9cM6jvSRXBwIQ.png" width="800">

 verilog代码

```verilog
module top_module ( 
    input p1a, p1b, p1c, p1d,
    output p1y,
    input p2a, p2b, p2c, p2d,
    output p2y );
    assign p1y = ~(p1a & p1b & p1c & p1d);
    assign p2y = ~(p2a & p2b & p2c & p2d);
endmodule
```

#### 3.1.1.8 Truth tables

 问题描述

<img alt="" height="294" src="https://s2.loli.net/2024/02/20/FGsizq3bRQXTK2I.png" width="178">

>  
 假设我们要构建上述电路，但我们仅限于使用一组标准逻辑门。您将如何构建任意逻辑函数（表示为真值表）？创建一个实现上述真值表的组合电路。 

<img alt="" height="260" src="https://s2.loli.net/2024/02/20/Lcz5GOX2Wp8vPdi.png" width="562">

verilog代码

```verilog
module top_module( 
    input x3,
    input x2,
    input x1,  // three inputs
    output f   // one output
);
    assign f = ((~x3) & x2 & (~x1))|((~x3) & x2 & x1)|(x3 & (~x2) & x1)|(x3 & x2 & x1);
endmodule
```

#### 3.1.1.9 Two bit equality

 问题描述

>  
 创建一个具有两个 2 位输入A[1:0]和B[1:0]的电路，并产生一个输出z。如果A = B ，则z的值应为 1 ，否则z应为 0。 


verilog代码

```verilog
module top_module ( input [1:0] A, input [1:0] B, output z ); 
    assign z = (A ==B) ? 1'B1 : 1'B0;
endmodule
```

#### 3.1.1.10 Simple circuit A

 问题描述

>  
 模块 A 应该实现函数z = (x^y)  &  x。实现这个模块。 


verilog代码

```verilog
module top_module (input x, input y, output z);
    assign z = (x^y)  &  x;
endmodule
```

#### 3.1.1.11 Simple circuit B

 问题描述

>  
 电路B可以用下面的仿真波形来描述，实现这个电路： 

<img alt="" height="172" src="https://s2.loli.net/2024/02/20/Lxus5aoZnEj8Y1Q.png" width="815">

verilog代码

```verilog
module top_module ( input x, input y, output z );
    assign z = ~(x^y);
endmodule
```

#### 3.1.1.12 Combine circuit A  and B

问题描述

>  
 顶层设计由子电路 A 和 B 的两个实例组成，如下所示。实现这个电路：  

<img alt="" height="316" src="https://s2.loli.net/2024/02/21/NkKeDmH96PWiaXR.png" width="396">

 verilog代码

```verilog
module top_module (input x, input y, output z);
    assign z = (((x^y) & x)|(x~^y))^(((x^y) & x) & (x~^y));
endmodule
```

#### 3.1.1.13 Ring or vibrate

 问题描述

>   假设您正在设计一个电路来控制手机的振铃器和振动电机。每当电话需要从来电中振铃 ( input ring) 时，您的电路必须打开振铃器 ( output ringer = 1) 或电机 (output motor = 1 )，但不能同时打开两者。如果手机处于振动模式 ( input vibrate_mode = 1)，请打开电机。否则，打开铃声。 
 尝试仅使用assign语句，看看您是否可以将问题描述转换为逻辑门的集合。 

<img alt="" height="170" src="https://s2.loli.net/2024/02/21/yIOlnNA7zJkcHph.png" width="518">

verilog代码

```verilog
module top_module (
    input ring,
    input vibrate_mode,
    output ringer,       // Make sound
    output motor         // Vibrate
);
    assign ringer = ring & (~vibrate_mode);
    assign motor = ring & vibrate_mode;
endmodule
```

#### 3.1.1.14 Thermostat

 问题描述

>  
 加热/冷却恒温器控制加热器（冬季）和空调（夏季）。实施一个可以根据需要打开和关闭加热器、空调和鼓风机的电路。 
 恒温器可以处于以下两种模式之一：加热 ( mode = 1) 和冷却 ( mode = 0)。在制热模式下，当天气太冷时打开加热器（too_cold = 1），但不要使用空调。在制冷模式下，空调过热时打开空调（too_hot = 1），但不要打开加热器。当加热器或空调打开时，还要打开风扇以循环空气。此外，fan_on = 1即使加热器和空调已关闭，用户也可以请求打开风扇 。 
 尝试仅使用assign语句，看看您是否可以将问题描述转换为逻辑门的集合。 


verilog代码

```verilog
module top_module (
    input too_cold,
    input too_hot,
    input mode,
    input fan_on,
    output heater,
    output aircon,
    output fan
); 
    assign heater = mode  &  too_cold  &  (~aircon);
    assign aircon = (~mode)  &  (~heater)  &  too_hot;
    assign fan = aircon | heater | fan_on;
endmodule
```

#### 3.1.1.15 3-bit population count

问题描述

>  
 “人口计数”电路计算输入向量中“1”的数量。构建一个输入为 3 位向量的人口计数电路。 


verilog代码

```verilog
module top_module( 
    input [2:0] in,
    output [1:0] out );
    integer i;
    always @(*) 
        begin
            out = 2'd0;
            for(i=0;i<3;i=i+1)
                begin
                    if(in[i]==1'b1)
                        out = out+2'd1;
                    else
                        out=out;
                end         
        end
endmodule

```

#### 3.1.1.16 Gates and Vectors

问题描述

>  
 在 [3:0] 中给定一个四位输入向量。我们想知道每个位与其邻居之间的一些关系： 
 - out_both：此输出向量的每个位都应指示相应的输入位及其左侧的邻居（较高的索引）是否都是“1” 。例如，out_both[2]应该表明in[2]和in[3]是否都为 1。由于in[3]左边没有邻居，所以答案很明显，所以我们不需要知道out_both[3 ] .
 - out_any：此输出向量的每个位应指示相应的输入位及其右侧的邻居是否为“1”。例如，out_any[2]应该指示in[2]或in[1]是否为 1。由于in[0]右侧没有邻居，因此答案很明显，因此我们不需要知道out_any[0 ] .
 - out_different：此输出向量的每个位都应指示相应的输入位是否与其左侧的邻居不同。例如，out_diff[2]应该指示in[2]是否与in[3]不同。对于这部分，将向量视为环绕，因此in[3]左侧的邻居是in[0]。 
 both 、any和different输出分别使用双输入 AND、OR 和 XOR 运算。使用向量，这可以在 3 个分配语句中完成。 


verilog代码

```verilog
module top_module( 
    input [3:0] in,
    output [2:0] out_both,
    output [3:1] out_any,
    output [3:0] out_different );
    integer i;
    always @(*)
        begin
            for(i=0;i<3;i=i+1)
                begin
                out_both[i] = in[i] & in[i+1];
                out_any[3-i] = in[3-i]|in[2-i];
                out_different[i] = in[i]^in[i+1];
                end
        end
    assign out_different[3] = in[3]^in[0];
endmodule
```

#### 3.1.1.17 Even longer vector

 问题描述

>  
 在 [99:0] 中给定一个 100 位的输入向量。我们想知道每个位与其邻居之间的一些关系： 
 - out_both：此输出向量的每个位应指示相应的输入位及其左侧的邻居是否都为“1”。例如，out_both[98]应该表明in[98]和in[99]是否都是 1。由于in[99]左边没有邻居，答案很明显，所以我们不需要知道out_both[99 ] .
 - out_any：此输出向量的每个位应指示相应的输入位及其右侧的邻居是否为“1”。例如，out_any[2]应该指示in[2]或in[1]是否为 1。由于in[0]右侧没有邻居，因此答案很明显，因此我们不需要知道out_any[0 ] .
 - out_different：此输出向量的每个位都应指示相应的输入位是否与其左侧的邻居不同。例如，out_diff[98]应该指示in[98]是否与in[99]不同。对于这部分，将向量视为环绕，因此in[99]左侧的邻居是in[0]。 
 上题中使用for实现，这里给出使用assign实现的方法： 


verilog代码

```verilog
module top_module( 
    input [99:0] in,
    output [98:0] out_both,
    output [99:1] out_any,
    output [99:0] out_different );
    assign out_both = in[98:0]  &  in[99:1];
    assign out_any = in[98:0] | in[99:1];
	assign out_different = in ^ {in[0], in[99:1]};
endmodule
```