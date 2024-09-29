# 包（Packages）

[包属性描述](#包属性描述)

[MQl 源文件](#MQl源文件)

[实体的定义路径](#实体的定义路径)

[单文件运行](#单文件运行)

[MirrorQL 编译器对包的处理方式](#mirrorql编译器对包的处理方式)

[使用方法](#使用方法)

## 包属性描述

包的属性由其根目录的`package.toml`描述：

```
[package]
name = "hello" # 包名（必须）
version = "0.1.0" # 版本号（必须）
entry = "main" # 入口点文件，缺省为"main"，即src/main.mql，对应一个容器路径（定义下面有）
author = "foo" # 作者（可选）
language = "java" # 分析语言（可选，仅作标识）
is_lib = false # 是否为库，缺省为false。若为库，则没有入口点

[deps] # 依赖列表
std = "1.0.0" # semver
java = "1.0.0"
```

以`hello`为例， 包的典型目录结构如下：

```
hello/
├── package.toml
└── src
    └── main.mql
```

## MQl 源文件

`MirrorQL`程序的最小单位是`MirrorQL`源文件。

```
[output]
fn out(x:str){
    x="Hello world"
}
```

`MirrorQL`源文件中可以包括以下元素：

- `use`语句
- 谓词定义
- 类(`class`)定义
- 模块(`module`)定义
- 带参数模块实例化（`use module`）
- `trait`定义与`impl`体
- 对`MirrorQL`程序行为无影响的语法元素，如注释
- `MirrorQL`文件中定义的实体包括谓词、类、模块、带参模块、实例化的模块、`trait`。之所以叫实体，是因为这些元素都是独立的功能单位，且都有一个名字。
  实体的名字是唯一的，一个文件中不能出现两个名字相同的实体，例如同时定义`class A`和谓词`A`是非法的，同时定义两个`class A`也是非法的。

## 实体的定义路径

`MirrorQL`文件定义多个实体。由此，我们不妨将 MirrorQL 文件称为容器。一个容器定义（包含）多个实体，一个`MirrorQL`文件对应一个容器。

而`MirrorQL`程序的基本单位是包，一个用户程序可能使用多个包（程序本身是一个包，用到的静态分析库是其他包），每个包中由有很多`MirrorQL`文件。为了精确定位实体，我们需要引入实体的定义路径。

首先，一个实体对应一个唯一的容器（即一个`MirrorQL`文件），我们首先需要描述容器的路径。

幸运的是，容器对应一个文件，文件有文件系统中的路径，例如`<lib search path>/java/src/PTA.mql`，只需要规定文件系统路径的表示方式。

`MirrorQL`用一个字符串表示容器的路径，`java::PTA`对应两个可能的路径：`<lib search path>/java/src/PTA.mql`或`<lib search path>/java/src/PTA/mod.mql`。编译器优先选择第一个路径，若第一个路径不存在才会选择第二个路径。第二个路径仍不存在则容器定位失败。

容器路径+实体名 可以定位绝大多数实体，但模块(`module`)在容器内引入了额外的层次结构。

假如`<lib search path>/java/src/PTA.mql`的内容如下：

```
mod A{ // 容器内定义路径 A
    fn foo(){} // 容器内定义路径 A::foo
    mod B{ // 容器内定义路径 A::B
        fn foo(){} // 容器内定义路径 A::B::foo
    }
}
```

此时`java::PTA + foo`是无法准确表示实体的，因为我们无法确定需要对应第一个定义的`foo`还是第二个定义的`foo`。

因此，还需要引入容器内的定义路径来精确描述`foo`在哪被定义。于是，第一个`foo`在容器内的定义路径就是`A::foo（module A中定义，实体名foo）`，第二个`foo`在容器内的定义路径就是`A::B::foo`。

综上所述，一个实体的定义路径可用元组（所在容器路径，容器内定义路径）描述。

## 单文件运行

为了便于开发者快速编写简短的`MirrorQL`程序，`MirrorQL`编译器也提供了单文件运行的能力。

假设要运行的文件名为`test.mql`，编译器行为等价于执行以下操作后再运行`test`包：

创建`user_prog`包，

`package.toml`内容如下：

```
[package]
name = "user_prog"
version = "0.1.0"
entry = "test"
is_lib = false
```

目录结构如下：

`hello`包的典型目录结构如下：

```
user_prog/
├── package.toml
└── src
    └── test.mql
```

## MirrorQL 编译器对包的处理方式

只有带有`package.toml`中含`entry`属性的`MirrorQL`包可以被编译运行，`entry`是入口点文件的容器路径。库中可能有很多文件，但真正被用户程序用到的`MirrorQL`文件只占少数。`MirrorQL`编译器只会处理真正被用户程序用到的`MirrorQL`文件。

`MirrorQL`编译器运行包时，首先进行以下操作：

解析`entry`对应的文件，加入集合`S`（当前程序用到的`MirrorQL`文件集合），并加入队列`Q`（待处理的文件列表）

从`Q`中取出一个文件，解析其中所有的`use`语句，每个`use`语句引用一个定义路径，从定义路径中得到容器路径，加入集合`S`，若插入成功，再将路径加入队列`Q`
重复步骤 2，直到队列`Q`为空

此时，集合`S`就是用户程序用到的所有`MirrorQL`文件列表。`MirrorQL`编译器只会编译集合`S`内的所有文件。

## 使用方法

在`MirrorQL`中，可以使用`use`语句导入其他`MirrorQL`文件中定义的实体。`use`语句可以导入本包文件的实体，也可以跨包导入实体。

`use`语句的具体用法与行为参见 [use 用法](Modules模块.md#use语句)
