# 表达式(Expressions)

[表达式(Expressions)](#表达式expressions)

[常量（Literal）](#常量literal)

[括号表达式(Parenthesized expressions)](#括号表达式parenthesized-expressions)

[范围表达式(Ranges)](#范围表达式ranges)

[注释(exegesis)](#注释exegesis)

&ensp;&ensp;[单行注释](#单行注释)

&ensp;&ensp;[多行注释](#多行注释)

[聚合（Aggregations）](#聚合aggregations)

&ensp;&ensp;[Any 表达式（Any）](#any表达式any)

[一元运算（Unary operations）](#一元运算unary-operations)

[二元运算（Binary operations）](#二元运算binary-operations)

&ensp;&ensp;[二元算术运算（Binary arithmetic operations）](#二元算术运算binary-arithmetic-operations)

&ensp;&ensp;[二元比较运算（Binary comparison operations）](#二元比较运算binary-comparison-operations)

&ensp;&ensp;[二元逻辑运算（Binary logic operations）](#二元逻辑运算binary-logic-operations)

["Donʼt-care"表达式(Donʼt-care expressions)](#donʼt-care表达式donʼt-care-expressions)

[类型判断](#类型判断)

[if-else 控制表达式](#if-else-控制表达式)

[量词](#量词)

&ensp;&ensp;[exists 量词](#exists-量词)

&ensp;&ensp;[forall 量词](#forall-量词)

## 表达式(Expressions)

表达式的计算结果为一组值，并且该值具有一个确定的数据类型。

例如，表达式`1+2`的计算结果为`3`，该表达式的类型为`int`。

## 常量（Literal）

可以在`MirrorQL`源码中直接表达某些值，例如数字、布尔值、字符串等。

1. 布尔值：有`yes` 和`no` 两种值表示

2. 整型数字：这些是十进制的数字（`0` 到 `9`）的序列，可能以减号（`-`）开头，例如：

```
0
42
-2048
```

3. 浮点数字：这些是由点号（`.`）分割的十进制数字序列，可能以减号（`-`）开头，例如：

```
2.0
123.456
-3.14
```

4. 字符串：这些可以通过讲字符括在引号（`"..."`）中来定义字符串。大部分的字符代表其本来字符值，但是可以根据需要使用反斜杠来`转义`一些字符，例如：

```
"hello world"
"They said, \"Please escape quotation marks!\""
```

## 括号表达式（Parenthesized expressions）

带括号的表达式是用括号`'('`和括起来的表达式`')'`。此表达式的类型和值与原始表达式完全相同。括号可用于将表达式分组在一起以消除歧义并提高可读性。

## 范围表达式(Ranges)

范围表达式表示在两个表达式之间排序的值的范围。它由两个表达式分隔，并用方括号（`[` 和`]`）括起来。

例如，`range(3, 7)`是有效的范围表达式。它的值是和之间`(`包括和本身`)`之间的任何整数 。

`[3 , 7]`表示`3`和`7`内所有的整数在有效范围内，开始和结束表达式是整数，浮点数。如果其中一个是整数，另一个是浮点数，则两者都将被视为浮点数。

详细参考[MirrorQL 内建谓词](./Predicates谓词.md#mql内建谓词mql-built-in-predicates)

## 注释（exegesis）

注释是计算机语言的一个重要组成部分，用于在源代码中解释代码的作用，可以增强程序的可读性，可维护性。

注释不会被编译器包含在最终的可执行程序中，因此不会影响程序的运行。

注释是良好的编程习惯，它们帮助程序员更容易地理解代码的用途和功能，并且在团队协作中非常有用。

`MirrorQL` 注释主要有两种类型：

1. 单行注释
2. 多行注释

### 单行注释

单行注释以双斜杠 `//` 开始：

```
// 这是一个单行注释
int x = 10; // 初始化一个变量x为10
```

### 多行注释

多行注释以 `/*`开始，以`*/`结束：

```
/*
这是一个多行注释
可以用来注释多行代码
*/
int y = 20; // 初始化一个变量y为20
```

## 聚合（Aggregations）

`MirrorQL`支持将多个值通过特定运算聚合为一个值，称为聚合操作。

目前支持的聚合操作有：

| 操作名 |                          用法                           |                       说明                        |
| :----- | :-----------------------------------------------------: | :-----------------------------------------------: |
| count  | `count( \| 需要计数的变量列表 \| 变量需要成立的逻辑式)` | 返回`int`，表示使得逻辑式成立的所有变量取值的个数 |
| min    |  `min( \| 需要取最小值的变量 \| 变量需要成立的逻辑式)`  |                  返回变量最小值                   |

操作示例如下：

```
fn getCount()->int{
    result == count(|x: int, y: int|x==range(1,3) and y==range(1,3) and any())
}
// getCount()返回值为 2*2 = 4

fn getCount1(v:int) -> int{
    v == range(1,3) and result == count(|x: int| x==range(1,3) and x>=v)
}
// getCount1(1)返回值为2（成立的x有1,2）
// getCount1(2)返回值为1 (成立的x有2)
// getCount1(3)不返回值（谓词不成立）
```

## Any 表达式（Any）

[MirrorQL 内建谓词](./Predicates谓词.md#mql内建谓词mql-built-in-predicates)。

## 一元运算（Unary operations）

在`MirrorQL`中定义了两种一元运算符 （`-`） 和`not`，其中:

`-`后面跟`int` 或者`float`表达式，表示算术取反；

`not`后面紧跟逻辑式，表示逻辑的否定。

例如：

```
-6.28
-(10-4)
not (1==1 and 2==2)
```

### 一元减号（-）

一元减号操作仅是对原表达式进行算术取反，无其他特殊含义；

!!! note "一元(-)的要求"
一元减号操作后面必须是`int` 或者`float`型的表达式

### 一元`not`

一元`not`运算作用于逻辑式，表示逻辑的否定。逻辑否定加在距离最近的逻辑式上。

当`not`作用于含有逻辑约束的表达式构成的逻辑式上时，将表达式隐含的逻辑约束视为与原逻辑式用`and`进行连接的，`not`作用于新的逻辑式上。

## 二元运算（Binary operations）

### 二元算术运算（Binary arithmetic operations）

`MirrorQL`支持`5`种算术运算，均有两个操作数，如下表所示：

| 运算符 |          示例          |                                           说明                                            |
| :----- | :--------------------: | :---------------------------------------------------------------------------------------: |
| +      | 1+1、1.0+3.14、"a"+"b" | 表示数字加法：(int,int) -> int，(float，float) -> float、表示字符串拼接：(str,str) -> str |
| -      |     1-1、1.0-3.14      |                  表示数字减法：(int,int) -> int，(float,float) -> float                   |
| \*     |    1\*1、1.0\*3.14     |                    表示乘法：(int,int) -> int，(float,float) -> float                     |
| /      |     1/1、1.0/3.14      |     <br>表示向下取整除法：(int,int) -> int,</br>表示普通除法：(float,float) -> float      |
| %      |          5%3           |                                  取模：(int,int) -> int                                   |

如果两个表达式都是数字，则这些运算符将充当标准算术运算符。

例如:`10.6 - 3.2`结果值为`7.4`，`123.456 * 0`值为`0`，`9%4`值为`1`（取余数）。

如果两个操作数均为整数，则结果为整数。否则，结果为浮点数。

还可以将`+`用于字符串的连接操作，但是两个操作数必须都是字符串。

### 二元比较运算（Binary comparison operations）

| 运算符 |     作用     |
| :----- | :----------: |
| ==     |   判断相等   |
| !=     |  判断不相等  |
| <      |   判断小于   |
| >      |   判断大于   |
| <=     | 判断小于等于 |
| >=     | 判断大于等于 |

!!! note "二元比较符对表达式的类型要求"

    二元比较运算符要求左侧和右侧的表达式类型[兼容](typing.md#类型兼容性)。此外，只有与`int`、`str`、`float`、`bool`兼容的类型才能使用大小判断的二元比较符。

    `"1"==1`非法，因为表达式左侧类型为`str`，右侧类型为`int`。两者不兼容。

#### in 语法糖

为了简化包含多个相等判断的程序，`MirrorQL`引入了 `in`语法糖。

`in`语法糖的语法为`<lhs_expr> in [expr1,expr2,...]`。与逻辑式`(lhs_expr == expr1 or lhs_expr == expr2 or ... )`等价。

例如，`callee_name in ["system","exe"+"cve"]`等价于`(callee_name == "system" or callee_name == "exe"+"cve")`。

### 二元逻辑运算（Binary logic operations）

| 运算符 |   作用   |
| :----- | :------: |
| and    |  逻辑与  |
| or     |  逻辑或  |
| not    | 逻辑否定 |

逻辑连接符可以连接多个逻辑式。例如：

```
1==1 and 2==2

1==1 or (2==3 and 4==5)

not (1==1 and 2==2)

```

!!! note "二元逻辑运算符对表达式的类型要求"

    在MirrorQL语言中，逻辑运算只能作用于逻辑表达式，也即两个表达式必须是逻辑表达式。

## "Donʼt-care"表达式(Donʼt-care expressions)

在 MirrorQL 语言中，用单下划线（`_`）来表示`Donʼt-care`表达式，它代表任何值，可以在谓词参数或者谓词的返回值处。

在使用其他谓词时，可能会遇到需要省略参数的情况，可以使用`_`代表参数省略。

与其他表达式不同，`“Donʼt-care”`表达式没有任何的类型，也就意味着`_`没有任何的成员谓词，不能调用`_.somePredicate()`。

例如，我们需要通过`people_age`谓词查询是否存在一个年龄为`28`的人。`age_query`展示了使用`don't care`表达式的查询方法。`age_query_exists`展示了使用量词的查询方法。

```
fn people_age(name:str,age:int){
    name=="Alice" and age==20
    or
    name=="Bob" and age==25
}

fn age_query() {
    people_age(_,28)
}

fn age_query_exists() {
    let some_name:str;
    people_age(some_name,28)
}
```

此外，`_`还可用于省略谓词的返回值，例如

```
fn age_query() {
    _ == get_people_by_age(28)
}
```

## 类型判断

`MirrorQL`引入了`instanceof`关键字，用来判断某个表达的值是否属于某个类型。

`instanceof`的语法为`表达式 instanceof 类型名`。示例说明`instanceof`的用法：

```
class SpecialNumber extends int{
    fn init() {
        self == 1
    }
}

fn test() {
    let y=2;
    y instanceof SpecialNumber
}

fn test2() {
    range(1,3) instanceof SpecialNumber
}
```

!!! note "instanceof 判断类型的要求"

    每一个表达式都有一个定义类型，合法的`instanceof`要求表达式的定义类型和欲判断的类型兼容。

    `1 instanceof str`是非法的，因为表达式`1`的定义类型为`int`，与`str`不兼容。

    `“string” instanceof str`合法且永远成立，因为表达式的定义类型为`str`，欲判断的类型也是`str`，判断永远成立。

## if-else 控制表达式

`MirrorQL`中可以使用 if 语句编写带有条件关系的逻辑式。一个`if`块相当于一个逻辑式。

`if`语句的典型语法包括：

不带`else`分支的`if`块：`if (条件逻辑式1) { 逻辑式2 }`；

以及带`else`分支的`if`块：`if (条件逻辑式1) { 逻辑式2 } else { 逻辑式3 }`；

以及带`else if`分支的`if`块：`if (条件逻辑式1) { 逻辑式2 } (else if(条件逻辑式) { 逻辑式})* else { 逻辑式 }`；

`if`语句的几种示例写法：

```
fn test(x:int) -> int{
    if (x==range(1,3)){
        result==x
    }
}

fn test2(x:int) -> int{
    if (x==range(1,3)){
        result==1
    } else{
        result==-1
    }
}

// 可在if语句前和条件内定义变量
fn test3(x:int) -> int{
    let y==range(1,3);
    if (x==y){
        let z=1;
        result==z
    }
}

// if可以嵌套
fn test4(x:int) -> int{
    if (x>1){
        if(x>2) {
            result==1
        }
    }
}

```

!!! note `if`表达式的误用情况

    `if`块本身是一个逻辑式。一个谓词体内只能有一个逻辑式，如果包含多个`if`块则非法。下面的程序是非法的。

    ```
    fn test(x:int) -> int{
        if (x==1) {
            result==1
        }
        if (x>2) {
            result==2
        }
    }
    fn test1(x:int) -> int{
        x!=1
        if (x>2) {
            result==2
        }
    }
    ```

    如果想表达如果`x==1`则返回`1`，否则若`x>2`则返回`2`，可以将程序修改为

    ```
    fn test(x:int) -> int{
        if (x==1) {
            result==1
        }else{
            if (x>2) {
                result==2
            }
        }
    }
    ```

    或

    ```
    fn test(x:int) -> int{
        if (x==1) {
            result==1
        }
        or
        if (x>2) {
            result==2
        }
    }
    ```

    此外，`if`的条件为一个逻辑式，如果填入表达式则语法错误：

    ```
    fn getTrue() -> bool{
        result=true
    }

    fn test() {
        if (getTrue()) { // 非法：getTrue()是表达式而非逻辑式
            any()
        }
    }
    ```

    应改为：

    ```

    fn getTrue() -> bool{
        result==true
    }

    fn test() {
        if (getTrue()==true) { // 逻辑式
            any()
        }
    }
    ```

## 量词

为了增强逻辑式的表达能力，`MirrorQL`允许为逻辑式引入量词。量词包括存在量词`exists`和全称量词`forall`。

### exists 量词

`MirrorQL`使用`exists`表示存在量词，其语法格式：

```
exists(|自由变量列表| 需要成立的逻辑式)
```

自由变量列表的每个变量可以写为`变量名:变量类型，或变量名=变量约束`。

此量化公式引入了一些新变量。它确定变量是否可以使用至少一组值来使主体中的公式为真。

例如：`exists(int i | i instanceof OneTwoThree)`，引入一个满足`OneTwoThree`类型的变量`i`。

`exists`的示例如下：

```
fn NumberInRange(x:int) {
    x == range(1,10)
}

fn test1() {
    exists(|x:int| NumberInRange(x))
}

fn test2() {
    not exists(|x:int| NumberInRange(x))
}

fn test3() {
    exists(|x=range(1,5)| NumberInRange(x))
}

fn test4() {
    let y:int; // 声明一个类型为int的自由变量y
    y=range(1,5) and exists(|x=y| NumberInRange(x))
}

fn test5() {
    exists(|x=range(1,2),y=range(1,3)| x==y)
}
```

### forall 量词

`MirrorQL`使用`forall`表示全称量词。其语法格式如下：

```
forall(|自由变量列表| 需要总成立的逻辑式)
或
forall(|自由变量列表| 约束自由变量范围的逻辑式,需要总成立的逻辑式)
```

和`exists`相似，自由变量列表的每个变量可以写为`变量名:变量类型，或变量名=变量约束`。

`forall`引入了一些新的变量，它对所有保持的值成立则算式成立。

例如：`forall(int i | i instanceof OneTwoThree ｜ i < 5）`,当`i`属于`OneTwoThree`并且`i<5`时，算式才成立；如果`i`有一个值大于`5`，则不成立。

**tips**：`forall(<vars> | <formula 1> | <formula 2>`和`not exists(<vars> | <formula 1> | not <formula 2>)`逻辑上相同（包含的概念`formula2`的集合包含`formula1`）。

示例：

```
fn NumberInRange(x:int) {
    x == range(1,10)
}

fn test1() {
    forall(|x:int| x==range(1,10) , NumberInRange(x))
}

fn test2() {
    forall(|x=range(1,10)| NumberInRange(x))
}

fn test3() {
    let y=range(1,3); // 声明类型为int的自由变量y，并给出范围约束
    forall(|x=range(1,10)| x>=y and NumberInRange(x))
}
```

#### forall 对自由变量的要求

`forall`要求所有的自由变量都要先明确限定范围。限定范围的方式包括：自由变量列表中写出变量约束、单独给出约束变量范围的逻辑式。

若自由变量没有明确限定范围，程序是非法的。

```
forall(|x=range(1,10)| P(x)) // 自由变量列表写出变量约束
forall(|x:int| x==range(1,10), P(x)) // 单独给出约束变量范围的逻辑式
forall(|x:int| P(x)) // 非法
```

#### forex 语法糖

需要注意的是，如果`forall`的某个自由变量的值域为空集，那么`forall`量词是成立的。

例如`forall(|x=1| x=2, P(x))`总成立，因为 `x==1 && x==2`的约束使`x`的值域为空集。

`forex`语法糖对`forall`的行为做了扩展，`forex`成立当且仅当`forall`成立且值域非空。

也就是说，`forex(|自由变量列表| 自由变量约束,需要总成立的逻辑式)`等价于`exists(|自由变量列表| 自由变量约束) and forall(|自由变量列表| 自由变量约束,需要总成立的逻辑式)`。
