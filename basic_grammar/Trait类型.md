# Trait

[类型Trait](#类型Trait)

&ensp;&ensp;[trait的声明](#trait的声明)

&ensp;&ensp;[内建trait](#内建trait)

&ensp;&ensp;[实现trait](#实现trait)

&ensp;&ensp;[使用trait](#使用trait)


## 类型Trait

`trait`描述了一类类型的特征，要求类型必须实现指定的成员谓词，从而允许实现类似接口类和泛型的功能。

### trait的声明

使用`trait`关键字声明一个`trait`。下面的代码声明了一个名为`HasFooBar`的`trait`，要求类型必须实现成员谓词`fn foo() -> str`和`fn bar(int)`

```
trait HasFoo {
    fn foo() -> str;
    fn bar(int);
}
```

### 使用trait约束基类

`trait`还支持约束类型的基类。注记`basetype`约束类型的基类。`trait` `ExtendsInt`要求类型必须直接或间接地`extends int`类型，且不是具名继承。

```
[basetype(int)]
trait ExtendsInt{

}
```

### 内建trait

`MirrorQL`中有部分内建`trait`，用于[在不兼容的类型之间转换](TypeSystem类型系统.md#在不兼容的类型间转换)
```
// int,str,bool,float实现
trait asInt{
    fn toInt() -> int;
}

// int,str,bool,float实现
trait asStr{
    fn toString() -> str;
}

// int,str,bool,float实现
trait asFloat{
    fn toFloat() -> float;
}
```

### 实现trait

使用`impl`语法实现`trait`。

```
trait TraitFoo{
    fn foo() -> str;
}

class One extends int {
    fn init(){
        self == 1
    }
}

// 对于同一个`class`和同一个`trait`，只能有一个`impl`体
impl TraitFoo for One {
    fn foo() -> str {
        result == "Foo:" + self.toString()
    }
}

```

若`class`本身已满足`trait`的要求，可以使用空`impl`体实现`trait`

```
trait TraitFoo{
    fn foo() -> str;
}

class One extends int {
    fn init(){
        self == 1
    }
    fn foo() -> str {
        result == "Foo:" + self.toString()
    }
}

// 对于同一个class和同一个trait，只能有一个impl体
impl TraitFoo for One;
```

!!! note 

    只有显式impl后，才认为某个类型实现了trait。

### 使用trait

`trait impl`体中定义的谓词会被添加到对应`class`的成员谓词列表中，因此`impl`体中定义的谓词也可作为成员谓词使用。

```
trait TraitFoo{
    fn foo() -> str;
}

class One extends int {
    fn init(){
        self == 1
    }
}

// 对于同一个class和同一个trait，只能有一个impl体
impl TraitFoo for One {
    fn foo() -> str {
        result == "Foo:" + self.toString()
    }
}

fn main() {
    let one:One;
    one.foo() == "Foo:1"
}
```

此外，`trait`还被用作[参数化函数、类、谓词](TypeParam参数化类型.md#参数化类型)中的[类型约束](TypeParam参数化类型.md#类型约束)。