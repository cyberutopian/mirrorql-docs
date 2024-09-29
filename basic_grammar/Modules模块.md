# 模块（Modules）

[模块（Modules）](#模块modules)

[定义模块(Defining a module)](#定义模块defining-a-module)

[模块主体(Module bodies)](#模块主体module-bodies)

&ensp;&ensp;[导入模块(Importing modules)](#导入模块importing-modules)

&ensp;&ensp;[use语句](#use语句)

&ensp;&ensp;&ensp;[示例](#示例)

&ensp;&ensp;&ensp;&ensp;[绝对路径](#绝对路径跨package使用)

&ensp;&ensp;&ensp;&ensp;[self方式](#self方式)

&ensp;&ensp;&ensp;&ensp;[super方式](#super方式)

&ensp;&ensp;&ensp;&ensp;[package方式](#package方式)

## 模块（Modules）

模块通过两相关类型，谓词和其他模块组合在一起，提供了一种组织`MirrorQL`代码的方法。

可以将模块倒入其他的文件，从而避免重复，并有助于将代码结构化为更易于管理的部分。

## 定义模块(Defining a module)

有多种方式来定义模块，这里是最简单的方法的例子，声明一个明确的模块名为`Example`含有类`OneTwoThree`：

```
mod Example {
   class OneTwoThree extends int {
    fn init() {
        self.(int) == 1
        or self.(int) == 2
        or self.(int) == 3
    }
  }
}
```

其中，模块的名称可以是任何以大写或者小写字母开头的标识符。

## 模块主体(Module bodies)

一个模块的主体是模块定义内的代码。通常，模块的主体可以包含以下的构造：

* [导入语句](#导入模块importing-modules)
* [谓词](./Predicates谓词.md)
* [类型](./BasicDataTypes基本数据类型.md)(包括用户自定义的[类](./Class类.md))

## 导入模块(Importing modules)

将代码存储在模块中的主要好处是可以在其他模块中重用它。要访问外部模块的内容，可以使用[use 语句](#use语句)导入模块。

导入模块时，这会将其名称空间内的所有名称都带入当前模块的名称空间内。

## use语句

导入语句用于导入模块。语法规范如下：

`use`语句的语法为`use 模块路径::实体名`，表示将对应模块的实体导入到本模块中。

模块路径有四种形式：

* 绝对路径，例如`java::PTA`，表示`java`包的`PTA`模块。
* `self`，表示当前模块，常用于导出`MirrorQL`文件的实体。
* `super`，表示上一级模块。
* `package`，表示当前包。例如`java`包文件中的`package::PTA`与`java::PTA`等价。
实体名是需要导入的模块、谓词、类型、`trait`的名称。也可以是通配符`*`，表示导入对应模块下的所有具名实体。

### 示例

#### 绝对路径（跨package使用）
该方法主要是为了跨`package`使用，例如：`java::PTA`，其中`java` 是`package.toml`文件中 `deps` 中的库名，`PTA`表示`java` 库中`src` 目录下的文件，各级目录用`::`分割，且最后一项是所引用文件中的`模块、谓词、类型、trait的名称`。

#### 注意：
* 使用该方式时，需要用`-L`来指定`java`库所在的目录。
* 使用该方式时，项目根目录下必须要有`package.toml`文件, 并且使用的库名必须在`deps` 列表中。

示例：
以  `example/dep_example`中的`test`为例, 该目录的结构如下：
```
test
├── package.toml
└── src
    └── test.mql
```
依赖的库`java`库目录结构如下：
```
java
├── package.toml
└── src
    ├── Other.mql
    └── edb
        ├── JavaDocComment.mql
        └── Literal.mql
```
执行如下的命令编译：`java -jar ../../compiler.jar -L ../ -c`, 其中依赖库`java` 存放在`../`路径下。编译后的目录结构：
```
.
├── main.mql.dl
├── package.toml
└── src
    └── main.mql
```
其中，命令行不指定`.mql`文件时，默认以`package.toml`中指定的文件执行。

#### self方式
该方式表示指定的路径与正在执行的`mql`文件在同一个目录下。以`example/self_example` 为例：
该示例的目录结构如下：
```
self_example
├── package.toml
└── src
    ├── Bar.mql
    └── main.mql
```

执行如下的命令编译：`java -jar ../../compiler.jar  -c`。编译后的目录结构：
```
.
├── main.mql.dl
├── package.toml
└── src
    ├── Bar.mql
    └── main.mql
```

#### super方式
该方式表示指定的路径与正在执行的`mql`文件的上一级目录下。以`example/super_example` 为例：

该示例的目录结构：
```
.
├── Bar.mql
├── package.toml
└── src
    └── main.mql
```
执行如下的命令编译：`java -jar ../../compiler.jar  -c`。编译后的目录结构：
```
.
├── Bar.mql
├── main.mql.dl
├── package.toml
└── src
    └── main.mql

```

#### package方式

该方式表示引用的符号还是在本 `package`内，其中`package`表示该项目的根目录，也即`package.toml` 所在的目录，以`example/package_example` 为例
该示例的目录结构如下：
```
package_example
├── package.toml
└── src
    ├── Bar.mql
    ├── java
    │   ├── Other.mql
    │   └── edb
    │       ├── JavaDocComment.mql
    │       └── Literal.mql
    └── main.mql
```
执行如下的命令编译：`java -jar ../../compiler.jar  -c`。编译后的目录结构：
```
.
├── main.mql.dl
├── package.toml
└── src
    ├── Bar.mql
    ├── java
    │   ├── Other.mql
    │   └── edb
    │       ├── JavaDocComment.mql
    │       └── Literal.mql
    └── main.mql
```