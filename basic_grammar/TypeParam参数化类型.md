# 参数化类型(TypeParam)

[参数化类型](#参数化类型)

[类型约束](#类型约束)

[参数化谓词](#参数化谓词)

&ensp;&ensp;[参数化谓词的声明](#参数化谓词的声明)

&ensp;&ensp;[参数化谓词的使用](#参数化谓词的使用)

[参数化类](#参数化类)

[参数化模块](#参数化模块)

&ensp;&ensp;[参数化模块的声明](#参数化模块的声明)

&ensp;&ensp;[参数化模块的实例化](#参数化模块的实例化)

## 参数化类型
`MirrorQL`允许将类型作为谓词、类、模块的参数使用，从而实现类似泛型的功能。

用示例说明参数化类型的用法。假设需要实现一个判断相等的谓词，一种可能的写法如下：

```
fn isEqu(x:int,y:int) {
    x==y
}
```

但`isEqu`谓词只能判断两个整数是否相等，无法判断两个字符串是否相等，若要判断，就要编写一个新的以`str`作为参数类型的谓词。

在`isEqu`谓词中，我们希望能处理所有类型相同的`x`和`y`，且不关心`x`和`y`具体是什么类型（只要两者类型相同即可）。

此时，可以考虑将`isEqu`的参数类型作为一个新的参数`T`。

要使用`isEqu`，需要给出`T`：`x y`两个参数的类型,`x:T`：参数1，`y:T`：参数2。

基于参数化类型的`isEqu`实现如下，可以处理任意类型的相等判断。

```
fn isEqu[T] (x:T,y:T) {
    x == y
}
```

`T`在代码中代表某一个未知的类型。

## 类型约束

再考虑一个新谓词`appendStr`，将参数1转换为字符串后再向结尾添加`"test"`并返回。类似的，可以用参数化类型写出一个实现（但这个实现是错误的）

```
fn appendStr[T] (x:T) -> str {
    // x.toString() 非法
    result == x.toString() + "test"
}
```

参数`x`的类型为`T`，编译器无法确定类型`T`是否有`toString`谓词，因此在类型检查时会报错。

此时需要对`T`增加类型约束，要求类型`T`必须实现`asStr` `trait`（也就是有`toString`成员谓词），新的写法为：

```
fn appendStr[T <: asStr] (x:T) -> str {
    result == x.toString() + "test"
}
```

若类型约束包含多个`trait`，可用加号分隔：

```
fn appendStr[T <: asStr+asInt] (x:T) -> str {
    // x.toString() 非法
    result == x.toString() + "test"
}
```

也可以指定多个类型参数：

```
fn appendStr[T <: asStr,T2 <: asStr] (x:T,y:T2) -> str {
    result == x.toString() + y.toString()
}
```

## 参数化谓词
### 参数化谓词的声明

要给谓词指定类型参数列表，只需要在谓词名后附加类型参数列表即可。例如谓词`appendStr`的类型参数列表为`[T <: asStr]`。

```
fn appendStr[T <: asStr] (x:T) -> str {
    result == x.toString() + "test"
}
```

### 参数化谓词的使用

使用参数化谓词时，可以显式给出类型参数：

```
fn appendStr[T <: asStr] (x:T) -> str {
    result == x.toString() + "test"
}

fn main(){
    // 给出类型参数
    appendStr[str] ("foo") == "footest"
}
```

编译器也可推导部分类型参数：

```
fn appendStr[T <: asStr] (x:T) -> str {
    result == x.toString() + "test"
}

fn main(){
    // 编译器推导 T = str
    appendStr("foo") == "footest"
}
```

## 参数化类

!!! warning

    参数化类在原版spec中并非标准实现，此处仅说明语法，不规定实现细节

与参数化谓词类似，参数化类的类型参数列表在1class1名后给出。使用参数化类时，在类名后给出类型参数列表。

```
class SomeClass [T <: asInt] extends int {
    fn init(){
        self == range(1,10)
    }
    fn get(x:T) -> int{
        result == self + x.toInt()
    }
}

fn main() {
    // 类名后给出类型参数列表
    let x:SomeClass[int];
    x.get(1) == 2
}
```

## 参数化模块

### 参数化模块的声明

在模块名后给出类型参数列表即可声明参数化模块：
```
mod SomeMod [T] {
    fn getSomeValue() -> T{
        let res: T;
        result == res
    }
}
```

### 参数化模块的实例化

要使用参数化模块，需要首先给出类型参数列表，称为实例化。与参数化谓词和参数化类不同，参数化模块不能直接使用，不支持参数化模块。

要使用参数化模块，需要用到`use module ... as ...`语法。

举例说明如何实例化`SomeMod`

```
class One extends int {
    fn init(){
        self == 1
    }
}

// 实例化SomeMod，实例化后的模块名为M
use SomeMod[One] as M;

fn main(){
    M::getSomeValue() == 1
}
```