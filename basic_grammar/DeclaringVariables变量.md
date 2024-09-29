# 变量(Variables)

[变量（Variables）](#变量variables)

[声明一个变量（Declaring a variable）](#声明一个变量declaring-a-variable)

[自由变量和约束变量（Free and bound variable）](#自由变量和约束变量free-and-bound-variable)

## 变量（Variables）

`MirrorQL`中的变量与代数或逻辑中的变量的使用方式相似；它们表示一组值，这些值通常受公式限制。

这与某些其他编程语言中的变量不同，在其他编程语言中，变量表示可能包含数据的内存位置，该数据还可以随着时间变化。

例如，在`MirrorQL`中，`n = n + 1` 是一个等式公式，仅当`n` 等于`n + 1` 时才成立（因此，实际上它不适用于任何数值）。

在`Java`中，`n = n + 1` 不是等式，而是通过添加`1` 到当前的值`n`，并进行新的赋值来改变`n`代表的值。

## 声明一个变量（Declaring a variable）

`MirrorQL`还支持在谓词体中使用`let`语句声明自由变量。

`let`语句有两种语法：**`let 变量名:变量类型;`** 或 **`let 变量名 = 表达式;`**。

对于第二种语法，`MirrorQL`编译器将自动推断变量类型。其中变量名是一系列合法的标识符字符；

1. 标识符字符是小写 `ASCII` 字母（`a` 到 `z`、`U+0061` 到 `U+007A`）、大写 `ASCII` 字母（`A` 到 `Z`、`U+0041` 到 `U+005A`）、十进制数字（`0` 到 `9`、`U+0030` 到 `U+0039`）和下划线（`_`、`U+005F`），标识符的第一个字符必须是字母。
2. 标识符不能具有与关键字相同的字符序列。

合法的标识符示例如下：

```
width
Window_width
window5000_mark_II
```

`let`语句相当于谓词顶层的`exists`量词。`let`的用法示例如下：

```
fn NumberInRange(x:int) {
    x == range(1, 10)
}

fn test1() {
    let x:int;
    NumberInRange(x)
    //相当于 exists(|x:int| NumberInRange(x))
}

fn test3() {
    let x = range(1, 5)
    NumberInRange(x)
    //相当于 exists(|x = range(1, 5)| NumberInRange(x))
}

fn test4() {
    let y = range(1, 5)
    let x = y;
    NumberInRange(x)
    //相当于 y = range(1, 5) and exists(|x = y| NumberInRange(x))
}

fn test5() {
    let x = range(1 ,2)
    let y = range(1, 3)
    x == y
    //相当于 exists(|x = range(1 ,2),y = range(1, 3)| x == y)
}
```

`MirrorQL`中的`keyword` 如下：

```
use
as
and
or
not
land
lor
class
exists
forall
instanceof
fn
for
if
else
impl
self
super
package
trait
let
mod
range
extends
yes
no
int
str
float
uint
count
min
max
enum
from
where
select
record
in
match
itos
ftos
toInt
toFloat
substr
len
cat
ord
none
ant
yon
inline
no_inline
magic
no_magic
result
```

## 自由变量和约束变量（Free and bound variable）

变量可以具有不同的作用。

有些变量是`free`，它们的值直接影响使用它们的表达式的值，或者使用它们的公式是否 成立。

其他变量（称为绑定变量）仅限于特定的值集。

在一个示例中，最容易理解这种区别。看一下以下表达式：

```
"hello".indexOf("l")

min(float f | f in [-3 .. 3])

(i + 7) * 3

x.sqrt()

```

第一个表达式没有任何变量。它找到`"l"` 字符串中出现位置的（从零开始的）索引`"hello"` ，因此它的结果为`2` 和`3`。

第二个表达式的计算结果为`[-3 .. 3]` 范围内的最小值`-3`。尽管此表达式使用变量`f` ，但它只是一个占位符或“虚拟”变量，不能为其分配任何值。

您可以使用其他变量替换 `f` 而无需更改表达式的含义。例如，`min(float f | f in [-3 .. 3])` 始终等于`min(float other | other in [-3 ..3])` 。这是**绑定变量**的示例。

表达式`(i + 7) * 3` 和 `x.sqrt()`有该是怎样的呢？在这两种情况下，表达式的值取决于什么值赋值给变量和分别。

换句话说，变量的值对表达式的值有影响。这些是自由变量的示例。同样，如果一个公式包含自由变量，则该公式可以保留还是不保留，取决于分配给这些变量的值。例如：

```
"hello".indexOf("l") = 1

min(float f | f in [-3 .. 3]) = -3

(i + 7) * 3 instanceof int

exists(float y | x.sqrt() = y)

```

第一个公式不包含任何的变量，并且永远不成立（因为它`"hello".indexOf("l") = 1` 具有值`2`和`3`， 从不为`1`）。

第二个公式仅包含一个绑定变量，因此不受该变量更改的影响。由于等于，所以该公式始终成立。

第三个公式包含一个自由变量`i`，公式是否成立，取决于分配给的值`i`。例如，如果`i`分配了`1`或者`2`（或者其他的`int`），则公式成立。另一方面，如果`i`已分配`3.5`，则它不成立。

最后一个公式包含一个自由变量`x` 和一个绑定变量`y`。如果`x`分配了一个非负数，则最终公式成立。另一个方面，例如，如果`x`指定值`-9`，则公式不成立。该变量`y`不影响公式是否成立。
