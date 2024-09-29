# 类（Class）

[类的定义](#类的定义)

[定义一个类](#定义一个类)

&ensp;&ensp;[类语法](#类语法)

&ensp;&ensp;[类要素](#类要素)

&ensp;&ensp;[类体（Class bodies）](#类体class-bodies)

&ensp;&ensp;[init谓词（Init predicates）](#init谓词init-predicates)

&ensp;&ensp;[成员谓词（Member predicates）](#成员谓词member-predicates)

&ensp;&ensp;[字段（Field）](#字段field)

&ensp;&ensp;[重写成员谓词（Overriding member predicates）](#重写成员谓词overriding-member-predicates)

&ensp;&ensp;[继承](#继承)

&ensp;&ensp;&ensp;&ensp;[多重继承](#多重继承)

&ensp;&ensp;&ensp;&ensp;[具名继承](#具名继承)


## 类的定义

除了基础的数据类型，你也可以在MirrorQL中定义自己的类型。一种实现方法是定义一个类`（Class）`，类提供了一种简单的方法来重用和构造代码。
例如：

1. 将相关联的值分组在一起
2. 根据这些值定义成员谓词
3. 定义覆盖成员谓词的子类

MirrorQL中的类不会“创建”新对象，而只是表示逻辑属性。如果值满足该逻辑属性，则该值属于特定类。


## 定义一个类

### 类语法

典型的`class`语法定义如下：
 - `class`的名称 - `class`的基类列表（即这个`class`基于哪些类型的交集创建） - `class`的`init`谓词（即筛选这个`class`的逻辑式） - `class`的成员谓词（可选） - `class`的成员变量（可选）

`class`的典型语法如下：

```
class 类名 extends 基类列表,... {
    let member_val1:int; 
    let member_val2:str; // 成员变量，可选

    fn init() {
        ... // class的init谓词
    }

    fn member1() -> int{
        result == self.member_val1 // self代表class本身
    }

    fn member2(x:str) {
        x == self.member_val2
    } // 成员谓词
}
```

### 类要素

要定义一个类，需要遵循以下的几条规则：

* 关键字`class`
* 类的名称，这是一个以字母开头的标识符
* 要扩展的类型
* 类的主体，用大括号括起来

例如：

```
class OneTwoThree extends int {
    fn init() {
        self.(int) == 1
        or self.(int) == 2
        or self.(int) == 3
    }
    
    fn getAString() -> str {
        result == "one, two or  three: " 
    }
}


```

这里定义一个类`OneTwoThree`继承自基本的数据类型`int`，特征谓词是约束变量需要满足值为`1、2、3`中的一个，`MirrorQL`并不会创建一个实例对象，而是通过逻辑条件约束变量所满足的条件。

`MirrorQL`中的类必须始终继承至少一个现有的类，类的值包含在基本类型`（int、float、bool、str）`的交集内。一个类从其基本类型继承所有的成员谓词。

一个类可以继承自多种类型，如果需要更多的信息，查看[多重继承](#多重继承) 章节。

一个有效的类必须包含以下的条件：

* 不能继承自这个类本身
  
* 不能继承不兼容的类型，更多相关信息查看[类型兼容](TypeSystem类型系统.md#类型兼容性)章节

### 类体（Class bodies）

类的主体（大括号括起的部分）可以包含：

* 一个特征谓词的声明
* 任意数量的成员谓词声明
* 任意数量的字段声明

当定义一个类时，该类还从其超类型继承所有非私有成员谓词和字段。您可以重写
这些谓词和字段以为其提供更具体的定义。

### init谓词（Init predicates）

`class`是从基类的交集中创建的，相当于取子集。`init`谓词的作用是指定取子集的逻辑式。

用例子说明`init`谓词的用法。

假如: 要定义一个`class SomeNumber`，要求每个元素满足特性：元素的值为两个在区间`[2,4]`内的数的乘积。

首先，`class SomeNumber`的基类型为`int`，即从`int`集合中筛选`SomeNumber`。再考虑`init`谓词的写法。

假如只需要实现一个判断是否为`SomeNumber`的谓词，可以用下面的代码：

```
fn is_SomeNumber(val:int) {
    let x=range(2,5);
    let y=range(2,5);
    val == x*y
}
```

`init`谓词和`is_SomeNumber`表达的逻辑相同：

```
class SomeNumber extends int {
    fn init(){
        let x=range(2,5);
        let y=range(2,5);
        self == x*y
    }
}
```
!!! info "缺省init谓词"

    若不定义init谓词，则为缺省情况：
    ```
    fn init(){
        any()
    }
    ```

!!! note "init谓词中self表达式的定义类型"

    在init谓词中，MirrorQL编译器认为self表达式的定义类型为新class(尽管此时新class没有被定义完全)。在实际编程中，应避免在init谓词中使用self的成员谓词，或将self作为新class类型传递给外部谓词。

    ```
    fn consume(x:Foo){
        ...
    }
    class Foo extends int{
        let v:int;
        fn init(){
            self.bar() // 不建议使用
            consume(self) // 不建议使用
            self==1 // 可用
            self.v==1  // 可用    
        }
        fn bar(){

        }
    }
    ```

### 成员谓词（Member predicates）

这些谓词仅适用于特定类别的成员。您可以根据值调用成员谓词。例如，您可以使用上述类中的成员谓词

```
1.(OneTwoThree).getAString()
```

此调用返回结果。`"One, two or three:"`


### 字段（Field）

这些是在类主体中声明的变量。一个类在其主体内可以有具体任意数量的字符声明（即变量声明）。

可以在类内部的谓词声明中使用这些变量。就像变量`this`一样，字段必须限制在特征谓词中。

### 重写成员谓词（Overriding member predicates）

如果类从超类型继承成员谓词，则可以覆盖继承的定义。

为此，您可以定义一个成员谓词，该成员谓词的名称和别名与继承的谓词相同。

如果要优化谓词以为子类中的值提供更具体的结果，这将很有用。

```
class OneTwoThree extends int {
    fn init() {
        self.(int) == 1
        or self.(int) == 2
        or self.(int) == 3
    }
}

class OneTwo extends OneTwoThree {
    fn init() {
        self.(int) == 1
        or self.(int) == 2
    }
}
```

### class 继承

定义一个`class`时，需要在`extends`关键词后给出一个基类列表。`class` `SomeNumber`的基类列表为`int`，则称`SomeNumber`继承`int`。

之所以称为"继承"，是因为新`class`会继承基类中所有的成员谓词（注意，成员变量没有被继承）。

此外，类型系统中也增加了"新class是基类的子类"这一类型信息。

子类可以重新定义从基类中继承而来的成员谓词，下面的例子说明了重定义基类成员谓词的用法：

```
class Pet extends str{
    fn init(){
        self == "pet"
    }
    fn getName() -> str{
        result == "some pet"
    }
}
class Dog extends Pet{
    // init谓词缺省为any()

    fn getName() -> str{ // 重写从Pet中继承的成员谓词
        result == "dog"
    }
}

fn test(){
    let x:Pet;
    x.getName() == "some pet" and x.(Dog).getName() == "dog" and test2(x.(Dog).(Pet))
}

fn test2(x:Pet){
    x.getName() == "some pet" // 成员谓词的解析只和表达式的定义类型有关
}
```

#### 多重继承

一个`class`可以同时从多个基类中继承，相当于从多个类中取交集。

例如`extends SmallInt,SomeNumber`表示，为`init`谓词选定的初始集合为`SmallInt`和`SomeNumber`的交集。

和单继承类似，新`class`将继承每个基类的成员谓词，但如果某些基类存在名称相同但定义不同的成员谓词，则继承时会出错，需要手动重写谓词，或使用具名继承来避免歧义。

下面的例子展示了多继承的用法：
```
class SmallInt extends int{
    fn init(){
        ...
    }
    fn getName() -> str{
        result == "small int"
    }
}

class SomeNumber extends int{
    fn init(){
        ...
    }
    fn getName() -> str{
        result == "some number"
    }
}

class NewClass extends SmallInt,SomeNumber {
    fn init(){
        ...
    }
    fn getName() -> str{ // 此处需要重新定义getName，否则会存在getName的二义性
        result == "new class"
    }
}
```

!!! note "名称相同但相同的成员谓词"

    在复杂的继承关系中，可能存在菱形继承的情况，此时在某个新class中可能存在名称相同、定义相同，但从两个不同基类继承而来的成员谓词。此时我们认为不存在二义性，不必重新定义成员谓词。

    下面的代码演示了菱形继承的情况：
    ```
    class A extends int{
        fn foo() {
            ...
        }
    }
    class B extends A{
        // 继承foo
    }
    class C extends A{
        // 继承foo
    }
    class D extends B,C{
        // 可以不重定义foo，B和C中的foo对应同一谓词，不存在歧义
    }
    ```

!!! note "class继承和多继承的要求"

    class继承如果存在环，则是非法的，例如：`A extends B; B extends A`非法。此外，某个class的基类列表中的各个类型必须兼容，`extends str,int`是非法的。

#### class具名继承

在多继承时，可能遇到继承的成员谓词存在二义性的情况。除了重新定义成员谓词以外，还可以使用具名继承解决问题。

具名继承与继承有两点差别：

1. 为继承的基类指定了名称，因此基类的成员谓词不会直接导入到此`class`中

2. 在类型系统中，不认为新`class`是具名继承基类的子类

要指定具名继承，只需要给`extends`列表的某一项加上名称即可。

下面的代码演示了具名继承的用法：
```
class SmallInt extends int{
    fn init(){
        ...
    }
    fn getName() -> str{
        result == "small int"
    }
}

class SomeNumber extends int{
    fn init(){
        ...
    }
    fn getName() -> str{
        result == "some number"
    }
}

class NewClass extends SmallInt,SomeNumber sn{
    fn init(){
        ...
    }
    // 此处会继承SmallInt的getName，无二义性

    fn getNameInSomeNumber() -> str{
        result == sn.getName() // 使用具名继承的成员谓词
    }

    fn asSomeNumber() -> SomeNumber{
        result == sn // sn相当于类型为SomeNumber的表达式，值与self相同
    }

    // self.(SomeNumber)非法，因为NewClass与SomeNumber不兼容
}
```

!!! note "具名继承的特殊情况：单具名继承"

    在某些情况下，需要使用具名的单继承。例如在定义数据库中的某个类型时，self应当对应行的主键ID，即一个`int`类型，但我们不认为此类型和`int`类型存在关系，此时需要具名继承`int`类型。

    示例代码如下：

    ```
    class TableRow extends int id{
        let col1:str;
        let col2:int;
        fn init(){
            table(id,col1,col2)
        }
        fn getId() -> int{
            result == id
        }
        fn getCol1() -> str{
            result==self.col1
        }
    }
    
    let x:TableRow; // x与int不兼容，x.(int)非法。
    ```
