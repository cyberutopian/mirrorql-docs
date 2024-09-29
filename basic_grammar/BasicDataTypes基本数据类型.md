# 基本数据类型

[引言](#引言)

[整型](#整型)

&ensp;&ensp;[内建操作](#内建操作)

[浮点类型](#浮点类型)

&ensp;&ensp;[内建操作](#内建操作)

[字符串类型](#字符串类型)

&ensp;&ensp;[内建操作](#内建操作)

[布尔型](#布尔型)

[基本数据类型转换](#基本数据类型转换)

## 引言

`MirrorQL`语言中的基本数据类型，包括整型、浮点型、布尔型以及字符串类型。

针对`MirrorQL`支持的数据类型，后续会扩展相关的内建函数--也即是系统已经帮用户写好的函数，用户可以直接拿过来使用。

## 整型

简单来说，整型变量用于保存整数，如`306`。例如：

```
let x: int = 306;
0
42
123
-2147483648
```

其中，整型使用`int` 关键字来标注。

### 内建操作

#### 整型转字符串

```
let x :int;
x.itos()
```

## 浮点类型

简单来说，而浮点型变量则用于保存浮点数，也就是带小数位的数，如`3.14`。例如：

```
let x: float = 3.14;
0.5
2.0
123.456
-100.5
```

其中，浮点类型使用`float`关键字来标注。

### 内建操作

#### 浮点转字符串

```
let x :float;
x.ftos()
```

## 字符串类型

&ensp;&ensp;字符串类型的变量用来存放以双引号开头和结尾的字符序列，即字符串。例如：

```
let x:str = "hello world"
let y = "hello"
let z:str = "He said, \"Logic clearly dictates that the needs of the many...\""
```

其中，字符串类型使用`str`关键字来标注;

`MirrorQL` 实现中支持字符串中包含相关的转义字符；

常见的转义字符如下：

```
n  \" 表示字符"
n  \\ 表示字符\
n  \n 表示换行符
n  \r 表示回⻋符
n  \t 表示制表符
```

### 内建操作

#### 正则匹配

基本类型`str`支持正则表达式匹配，对应`str`类型的成员谓词`match`。

用法为:`字符串.match(str类型的正则表达式)`。示例：

```
"aaa".match("*") 成立
"aaa".match("bbb") 不成立。
```

欲匹配的正则表达式不一定是字面量，`x=="bbb" and "aaa".match(x)`也是合法的用法。

#### 拼接

- `str`类型可以直接使用`+`运算符进行拼接。示例：

```
"aaa" + "bbb" == "aaabbb"
```

- `str`类型也可以通过方法`cat`来进行拼接。示例：

```
let x: str;
let y: str;
let z: str;
z == x.cat(y)
```

#### 获取长度

`str`类型的成员谓词`len`用来获取字符串的长度。

用法为`字符串.len()`，返回一个`int`。示例：

```
"aaa".len()==3
```

### 字符串转整型

`str`类型的成员谓词`toInt`，将字符串转整型。示例：

```
    let s: str;
    s == "123890"
    and
    i == s.toInt()
```

### 字符串转浮点

`str`类型的成员谓词`toFloat`，将字符串转浮点。示例：

```
    let s: str;
    s == "12389.98"
    and
    i == s.toFloat()
```

### 获取子串

`str`类型的成员谓词`substr(start, num)`，获取字串。其中`start` 表示字串的起始位置， `num`表示要获取的子串的个数。
示例：

```
    let s: str;
    s == "12389078qwert"
    i == s.substr(2,10)
```

!!! note "子串截取规则"

    当给出的截取长度大于实际可以截取的长度时，只会截取实际能截取的长度。例如`"abc".substr(1,100)=="bc"`。

    当给出的起始下标越界时，substr会返回一个空串，并在运行时给出警告。

## 布尔型

布尔型变量用来存放布尔值，即`no`（假）或者 `yes`（真）。

为了便于读者理解，这里举例说明：

```
let x: yon = yes;
```

在上面的代码中，我们定义了一个布尔型变量`x`，并将其赋值为`yes`。

其中，布尔类型使用`yon`关键字标注。

## 基本数据类型内建操作

基本数据类型的基本内建操作如下表：

|     实现的接口     | 内建 impl 的类型 |                            说明 |
| :----------------: | :--------------: | ------------------------------: |
|   toInt() -> int   |       str        |      转换为带符号 32 位整数类型 |
|   itos() -> str    |       int        |                整型转换为字符串 |
|   ftos() -> str    |      float       |                浮点转换为字符串 |
| toFloat() -> float |       str        | 换为带符号 32 位 IEEE754 浮点数 |
|        cat         |       str        |                      字符串连接 |
|        len         |       str        |                  获取字符串长度 |
|       substr       |       str        |                        获取子串 |

用示例说明使用这些内建方法：

```
fn intToStr() -> str{
    let x=range(1,10); // x为int
    result == x.itos() // x转换为string
}

fn floatToStr() -> str {
    let x: float;
    x == 19.89
    and
    result == x.ftos()
}

fn getInt() -> int{
    result == "1".toInt()
}

fn getFloat() -> float{
    result == 1.0 + toFloat("1")
}

fn catTest() -> str {
    let x: str = "xx";
    let y: str = "yyy";
    result == x.cat(y)
}

fn getlen() -> int{
    let x:str = "xxxxxxx";
    result == x.len()
}

fn getSubStr() -> str {
    let x: str = "xxxxxxx";
    result == x.substr(2, 3)
}

```

此外，`MirrorQL`还提供了内建的`itos`，`ftos`,`cat`,`len`,`substr`,`toInt`，`toFloat`谓词.

```
fn intToStr() -> str{
    let x=range(1,10); // x为int
    result == itos(x) // x转换为string
}

fn floatToStr() -> str {
    let x: float;
    x == 19.89
    and
    result == ftos(x)
}

fn getInt() -> int{
    result == toInt("1")
}

fn getFloat() -> float{
    result == 1.0 + toFloat("1")
}

fn catTest() -> str {
    let x: str = "xx";
    let y: str = "yyy";
    result == cat(x,y)
}

fn getlen() -> int{
    let x:str = "xxxxxxx";
    result == len(x)
}

fn getSubStr() -> str {
    let x: str = "xxxxxxx";
    result == substr(x, 2, 3)
}
```
