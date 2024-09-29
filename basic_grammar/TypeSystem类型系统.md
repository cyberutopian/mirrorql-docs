# TypeSystem 类型系统

[类型](#类型)

[类型兼容性](#类型兼容性)

[类型转换](#类型转换)

&ensp;&ensp;[在不兼容的类型间转换](#在不兼容的类型间转换)

## 类型

`MirrorQL`的类型包括[基本类型](BasicDataTypes基本数据类型.md#基本数据类型)、[`class`类型](Class类.md#类class)、`record class`类型(待定)。`MirrorQL`中，类型定义为值的集合。某个值如果同时为多个集合的成员，那么此值可能有多个类型。

例如，在定义"奇数"类型的情况下，1的类型既可以是`int`，也可以是"奇数"。

每个表达式都有一个定义类型`T`，表示表达式求值的值集中每个值的类型一定可以是`T`。

表达式`range(1,4)`的定义类型为`int`，其值集`{1,2,3}`的每个值的类型一定可以是`int`。

表达式`range(1,4).(Odd)`的定义类型为`Odd`，其值集`{1,3}`的每个值的类型一定可以是`Odd`，这不妨碍其中每个值也可以是`int`。

## 类型兼容性

在实际编程中，会遇到判断两个值是否相等的情况，`1==1`是显然成立的，但`"1"==1`难以判断是否成立。

根本问题是，等式左边为`str`类型，等式右边为`int`类型，`str`类型和`int`类型的值域不同。

实际上，`str`和`int`类型不兼容，`MirrorQL`不允许不兼容的类型之间做比较。

`MirrorQL`的类型要么是内建的（基本类型），要么是直接创建的`（record class）`，要么是在已有类型中做子集操作，生成的新类型（`class类型`）。

类型的兼容是等价关系。类型兼容的直接条件是类型`A`继承类型`B`（非具名继承），那么`A`与`B`兼容。

用例子说明：

```
record class RA { ... }
record class RB { ... }
class A extends int { ... }
class B extends str { ... }
class C extends A,int { ... }
class D extends RA { ... }

// 类型兼容的等价类为： {RA,D} , {RB} , {int,A,C} , {str,B}
```

## 类型转换

类型转换用来修改表达式的定义类型，不改变表达式产生的值本身。

对于定义类型为`T`，取值集合为`S`的原表达式`expr`，类型转换后的表达式`expr.(T1)`的定义类型为`T1`，取值集合为`S'`。

类型转换的过程可以理解为，遍历`S`中的每一个元素（元素定义类型为`T`），若元素的类型还可以是`T1`，则将元素加入`S'`中。最终`S'`中的每一个元素的定义类型均为`T1`。

原定义类型和新定义类型必须兼容，否则类型转换是非法的。


### 在不兼容的类型间转换

在实际编程中，存在`int`和`str`之间相互转换的需求。

但`int`类型和`str`类型不兼容，无法直接通过类型转换表达式进行转换。

MirrorQL为基础类型间相互转换的需求内建了[trait](Trait类型.md#trait) 。

---
| trait名 | 实现的方法         | 内建impl的类型     | 说明                                                                   |
| ------- | ------------------ | ------------------ | ---------------------------------------------------------------------- |
| asInt   | toInt() -> int     | int,str,bool,float | 转换为带符号32位整数类型，浮点数向0取整，bool的True和False分别对应1、0 |
| asStr   | toString() -> str  | int,str,bool,float | 转换为字符串                                                           |
| asFloat | toFloat() -> float | int,str,bool,float | 换为带符号32位IEEE754浮点数，bool的True和False分别对应1、0             |

用示例说明使用这些内建trait的方法：

```
fn getRangeStr() -> str{
    let x=range(1,10); // x为int
    result == x.toString() // x转换为string
}
fn getInt() -> int{
    result == "1".toInt()
}

fn getFloat() -> float{
    result == 1.0 + 1.toFloat()
}
```
此外，MirrorQL还提供了内建的`toString`，`toInt`，`toFloat`谓词，分别对应trait `asString`，`asInt`，`asFloat`。

```
toString(1) == "1"
toInt("1") == 1
```

!!! note "类型转换对新类型的要求"

    类型转换要求原表达式的定义类型和新类型兼容。`1.(str)`和`"1".(int)`都是非法的。
    
    类型转换不改变表达式产生的值本身，只可能缩小表达式产生的值集合。
    
    如`let t:str; t in ["a","system"]`中，表达式`t`的值集合为`{"a","system"}`，而类型转换后的`t.(SomeString)`值集合缩小为`{"system"}`.
    
    如需实现整数、字符串、浮点数之间的互转，需要使用内建的类型转换函数，如`toString(1)`、`toInt("1")`，并不属于表达式中类型转换的范畴。
    
    原定义类型和新定义类型必须兼容，否则类型转换是非法的。