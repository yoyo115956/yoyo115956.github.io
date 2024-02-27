---
layout:       post
title:        'HDLbits刷题中文完整版-12'
date:         2024-03-12
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

#### 3.2.5.19 Q5a:Serial two's complementer(Moore FSM)

问题描述

>  
 你要设计一个单输入单输出串行2的补码**摩尔**状态机。输入 (x) 是一系列位（每个时钟周期一个），从数字的最低有效位开始，输出 (Z) 是输入的 2 的补码。机器将接受任意长度的输入数字。该电路需要异步复位。转换在释放复位时开始，在复位时停止。 
 例如： 

<img alt="" src="https://s2.loli.net/2024/02/26/GVZT9hdUwyDX854.png">

<img alt="" height="226" src="https://s2.loli.net/2024/02/26/1ZbK2rzcC7ywgU8.png" width="520">

verilog代码

```verilog
module top_module (
    input clk,
    input areset,
    input x,
    output z
); 
    parameter A=2'd0,B=2'd1,C=2'd2;
    reg [1:0] state,next_state;
    always @(posedge clk,posedge areset)
        begin
            if(areset)
                state <= A;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                A:next_state = x?B:A;
                B:next_state = x?C:B;
                C:next_state = x?C:B;
                default:next_state = A;
            endcase
        end
    assign z = (state == B);
endmodule
```

#### 3.2.5.20 Q5b:Serial two's complementer(Mealy FSM)

问题描述

下图是 2 的补码的**Mealy**机器实现。使用 one-hot 编码实现。

<img alt="" src="https://s2.loli.net/2024/02/26/pXzd2jCZLwDYVBA.png">

<img alt="" src="https://s2.loli.net/2024/02/26/1GDPegyRBcC53rN.png">

verilog代码

```verilog
module top_module (
    input clk,
    input areset,
    input x,
    output z
); 
    parameter A=1'b0,B=1'b1;
    reg state,next_state;
    always @(posedge clk,posedge areset)
        begin
            if(areset)
                state <= A;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                A:next_state = x?B:A;
                B:next_state = B;
            endcase
        end
    always @(*)
        begin
            case(state)
                A:z = x;
                B:z = ~x;
            endcase
        end
endmodule
```

知识点

>   关于2进制补码的求法，找到其中的规律，然后用状态机实现。 
>                    低位 ---> 高位 
>            例：x --> 0 0 0        其反码 为 0 0 0，则其补码z如下 
>                   z --> 0 0 0 
>                   x --> 0 0 0 1     其反码为 1 1 1 0，则其补码为反码+1 
>                   z --> 0 0 0 1 
>                   x --> 0 0 0 1 0  其反码为 1 1 1 0 1，则其补码为反码+1 
>                   z --> 0 0 0 1 1 
>                   x --> 0 0 0 1 1  其反码为 1 1 1 0 0，则其补码为反码+1 
>                   z --> 0 0 0 1 0 
>            我们从中可以找出规律，既数据从 0 开始时，则其补码输出也为 0 ，当出现第一个 1 时， 
>    补码对应位输出为1。此后当输入为 0 ，则其对应位输出为 1；输入为 1 则其对应位输出为 0。 


#### 3.2.5.21 Q3a:FSM

问题描述

>  
 考虑一个具有输入s和w的有限状态机。假设 FSM 以称为A的复位状态开始，如下所示。只要s = 0， FSM 就保持在状态A ，当s = 1 时，它移动到状态 B。一旦处于状态B，FSM在接下来的三个时钟周期内检查输入w的值。如果w = 1 在恰好两个时钟周期中，则 FSM 必须 在下一个时钟周期中将输出z设置为 1。否则z必须为 0。FSM 继续检查w接下来的三个时钟周期，依此类推。下面的时序图说明了不同w值所需的z值。 
 使用尽可能少的状态。请注意，s输入仅用于状态A，因此您只需要考虑w输入。 

<img alt="" height="287" src="https://s2.loli.net/2024/02/26/NjyFpVCI9HSqOis.png" width="484">

verilog代码

```verilog
module top_module (
    input clk,
    input reset,   // Synchronous reset
    input s,
    input w,
    output z
);
    parameter A=1'b0,B=1'b1;
    reg state,next_state;
    always @(posedge clk)
        begin
            if(reset)
                state <= A;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                A:next_state = s?B:A;
                B:next_state = B;
            endcase
        end
    reg w1,w2;
    always @(posedge clk)
        begin
            if(reset) begin
                w1 <= 1'b0;
                w2 <= 1'b0;
            end
            else if(next_state == B) begin
                w1 <= w;
                w2 <= w1;
            end
            else begin
                w1 <= 1'b0;
                w2 <= 1'b0;
            end
        end
    always @(posedge clk)
        begin
            if(reset)
                z <= 1'b0;
            else if((state == B) &  & (counter == 2'd0)) begin
                if(w & w1 & ~w2 | w & ~w1 & w2 | ~w & w1 & w2) begin
                    z <= 1'b1;
                end
                else begin
                    z <= 1'b0;
                end
            end 
            else
                z <= 1'b0;
        end
    reg [1:0] counter;
    always @(posedge clk)
        begin
            if(reset)
                counter <= 2'd0;
            else if(counter == 2'd2)
                counter <= 2'd0;
            else if(next_state == B)
                counter <= counter + 1'b1;
        end     
endmodule
```

#### 3.2.5.22 Q3b FSM

问题描述

>  
 给定如下所示的状态分配表，实现有限状态机。重置应该将 FSM 重置为状态 000。 

<img alt="" height="247" src="https://s2.loli.net/2024/02/26/ZecxBM9NdunYvtV.png" width="277">

verilog代码

```verilog
module top_module (
    input clk,
    input reset,   // Synchronous reset
    input x,
    output z
);
    parameter A = 3'b000,B = 3'b001,C = 3'b010,D = 3'b011,E = 3'b100;
    reg [2:0] state,next_state;
    always @(posedge clk)
        begin
            if(reset)
                state <= A;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                A:next_state = x?B:A;
                B:next_state = x?E:B;
                C:next_state = x?B:C;
                D:next_state = x?C:B;
                E:next_state = x?E:D;
            endcase
        end
    assign z = ((state == D)||(state == E));
endmodule
```

#### 3.2.5.23 Q3c:FSM logic

问题描述

>  
 给定如下所示的状态分配表，实现逻辑函数 Y[0] 和 z。 

<img alt="" height="212" src="https://s2.loli.net/2024/02/26/v8GYDksulEANVIX.png" width="237">

verilog代码

```verilog
module top_module (
    input clk,
    input [2:0] y,
    input x,
    output Y0,
    output z
);
    parameter A = 3'b000,B = 3'b001,C = 3'b010,D = 3'b011,E = 3'b100;
    reg [2:0] next_state;
    always @(*)
        begin
            case(y)
                A:next_state = x?B:A;
                B:next_state = x?E:B;
                C:next_state = x?B:C;
                D:next_state = x?C:B;
                E:next_state = x?E:D;
            endcase
        end
    assign Y0 = ((next_state == B)||(next_state == D));
    assign z = ((y == D)||(y == E));
endmodule
```

#### 3.2.5.24 Q6b:FSM next _state logic

问题描述

>  
 考虑如下所示的状态机，它有一个输入w和一个输出z。 

<img alt="" height="345" src="https://s2.loli.net/2024/02/26/6c8t1VFUpWaSEvs.png" width="348">

>  
 假设您希望使用三个触发器和状态代码y[3:1] = 000, 001, ..., 101 分别用于状态 A、B、...、F 来实现 FSM。显示此 FSM 的状态分配表。导出触发器y[2]的下一个状态表达式。 
 只为y[2]实现下一个状态逻辑。 


verilog代码

```verilog
module top_module (
    input [3:1] y,
    input w,
    output Y2);
    parameter A = 3'b000,B = 3'b001,C = 3'b010,D = 3'b011,E = 3'b100,F = 3'b101;
    reg [2:0] next_state;
    always @(*)
        begin
            case(y)
                A:next_state = w?A:B;
                B:next_state = w?D:C;
                C:next_state = w?D:E;
                D:next_state = w?A:F;
                E:next_state = w?D:E;
                F:next_state = w?D:C;
            endcase
        end
    assign Y2 = ((next_state == C)||(next_state == D));
endmodule
```

#### 3.2.5.25 Q6c:FSM one-hot next-state log

问题描述

>  
 考虑如下所示的状态机，它有一个输入**w**和一个输出**z**。 

<img alt="" height="292" src="https://s2.loli.net/2024/02/26/hGqT8cbps47Uw56.png" width="275">

verilog代码

```verilog
module top_module (
    input [6:1] y,
    input w,
    output Y2,
    output Y4);
    parameter A = 3'd1,B = 3'd2,C = 3'd3,D = 3'd4,E = 3'd5,F = 3'd6;
    reg [6:0] next_state;
    always @(*)
        begin
            case(y)
                A:next_state = w?A:B;
                B:next_state = w?D:C;
                C:next_state = w?D:E;
                D:next_state = w?A:F;
                E:next_state = w?D:E;
                F:next_state = w?D:C;
                default:next_state = A;
            endcase
        end
    assign Y2 = ~w & y[A];
    assign Y4 = w & (y[B]|y[C]|y[E]|y[F]);
endmodule
```

#### 3.2.5.26 Q6:FSM

问题描述

>  
 考虑如下所示的状态机，它有一个输入**w**和一个输出**z**。 

<img alt="" height="292" src="https://s2.loli.net/2024/02/26/kLiNovIOySKDZRB.png" width="290">

>  
 实现状态机。（这部分不在期中，但编写 FSM 是一种很好的做法）。 


verilog代码

```verilog
module top_module (
    input clk,
    input reset,     // synchronous reset
    input w,
    output z);
    parameter A = 3'b000,B = 3'b001,C = 3'b010,D = 3'b011,E = 3'b100,F = 3'b101;
    reg [2:0] state,next_state;
    always @(posedge clk)
        begin
            if(reset)
                state <= A;
            else
                state <= next_state;
        end 
    always @(*)
        begin
            case(state)
                A:next_state = w?A:B;
                B:next_state = w?D:C;
                C:next_state = w?D:E;
                D:next_state = w?A:F;
                E:next_state = w?D:E;
                F:next_state = w?D:C;
            endcase
        end
    assign z = ((state == E)||(state == F));
endmodule
```

#### 3.2.5.27 Q2a:FSM

问题描述

>  
 考虑如下所示的状态图。 

<img alt="" height="338" src="https://s2.loli.net/2024/02/26/BVuTH4oFntQcEq3.png" width="333">

>  
 编写代表此 FSM 的完整 Verilog 代码。就像在讲座中所做的那样，对状态表和状态触发器使用单独的**always块。**使用连续赋值语句或**always**块（由您自行决定）描述 FSM 输出，称为**z 。** 


verilog代码

```verilog
module top_module (
    input clk,
    input reset,   // Synchronous active-high reset
    input w,
    output z
);
    parameter A = 3'b000,B = 3'b001,C = 3'b010,D = 3'b011,E = 3'b100,F = 3'b101;
    reg [2:0] state,next_state;
    always @(posedge clk)
        begin
            if(reset)
                state <= A;
            else
                state <= next_state;
        end 
    always @(*)
        begin
            case(state)
                A:next_state = w?B:A;
                B:next_state = w?C:D;
                C:next_state = w?E:D;
                D:next_state = w?F:A;
                E:next_state = w?E:D;
                F:next_state = w?C:D;
            endcase
        end
    assign z = ((state == E)||(state == F));
endmodule
```

#### 3.2.5.28  Q2b:One-hot FSM equations

问题描述

>  
 这个问题的状态图再次显示在下面。 



<img alt="" height="410" src="https://s2.loli.net/2024/02/26/bnESMThFiIusD6Z.png" width="420">

>   假设在状态分配y[5:0] = 000001(A), 000010(B), 000100(C), 001000(D), 010000(E), 100000(F) 中使用 one-hot 代码 
 为信号Y1写一个逻辑表达式，它是状态触发器y[1]的输入。 
 为信号Y3写一个逻辑表达式，它是状态触发器y[3]的输入。 
 （通过假设 one-hot 编码的检查推导出逻辑方程。测试台将使用非 one hot 输入进行测试，以确保您不会尝试做更复杂的事情）。 
 单热状态转换逻辑的逻辑方程可以通过查看状态转换图的边缘来导出。 


verilog代码 

```verilog
module top_module (
    input [5:0] y,
    input w,
    output Y1,
    output Y3
);
    parameter A = 3'b000,B = 3'b001,C = 3'b010,D = 3'b011,E = 3'b100,F = 3'b101;
    assign Y1 = w & y[A];
    assign Y3 = ~w & (y[B]|y[C]|y[E]|y[F]);
endmodule
```

#### 3.2.5.29 Q2a:FSM

问题描述

>  
 考虑下图所示状态图描述的 FSM：  

<img alt="" height="426" src="https://s2.loli.net/2024/02/26/YPE2oXc8rAC75n6.png" width="300">

>   该 FSM 充当仲裁电路，控制三个请求设备对某种类型资源的访问。每个设备通过设置信号r[i] = 1 来请求资源，其中r[i]是r[1]、r[2]或r[3]。每个 r[i] 是 FSM 的输入信号，代表三个设备之一。只要没有请求，FSM 就会保持在状态A。当一个或多个请求发生时，FSM 决定哪个设备接收到使用资源的授权，并更改为将该设备的g[i]信号设置为 1 的状态。每个g[i]是 FSM 的输出。有一个优先级系统，设备 1 的优先级高于设备 2，设备 3 的优先级最低。因此，例如，如果设备 3 是在 FSM 处于状态A时发出请求的唯一设备，则设备 3 将仅接收授权。一旦设备i被 FSM 授予授权，只要其请求r[i] = 1，该设备就会继续接收授权。 
 编写代表此 FSM 的完整 Verilog 代码。就像在讲座中所做的那样，对状态表和状态触发器使用单独的 always 块。使用连续赋值语句或 always 块（由您自行决定）描述 FSM 输出g[i] 。 


 verilog代码

```verilog
module top_module (
    input clk,
    input resetn,    // active-low synchronous reset
    input [3:1] r,   // request
    output [3:1] g   // grant
); 
    parameter A=0,B=1,C=2,D=3;
    reg [1:0] state;
    reg [1:0] next_state;
    always @(*) begin
            case(state)
                A:begin
                    if(r[1])
                        next_state = B;
                    else if(r[2])
                        next_state = C;
                    else if(r[3])
                        next_state = D;
                    else
                        next_state = A;
                end
                B:begin
                    if(~r[1])
                        next_state = A;
                    else
                        next_state = B;
                end
                C:begin
                    if(~r[2])
                        next_state = A;
                    else
                        next_state = C;
                end
                D:begin
                    if(~r[3])
                        next_state = A;
                    else
                        next_state = D;
                end
            endcase
        end

    always @(posedge clk) begin
        if(!resetn)
            state<=A;
        else
            state<=next_state;
    end
    assign g[1]=(state==B);
    assign g[2]=(state==C);
    assign g[3]=(state==D);
endmodule

```

#### 3.2.5.30 Q2b:Another FSM

问题描述

>  考虑一个用于控制某种电机的有限状态机。FSM 具有 来自电机的输入x和y ，并产生控制电机的输出f和g。还有一个称为clk的时钟输入和一个称为resetn的复位输入。 
 FSM 必须按如下方式工作。只要复位输入被置位，FSM 就保持在开始状态，称为状态A。当复位信号无效时，在下一个时钟沿之后，FSM 必须将输出f设置为 1 一个时钟周期。然后，FSM 必须监控 x输入。当x在三个连续的时钟周期中产生值 1、0、1 时，应在下一个时钟周期将g设置为 1。在保持g = 1 的同时，FSM 必须监控y 输入。如果y在最多两个时钟周期内为 1，则 FSM 应保持g= 1 永久（即，直到重置）。但如果y在两个时钟周期内未变为 1，则 FSM 应永久设置g = 0（直到复位）。 
 （最初的考试问题只要求提供状态图。但在这里，实现 FSM。） 
 FSM 直到f之后的周期为 1才 开始监视x输入。 


verilog代码

```verilog
module top_module (
    input clk,
    input resetn,    // active-low synchronous reset
    input x,
    input y,
    output f,
    output g
); 
    parameter FOUT=4'd0,A=4'd1,B=4'd2,C=4'd3,D=4'd4,E=4'd5,FONE=4'd6,FZERO=4'd7,IDEL=4'd8;
    reg [3:0] state,next_state;
    always @(posedge clk)
        begin
            if(!resetn)
                state <= IDEL;
            else
                state <= next_state;
        end
    always @(*)
        begin
            case(state)
                IDEL:next_state = FOUT;
                FOUT:next_state = A;
                A:next_state = x?B:A;
                B:next_state = x?B:C;
                C:next_state = x?D:A;
                D:next_state = y?FONE:E;
                E:next_state = y?FONE:FZERO;
                FONE:next_state = FONE;
                FZERO:next_state = FZERO;
                default : next_state = IDEL;
            endcase
        end
    assign f = (state == FOUT);
    assign g = ((state == D)||(state == E)||(state == FONE));
endmodule

```