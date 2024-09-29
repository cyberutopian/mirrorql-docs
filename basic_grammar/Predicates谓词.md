# 谓词(Predicates)

[命题与谓词的概念](#命题与谓词的概念)

[谓词(Predicates)](#谓词predicates)

[定义谓词(Defining a predicate)](#定义谓词defining-a-predicate)

&ensp;&ensp;[有结果的谓词(Predicates with result)](#有结果的谓词predicates-with-result)

&ensp;&ensp;[递归谓词（Recursive predicates）](#递归谓词recursive-predicates)

&ensp;&ensp;[谓词注记(Predicate annotation)](#谓词注记predicate-annotation)

&ensp;&ensp;&ensp;&ensp;[输入谓词](#输入谓词)

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;[从数据库中输入](#从数据库中输入)

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;[从其他 datalog 程序中输入](#从其他datalog程序中输入)

&ensp;&ensp;&ensp;&ensp;[输出谓词](#输出谓词)

&ensp;&ensp;[绑定行为（Binding behavior）](#绑定行为binding-behavior)

&ensp;&ensp;&ensp;&ensp;[绑定(bingding)](#绑定binding)

&ensp;&ensp;[MirrorQL 内建谓词(MirrorQL built-in predicates)](#mirrorql-内建谓词mirrorql-built-in-predicates)

## 命题与谓词的概念

命题是一个陈述表达的判断，可以判断真假。

例如，`”main.java文件有50行“`是一个命题，这句话是对于`main.java`文件性质的判断，可以判定真假。

而`"main.java文件是否被引用"`不是一个命题，因为这句话没有表达判断的意思。

`“main.java被引用”`和`"main.java没有被引用"`都是命题。命题表示一个固有的，可以判断真假的逻辑概念。

`MirrorQL`的谓词则为命题引入了变元。`“P(x):文件x有50行”`和`“Q(x,y): 文件x有y行”`定义了两个谓词`P`和`Q`。

谓词本身没有真假的概念，也不能独立构成命题。`"文件x有50行"`并不是一个命题，因为`x`带入一个特定的文件后才能判断这句话的真假。

谓词作用于常量后才能判断真假，并构成一个命题。`P("main.java")`是一个命题，若命题为真，则`main.java`文件有`50`行。

`Q("main.java", 50)`也是一个命题，若命题为真，则`main.java`文件有`50`行。

`MirrorQL`用关系表示谓词。谓词`P`对应一个一元关系，谓词`Q`对应一个二元关系。关系可以被描述为元组的集合。假设欲查询的数据库中存在三个文件，文件名合格行数定义如下：

| 文件名    | 行数 |
| :-------- | :--: |
| Main.java |  50  |
| Foo.java  |  20  |
| Bar.java  |  30  |

那么，谓词`P`对应关系`P`为`${('Main.java')}$`,谓词`Q`对应关系`Q`为`${('Main.java',50),('Foo.java',20),('Bar.java',30)}$`。

`MirrorQL`规定关系对应的集合必须是有限的。谓词`"R(x): 整数x大于1"`不是一个合法的谓词，因为关系`R`的集合是无限的（大于 1 的整数有无穷多个）。

谓词`"S(x): 整数x大于1，且x在范围[0,100]"`是一个合法的谓词。

特殊地，`MirrorQL`中也有 0 元谓词，例如`"T() : Main.java文件有50行"`，对应的关系`T`为`${()}$`，即存在一个空元组的集合。

总不成立的 0 元谓词对应的关系为一个空集合。

为了便于编程，`MirrorQL`还引入了带返回值的谓词。`"P(x) -> int : 返回值为x"`表示一个返回整数的一元谓词。

将谓词`P`施加到一些值上将得到另一些值（而非命题），`P(1)`得到一个值`1`，`P(P(2))`得到一个值`2`。

带返回值的谓词同样有一个对应的有限关系集合，作用于常量后同样可以判断真假。带返回值的谓词可以看作是普通谓词的语法糖。

谓词`p`可以等价地写为谓词`"Q(r,x) : x==r"`，`P(1)`等价于`Q(表达式值,1)`，`P(P(2))`等价于`Q(表达式值,tmp) and Q(tmp,2)`。

## 谓词(Predicates)

谓词用于描述组成`MirrorQL`程序的逻辑关系。

严格来说，谓词对一组元组求值。例如，考虑以下的两个谓词定义：

```
fn isCountry(country: str) {
  country == "Germany"
  or
  country == "Belgium"
  or
  country == "France"
}

fn hasCapital(country: str, capital: str) {
  country == "Belgium" and capital == "Brussels"
  or
  country == "Germany" and capital == "Berlin"
  or
  country == "France" and capital == "Paris"
}
```

谓词`isCountry`是一元组的集合`{("Belgium"),("Germany"),("France")}`，

而谓词是二元组`hasCapital`的集合`{("Belgium","Brussels"),("Germany","Berlin"),("France","Paris")}`。

通常，谓词中的所有元组都具有相同数量的元素。谓词的多样性是不包含可能的`result`变量的元素数量。有关更多信息，请参⻅[有结果的谓词](Predicates谓词.md#有结果的谓词predicates-with-result)。

`MirrorQL`中有许多[内置谓词](#mirrorql-内建谓词mirrorql-built-in-predicates)。您可以在任何查询中使用它们，而无需导入 任何其他模块。除了这些内置谓词，还可以定义自己的谓词。

## 定义谓词(Defining a predicate)

定义谓词时，需要指定：

- 关键字 `fn`
- 谓词的名称。这是一个以字母开头的标识符
- 谓词的参数（如果有）以`,`分割。对于每一个参数，指定参数变量的标识符和参数类型
- 如果谓词有返回值，则在参数列表尾部增加->返回值类型，返回值在逻辑式内通过`result`表示
- 谓词体，用大括号括起来的逻辑式

示例：

```
fn people_age(name:str,age:int){
 name=="Alice" and age==20
 or
 name=="Bob" and age==25
}

fn get_age(name:str) -> int{
 name=="Alice" and result==20 // result表示返回值
 or
 name=="Bob" and result==25
}
```

## 有结果的谓词(Predicates with result)

带返回值的谓词也是一种谓词，但是与普通谓词存在行为上的差异，并引入一个特殊的变量`result`。
例如：

```
fn getSuccessor(i: int) -> int {
  result == i + 1 and i in range(1,9)
}
```

如果`i`是小于`10`的正整数，则谓词的结果为后继的`i`。

带返回值的谓词也和传统意义的函数不同。带返回值的谓词可能返回零或多个值。示例：

```
fn get_age(name:str) -> int{
 name=="Alice" and result==20 // result表示返回值
 or
 name=="David" and result==20
}

fn age_to_name(age:int) -> str{
 result=="Alice" and age==20 // result表示返回值
 or
 result=="David" and age==20
}
```

`get_age("Alice")`得到值集`{20}`

`get_age("Bob")`得到空集（没有返回值）

`age_to_name(20)`得到值集`{"Alice","David"}`（多个返回值）

## 递归谓词（Recursive predicates）

谓词可以递归定义，此时谓词可能对应多个集合。下面的例子定义了两个递归谓词：

```
fn P(){
 Q()
}
fn Q(){
 P()
}
```

此时，谓词`P`对应`{()}`（即成立）、谓词`Q`对应`{()}`（即成立） 和 谓词`P`对应`{}`（即不成立）、谓词`Q`对应`{}` （即不成立） 都符合逻辑规则。

但`MirrorQL`编译器会求得最小的不动点解`least fixed point）`。对于上面的程序，`MirrorQL`编译器应当给出结果：`P对应{}，Q对应{}`

!!! info "不合法的递归情况"

    MirrorQL不允许声明带有逻辑矛盾的谓词。下面的程序是非法的。
    ```
    fn P() {
    	not P()
    }

    ```

## 谓词注记(Predicate annotation)

为了拓展谓词的功能，`MirrorQL`支持给谓词增加注记。注记内容需要用`[]`包裹，写在谓词定义的前一行。

```
[inline] // inline注记
fn inline_me() -> int{
 result == 1
}
```

`MirrorQL`支持的注记列表如下：

| 注记名   |           注记用法            |                                    备注                                     |
| :------- | :---------------------------: | :-------------------------------------------------------------------------: |
| eqrel    |    谓词为一个二元等价关系     |                 谓词不能有返回值，必须有两个类型相同的参数                  |
| inline   |       谓词规则需要内联        |                                                                             |
| input    |    谓词内容从数据库中输入     | 可选参数 id="数据表名"，缺省与谓词名相同，详情参见[输入谓词](#输入谓词)一节 |
| output   |       谓词内容需要输出        |                      详情参见[输出谓词](#输出谓词)一节                      |
| external | 谓词在外部 datalog 程序中实现 |      详情参见[从其他 datalog 程序中输入](#从其他datalog程序中输入)一节      |

!!! info "注记使用说明"

    对于同一个谓词，以上五个注记只能同时出现一个。

### 输入谓词

`MirrorQL`是一个面向数据库的查询语言，通过输入谓词从数据库中读取数据，通过输出谓词呈现查询的结果。

#### 从数据库中输入

谓词注记`input`定义一个从数据库输入的谓词。通过注记设置需要读取的数据表名。

下面的示例从`People`表中读取数据。`People`表共两列，第一列为字符串，第二列为整型。

```
[input,id="People"]
fn people_age(name:str,age:int); // 分号结尾表示谓词体为空
```

输入谓词的每一个参数依次对应数据库的每一列。因此，参数的个数和类型必须和数据表结构相应。

此外，输入谓词不存在返回值，也没有逻辑式，定义返回值类型或者谓词体是非法的。

```
[input,id="People"]
fn people_age(name:str,age:int) -> int; // 非法定义：输入谓词不能有返回值

[input,id="People"]
fn people_age(name:str,age:int){
 any()
} // 非法定义：不能有谓词体
```

#### 从其他 datalog 程序中输入

谓词标注`external`定义一个外部`datalog`程序实现的谓词。

编译器首先将`MirrorQL` 程序编译为`datalog`程序，`external`运行`MirrorQL`程序使用外部的`datalog`程序定义的谓词，而无需在`MirrorQL`程序中定义谓词的逻辑。

同样，`external`谓词不存在为不能定义谓词体。但是`external`谓词支持返回值，返回值被翻译为`datalog`谓词的第一个参数。

```
[external]
fn foreign_pred(x:int) -> str;

fn main(){
 foreign_pred(1) == "a" // 允许MirrorQL程序直接使用external谓词
}
// 对应datalog的.decl foreign_pred(result:str,x:int)
```

### 输出谓词

谓词注记`output`定义`MirrorQL`程序输出的查询结果。谓词的每一个参数对应输出表格的每一列。

下面的示例程序输出一个两列的表格，第一列卫姓名，第二列为年龄。

```
fn get_age(name:str) -> int{
 name=="Alice" and result==20 // result表示返回值
 or
 name=="David" and result==20
}

[output]
fn query_output(name:str,age:int){
 age=get_age(name)
}
```

需要注意的是，输出谓词不存在返回值。若指定返回值，输出谓词是非法的。

此外，输出谓词的每一个参数必须能转换为字符串（即必须实现**`trait Print`**），否则将无法编译。

```
[output]
fn bad_query(name:str) -> int { // 非法：存在返回值
 ...
}

[output]
fn bad_query2(arg:unit) { // 非法：unit类型未实现trait Print
 ...
}
```

## 绑定行为（Binding behavior）

必须有可能在有限的时间内评估谓词，因此通常不允许它描述的集合是无限的。换句话说，谓词只能包含有限数量的元组。

当 MirrorQL 编译器可以证明谓词包含的变量不限于有限数量的值时，将报告错误。有更多信息参考[绑定](#绑定binding)

以下时无限谓词的一些示例：

```
/*
   Compilation errors:
   "i" is not bound to a value.
   "result" is not bound to a value.
   expression "i * 4" is not bound to a value.
 */
 fn multiplyBy4(i: int) -> int {
   result == i * 4
 }

 /*
   Compilation errors:
   "str" is not bound to a value.
   expression "str.length()" is not bound to a value.
 */
 fn shortString( data: str) {
   data.len() < 10
 }

```

在`multiplyBy4`接口中，参数`i`声明为`int`，这是一个无限的类型。它同在二元操作`*`中，该操作不绑定其他的操作数。`result`首先是未绑定的，并且由于与相等的检查中使用了它，因此也保持未绑定的状态

在`shortString`接口中，`data`由于使用无限类型声明，因此保持未绑定的状态`str`，并且内置函数`len()`不会绑定它。

### 绑定（Binding）

若要避免查询中的无限关系，必须确保没有未绑定的变量。为此，您可以使用以下机制：

1. 有限类型：有限类型的变量是绑定的。特别是，任何非原始类型都是有限的。若要为变量提供有限类型，可以使用有限类型声明该变量、使用强制转换或使用类型检查。

2. 谓词调用：有效的谓词通常是范围限制的，因此它绑定了所有参数。因此，如果对变量调用谓词，则该变量将受到约束。

3. 绑定运算符：大多数运算符（如算术运算符）都要求绑定其所有操作数。例如，您不能在`MirrorQL`中添加两个变量，除非您对这两个变量都有一组有限的可能值。

但是，有一些内置运算符可以绑定其参数。例如，如果相等性检查的一侧（使用`==`）是绑定的，而另一侧是变量，则该变量也会绑定。有关示例，请参阅下表。

直观地说，绑定事件将变量限制为一组有限的值，而非绑定事件则不会。以下是一些示例，用于阐明变量的绑定和非绑定实例之间的区别：

|       绑定行为       |                                                                              说明                                                                               |
| :------------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|        x == 1        |                                                                      绑定：将 x 限制为值 1                                                                      |
|   x != 1, not x==1   |                                                                             非绑定                                                                              |
| x == 2+3, x + 1 == 3 |                                                                              绑定                                                                               |
|   x in range(0,3)    |                                                                              绑定                                                                               |
|       p(x,\_)        |                                                                绑定，因为 p（） 是对谓词的调用。                                                                |
|   x = y, x = y + 1   |                                             当且仅当变量 y 被绑定时，x 的绑定。当且仅当变量 x 被绑定时，y 的绑定。                                              |
|     x == y \* 2      |                                                         如果绑定了变量 y，则绑定 x。对 y 不具有约束力。                                                         |
|        x > y         |                                                                         不绑定 x 或 y。                                                                         |
| "string".matches(x)  |                                                                           不绑定 x。                                                                            |
|     x.matches(y)     |                                                                         不绑定 x 或 y。                                                                         |
|   x == 1 or y == 1   | 不绑定 x 或 y。第一个子表达式 x = 1 绑定 x，第二个子表达式 y = 1 绑定 y。但是，将它们与分离相结合仅对所有分离都绑定的变量具有约束力，在这种情况下，这不是变量。 |

虽然变量的出现可以是绑定的，也可以是非绑定的，但变量的“绑定”或“未绑定”属性是一个全局概念——单个绑定的出现就足以使变量绑定。

因此，您可以通过提供绑定实例来修复上面的“无限”示例。

例如，你可以这样写，而不是 `fn timesTwo（n: int）-> int { result == n * 2 }`：

```
fn timesTwo(n: int) -> int {
  n in [0 .. 10] and
  result == n * 2
}
```

谓词现在绑定 `n`，变量 `result` 自动绑定计算 `result == n * 2`。

## MirrorQL 内建谓词(MirrorQL built-in predicates)

`MirrorQL`的内建谓词包括：

- `none()`: 表示从不成立的谓词，对应空集
- `any()`: 表示总成立的谓词，对应非空集合
- `range(start, end)`: 返回一个整数区间

示例：

```
fn nothing() -> int { // 无返回值
 result==1 and none()
}
fn getOne() -> int { // 返回值集合仍然为{1}
 result==1 and any()
}

fn getRange() -> int { // 返回值集合为{1,2,3,4}
 result == range(1,5)
}
```
