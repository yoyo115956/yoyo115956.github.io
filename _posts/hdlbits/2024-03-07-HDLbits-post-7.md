---
layout:       post
title:        'HDLbits刷题中文完整版-7'
date:         2024-03-07
header-style: text
catalog:      true
mathjax:      true
tags:
  - HDLbits
---

#### 2.2.2 Counters

#### 3.2.2.1 Four-bit binary counter

问题描述

>  
 构建一个从 0 到 15（含）计数的 4 位二进制计数器，周期为 16。复位输入是同步的，复位将计数器复位为 0。 

<img alt="" height="89" src="https://s2.loli.net/2024/02/22/ybrCNIo3nzgs7Qh.png" width="356">

verilog代码

```verilog
module top_module (
    input clk,
    input reset,      // Synchronous active-high reset
    output [3:0] q);
    always @(posedge clk) begin
        if(reset)
            q<=4'd0;
        else if(q==4'd15)
            q<=4'd0;
        else
            q<=q+4'd1;
    end
endmodule
```

#### 2.2.2.2 Decade counter

问题描述

>  
 构建一个从 0 到 9（含）计数的十进制计数器，周期为 10。复位输入是同步的，应将计数器复位为 0。 

<img alt="" height="82" src="https://s2.loli.net/2024/02/22/7LpPZYghwJ2OVev.png" width="318">

 verilog代码

```verilog
module top_module (
    input clk,
    input reset,        // Synchronous active-high reset
    output [3:0] q);
    always @(posedge clk)
        begin
            if(reset || q>=4'd9)
                q <= 4'b0;
            else
                q <= q+1'b1;
        end
endmodule
```

#### 3.2.2.3 Decade counter again

 问题描述

>  
 制作一个从 1 到 10 的十进制计数器，包括 1 到 10。复位输入是同步的，应将计数器复位为 1。 

<img alt="" height="79" src="https://s2.loli.net/2024/02/22/PAt17MvKegDpjGc.png" width="307">

 verilog代码

```verilog
module top_module (
    input clk,
    input reset,
    output [3:0] q);
    always @(posedge clk) begin
        if(reset) 
            q <= 4'd1;
        else if(q==4'd10)
            q <= 4'd1;
        else
            q <= q + 4'd1;
    end
endmodule
```

#### 3.2.2.4 Slow decade counter

 问题描述、

>   构建一个从 0 到 9 计数的十进制计数器，周期为 10。复位输入是同步的，应该将计数器复位为 0。我们希望能够暂停计数器，而不是总是在每个时钟周期递增，所以slowena输入指示计数器何时应该增加。 
 这是一个带有启用控制信号的常规十进制计数器 

<img alt="" height="98" src="https://s2.loli.net/2024/02/22/C8OnTejfb7w9VBt.png" width="316">

 verilog代码

```verilog
module top_module (
    input clk,
    input slowena,
    input reset,
    output [3:0] q);
    always @(posedge clk) begin
        if(reset)
            q <= 4'b0;
        else if(slowena)
            begin
                if(q>=4'd9)
                    q <= 4'b0;
                else
                    q <= q+1'b1;
            end
        else
            q <= q;
    end
endmodule
```

#### 3.2.2.5 Counter 1-12

问题描述

>  
 设计一个具有以下输入和输出的 1-12 计数器： 
 - 复位同步高电平有效复位，强制计数器为 1
 - 启用设置为高以使计数器运行
 - clk上升沿触发时钟输入
 - Q[3:0]计数器的输出
 - c_enable, c_load, c_d[3:0]控制信号进入提供的 4 位计数器，它们的目的是允许检查这些信号的正确性。 

您有以下可用组件： 

 - 下面的 4 位二进制计数器 ( count4 )，它具有启用和同步并行加载输入（加载的优先级高于启用）。count4模块提供给您。在你的电路中实例化它。 

 - 逻辑门 

    ```verilog
    module count4(
    	input clk,
    	input enable,
    	input load,
    	input [3:0] d,
    	output reg [3:0] Q
    );
    ```

    c_enable 、c_load和c_d输出是分别进入内部计数器的enable、load和d输入的信号。它们的目的是允许检查这些信号的正确性。 


verilog代码

```verilog

module top_module (
    input clk,
    input reset,
    input enable,
    output [3:0] Q,
    output c_enable,
    output c_load,
    output [3:0] c_d
);
    assign c_enable = enable;
    assign c_load = ((Q >= 4'd12) &  & (enable == 1'b1))|reset;
    assign c_d = c_load?4'd1:4'd0;
    count4 the_counter (clk, c_enable, c_load, c_d , Q);
endmodule
```

#### 3.2.2.6 Counter 1000

 问题描述

>   从 1000 Hz 时钟导出一个称为OneHertz的 1 Hz 信号，该信号可用于驱动一组小时/分钟/秒计数器的启用信号，以创建数字挂钟。由于我们希望时钟每秒计数一次，因此OneHertz信号必须每秒准确地断言一个周期。使用模 10 (BCD) 计数器和尽可能少的其他门构建分频器。还要从您使用的每个 BCD 计数器输出使能信号（c_enable[0] 为最快的计数器，c_enable[2] 为最慢的）。 
>   为您提供以下 BCD 计数器。Enable必须为高电平才能使计数器运行。复位是同步的并设置为高以强制计数器为零。电路中的所有计数器必须直接使用相同的 1000 Hz 信号。 


verilog代码

```verilog

module top_module (
    input clk,
    input reset,
    output OneHertz,
    output [2:0] c_enable
); 
    wire [3:0]q0,q1,q2;
    assign c_enable[0]=1'b1;
    assign c_enable[1]=~reset&q0==4'd9;
    assign c_enable[2]=~reset&q1==4'd9&q0==4'd9;
    assign OneHertz=q2==4'd9&q1==4'd9&q0==4'd9;
    bcdcount counter0 (clk, reset, c_enable[0],q0);
    bcdcount counter1 (clk, reset, c_enable[1],q1);
    bcdcount counter2 (clk, reset, c_enable[2],q2);	
endmodule
```

#### 3.2.2.7 4-digit decimal counter

 问题描述

>  
 构建一个 4 位 BCD（二进制编码的十进制）计数器。每个十进制数字使用 4 位编码：q[3:0] 是个位，q[7:4] 是十位等。对于数字 [3:1]，还输出一个使能信号，指示个位，十位，百位何时应加1。 

<img alt="" height="123" src="https://s2.loli.net/2024/02/22/VbEzjhTRMUcDoC3.png" width="289">

verilog代码

```verilog

module top_module (
    input clk,
    input reset,   // Synchronous active-high reset
    output [3:1] ena,
    output [15:0] q);
    reg [3:0] one;
    reg [3:0] ten;
    reg [3:0] hundred;
    reg [3:0] thousand;
    always @(posedge clk)
        begin
            if(reset || one==4'd9)
                one <= 4'b0;
            else
                one <= one+1'b1;
        end
    always @(posedge clk)
        begin
            if(reset || ((one == 4'd9) &  & (ten == 4'd9)))
                ten <= 4'b0;
            else if(one == 4'd9)
                begin
                      ten <= ten+1'b1;
                end
        end
     always @(posedge clk)
        begin
            if(reset || ((one == 4'd9) &  & (ten == 4'd9) &  & (hundred == 4'd9)))
                hundred <= 4'b0;
            else if((one == 4'd9) &  & (ten == 4'd9))
                hundred <= hundred+1'b1;
        end
     always @(posedge clk)
        begin
            if(reset || ((one == 4'd9) &  & (ten == 4'd9) &  & (hundred == 4'd9) &  & (thousand == 4'd9)))
                thousand <= 4'b0;
            else if((one == 4'd9) &  & (ten == 4'd9) &  & (hundred == 4'd9))
                thousand <= thousand+1'b1;
        end
   
    assign q = {thousand,hundred,ten,one};
    assign ena[1] = (one == 4'd9)?1'b1:1'b0;
    assign ena[2] = ((one == 4'd9) &  & (ten == 4'd9))?1'b1:1'b0;
    assign ena[3] = ((one == 4'd9) &  & (ten == 4'd9) &  & (hundred == 4'd9))?1'b1:1'b0;
endmodule
```

#### 3.2.2.8 12-hour clock

问题描述

>  
 创建一组适合用作 12 小时制的计数器（带有上午/下午指示器）。您的计数器由快速运行的clk计时，只要您的时钟增加（即每秒一次），就会 在ena上显示一个脉冲。 
 reset将时钟重置为 12:00 AM。pm对于 AM 为 0，对于 PM 为 1。hh、mm和ss是两个BCD（二进制编码的十进制）数字，分别表示小时 (01-12)、分钟 (00-59) 和秒 (00-59)。重置的优先级高于启用，即使未启用也可能发生。 
 以下时序图显示了从上午 11:59:59到下午 12:00:00的翻转行为以及同步复位和启用行为。 
  请注意， 11:59:59 PM提前到12:00:00 AM，12:59:59 PM提前到01:00:00 PM。没有 00:00:00。 


verilog代码

```verilog
module top_module(
    input clk,
    input reset,
    input ena,
    output pm,
    output [7:0] hh,
    output [7:0] mm,
    output [7:0] ss); 
    reg pm_temp;
    reg [3:0] ss_one;
    reg [3:0] ss_ten;
    reg [3:0] mm_one;
    reg [3:0] mm_ten;
    reg [3:0] hh_one;
    reg [3:0] hh_ten;
    wire pm_ding;
    always @(posedge clk)
        begin
            if(reset)
                begin
                    ss_one <= 4'b0;
                end
            else if(ena)
                begin
                    if(ss_one == 4'd9)
                        ss_one <= 4'b0;
                    else
                        ss_one <= ss_one+1'b1;
                end
        end
    always @(posedge clk)
        begin
            if(reset)
                begin
                    ss_ten <= 4'b0;
                end
            else if((ena) &  & (ss_one == 4'd9))
                begin
                    if(ss_ten == 4'd5)
                        ss_ten <= 4'b0;
                    else
                        ss_ten <= ss_ten+1'b1;
                end
        end
    always @(posedge clk)
        begin
            if(reset)
                begin
                    mm_one <= 4'b0;
                end
            else if((ena) &  & (ss_one == 4'd9) &  & (ss_ten == 4'd5))
                begin
                    if(mm_one == 4'd9)
                        mm_one <= 4'b0;
                    else
                        mm_one <= mm_one+1'b1;
                end
        end
    always @(posedge clk)
        begin
            if(reset)
                begin
                    mm_ten <= 4'b0;
                end
            else if((ena) &  & (ss_one == 4'd9) &  & (ss_ten == 4'd5) &  & (mm_one == 4'd9))
                begin
                    if(mm_ten == 4'd5)
                        mm_ten <= 4'b0;
                    else
                        mm_ten <= mm_ten+1'b1;
                end
        end
    always @(posedge clk)
        begin
            if(reset)
                begin
                    hh_one <= 4'd2;
                end
            else if((ena) &  & (ss_one == 4'd9) &  & (ss_ten == 4'd5) &  & (mm_one == 4'd9) &  & (mm_ten == 4'd5))
                begin
                    if(hh_one == 4'd9)
                        hh_one <= 4'b0;
                    else if((hh_one == 4'd2) &  & (hh_ten == 4'd1))
                        begin
                            hh_one <= 4'b1;
                        end
                    else
                        hh_one <= hh_one+1'b1;
                end
        end   
    always @(posedge clk)
        begin
            if(reset)
                begin
                    hh_ten <= 4'd1;
                end
            else if((ena) &  & (ss_one == 4'd9) &  & (ss_ten == 4'd5) &  & (mm_one == 4'd9) &  & (mm_ten == 4'd5))
                begin
                    if((hh_one == 4'd2) &  & (hh_ten == 4'd1))
                        hh_ten <= 4'b0;
                    else if(hh_one == 4'd9)
                        begin
                            hh_ten <= hh_ten+1'b1;;
                        end
                end
        end
    always @(posedge clk)
        begin
            if(reset)
                begin
                    pm_temp <= 1'b0;
                end
            else if(pm_ding)
                begin
                    pm_temp <= ~pm_temp;
                end
        end
    assign pm_ding = (hh_ten == 4'd1) &  & (hh_one == 4'd1) &  & (ena) &  & (ss_one == 4'd9) &  & (ss_ten == 4'd5) &  & (mm_one == 4'd9) &  & (mm_ten == 4'd5);
    assign ss = {ss_ten,ss_one};
    assign mm = {mm_ten,mm_one};
    assign hh = {hh_ten,hh_one};
    assign pm = pm_temp;
endmodule
```