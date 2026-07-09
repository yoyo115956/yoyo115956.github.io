---
layout:       post
title:        'IC验证面试题（一）：SystemVerilog 语言基础'
date:         2025-03-01
header-style: text
catalog:      true
mathjax:      false
tags:
  - 芯片验证
  - SystemVerilog
  - 面试
  - 核心框架
---

> 本系列整理 IC 验证工程师面试高频考点，建议先自测再对照答案。本篇覆盖 SV 数据类型、多线程、面向对象、任务/函数、约束随机等基础语法。

---

## 一、数据类型

### Q1：定宽数组、动态数组、关联数组、队列各自特点是什么？

**定宽数组（Fixed-size Array）**
- 编译时确定大小，属于静态数组
- 分为压缩（packed）和非压缩（unpacked）两种：
  ```systemverilog
  bit [7:0][3:0] packed_arr;   // 压缩：定义在类型后、名字前
  bit [7:0] unpacked_arr[3:0]; // 非压缩：定义在名字后
  ```

**动态数组（Dynamic Array）**
- 运行时确定大小，使用前需要 `new[]` 分配空间
  ```systemverilog
  int dyn[];
  dyn = new[10];          // 分配 10 个元素
  dyn = new[20](dyn);     // 扩展到 20，保留原内容
  dyn.delete();           // 释放
  ```

**关联数组（Associative Array）**
- 类似哈希表（Hash/Dictionary），索引可以是任意类型且必须唯一
- 针对超大稀疏空间，只为实际写入的元素分配内存
  ```systemverilog
  int assoc[string];
  assoc["key1"] = 100;
  if (assoc.exists("key1")) ...
  assoc.delete("key1");
  ```

**队列（Queue）**
- 结合链表和数组的优点，可以在任意位置插入/删除元素
  ```systemverilog
  int q[$];
  q.push_back(1);   // 尾部插入
  q.push_front(0);  // 头部插入
  int v = q.pop_front();   // 取出头部
  int v = q.pop_back();    // 取出尾部
  q.insert(2, 99);         // 在索引 2 处插入
  q.delete(2);             // 删除索引 2
  ```

---

### Q2：队列的 `push_back` 和 `pop_front` 的区别？

- `push_back`：在队列**尾部**插入元素（入队）
- `pop_front`：从队列**头部**取出并移除元素（出队）
- 两者配合实现 FIFO 行为：`push_back` 入队，`pop_front` 出队

---

## 二、多线程

### Q3：`fork join` / `fork join_any` / `fork join_none` 的区别？

| 语法 | 行为 | 说明 |
|------|------|------|
| `fork join` | 等待所有子线程结束 | 所有 begin-end 并行，全部完成才继续 |
| `fork join_any` | 等待任意一个子线程结束 | 第一个完成即继续，其余仍在后台运行 |
| `fork join_none` | 不等待，立即继续 | 子线程在后台异步运行 |

```systemverilog
// 附加控制语句
wait fork;    // 阻塞，等待当前进程的所有子进程结束
disable fork; // 终止当前进程的所有活跃子进程
```

---

### Q4：多线程同步的三种机制？

**Event（事件）**：两个线程间的同步触发
```systemverilog
event ev;
// 线程 A：等待
@(ev);              // 等待事件触发（边沿敏感）
wait(ev.triggered); // 等待事件（电平敏感，同一时刻也可捕获）
// 线程 B：触发
-> ev;
```

**Semaphore（旗语）**：控制对共享资源的互斥访问，基于 key 机制
```systemverilog
semaphore sem = new(1); // 初始 1 个 key
sem.get(1);   // 获取 1 个 key（阻塞直到有空闲）
// ... 访问共享资源 ...
sem.put(1);   // 归还 key
```

**Mailbox（邮箱）**：两线程间的数据传递，支持有界/无界缓冲
```systemverilog
mailbox #(int) mbx = new(4); // 容量为 4 的有界邮箱
mbx.put(data);    // 发送（满则阻塞）
mbx.get(data);    // 接收（空则阻塞）
mbx.peek(data);   // 查看但不移除
```

---

## 三、Task 与 Function

### Q5：Task 和 Function 有哪些核心区别？

| 特性 | Function | Task |
|------|----------|------|
| 耗时 | 不可包含延迟/事件（仿真时刻 0 执行）| 可以有延迟、事件、时序控制 |
| 输入输出 | 至少一个 input，只能返回一个值，无 output/inout | 可以有 0 到多个 input/output/inout |
| 调用关系 | 可以调用 function，不能调用 task | 可以调用 function 和 task |
| 返回值 | 通过函数名返回单一值 | 通过 output/inout 传递多个值，不直接返回 |

---

## 四、面向对象（OOP）

### Q6：OOP 的三大特性？

**封装（Encapsulation）**：将数据和操作数据的方法封装在类中，隐藏内部实现细节。

**继承（Inheritance）**：子类（派生类/扩展类）继承父类（基类）的属性和方法，可以扩展或重写。

**多态（Polymorphism）**：基类句柄指向子类对象时，通过 `virtual` 方法声明，自动调用子类的实现。
```systemverilog
class Animal;
    virtual function void speak(); $display("..."); endfunction
endclass

class Dog extends Animal;
    function void speak(); $display("Woof!"); endfunction
endclass

Animal a = new Dog();  // 上转型
a.speak();             // 调用 Dog::speak()（多态）
```

---

### Q7：类的 `public`、`protected`、`local` 访问修饰符区别？

| 修饰符 | 本类 | 子类 | 外部 |
|--------|------|------|------|
| `public`（默认）| ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `local` | ✅ | ❌ | ❌ |

---

### Q8：深拷贝（deep copy）和浅拷贝（shallow copy）的区别？

**浅拷贝**：复制对象本身，但对象内部的引用（句柄）仍指向同一个地址。修改子对象会影响原对象。
- SV 中的默认 `copy()` 是浅拷贝

**深拷贝**：递归复制所有内容，包括内部引用指向的对象，新旧对象完全独立。
```systemverilog
function Packet deep_copy();
    Packet p = new();
    p.addr = this.addr;
    p.data = this.data;
    // 如果有内嵌对象，也要 new 一个新对象并拷贝
    p.sub_pkt = this.sub_pkt.deep_copy();
    return p;
endfunction
```

---

### Q9：`ref` 参数类型有什么特点？

- `ref` 传递的是变量的**引用**（地址），子程序中对其修改立即对调用者可见
- 传递数组时应尽量使用 `ref` 以获得最佳性能（避免复制大数组）
- 若不想让子程序修改数组，使用 `const ref`
```systemverilog
task fill_array(ref int arr[]);   // 子程序修改 arr 会影响调用方
task read_array(const ref int arr[]); // 只读，不可修改
```

---

## 五、约束随机

### Q10：`rand` 和 `randc` 的区别？

| 修饰符 | 行为 |
|--------|------|
| `rand` | 每次 `randomize()` 在取值范围内均匀随机（有放回抽样，类似掷骰子）|
| `randc` | 周期性随机：所有可能值都出现一次后才开始新一轮循环（无放回抽样）|

`randc` 适用于需要枚举所有模式的场景，如遍历所有寄存器地址。

---

### Q11：约束的几种形式？

```systemverilog
// 1. 范围约束（inside）
constraint c1 { addr inside {[8'h00:8'hFF]}; }

// 2. 权重约束（dist）
// := 每个值独立权重 n；:/ 权重 n 在范围内平分
constraint c2 { data dist {0 := 10, [1:254] :/ 80, 255 := 10}; }

// 3. 条件约束（if-else 或 ->）
constraint c3 { if (rw) data != 0; }
constraint c4 { rw -> addr[1:0] == 2'b00; } // rw 为真时 addr 必须对齐

// 4. 唯一约束（unique）
rand bit [3:0] a, b, c;
constraint c5 { unique {a, b, c}; } // a、b、c 互不相等

// 5. soft 约束（可被覆盖的默认约束）
constraint c6 { soft mode == 2'b00; } // 默认 mode=0，可在 inline 中覆盖
```

---

### Q12：如何关闭约束？

```systemverilog
obj.c_addr.constraint_mode(0);  // 关闭名为 c_addr 的约束块
obj.c_addr.constraint_mode(1);  // 重新打开

// inline 覆盖（优先级最高）
assert(obj.randomize() with { addr == 8'h20; });
```

---

## 六、Interface 与 Clocking Block

### Q13：在 Testbench 中使用 Interface 和 Clocking Block 的好处？

**Interface 的好处**：
- 将一组信号封装成一个接口，避免在各层次重复声明端口
- 信号变更时只需修改接口定义，降低出错率
- 提高可重用性，同一接口可在不同 TB 中复用

**Clocking Block 的好处**：
- 明确指定信号的**采样时刻**（提前于时钟沿）和**驱动时刻**（滞后于时钟沿）
- 避免 TB 与 DUT 之间的竞争冒险（Race Condition）
- 确保 TB 驱动的信号不会在同一时刻被 DUT 采样到中间状态

```systemverilog
interface apb_if(input clk);
    logic psel, penable, pwrite;
    logic [31:0] paddr, pwdata, prdata;
    
    clocking driver_cb @(posedge clk);
        default input #1step output #1; // 采样提前 1step，驱动延迟 1 个时间单位
        output psel, penable, pwrite, paddr, pwdata;
        input  prdata;
    endclocking
endinterface
```

---

## 七、`break` / `continue` / `return`

### Q14：三者的含义，`return` 之后函数剩余语句还会执行吗？

| 语句 | 作用范围 | 说明 |
|------|----------|------|
| `break` | 循环 | 立即结束整个循环，跳出 for/while/foreach |
| `continue` | 循环 | 跳过本次迭代，继续执行下一次循环 |
| `return` | 函数/任务 | 立即终止函数/任务执行，可携带返回值 |

**`return` 之后函数里剩下的语句不会执行**。`return` 会立即终止函数，跳回调用处。
