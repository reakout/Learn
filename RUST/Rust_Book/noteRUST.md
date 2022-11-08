# `RUST`权威指南读书笔记

[TOC]

## Char3 通用编程概念

### 变量与可变性

#### 变量

用let声明的变量默认是不可变的，如以下会报错

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

如果想要改变值的话，需要声明时加上`mut`

`let mut x = 5;`

#### 常量

`const`关键字，不允许使用`mut`也不能修改值，只能设置为**常量表达式**，且**类型必须注明**，也不能使用函数调用结果或者运行时结果

常量命名应该用全大写加下划线，如下:

`const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;`

#### 遮蔽

可以用`let`重复使用相同的变量名来遮蔽变量，如

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}
```

外层作用域里遮蔽一次，内层作用域里遮蔽一次，但传不出来，所以两次输出结果应该是12和6

遮蔽可以完成一些值的转换，但完成之后还是不可变的

与`mut`的区别，用`let`本质是再创建了一个新的变量，但是重复使用了相同的名称，因此用`let`遮蔽可以同名换类型，但`mut`不允许更改类型，如

```rust
fn main() {
    let spaces = "   ";
    let spaces = spaces.len();
}
```

```rust
fn main() {
    let mut spaces = "   ";
    spaces = spaces.len();
}
```

第一个ok第二个报错。

### 数据类型

`rust`是静态类型语言，即需要在编译器而非运行时得到所有的变量类型

#### 标量类型

表示单个值的类型（scalar），在`rust`中有整型、浮点、布尔和字符四种

 ##### 整型

| 长度   | 有符号类型 | 无符号类型 |
| ------ | ---------- | ---------- |
| 8 位   | `i8`       | `u8`       |
| 16 位  | `i16`      | `u16`      |
| 32 位  | `i32`      | `u32`      |
| 64 位  | `i64`      | `u64`      |
| 128 位 | `i128`     | `u128`     |
| arch   | `isize`    | `usize`    |

`isize`和`usize`取决于系统架构，64位则是64位，32位则32位

可能属于多个数字类型的数字字面值允许使用类型后缀指定类型，如`58u8`。数字字面值还可以用`_`作为不影响大小和正确性的可视分割符，如`1_000`

不同进制还可以直接用于字面量赋值

| 数字字面量         | 示例          |
| ------------------ | ------------- |
| 十进制             | `98_222`      |
| 十六进制           | `0xff`        |
| 八进制             | `0o77`        |
| 二进制             | `0b1111_0000` |
| 字节 (仅限于 `u8`) | `b'A'`        |

**整形默认是`i32`**

超出了表示范围时会发生整型溢出，在debug版本会检查整型溢出（how？？这需要运行时进行啊）如果发现存在，则会报panic退出，panic实际上就是运行时的操作。在release版本不会进行检查，如果溢出会进行two's complement srapping二进制补码包裹（比如255上限时对应256会溢出成0，类似一个循环队列）

##### 浮点类型

`rust`中的浮点类型有`f32`和`f64` 两类，默认为`f64`，因为速度与`f32`几乎相同，但精度更高，所有浮点数都有符号

采用`IEEE-754`标准，`f32`是单精度浮点型`f64`是双精度浮点型

##### 数字运算

支持加减乘除取模，且整数除法向下取整

##### 布尔类型

`bool`大小1Byte，取值`true`和`false`

##### 字符类型

`char`  跟cpp类似，字符用单印号，字符串用双引号

`rust`的字符类型大小为4Byte，采用Unicode标量，一个字节可以容纳范围远超ASCII，比如可以放下emoji 😎

##### 复合类型

复合类型指的是把多个值组合成一个类型。`rust`两种基本的符合类型是元组和数组

###### 元组

多种类型的多个值放到一个复合类型中，**长度固定**，语法是在小括号中用逗号间隔

如`let tup: (i32, f64, u8) = (500, 6.4, 1)`

想要从元组中取值需要使用模式匹配来解构（类似python写法）

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

也可以按顺序用索引加`.`取得，下标从**0**开始，如取`f64`类型的元素应该使用`tup.1`

空元组是一种特殊类型，也写成`()`，被称为**单元类型**，该值被称为**单元值**。如果表达式部返回任何其他值，就**隐式返回单元值**



###### 数组

数组中每个元素必须有相同类型，且`rust`中的数组也是**定长**的，使用方法是在方括号中用逗号分割定义。如`let a = [1, 2, 3, 4]`

数组一般用于需要将数据分配到**栈而非堆**时，`rust`中也有`vector`，应优先使用`vector`

也可以显式指定类型和长度定义数组，如`let a: [i32; 5] = [1, 2, 3, 4, 5]`

如果数组中每个元素都相同，可以直接值和个数定义，如`let a = [3; 5]`定义了5个3组成的数组（着一块和matlab语法很像。

数组的元素访问用`[]`

在运行时访问超出范围的值时会`panic`而不是像C一样稀里糊涂运行着。



### 函数

`main`函数是程序入口点，`rust`中函数使用下划线命名法，即所有字母都是小写的并且用下划线来分割单词。`rust`不在意函数定义位置，只要定义过就行。

函数必须声明每个参数的类型(类似c/cpp)，多个参数时用逗号分隔。

#### 语句和表达式

函数体由一系列语句组成，但也可以选择用表达式结尾。`rust`是一门特别的**基于表达式**的语言，选择语句或者表达式将会影响到函数体。

**语句**(statement)执行操作但不返回值。如`let y = 6;`

**表达式**(expression)计算并产生一个值，语句中的一部分也可以是表达式，比如上面的`6`就是一个表达式。函数**调用**是表达式，宏调用也是表达式，创建新作用域的大括号`{}`也是表达式

函数**定义**本身也是语句。和c赋值语句不同，语句没有返回值，所以不能写连等如`let x = (let y = 6);`



表达式的结尾没有分号。如果在表达式末尾加上分号，就会转换为不会返回值的语句

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1   // 没有;
    };

    println!("The value of y is: {}", y);
}
```

如其中y后的那一串`{}`里的就是表达式

#### 带返回值的函数

`rust`不像c一样写显式返回值，默认可以用隐式返回最后一个表达式的值，如果用`return`关键字和指定值，可以从函数提前返回。函数返回值的类型写在参数列表后用 `->`接类型表示。

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

如果带上分号，会让函数从表达式变成语句，没有返回值（即会返回一个空元组类型表示没有返回值），导致类型和i32不匹配报错。



### 注释

`rust`通用注释是`//`，还有一种文档注释留待之后再说。



### 控制流

#### if表达式

if else用法类似c中，但是if后必须是一个`bool`类型的表达式，`rust`不会自动把非布尔值转换成布尔值。

在多个if串else if中会检测第一个为真的if条件执行

##### let中使用if

类似于cpp中的三目运算符 ?:

如

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {}", number);
}
```

if表达式在if分支和else分支的结果必须是相同类型，不匹配的话会报错。

#### 循环

`rust`有三种循环，`loop` `while` `for`

##### loop

无条件循环，可用continue和break跳出，使用时可以带上循环标签（`loop label`）配合使用来跳出指定的循环而非默认的内层循环

```rust
// 无控制loop
fn main() {
    loop {
        println!("again!");
    }
}
```

```rust
// 带控制loop
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;

        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}
```

##### while

用法类似c，不过条件不用打括号，且必须为`bool`类型

##### for

`rust`中的for循环可以实现类似`python`中用法，主要用于遍历时有效避免`panic`

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {}", element);
    }
}
```

一个反转区间倒计时的例子

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

以上代码中rev是反转方法。



## Char4 所有权

所有权让`rust`无需垃圾回收器（GC-garbage collector）就可以实现内存安全。

### 什么是所有权

对于系统内存的控制，GC是不断寻找不再使用的内存，C/CPP中是需要开发者手动分配和释放，而`rust`使用所有权来管理系统内存，在编译时通过一系列检查，在**运行时，不会造成开销**

以下示例，基于字符串说明。



#### 堆和栈

`rust`中需要显式考虑数据放在堆还是栈上，两者结构不同。

`stack`栈是后进先出的，栈上所有的数据必须占用**已知且固定的大小**。如果在编译时大小未知或者可能变化的数据应该放在堆上。**把数据推入栈不被认为是分配**。

`heap`堆中的数据是无组织的。往堆里放数据时，先请求一定大小的内存空间，然后内存分配器（memory allocator）会在堆里某处找到一个足够大的空位，标记为已使用，然后返回一个指针。该过程称为在堆上分配内存（allocating on the heap）

入栈比堆上分配内存更快，因为不需要分配器搜索空间，位置总是在栈顶。

访问堆上的数据比栈上的更慢，因为需要通过指针访问。

与C/CPP类似，代码调用一个函数时，传给函数的值和函数的局部变量入栈。函数结束时，这些值出栈。

跟踪正在使用的堆上的数据（C/CPP中内存泄漏的原因），最大限度减少堆上重复数据，并清理堆上的死数据，这就是所有权系统处理的问题。

#### 所有权规则

三条基本规则：

* `rust`中的每一个值都有一个被称为其`所有者`(owner)的变量
* 值在任一时刻有且只有一个所有者
* 当所有者（变量）离开作用域，这个值将被丢弃

#### 变量作用域

**作用域**（scope）是一个**项**（item）在程序中有效的范围，如以下代码所示

```rust
fn main() {
    {                      // s 在这里无效, 它尚未声明
        let s = "hello";   // 从此处起，s 开始有效
        				   // s绑定了一个硬编码绑定进程序代码的字符串字面量

        // 使用 s
    }                      // 此作用域已结束，s 不再有效
}
```

从s进入作用域时，它就是有效的，一直持续到离开作用域为止

#### 内存与分配

字符串字面量是硬编码进程序的，不可变。用`String`来管理被分配到堆上的数据，用于存储在编译时位置大小的文本，使用`from`创建方法如下

`let s = String::from("hello");`

`::`用法类比CPP中命名空间

对其进行修改如下

```rust
fn main() {
    let mut s = String::from("hello");
    s.push_str(", world!"); // push_str() 在字符串后追加字面值
    println!("{}", s); // 将打印 `hello, world!`
}
```

对于`String`为了支持一个可变，可增长的文本片段，需要在堆上分配一块可改变大小的空间：

* 必须在运行时向内存分配器请求内存
* 需要一个处理完`String`后把内存返回给分配器的方法（类似于析构）

第一部分各个语言比较类似，第二部分千差万别，带GC的用GC处理，手动分配需要精确一个`allocate`配对一个`free`

`rust`使用策略是：内存在拥有它的变量；离开作用域后就被自动释放

```rust
fn main() {
    {
        let s = String::from("hello"); // 从此处起，s 开始有效

        // 使用 s
    }                                  // 此作用域已结束，
                                       // s 不再有效
}
```

`rust`中变量离开作用域时，调用的特殊函数被称为`drop`，`rust`会在结尾的`}`处自动调用`drop`。在CPP中，这种结束时释放资源的模式为资源获取即初始化（RAII-Resource Acquisition Is Initialization）

##### 变量与数据交互（一）：移动

```rust
fn main() {
    let x = 5;
    let y = x;
}
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
}
```

两种，第一个是整数，有已知固定大小的简单值，所以两个都被放在栈上。

第二个看上去类似，但是底层实现不一样。

`String`的底层实现如下，左边三个放在栈上，分别是指向堆上的指针，长度，容量，指针指向的数据放在堆上。

![String in memory](https://rustwiki.org/zh-CN/book/img/trpl04-01.svg)

在赋值时会复制栈上数据，而不会复制堆上数据，结果如下

![s1 and s2 pointing to the same value](https://rustwiki.org/zh-CN/book/img/trpl04-02.svg)

参考前面提到的，当一个变量离开作用域后会调用`drop`但是复制后两个指针指向了同一位置，此时s1，s2都离开时会二次释放(double free)一个区域。

为了避免这种情况，在`let s2 = s1`后会认为`s1`不再有效，故不会在`s1`离开作用域后清理任何东西。如下代码

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1);// 会因为无效引用而报错
}
```

只复制指针而不复制值很像浅拷贝，但不同的是这里让第一个变量无效了，所以被称为移动(move)。

同时也隐含了一个设计选择：`rust`永远不会自动创建数据的深拷贝，任何自动的复制可以忽略对性能的影响。

##### 变量与数据交互的方式(二)：克隆

用于深拷贝时，使用`clone`的通用函数，使用如下所示

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
}
```

调用`clone`时可能会出现大量性能消耗

与整型不同，整型或者字面量这种在编译时已知大小的类型被整个存储在栈上，其拷贝不会区分深浅拷贝，也不会在拷贝后让前者无效。

`rust`在底层使用的是`Copy trait`的特殊标注，用了这样标注的类型，在赋值后不会使旧变量无效。同时`rust`不允许自身或其任何部分实现了`Drop trait`的类型使用`Copy trait`，以下是一些常见的实现了`copy`的类型：

* 所有的整数类型
* 布尔类型
* 所有的浮点类型
* 字符类型
* 元组，当其所包含的类型都实现了`Copy`时

#### 所有权与函数

把值传递给函数在语义上和给变量赋值类似，可以发生移动和克隆。如下所示

```rust
fn main() {
  let s = String::from("hello");  // s 进入作用域

  takes_ownership(s);             // s 的值移动到函数里 ...
                                  // ... 所以到这里不再有效

  let x = 5;                      // x 进入作用域

  makes_copy(x);                  // x 应该移动函数里，
                                  // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
  println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
  println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```

返回值也可以转移所有权，如下代码

```rust
fn main() {
  let s1 = gives_ownership();         // gives_ownership 将返回值
                                      // 移给 s1

  let s2 = String::from("hello");     // s2 进入作用域

  let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                      // takes_and_gives_back 中,
                                      // 它也将返回值移给 s3
} // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃

fn gives_ownership() -> String {           // gives_ownership 将返回值移动给
                                           // 调用它的函数

  let some_string = String::from("yours"); // some_string 进入作用域

  some_string                              // 返回 some_string 并移出给调用的函数
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

  a_string  // 返回 a_string 并移出给调用的函数
}
```

所有权遵循的相同模式：将值赋给另一个变量时移动它。当持有堆里数据值的变量离开作用域时用`drop`清理掉。

一个常用的场景是怎么使用值而不需要专门返回所有权，为此提供的功能是引用(reference)

### 引用与借用

#### 基本概念

不使用引用时为了将参数的使用权返回，一个常用方法是返回一个元组，如计算字符串长度的代码如下

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度

    (s, length)
}
```

显得非常形式主义，使用引用的改进方案如下

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

引用（`&`）允许使用值而不获取使用权

与CPP类似，有解引用操作（dereferencing）使用`*`运算符

以上使用引用的示意图如下

![&String s pointing at String s1](https://rustwiki.org/zh-CN/book/img/trpl04-05.svg)

`&s1`创建了一个指向值`s1`的引用，但是并不拥有值，故引用停止时，其指向的值也不会被丢弃，函数使用引用作为参数时，无需返回值来交还所有权，因为压根儿未曾拥有过。

**创建一个引用的行为称为借用（borrow）**

与CPP不同，修改引用指向的变量是不行的，变量默认不可变，引用也一样。

#### 可变引用

同样加上`mut`关键字就可以实现对引用变量的修改

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

可变引用有一个限制，同一时间只能有一个对某一特定数据的可变引用，很像os中的读写同步问题，用于避免数据竞争（data race），其发生条件：

* 两个或更多指针同时访问同一数据
* 至少有一个指针用来写入数据
* 没有同步数据访问的机制

数据竞争会导致未定义行为，且几乎无法追踪和修复，`rust`直接选择不编译存在数据竞争的代码

可以通过大括号来创建新作用域来避免在一个作用域中的可变引用限制，如

```rust
fn main() {
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用
    // 注意到这里r1离开作用域后，这个变量实际被释放不可使用了

    let r2 = &mut s;
}
```

同样的规则也存在于同时使用可变与不可变引用中，即无法同时拥有可变引用和不可变引用。

一个引用的作用域是从声明的地方一直持续到最后一次使用（或者离开作用域被释放）为止，故如下代码可正确执行

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // 没问题
    let r2 = &s; // 没问题
    println!("{} and {}", r1, r2);// 可以把引用当成一个指针对象，被调用后就drop了
    // 此位置之后 r1 和 r2 不再使用

    let r3 = &mut s; // 没问题
    println!("{}", r3);
}
```

`rust`使用非词法作用域声明周期(Non-Lexical Lifetimes NLL)来判断作用域结束前是否不再使用。

#### 垂悬引用（Dangling Reference）

类似于C/CPP中的垂悬指针。即指针指向的内存可能已经被分配给其他持有者，`rust`编译器确保引用永远不会垂悬，即当拥有数据引用时，编译器确保**数据不会在引用之前离开作用域**

以下是一个报错示例

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle 返回一个字符串的引用

    let s = String::from("hello"); // s 是一个新字符串

    &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。
  // 危险！
```

#### 总结

* 在任意给定时刻，要么只能有一个可变引用，要么只能有多个不可变引用
* 引用必须总是有效的

### 切片Slice类型

`Slice`是另一个没有所有权的数据类型，用来引用集合中一段连续的元素序列，而不用引用整个集合

以一个例子来引入：编写一个函数，接收一个字符串，并返回在该字符串中找到的第一个单词

不需要得到`string`的所有权，所以参数用引用，但是没有真正获取部分字符串的办法，但可以妥协一波返回单词结尾的索引

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();// 将`String`转成字节数组

    for (i, &item) in bytes.iter().enumerate() {
        // .item()方法构建迭代器，返回集合中的每一个元素
        // enumerate返回一个元组，其中i是索引,&item是单个字节
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word 的值为 5

    s.clear(); // 这清空了字符串，使其等于 ""

    // word 在此处的值仍然是 5，
    // 但是没有更多的字符串让我们可以有效地应用数值 5。word 的值现在完全无效！
}

```

生成得到的word和s是完全没有关系的，这样需要时刻担心同步关系，啰嗦且易错

为此使用`Slice`方法来解决

#### 字符串`slice`

`String`中部分值的引用，使用方法如下

```rust
    let s = String::from("hello world");
    let hello = &s[0..5];
    let world = &s[6..11];
```

其中索引为左闭右开区间，实际创建的索引如下

![world containing a pointer to the byte at index 6 of String s and a length 5](https://rustwiki.org/zh-CN/book/img/trpl04-06.svg)

第一个索引可以省略表示从0开始，第二个索引也可以省略表示到最后

同时省略只留点时表示取整个字符串的`slice`故以下几种写法等价

```rust
let s = String::from("hello");
let slice = &s[0..2];
let slice = &s[..2];

let len = s.len();
let slice = &s[2..len];
let slice = &s[2..];

let slice = &s[0..len];
let slice = &s[..]
```



字符串`slice`的索引在ASCII字符集使用正常，在UTF-8字符使用时需要额外注意，在后续补充。

再回头看之前的需求，用`slice`实现如下

注意到slice的类型声明是`&str`

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!

    println!("the first word is: {}", word);
}
```

此时该部分就会报错了，因为拥有了`word`是`s`的不可变引用，而`clear`会试图调用获取一个可变引用，而最后的`println!`又会使用`word`的不可变引用，即该不可变引用在此时必须还是有效，即此时会同时存在可变和不可变引用，会编译失败。

#### 字符串字面量就是`slice`

字符串字面量被储存在二进制文件中，声明一个字符串字面量

`let s = "hello world"`时，`s`的类型是`&str`它会指向二进制程序特定位置的`slice`。这也导致字符串字面量是不可变的（因为变就意味着会需要另外一个引用）；`&str`是一个不可变引用

#### 用字符串`slice`作为参数

上面的代码中可以修改函数签名为

`fn first_word(s: &str) -> &str {`

这样利用了`deref coercions`的优势，在后面章节中会介绍。

现在只需要了解，定义获取一个字符串`slice`而不是`String`引用的函数可以使得我们的API更加通用且不会丢失任何功能

#### 其他类型的`slice`

其他类型数组也有`slice`，如

```rust
let a = [1, 2, 3, 4];
let slice = &a[1..2];
```

此时这个`slice`的类型是`&[i32]`，在`vector`部分会进一步详细描述。



## Char5 使用结构体组织相关联的数据

### 定义与实例化结构体

#### 定义

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

各项称为字段（`field`）是以`,`分隔

#### 创建

```rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
}
```

给各个字段赋初值，以`key: value`的形式完成

使用字段值使用`.`取值

`rust`中的不允许某个字段为可变的，必须整个**实例**（注意不是定义时）是可变的

#### 变量与字段同名时的字段初始化简写语法

当参数名与字段名完全相同时，可以使用字段初始化简写语法(field init shorthand)，如以下两个分别是简化前后的构造函数写法

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

#### 使用结构体更新语法从其他实例创建实例

结构体更新语法(struct update syntax)使用旧实例的大部分值然后只改变部分值来创建新实例时使用。

如已经有一个`user1`时，以下两种写法相同，后者使用结构体更新语法

```rust
let user2 = User {
    active: user1.active,
    username: user1.username,
    email: String::from("another@example.com"),
    sign_in_count: user1.sign_in_count,
};
let user2 = User {
    email: String::from("another@example.com"),
    ..user1
};
```

使用结构体更新语法就像使用`=`的赋值，因为移动了数据，这里由于用了`String`的赋值，会导致`user2`创建后`user1`中的该部分失效，但是如果这里只是改了`u32`和`bool`类型的值时，则仍然有效。两种写法本质一样，使用更新语法赋值本质是值的移动，参见之前的内容，如果该类型实现了`copy trait`就仍可以在更新语法赋值后使用。

#### 使用没有命名字段的元组结构体来创建不同类型

可定义与元组类似的结构体，称为元组结构体(tuple struct)，有结构体本身名称和字段类型，但是没有字段名，如

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

这里虽然Color和Point的成员类型相同，但是属于不同类型。

#### 没有任何字段的类单元结构体

定义一个无字段的结构体，称为**类单元结构体(unit-like structs)**，类似于**unit类型()**，多用于后面实现`trait`时，一个定义使用例子如下

```rust
struct AlwaysEqual;
fn main() {
    let subject = AlwaysEqual;
}
```

#### 结构体数据的所有权

成员用自身拥有所有权的类型而非引用，可以保证只要整个结构体有效则数据都有效，如果使用引用类型来定义，需要加上**生命周期(lifetime)**



### 结构体使用示例

#### 示例

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

`area`函数使用结构体的不可变借用，这样`main`会保持`rect1`的所有权并继续使用，所以函数签名用了`&`

#### 通过派生`trait`增加功能

##### 使用println打印调试信息

如果想要在调试程序时打印出所有结构体的字段，尝试使用`printfln!`宏

直接使用`println!("rect1 is {}", rect1);`会报错

提示`error[E0277]: Rectangle doesn't implement std::fmt::Display`

原因是这里`{}`会默认告诉`println!`使用`Display`格式，但是对于结构体因为格式不固定，故没有实现`Display`实现。

但是报错信息中有指明一条路

`note: in format strings you may be able to use {:?} (or {:#?} for pretty-print) instead`

于是将输出格式改成如下形式`println!("rect1 is {:?}", rect1)`

在`{}`中加入`:?`会告诉`println!`使用`Debug`输出格式，后者是一个`trait`，允许打印结构体，以在调试时看到结构体的值。

但此时直接编译仍会报错`error[E0277]: Rectangledoesn't implement Debug`

同样给出了提示信息`note: add #[derive(Debug)] to Rectangle or manually impl Debug for Rectangle`

`Rust`中有实现了打印调试信息功能，但是必须为结构体显式选择该功能，为此在结构体定义之前加上属性 `#[derive(Debug)]`

最后得到的代码如下

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}
```

最后的格式控制也可以使用`{:#?}`这样会让结构体输出结果更易读

##### 使用`dbg!`宏打印调试信息

该宏会接收一个表达式的所有权，打印出代码中调用`dbg!`宏时所在的文件和行号，表达式的结果，最后返回该值的所有权。但是使用`dbg!`宏会打印到标准错误控制台流(`stderr`)，相对的`println!`会打印到标准输出控制流(`stdout`)

下面是使用例子

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```

这里使用`dbg!(30*scale)`时由于会返回使用权，所以用起来就像没有使用该调用一样。下面使用引用以避免`dbg!`拥有所有权。

最后得到的输出如下

```bash
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```

除了该`trait`，`rust`还提供了很多通过`derive`属性来使用的`trait`，具体在char10会分析。



### 方法语法

与函数类似，使用`fn`关键字，拥有参数和返回值。但是其在结构体的上下文中被定义，且第一个参数总是`self`，代表调用该方法的结构体（类似`Python`）

#### 定义方法

把之前的`area`改成`Rectangle`中的方法，代码如下

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

其中`impl`是`implementation`的缩写，标记该块与`Rectangle`类型相关联，`self`在这里就相当于`Rectangle`故需要再加一个`&`表示使用的是引用。如果想要改变，一般推荐使用可变引用`&mut self`而不是直接获取所有权，后者通常用在当方法把`self`转换成别的实例的情况时，用来防止调用者在转换后再使用原来的实例。

方法与成员可以同名，但通常用来实现返回特定字段的值用，用来实现类似`CPP`中用`public`方法打印`private`成员的效果，这样的方法一般被称为`getters`

与`CPP`中不同，`rust`里由于`self`会在三种格式中固定选择，所以实现了自动解引用，不需要对象本身用`.`，对象的指针用`->`

#### 带有更多参数

增加一个需要获取另一个`Rectangle`实例作为参数的方法`can_hold`用来判断本长方形是否能完全包含另一个长方形。

增加后如下

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

#### 关联函数

所有在`impl`块中定义的函数称为关联函数(associated function)，由于其可以不以`self`为第一参数，所以其也不是方法，调用使用`::`，比如 `String::from`就是在`String`类型上定义的关联函数。

关联函数常用来作为某些构造函数，比如传一个参数，构造正方形`Rectangle`

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}

fn main() {
    let sq = Rectangle::square(3);
}
```

#### 多个`impl`块

每个结构体允许有多个`impl`块，不影响语义，在泛型和`trait`中会比较实用。



## Char6 枚举和模式匹配

枚举(enumeriations，简记为enums)允许列举可能的成员(variants)来定义类型。`Rust`中的枚举与`F#` `OCaml` `Haskell`这样的函数式编程语言中的代数数据类型最相似。

### 定义枚举

此处举例使用IP4和IP6标准处理

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

#### 枚举值

创建实例使用`::`如`let four = IpAddrKind::v4;` `let six = IpAddrKind::v6`

即枚举的成员位于其标识符的命名空间内，且`four`和`six`都是`IpAddrKind`类型的，可以用一个函数处理两个情况`fn route(ip_type: IpAddrKind) {}`，且对两个成员都能作为参数调用该函数。

使用枚举还能优化结构体

```rust

#![allow(unused)]
fn main() {
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
}
```

可以用更简洁方式表达相同概念，直接使用枚举把**数据**放进每个枚举成员，而不是把枚举作为结构体的一部分，如下

```rust
#![allow(unused)]
fn main() {
    enum IpAddr {
        V4(String),
        V6(String),
    }
    let home = IpAddr::V4(String::from("127.0.0.1"));
    let loopback = IpAddr::V6(String::from("::1"));
}
```

IpAddr枚举的新定义表明了`V4`和`V6`成员都关联了`String`值，这样就不需要一个额外的结构体了。

用枚举的另一个好处是每个成员可以处理不同类型和数量的数据，如IPv4总是由4个在0～255间的值组成，故此处可以存成4个u8而不是一个String，如下

```rust
#![allow(unused)]
fn main() {
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }
    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
}
```

实际上`Rust`标准库有定义`IpAddr`，它使用同样的枚举和成员，不过把成员中的地址数据嵌到了两个不同形式的结构体内

```rust
#![allow(unused)]
fn main() {
    struct Ipv4Addr {
        // --snip--
    }
    struct Ipv6Addr {
        // --snip--
    }
    enum IpAddr {
        V4(Ipv4Addr),
        V6(Ipv6Addr),
    }
}
```

尽管标准库中有包含`IpAddr`的定义，但仍可以创建和使用自定义的类型，因为我们并为将标准库中的定义引入作用域。

枚举包含多个结构体，主要优点在于可以轻易定义一个处理这些不同类型结构体的函数，如以下两种写法

```
#![allow(unused)]
fn main() {
	enum Message {
    	Quit,
    	Move { x: i32, y: i32 }, //此处是一个匿名结构体
    	Write(String),
    	ChangeColor(i32, i32, i32),
	}
    struct QuitMessage; // 类单元结构体
    struct MoveMessage {
        x: i32,
        y: i32,
    }
    struct WriteMessage(String); // 元组结构体
    struct ChangeColorMessage(i32, i32, i32); // 元组结构体
}
```

枚举与结构体类似，可以使用`impl`来为枚举定义方法，如下在`message`上定义了一个`call`方法

```rust
#![allow(unused)]
fn main() {
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }
    impl Message {
        fn call(&self) {
            // 在这里定义方法体
        }
    }
    let m = Message::Write(String::from("hello"));
    m.call();
}
```

#### Option枚举和其相对于空值的优势

`Option`是标准库定义的枚举，编码了一个非常普遍的场景，即一个值要么有值要么没值（ a value could be something or it could be nothing.For example, if you request the first of a list containing items, you would get a value. If you request the first item of an empty list, you would get nothing.）

`Rust`中并没有其他语言中存在的空值功能。**空值（NULL）**是一个代表没有值的值。在有空值的语言中，变量总是两个状态之一：空值和非空值。问题在于当尝试像非空值一样使用空值时会出现错误。

问题在于实现而非概念，`Rust`中虽然没有空值，但有一个可以编码存在不存在概念的枚举`Option<T>`定义在标准库中，如下

```rust
#![allow(unused)]
fn main() {
    enum Option<T> {
        Some(T),
        None,
    }
}
```

该枚举已经被包含在`prelude`（`prelude` 是 `Rust` 自动导入每个 `Rust` 程序的内容的列表。 它保持尽可能的小，并专注于几乎在每个 `Rust` 程序中使用的东西，尤其是 `traits`。）中，不需要显式将其引入作用域。其成员也是如此，不需要`Option::`前缀就可以直接使用`Some`和`None`。使用如下

```rust
#![allow(unused)]
fn main() {
let some_number = Some(5);
let some_string = Some("a string");
let absent_number: Option<i32> = None;
}
```

`<T>`是泛型参数，暂时可以按模板理解，对于`Some`成员可以包含任意类型，因为编译器可以推断出来。而`None`需要指定类别，因为编译器推断不出来。

使用`Option<T>`比空值好的原因是：`Option<T>`和`T`是不同类型，编译器将不允许项有效值一样使用`Option<T>`，如以下代码不能编译

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);
let sum = x + y;
```

换句话说，对于`Option<T>`进行`T`的运算前需要将其转换为`T`，这通常能帮助我们捕获到空值最常见的问题之一：假设某值不为空但实际上为空的情况。

`Option<T>`其他用法参见[文档](https://rustwiki.org/zh-CN/std/option/enum.Option.html)



### match控制流运算符

`match`控制流运算符允许我们将一个值同一系列模式相比较，并根据相匹配的模式执行相应代码。模式可由字面量、变量、通配符和其他内容组成。

值会通过`match`的每一个模式，并且在遇到**第一个**"符合"的模式时，值进入到相关联的代码块中并被使用。

以硬币分类为例演示`match`如下

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {}
```

与`if`不同，前者必须返回一个布尔值，而`match`可以返回任何类型

`match`的分支由两部分组成：一个模式和一部分代码，两者用`=>`分隔，不同分支之间用`,`分隔。

从上往下执行，如果模式不匹配就执行下一个分支，每个分支的相关联代码是一个表达式，其结果值将作为整个`match`表达式的返回值。

分支后的代码如果较长可以使用大括号，但此时与下一个分支之间不需要使用`,`分隔开，如下

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
fn main() {}
```

#### 绑定值的模式

匹配分支另一个功能是可以绑定匹配的模式的部分值。这也是从枚举变量中提取值的方法。

如25美分硬币与州相关，体现在枚举中如下

```rust
#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
fn main() {}
```

在匹配分支的模式中增加一个叫`state`的变量用于匹配到`Coin::Quarter`时绑定，然后在分支的代码中可以使用该变量，如下

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

以上代码中由于枚举`UsState`未实现`display`故想要用`println`输出需要用调试模式。

#### 匹配`Option<T>`

可以像处理`Coin`枚举一样把`Some`中的值绑定出来使用，如编写一个函数，尝试获取`Option<i32>`如果其中有值，就将其加一。否则返回`None`值。

```rust
fn main() {
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }
    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
}
```

`Rust`中的匹配必须是穷尽的，如果忽略了某几种模式会编译报错。

#### 通配模式和_占位符

希望对一些特定的值采取特殊操作，而对其他值采取默认操作时。

```rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
}
```

使用`other`表示其他所有情况，类似于`default`

一般放在最后，否则会警告，因为此后的分支永远不会被匹配到。

当不想使用通配符获取的值时，使用`_`，这是一个特殊的模式，可以匹配任意值而不绑定到该值。告诉`Rust`该值不会被使用，故也不会得到相关警告

也可以使用单元值(空元组)来作为`_`分支的代码，如下

```rust
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
}
```

直接可以理解为其余分支什么都不做。

### if let 简单控制流

