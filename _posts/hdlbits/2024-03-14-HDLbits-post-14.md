---
layout:       post
title:        'HDLbits刷题中文完整版-14'
date:         2024-03-14
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

## 4.Verification: Reading Simulations

### 4.1 Finding bugs in code

#### 4.1.1 Mux

 问题描述

>  
 这个 8 位宽的 2 对 1 多路复用器不起作用。修复错误。 

<img alt="" height="203" src="https://s2.loli.net/2024/02/27/kT5YdES3HbvgJhp.png" width="345">

verilog代码

```verilog
module top_module (
    input sel,
    input [7:0] a,
    input [7:0] b,
    output [7:0] out  );

    assign out = sel?a:b;

endmodule
```

#### 4.1.2 NAND

问题描述 

>  
 这个三输入与非门不起作用。修复错误。 
 您必须使用提供的 5 输入与门： 
 <pre>module andgate ( output out, input a, input b, input c, input d, input e );
</pre> 
```verilog
module top_module (input a, input b, input c, output out);
    wire out_temp;
    andgate inst1 (.out(out_temp),.a(a),.b(b),.c(c),.d(1'b1),.e(1'b1));
    assign out = ~out_temp;
endmodule
```

#### 4.1.3 Mux 

问题描述 

>  
 这个 4 对 1 多路复用器不起作用。修复错误。 
 为您提供了一个无错误的 2 对 1 多路复用器： 
 module mux2 (     input sel,     input [7:0] a,     input [7:0] b,     output [7:0] out ); 

<img alt="" height="257" src="https://s2.loli.net/2024/02/27/BkxdHz6EnIZ2V79.png" width="307">

 verilog代码

```verilog
module top_module (
    input [1:0] sel,
    input [7:0] a,
    input [7:0] b,
    input [7:0] c,
    input [7:0] d,
    output [7:0] out  ); 
    wire [7:0] mux0, mux1;
    mux2 u1 ( sel[0],    a,    b, mux0 );
    mux2 u2 ( sel[0],    c,    d, mux1 );
    mux2 u3 ( sel[1], mux0, mux1,  out );
endmodule
```

####  4.1.4 Add/Sub

问题描述

>  
 以下带有零标志的加减法器不起作用。修复错误。 

<img alt="" height="189" src="https://s2.loli.net/2024/02/27/gowRVhF1XzTxarD.png" width="322">

<img alt="" height="290" src="https://s2.loli.net/2024/02/27/EUcBLHKIAlvem8h.png" width="277">

verilog代码 

```verilog
module top_module ( 
    input do_sub,
    input [7:0] a,
    input [7:0] b,
    output reg [7:0] out,
    output reg result_is_zero
);
    always @(*) begin
        case (do_sub)
          0: out = a+b;
          1: out = a-b;
        endcase
        if (out == 8'b0)
            result_is_zero = 1'b1;
        else
            result_is_zero = 1'b0;
    end
endmodule
```

#### 3.1.5 Case Statement

>  
 该组合电路应该识别键 0 到 9 的 8 位键盘扫描码。它应该指示 10 种情况中的一种是否被识别（有效），如果是，则检测到哪个键。修复错误。 

<img alt="" height="162" src="https://s2.loli.net/2024/02/27/ruQ1l4qjFNpcn9B.png" width="287">

<img alt="" height="360" src="https://s2.loli.net/2024/02/27/EScqb2LZwvH4gGO.png" width="254">

 verilog代码

```verilog
module top_module (
    input [7:0] code,
    output reg [3:0] out,
    output reg valid );
 
    always @(*)begin
        case (code)
            8'h45: out = 0;
            8'h16: out = 1;
            8'h1e: out = 2;
            8'h26: out = 3;
            8'h25: out = 4;
            8'h2e: out = 5;
            8'h36: out = 6;
            8'h3d: out = 7;
            8'h3e: out = 8;
            8'h46: out = 9;
            default: out = 0;
        endcase
        if((out == 0) & (code != 8'h45))
            valid = 1'b0;
        else
            valid = 1'b1;
    end
endmodule
```

### 4.2 Build a circuit from a simulation wavefrom

#### 4.2.1 combinational circuit 1

问题描述 

>  
 这是一个组合电路。阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" src="https://s2.loli.net/2024/02/27/41oxjQ9NyI5kcEZ.png">

verilog代码 

```verilog
module top_module (
    input a,
    input b,
    output q );//

    assign q = a & b; // Fix me

endmodule
```

#### 4.2.2 combinational circuit 2

问题描述 

>  
 这是一个组合电路。阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" height="158" src="https://s2.loli.net/2024/02/27/39lJYTUKvHCEAP7.png" width="570">

 verilog代码

```verilog
module top_module (
    input a,
    input b,
    input c,
    input d,
    output q );
	assign q = (~a & ~b & ~c & ~d)|(a & b & ~c & ~d)|(~a & b & ~c & d)|(a & ~b & ~c & d)|(~a & ~b & c & d)|(a & b & c & d)|(~a & b & c & ~d)|(a & ~b & c & ~d);
endmodule
```

#### 4.2.3 combinational circuit 3

问题描述 

>  
 这是一个组合电路。阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" height="147" src="https://s2.loli.net/2024/02/27/7A2BSfeb9Tr5xFt.png" width="520">

verilog代码 

```verilog
module top_module (
    input a,
    input b,
    input c,
    input d,
    output q );
    assign q = (b & d)|(a & d)|(b & c)|(a & c);
endmodule
```

#### 4.2.4 combinational circuit 4

问题描述 

>  
 这是一个组合电路。阅读仿真波形以确定电路的作用，然后实现它。 


<img alt="" height="143" src="https://img-blog.csdnimg.cn/d85fb4221b03499b9d4093b9b8c388d6.png" width="501">

verilog代码 

```verilog
module top_module (
    input a,
    input b,
    input c,
    input d,
    output q );
    assign q = b|c;
endmodule
```

#### 4.2.5  combinational circuit 5

问题描述 

>  
 这是一个组合电路。阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" src="https://s2.loli.net/2024/02/27/rOoe2C143GidyXH.png">

 verilog代码

```verilog
module top_module (
    input [3:0] a,
    input [3:0] b,
    input [3:0] c,
    input [3:0] d,
    input [3:0] e,
    output [3:0] q );
    always @(*)
        begin
            case(c)
                4'b0000: q = b;
                4'b0001: q = e;
                4'b0010: q = a;
                4'b0011: q = d;
                default q = 4'b1111;
            endcase
        end
endmodule
```

#### 4.2.6  combinational circuit 6

问题描述 

>  
 这是一个组合电路。阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" src="https://s2.loli.net/2024/02/27/T2Q8eAkvwPz3yGL.png">

 verilog代码

```verilog
module top_module (
    input [2:0] a,
    output [15:0] q ); 
    always @(*)
        begin
            case(a)
                3'b000: q = 16'h1232;
                3'b001: q = 16'haee0;
                3'b010: q = 16'h27d4;
                3'b011: q = 16'h5a0e;
                3'b100: q = 16'h2066;
                3'b101: q = 16'h64ce;
                3'b110: q = 16'hc526;
                3'b111: q = 16'h2f19;
            endcase
        end
endmodule
```

#### 4.2.7 Sequential cricuit 7 

问题描述 

>  
 这是一个时序电路。阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" src="https://s2.loli.net/2024/02/27/lgK3iRI4kS6yuDv.png">

 verilog代码

```verilog
module top_module (
    input clk,
    input a,
    output q );
    always @(posedge clk)
        begin
            if(a)
                q <= 1'b0;
            else
                q <= 1'b1;
        end
endmodule
```

#### 4.2.8 Sequential cricuit 8

问题描述 

>  
 这是一个时序电路。阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" height="127" src="https://s2.loli.net/2024/02/27/LqFH837XpJlVerP.png" width="445">

 verilog代码

```verilog
module top_module (
    input clock,
    input a,
    output p,
    output q );
    assign p = clock?a:p;
    always @(negedge clock)
        begin
            q <= p;
        end
endmodul
```



#### 4.2.9 Sequential cricuit 9

问题描述 

>  
 这是一个时序电路。阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" src="https://s2.loli.net/2024/02/27/JGgF6Z1OXk8mbMB.png">

 verilog代码

```verilog
module top_module (
    input clk,
    input a,
    output [3:0] q );
    always @(posedge clk)
        begin
            if(a) begin
                q <= 4'd4;
            end
            else if(q>=4'd6) begin
                q <= 4'd0;
            end
            else
                q <= q+1'b1;
        end            
endmodule
```

#### 4.2.10 Sequential cricuit 10

问题描述 

>  
 这是一个时序电路。该电路由组合逻辑和一位存储器（即一个触发器）组成。触发器的输出可以通过输出状态观察到。 
 阅读仿真波形以确定电路的作用，然后实现它。 

<img alt="" height="161" src="https://s2.loli.net/2024/02/27/QM64hJcgltz83dT.png" width="470">

verilog代码 

```verilog
module top_module (
    input clk,
    input a,
    input b,
    output q,
    output state  );
    assign q = a^b^state;
    always @(posedge clk)
        begin
            if(a & b) begin
                state <= 1'b1;
            end
            else if(~a & ~b) begin
                state <= 1'b0;
            end
            else
                state <= state;
        end
endmodule

```