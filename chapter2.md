## 第二章  Q\#编程语言

谈到量子计算，一个自然而然的观点是将量子计算机看作一种协处理器，就像如今的GPU、FPGA一样，程序运行时，经典的通用计算机完成基本的逻辑控制，在特定的情况下，调用运行在协处理器上的子程序处理相应的任务，协处理器完成任务后主程序获取子程序的运行结果，然后进行后续处理。


在这种主机+协处理器的模型中，计算被分为了三个层次：

1. 经典计算。完成数据的输入/输出，设置和触发量子计算机并处理计算结果等经典计算机能够处理的任务。
2. 量子计算。直接在量子计算设备上应用量子算法完成计算任务。
3. 量子计算在处理过程中调用经典计算。

以上三个层次并不需要全部使用同一种语言来实现，实际上，由于量子计算机有着独特的控制结构和资源管理机制，因此采用专门的编程语言在量子算法中实现通用的模型可能更自然流畅。

将经典计算与量子计算分离的模型意味着量子编程语言可能会受到更多更严格的限制，但是这些限制也给优化程序使程序运行更快创造了条件。

Q\#（`Q-Sharp`）语言就是上面模型下的产物，它是一种用来表达量子算法的特定领域编程语言，用来编写运行与量子协处理器上的子程序，并受经典计算机和程序操控。

Q\#提供了一个小的基本数据类型集合和两种创建新的结构体类型的方法，它同时也支持基本的循环和条件分支结构Q\#的顶层结构包含了用户自定义类型、`operation`和函数。

本章接下来的内容包括以下部分：

* Q\#语言的类型模型
* 表达式
* 语句
* 文件结构

### 2.1 类型模型

#### 2.1.1 基本数据类型

Q\#提供的基本数据类型如下，出去这些基本类型，其他类型都属于结构化类型。

* `Int`：64位有符号整型
* `Double`：双精度浮点数类型
* `Bool`：布尔类型，取值为`true`或`false`
* `Qubit`：`Qubit`表示一个量子比特或量子位。`Qubit`对用户是不透明的，除了将它们传递给另一个`operation`外，唯一的操作可能就是验证其相同性了。最终，所有在`Qubit`上的操作都需要通过Q\#的标准库来实现。
* `Pauli`：`Pauli`表示一个单量子位群中的一个元素， 这种类型用于表示旋转的基本操作，并指定测量的基准，并且它是由四个可能的取值构成的可区分的联合体，这四个值分别是：`PauliI`、`PauliX`、`PauliY`和`PauliZ`。
* `Result`：`Result`表示测量得到的结果，它是由两个可能的取值构成的可区分的联合体，这两个值分别是：`One`和`Zero`，`Zero`表示特征值+1，`One`表示特征值-1。
* `Range`：`Range`表示整数序列，如`1..5`这个`Range`就表示`（1,2,3,4,5）`这个整数序列，注意在Q\#中，`Range`所表示的范围是一个闭区间，包含头尾的数值。
* `String`：Unicode字符序列。

#### 2.1.2 数组（Array）

任何一个合法的Q\#数据类型都可以构成数组，表示为`T[]`，其中`T`为任意合法类型，例如`Qubit[]`、`Int[][]`，注意这里的`Int[][]`并不像其他编程语言中的二维数组是一个矩形，在Q\#中，这个二维数组是锯齿形的，也就是说其每一行的数据数量是不同的，目前来看，Q\#中并不支持矩形的二维数组。

#### 2.1.3 元组（Tuple）

给定任意数量合法的Q\#数据类型，他们能够组成元组类型，表示为`(T1,T2,T3,...)`，其中`T1,T2,T3`是任意合法的数据类型，除了基本类型，它们还可以是`Array`、`Tuple`等，一个空的括号`()`表示一个空的元组。相信熟悉`Python`的朋友对元组应该不陌生。

** 元组一旦定义就不能改变其内容。 **

同时需要注意一下只含有一个元素的元组`(T)`，在Q\#中，单个元素的元组与这个元素本身是等价的，例如`(5)`和`([1,2,3])`，`(5)`和5以及`([1,2,3])`和`[1,2,3]`没有任何区别，同样的`(((5)))`与5，`(5,(6))`与`(5,6)`也都是一样的。这个性质在Q\#中称为**单元素元组等价**（`Singleton Tuple Equivalence`）

#### 2.1.4 用户自定义类型

Q\#中可以基于基本类型和合法的用户自定义类型来创建新的类型结构，其语法如下：

```
//T1,T2位基本类型或用户定义的其他类型
newtype TypeName = (T1,T2,...);
```

用户自定义类型有两个限制，一是不能定义递归结构的类型，也就是说新的类型结构中不能包含该类型本身，如下定义的类型就是非法的递归结构类型：

```
newtype Type1 = (Int,Type1);
```

二是定义的类型之间不能存在循环依赖，这一点可以通过以下代码来说明：

```
//非法，存在循环依赖
newtype TypeA = (Int, TypeB);
newtype TypeB = (Double, TypeC);
newtype TypeC = (TypeA, Range);
```

用户自定义类型的可变性由其基类型决定，如果其基类为不可变类型，例如元组，那么这个类型也是不可变的，如果其基类是可变类型，如数组，那么这个类型也是可变的。

---

##### 问题：用户自定义类型是否必须是元组的形式，上面所说的不可变性中提到数组是否意味着自定义类型也可以是数组的形式或者其他

---

##### 用户自定义类型的兼容性

用户自定义类型是其基类的子类，因此凡是可以使用其基类的地方也能使用这个自定义类型，并且这个性质是递归定义的，例如一个新的类型`IntPair`基于`(Int,Int)`定义，`IntPair2`基于`NewPair`定义，那么凡是可以使用`(Int,Int)`、`NewPair`的地方都能使用`NewPair2`。

但是不同的用户自定义类型是不同的不兼容的，即使它们基于相同的基类进行定义。例如假设一个新的类型`IntPair3`也基于`(Int,Int)`进行定义，那么能够使用上面例子中`IntPair`的地方并不能够使用`IntPair3`。

#### 2.1.5 operation和function

---

**问题：这里并没有给出operation和function的具体定义形式，容易曹成误解。以后需要补充。**

---

Q\#中，`operation`是一个量子计算子例程，也就是说它是一个可调用的包含量子操作的例程。

`function`则是用于量子计算中的经典子例程，它可以包含经典程序代码甚至其中可以没有量子操作代码，在`function`中可能不会分配量子位，也不会调用`operation`，然而，量子位和`operation`可能会被传进`function`中进行处理。

`operation`和`function`在Q\#中都是** 可调用类型 **，他们统称为`callable`。

Q\#中所有的可调用对象都可以看作是接收一个输入然后产生一个输出，这里的输入输出都有元组进行表示，当然，没有输入的可调用对象以空元组为输入，不产生结果的可调用对象返回空元组。

可调用对象的基签名式为`('Tinput => 'Tresult)`或`('Tinput -> 'Tresult)`，其中`'Tinput`和`'Tresult`是相应的数据类型，第一种形式用于`operation`，第二种形式用于`function`。例如，`((Qubit,Pauli) => Result)`表示一个单量子位的测量`operation`。

`function`类型完全由其签名式决定，例如一个计算某个角度`sine`值的函数可表示为`(Double -> Double)`。`operation`类型则由其签名式和其支持的`functor`共同决定（关于`functor`马上就会讲到），`operation`允许应用一个或多个functor。例如`Pauli X operation`的类型表示为：`(Qubit => () : Adjoint, Controlled)`，一个不支持任何functor的operation只需要写出其签名式即可，不需要之后的冒号以及`functor`列表。

##### 参数化类型的operation和function

可调用类型的签名式中可以包含参数化类型。参数化类型中使用一个单引号：`'`作为前缀，例如`'A`就是一个合法的类型参数。这样的类型参数有些类似于其他编程语言中的泛型，但实际上Q\#并没有提供对泛型的完全支持。

类型参数可以在一个签名式中多次出现。例如，一个`function`对数组中的元素应用另一个`function`，并返回运行结果组成的数组，那么这个`function`的签名式就是：`(('A[], 'A -> 'A) -> 'A[])`，类似的，一个返回由两个`operation`组合而成的结果的`function`其签名式可能为：`((('A => 'B), ('B => 'C)) -> ('A => 'C))`。

调用参数化类型的operation和function时，所有拥有相同参数类型的参数类型也必须是相同的，或者是兼容的。

对于一个类型参数，Q\#没有提供相应的机制来约束可以替换该参数的类型，因此，参数化类型对于作用于数组的`function`和组合的可调用类型更有用。

##### 可调用对象的类型兼容性

* 一个支持多个`functor`的`operation`可以用在使用另一个支持较少`functor`的`operation`的场合，只要这两个`operation`拥有相同的签名式。例如，一个类型为`(Qubit => () : Adjoint)`的`operation`可以用于使用类型为`(Qubit => ())`的`operation`的场合。

* 一个返回类型为`'A`可调用类型与接受相同输入参数类型而`'A`与其返回结果兼容的可调用类型是兼容的，这叫做返回值协变性。

* 一个输入参数类型为`'A`的可调用类型与另一个拥有相同返回类型且输入参数与`'A`兼容的可调用类型是兼容的，这叫做输入类型逆变性。

根据以上所述，给出下面几个类型：

```
newtype IntPair : (Int, Int) 
newtype IntPairTransform : ((Int, Int) -> (Int, Int))
newType IntPairTransform2 : ((Int, Int) -> IntPair)   
newType IntPairTransform3 : (IntPair -> (Int, Int))
```

其中：

* 一个类型为`IntPairTransform`的可调用对象可以被拥有一个`IntPair`参数的类型调用
* 调用一个类型为`IntPairTransform`的`function`得到的结果可能无法用于需要`IntPair`类型的场合
* 一个类型为`IntPairTransform2`的`function`可以用于需要`IntPairTransform`的场合中，但反过来就不行
* 一个类型为`IntPairTransform`的`function`可以用于需要`IntPairTransform3`的场合中，但反过来就不行

#### 2.1.6 functor

在Q\#中，`functor`是根据一个`operation`生成另一个`operation`的工厂。当实现一个新的`operation`时，`functor`可以访问基本`operation`的实现，从而能够执行比传统的高层次函数更复杂的功能。

一个`functor`能应用于一个`operation`并返回一个新的`operation`。例如，一个通过将`functor Adjoint`应用于`Y operation`而得到的`operation`可以写作：`(Adjoint Y)`，这个新的`operation`可以像其他`operation`一样被调用。因此，`(Adjoint Y)(q1)`就是讲`Adjoint`应用于`Y`产生一个新的operation，并用这个新的`operation`作用于`q1`得到最终的结果。类似的还有`(Controlled X)(controls, target)`等。

在Q\#中，有两个标准的`functor`：`Adjoint`和`Controlled`。

##### Adjoint

在量子计算中，一个操作的伴随矩阵是该操作的复共轭矩阵，对于实现幺正运算的操作，其伴随矩阵就是逆矩阵，对于只是调用了一系列作用于一组量子位上的幺正运算的操作，其伴随矩阵可以以相反的顺序，通过应用在同一量子位上的子运算而计算出来。

给定一个`operation`表达式，通过使用`Adjoint`修饰并用括号括起来就形成了一个新的`operation`。这个新的`operation`拥有与其父`operation`相同的签名式，且允许使用`Adjoint`继续生成新的`operation`，但只有在父`operation`支持`Controlled`时，新的`operation`才支持`Controlled`。

例如，`(Adjoint QFT)`指定了`QFT operation`的伴随矩阵。

##### Controlled

一个`operation`的受控形式是一个能够高效地将这个`operation`应用于所有量子位的新的`operation`，但是这些量子位必须处于同一个状态。如果控制量子位处于叠加状态，那么最初的`operation`将被连贯地用于适当的叠加部分，因此受控`operation`经常用于产生纠缠态。

在Q\#中，受控`operation`总是采用一个量子位数组，并且对于控制量子位，它们总是处于指定的`One`状态：`|1>`。对处于其他状态的量子位进行操控可以通过在受控`operation`发生作用前应用合适的`Clifford operation`来完成，并在受控`operation`作用后应用逆`Clifford operation`。例如，对于一个控制量子位，在受控operation作用前后对其施加X operation将导致控制在Zero状态下进行，应用`H operation`将导致控制在`PauliX Zero`状态`|+> := ((|0⟩+|1⟩)/√2` 下执行而非`PauliZ Zero`状态。

与`Adjoint`类似，通过用`Controlled`对一个`operation`进行修饰并且使用括号括起来就产生了一个新的`operation`，新的`operation`的签名式基于父`operation`，其结果类型与父`operation`相同，但其输入参数类型为由两个元素组成的元组，第一个元素是由控制量子位组成的数组，父`operation`的输入参数为其第二个元素，如果父`operation`的输入参数为空`()`，那么新的`operation`的参数仅有控制量子位组成的数组构成。新的`operation`允许使用`Controlled`继续生成新的`operation`，但只有在父`operation`支持`Adjoint`时，新的`operation`才支持`Adjoint`修饰。

如果父`operation`只接受单个参数作为输入，那么**单元素元组等价**就会在此时起作用。例如，`Controlled(X)`是`X operation`的受控形式，`X`的类型为：`(Qubit => () : Adjoint, Controlled)`，那么`Controlled(X)`的类型为：`((Qubit[], (Qubit)) => () : Adjoint, Controlled)`，因为单元素元组等价的原因，Controlled\(X\)的类型与`((Qubit[], Qubit) => () : Adjoint, Controlled)`等价。类似的，`Controlled(Rz)`是`Rz`operation的受控形式，Rz的类型为`((Double, Qubit) => () : Adjoint, Controlled)`，所以`Controlled(Rz)`的类型为`((Qubit[], (Double, Qubit)) => () : Adjoint, Controlled)`，那么`((Controlled(Rz))(controls, (0.1, target))`就是一个合法的`Controlled(Rz)`调用。

### 2.2 表达式

#### 2.2.1 分组

给定任意一个合法的表达式，将其用一对括号括起来构成一个新的表达式，那么这两个表达式在Q\#中具有相同的类型。例如`(7)`是一个`Int`型的表达式，`([1;2;3])`的类型是`Int`型的数组，`((1,2))`是`(Int,Int)`型的元组。

#### 2.2.2 符号

将一个类型为`'T`值绑定或赋值给一个符号，那么这个符号的类型也是`'T`，例如，将5赋值给`count`，那么`count`就是整数型的表达式。

#### 2.2.3 代数表达式

Q\#中代数表达式的类型是只能是`Int`或`Double`。

`Int`型数据的字面量在Q\#和C\#中是一样的，不过Q\#中不需要在数值字面量后加`l`或`L`。16进制数需要在字面量前加0x前缀。

`Double`型数据字面量在Q\#和C\#中也是一样的，不过在Q\#中不需要在数值字面量后加`d`或`D`。

对于一个任意类型的数组，内置函数`Length`返回该数组的元素数量（`Int`型）。例如，`a`是一个数组，`Length(a)`返回`a`中元素的数量，`b`的类型为`Int[][]`，那么`Length(b)`返回`b`中自数组的数量，`Length(b[1])`返回`b`中第二个子数组中元素的数量。

代数运算中运算符`+、-、*、/`与操作数一起构成新的代数表达式，如果操作数中有一个是`Double`类型那么这个表达式的类型也是`Double`类型，如果两个操作数都是`Int`类型，那么该表达式也是`Int`类型。

对于整型数而言，其支持的运算和运算符还包括：`%`（取模）、`^`（乘方）、`&&&`（按位与）、`|||`（按位或）、`^^^`（按位异或）、`~~~`（按位取反）、`<<<`（算术右移）、`>>>`（算术左移）。在移位操作中，第二个操作数必须大于等于0，如果小于0，对应的行为是未定义的。

#### 2.2.4 量子位表达式（Qubit expressions）

Q\#中量子位没有对应的字面量，将量子位或量子位数组绑定到一个符号上就构成了一个量子位表达式。

#### 2.2.5 泡利表达式（Pauli expressions）

四个`Pauli`值：`PauliI`、`PauliX`、`PauliY`和`PauliZ`都是合法的泡利表达式，将这些值或这些值组成的数组绑定至一个符号上也构成合法的泡利表达式。

#### 2.2.6 结果表达式（Result expressions）

`Result`有两个取值：`One`和`Zero`，除了这两个值，将这两个值或由其组成的数组绑定至一个符号也构成合法的结果表达式。需要注意的是，这里的`One`并不等于整数1，两者之间也不存在任何直接的转换方法，但是`Zero`与0是相等的。

#### 2.2.7 范围表达式（Range expressions）

使用三个整数就能构成一个范围表达式，这三个整数分别表示起始值、步长和终止值，范围表达式形式为：`start..step..stop`。范围表达式的其实值就是`first`，第二个值是`first+step`，第三个值是`first+step*2`，以此类推，知道数值到达或超过了`stop`。范围表达式也可以表示一个空的范围，例如当`step`为正并且`start<stop`。范围表达式是一个闭区间，其包括最后的`stop`。在范围表达式中，`step`可以省略，其省略时步长取默认值1。下面来看一些范围表达式的例子：

* `1..3`表示范围`1,2,3`
* `2..2..5`表示范围`2,4`
* `2..2..6`表示范围`2,4,6`
* `6..-2..2`表示范围`6,4,2`
* `2..1`是一个空的范围
* `2..7..6`表示范围`2`
* `2..2..1`是一个空的范围
* `1..-1..2`是一个空的范围

#### 2.2.8 可调用表达式

一个可调用类型的表达式就是`operation`或`function`的名字。例如，`X`就表示标准库中的`X operation`，`Message`则是标准库中的`Message`函数。

如果一个`operation`支持`Adjoint functor`，那么`(Adjoint op)`也是一个`operation`表达式，类似的，如果一个`operation`支持`Controlled functor`，那么`(Controlledop)`也是一个`operation`表达式。

##### 调用可调用表达式

给定一个可调用表达式（`operation`或`function`）和相应的输入数据表达式，那么将输入数据后接于可调用表达式就构成了对可调用表达式的调用，该表达式的类型为可调用表达式的返回类型。例如，一个名为Op签名式为`((Int, Qubit) => Double)`的`operation`，`Op(3,qubit1)`就是对`Op`的调用，这个表达式的类型为`Double`。

如果一个可调用表达式的返回值仍然是一个可调用对象，那么调用这个返回值时需要多加一对括号，如下代码所示：

```
(Adjoint(Op))(3, qubit1)
```

##### 部分调用

给定一个可调用表达式，可以传递给它一部分参数从而构造一个新的可调用对象，这就叫做** 部分调用** 。

在Q\#中是通过使用下划线\_代替一个可调用对象的参数，其它部分保持不变来表示一个部分调用的表达式的，这个新的表达式拥有和原表达式相同的返回类型，如果远表达式一个个`operation`，那么新的表达式有相同的参数变量，对于这个新的表达式，其输入参数类型是除去指定参数类型后剩下的部分，下面的例子说明了输入参数的类型与原表达式的关系。

Op的类型为：`((Int, ((Qubit,Qubit), Double)) => ():Adjoint)`，那么：

* `Op(5,(_,_))`和`Op(5,_)`的类型均为：`(((Qubit,Qubit), Double) => ():Adjoint)`
* `Op(_,(_,1.0))`的类型为：`((Int, (Qubit,Qubit)) => ():Adjoint)`
* `Op(_,((q1,q2),_))`的类型为：`((Int,Double) => ():Adjoint)`

##### 递归

Q\#中的可调用对象支持递归，这也就是说，一个`operation`或`function`可以直接调用其自身。使用递归时，有两个注意事项：

1. 递归的使用有可能妨碍对程序进行优化，这个对算法的运行时间会造成实质性的影响，此时编译器应该向用户发出警告。
2. 当在一个特定的量子计算设备上使用递归程序时，如果递归深度较大亏导致运行时错误，特别的是，在Q\#中，编译和运行时尾递归都不能被识别出来也不会对其进行优化。

#### 2.2.9 元组表达式

一个元组的字面量是由括号括起来的一系列各种类型数据的组合，数据之间用逗号进行分隔。例如`(1,One)`就是一个类型为`(Int,Result)`的元组表达式。

除了上面所说的字面量，如果一个符号绑定了元组表达式、元组数组中的元组元素以及返回元组类型的可调用对象都是合法的元组表达式。

#### 2.2.10 用户自定义类型表达式

用户自定义类型的字面量由元组表达式和在其中的类型名称组成。例如，用户自定义类型`IntPair`基于`(Int,Int)`，那么`IntPair(2,3)`就是一个合法的`IntPair`类型的字面量。

#### 2.2.11 数组表达式

一个数组的字面量是由中括号括起来的一系列特定类型数据的组合，数据之间由分好分隔。数组中所有的类型必须与数组定义时的类型兼容，如果数组定义的元素类型是`operation`或`function`，那么每个元素必须拥有相同的签名式，对于`operation`而言还必须拥有支持相同的`functor`。

`[]`在Q\#中不是一个空的数组且其是非法的表达形式，要想定义一个长度为0的数组，使用以下形式：`new ★[0]`，此处`★`是类型的占位符。

给定两个相同元素类型的数组，操作符+将第二个数组中的元素附加在第一个数组后面，从而构造出一个新的数组。例如：`[1;2;3] + [4;5;6]`得到新数组：`[1;2;3;4;5;6]`

##### 数组的创建

给出一个类型T和一个整型表达式，使用`new`操作符就能创建一个元素类型为`T`，长度为整型表达式值的数组。数组中元素根据类型而被初始化为相应的值，通常默认的初始化值都为0。对于量子位和可调用类型而言，因为他们是引用类型，不存在一个合理的默认值，因此它们被初始化为一个非法的引用，如果直接使用将会导致运行错误，这类似于C\#或Java中的`null`，要正确使用它们，必须使用`set`语句对每个元素进行赋值。

数组类型只有被声明为`mutable`时才能被赋值，例如：`mutable array = new Int[5]`，数组被当做参数传递时是不可改变的。

下标列出了部分类型的默认值：

| 类型 | 默认值 |
| :--- | :--- |
| Int | 0 |
| Double | 0.0 |
| Bool | false |
| String | "" |
| Qubit | invalid qubit |
| Pauli | PauliI |
| Result | Zero |
| Range | 空范围：1..1..0 |
| Callable | invalid callable |
| Array\['T\] | 'T\[0\] |

元组类型中元素一个接一个被初始化。

##### 数组切片

Q\#中数组元素的次序是从0开始的。给定一个数组和一个范围表达式，可以得到该数组中在此范围中的元素构成的新的数组，这就是数组的切片，其表达形式为：`a[range expression]`，其中`a`为一个数组，得到的数组拥有与原数组相同的类型。例如，假设`a`是一个元素类型为`Double`的数组，`a[3..1..0]`得到一个新的数组，新数组中包含原数组的前四个元素，但是这四个元素的顺序与原来相反。如果范围表达式是一个空范围，那么得到的将会是一个长度为0的数组。

如果只想得到数组中某个位置的单个值，那么直接在中括号中指定元素的次序即可：`a[1]`就得到了数组`a`的第二个元素。

#### 2.2.12 布尔表达式

布尔类型的两个字面量是`true`和`false`。

对于两个与相同的基本类型兼容的变量而言，`==`和`!=`等比较运算符会得到一个布尔类型的结果。

对于大小和相同性的比较，用户自定义类型会被转换为它们的父类型然后进行比较。

对量子位类型进行比较时，实际比较的是它们是否是同一个量子位，量子位的状态不会被该操作比较、访问、测量和改变。

对`Double`类型进行比较时，要注意精度损失造成的结果不准确的问题。

`&&`和`||`是逻辑与或操作符，其规则与C\#相同。

`!`为逻辑非操作符。

##### 操作符优先级

下标列出了Q\#中操作符的优先级，从上到下优先级依次降低

| 操作符 | 元数 | 简介 |
| :--- | :--- | :--- |
| -,~~~,! | 单目 | 负数，按位取反，逻辑非 |
| ^ | 双目 | 乘方 |
| /,\*,% | 双目 | 除法，乘法，取模 |
| +,- | 双目 | 加减，数组相加，字符串相加 |
| &lt;&lt;&lt;,&gt;&gt;&gt; | 双目 | 算数右移，算数左移 |
| &lt;,&lt;=,&gt;,&gt;= | 双目 | 小于，小于等于，大于，大于等于 |
| ==,!= | 双目 | 相等，不等 |
| &&& | 双目 | 按位与 |
| ^^^ | 双目 | 按位异或 |
| \|\|\| | 双目 | 按位或 |
| && | 双目 | 逻辑与 |
| \|\| | 双目 | 逻辑或 |

#### 2.2.13 字符串表达式

Q\#中字符串可以用于`fail`语句和`Log`函数中，Q\#中字符串的语法是C\#中字符串语法的子集，这里不再进行过多介绍。

### 2.3 语句和其他结构

#### 2.3.1 注释

Q\#中有两种注释：

##### 单行注释

以//开头后跟注释内容，这样的注释在其他语言中也应用广泛，如C/C++、Java等。

##### 文档注释

以///开头的注释为文档注释，文档注释出现在`operation`、`function`和类型定义之前，他们会得到编译器的特殊关照，注释内容会作为可调用类型或用户自定义类型的说明文档。

文档注释的文本是API文档的一部分，它使用[Markdown文档格式](https://daringfireball.net/projects/markdown/syntax)，文档中不同的部分会使用相应的命名标题标识出来。作为对Markdown文档格式的扩展，对`operation`、`function`和用户自定义类型的交叉引用可以使用`@"<ref target>"`来引入，其中`<ref target>`是要引用的类型的全限定名称。作为可选择的功能，文档引擎也可以支持其他Markdown格式的扩展。

以下代码展示了文档注释的模样：

    /// # Summary
    /// Given an operation and a target for that operation,
    /// applies the given operation twice.
    ///
    /// # Input
    /// ## op
    /// The operation to be applied.
    /// ## target
    /// The target to which the operation is to be applied.
    ///
    /// # Type Parameters
    /// ## 'T
    /// The type expected by the given operation as its input.
    ///
    /// # Example
    /// ```Q#
    /// // Should be equivalent to the identity.
    /// ApplyTwice(H, qubit);
    /// ```
    ///
    /// # See Also
    /// - Microsoft.Quantum.Primitive.H
    operation ApplyTwice<'T>(op : ('T => ()), target : 'T) : () {
        body {
            op(target);
            op(target);
        }
    }

下面的名称是文档注释中的命名标题。

* `Summary`：对定义的`operation`、`function`和自定义类型的简要概括
* `Input`：对`operation`、`function`输入数据的描述，在其后面可以跟子段落以对各个输入参数进行简要说明，如上面代码中的`##op`、`##target`
* `Output`：对输出数据的描述
* `Type Parameter`：这是一个空段，它包含了对泛型参数类型进行描述的子段落
* `Example`：一个简短的说明`operation`、`function`和自定义类型如何使用的示例
* `Remarks`：描述`operation`、`function`和类型的某些方面的文字
* `See Also`：列出相关联的`operation`、`function`和类型的全限定名称
* `Reference`：记录引用和参考到的资料

### 2.3.2 命名空间

Q\#中，每一个`operation`、`function`和用户自定义类型都定义在某个命名空间中，Q\#遵循与其他.NET语言相同的命名规则，但Q\#中不支持嵌套的命名空间。特别地，对于两个定义的命名空间`NS.Name1`和`NS.Name2`，只用使用全部名称时才能被打开，直接打开`open NS`是错误的。

每个Q\#文件必须包含至少一个命名空间声明，声明命名空间时以`namespace`关键字引出，后跟名称和一对大括号`{}`，除了注释以外，所有代码都必须写在大括号里面。

在一个命名空间中，`open`指令可以允许缩短对其他命名空间中结构的引用。使用时在`open`指令后跟要打开的命名空间。`open`指令必须出现在任何`operation`、`function`和`newtype`指令之前，且`open`指令的作用域为整个命名空间。如果一个命名空间未被打开，当要引用其中的结构时必须使用全限定名称，例如`X.Y`命名空间中有一个名为`Op`的`operation`，要引用该`operation`，则必须指定其全限定名称：`X.Y.Op`，如果这个命名空间已被打开，那么可以直接引用`Op`。

一般来说，我们更倾向于使用`open`指令打开一个命名空间，但这可能带来潜在的命名冲突问题，在这种情况下，就要使用全限定名称来引用其他命名空间中的结构了。

### 2.3.4 格式化

大部分Q\#中的语句和指令都使用分号;作为结束符号。后跟代码块的语句和指令，例如`operation`和`for`语句不需要使用分号来终结。

### 2.3.5 语句块

Q\#的语句以语句块进行分组，一对大括号和其中的代码就构成了一个语句块，语句块中还可以包含其他语句块，这些被包含的语句块叫做子语句块，也叫作内部语句块，而包含内部语句块的语句块就是外部语句块。

### 2.3.6 符号绑定和赋值

Q\#中包括可变和不可变的符号，通常而言，我们更倾向于使用不可变的符号因为编译器能对其进行更好的优化。

对数组来说，可变性和不可变性共同作用于数组整体和其中的元素，这意味着，一个不可变的数组中的元素也是不可变的。

元组和基于元组的用户自定义类型，无论被绑定的符号本身是否是可变的，他们中的内容都是不可变的。如果一个元组绑定到了一个可变的符号上，那么这个符号可以被重新复制为其他元组，但最初的元组中的内容也是不可变的。

一个`operation`和`function`的参数是不可变的，这也意味着Q\#中不存在“传出参数”这样的概念，而且作为参数传递给`operation`或`function`的数组也是不可变的。

##### 不可变符号

不可变符号使用`let`进行绑定，这种操作大体与C\#中的变量声明和初始化相同，但在Q\#中这个符号一旦被绑定，其就是不可变的。

下面的代码展示了如何使用`let`对一个符号进行绑定，这句代码将符号`i`绑定到了一个值为5的`Int`型数据。

```
let i = 5;
```

如果代码右侧是一个元组或用户自定义类型，那么可以使用一种扩展语法对其进行方便的解构，如下代码所示：

```
let (i, f) = (5, 0.1);
```

上面代码将5绑定到了`i`，将0.1绑定到了`f`。

##### 可变符号

可变符号使用`mutable`关键字进行声明和初始化，如下面代码所示：

```
mutable counter = 0;
```

上面代码定义了一个可变符号`counter`，并将其赋值为0。

`mutable`语句不支持对元组进行解构。

如果一个可变符号被绑定到了一个不可变的数组上，此时会产生一个该数组的副本赋值给这个可变符号，因此对该数据进行更改不会对原数组造成影响。

##### 更新可变符号

使用`set`语句可以改变一个可变符号的值，如下代码所示对可变符号`counter`的值增加了1：

```
set counter = counter + 1;
```

`set`语句也可以对可变数组中的元素重新赋值：

```
set result[1] = One;
```

以上代码将数组`result`的第二个元素的值改为`One`，此处的`result`必须是一个声明为`mutable`的数组。

##### 作用域

一般而言，一个符号在在其被定义的代码块的结尾处超出作用范围并且不能再被操作，但这里有两个例外：

* `for`循环中绑定的循环变量作用域`for`循环体中，而不是`for`循环结束的地方。
* 一个`repeat/until`循环的三部分（循环体、测试条件、修正操作）被看做是一个作用域，所以循环体中定义的符号，在测试条件和修正操作中都是可见的。

内部代码块会继承在外部代码块中定义的符号；在一个代码块中，一个符号只能被定义一次。下面的代码是合法的：

```
if a == b {
    ...
    let n = 5;
    ...             // n is 5
}
let n = 8;
...                 // n is 8
```

```
if a == b {
    ...
    let n = 5;
    ...             // n is 5
} else {
    ...
    let n = 8;
    ...             // n is 8
}
```

而下面的代码是非法的：

```
let n = 5;
...                 // n is 5
let n = 8;          // Error!!
...
```

```
let n = 5;
...                 // n is 5
let n = 8;          // Error!!
...
```

### 2.3 控制结构

#### 2.3.1 for循环

`for`循环在一个整数范围中进行迭代，一个`for`循环由关键字`for`引出，后跟一个括号，括号中分为三部分，一是循环变量的名字，二是`in`关键字，最后是一个`Range`表达式，括号后紧接着就是相应的代码段。如果`Range`表达式代表了一个空范围，那么`for`循环语句将不会做任何事。

`for`循环中，`Range`表达式在循环开始之前完成全部的求值，所以在循环过程中，`Range`表达式所代表的范围是不会产生变化的。

下面展示了一个`for`循环代码块：

```
for (index in 0 .. n-2) {
    set results[index] = Measure([PauliX], [qubits[index]]);
}
```

#### 2.3.2 Repeat-Until-Success循环

`repeat`语句支持量子计算中的`“repeat until success”`（重复直至成功）模型，它包括`repeat`关键字，后跟一个代码块，该代码块就是要执行的循环体，在循环体后是`until`关键字和一个条件表达式，最后是`fixup`关键字和另一个代码块。`fixup`代码块是必须要有的，无论其是否为空，如果`fixup`代码块不做任何工作，那么只需在其中写一个空的表达式`()`即可。

`repeat`语句的执行过程为：首先执行循环体，然后测试条件表达式是否为`true`，如果为`true`，那么这个`repeat`语句就执行完毕了，如果为`false`，则执行`fixup`语句中的内容，然后重新开始`repeat`语句的执行，如此循环直至结束。

下面的代码展示了`repeat`语句的使用，该代码用 `Hadamard`门和`T`门实现了重要的翻转门`V3=(1+2iZ)/√5`，并应用该翻转门完成了一个[概率电路](http://oai.dtic.mil/oai/oai?verb=getRecord&metadataPrefix=html&identifier=AD0603249)。

```
using ancilla = Qubit[1] {
    repeat {
        let anc = ancilla[0];
        H(anc);
        T(anc);
        CNOT(target,anc);
        H(anc);
        (Adjoint T)(anc);
        H(anc);
        T(anc);
        H(anc);
        CNOT(target,anc);
        T(anc);
        Z(target);
        H(anc);
        let result = M([anc],[PauliZ]);
    } until result == Zero
    fixup {
        ();//空语句
    }
}
```

#### 2.3.3 条件表达式

`if`语句支持条件执行，条件表达式由if引出，后跟一个布尔表达式和相应的代码块，一个if语句块之后可以继续添加其他条件表达式，使用`elif`关键字引出，然后是相应的布尔表达式和语句块，在最后，可以使用`else`引出不满足上面所有条件时要执行的操作。

条件表达式的使用请看下面的代码：

```
if (result == One) {
    X(target);
} else {
    Z(target);
}
```

```
if (i == 1) {
    X(target);
} elif (i == 2) {
    Y(target);
} else {
    Z(target);
}
```

#### 2.3.4 return语句

`return`语句结束`operation`和`function`的执行，并将结果返回给调用者。`return`语句以`return`关键字开始，后跟一个表达式和分号。一个返回值为空`()`的可调用对象不需要`return`语句，如果需要在一定条件下提前结束执行，可以使用`return ()`来返回一个空值。`return`语句的使用如下面代码所示：

```
return 1;
```

```
return ();//返回空值
```

```
return (results, qubits);
```

#### 2.3.5 fail语句

`fail`语句技术一个`operation`的执行，并向调用者返回一个出错信息。`fail`语句由`fail`关键字和一个代表错误信息的字符串组成。下面的代码展示了`fail`语句的写法：

```
fail $"Impossible state reached";
```

```
fail $"Syndrome {syn} is incorrect";
```

### 2.4 量子位管理

接下来所讲的量子位管理相关语句不能用于`function`中。

#### 2.4.1 干净的量子位

`using`语句用于在语句块中获得新的量子位。这些新获得的量子位被保证处于可计算的`Zero`状态，并且这些量子位在语句块结束时也应该确保处于可计算的`Zero`状态。模拟器对这一点也被鼓励强制执行。

`using`语句包括以下几个部分：首先由关键字`using`引出，随后是量子位将要绑定到的符号以及用于赋值的等号，随后是一个元素类型为`Qubit`的数组，如下代码所示：

```
using (qubits = Qubit[bits * 2 + 3]) {
    ...
}
```

#### 2.4.2 污浊的量子位

`borrowing`语句用于在代码块中分配临时使用的量子位，用户应保证这些借来的量子位与借来时的状态相同。关于污浊的量子位的用法可以参看这个示例：[_Factoring using 2n+2 qubits with Toffoli based modular multiplication_](https://arxiv.org/abs/1611.07995) 。

`borrowing`语句的使用方法如下所示：

```
borrowing qubits = Qubit[bits * 2 + 3] {
    ...
}
```

当借用量子位时，系统会首先使用那些正在使用中但在当前`borrowing`语句所在语句块中不可访问的量子位来填充请求，如果这些量子位不足，系统会重新分配量子位以满足需求。

### 2.5 文件结构

一个Q\#文件中包含一个或多个命名空间，每一个命名空间中都包含有operation、function和用户自定义类型等的定义，一个命名空间中可以有多个这样的定义，但是最少必须有一个。

#### 2.5.1 用户自定义类型

关于用户自定义类型，请参考章节2.1.4。

#### 2.5.2 operation定义

operation是Q\#的核心，大体相当于其它语言中的函数。

使用关键字operation定义一个新的operation，后面紧跟operation的名称，名称之后是两个元组，第一个元组为operation的输入参数列表，第二个元组描述了operation的结果的类型，两个元组中间使用冒号:分隔，最后是一对大括号{}，在这对大括号中是operation的主体代码，代码分为4个可选的组成部分：

* Body主体代码部分
* Adjoint定义部分
* Controlled定义部分
* Controlled Adjoint定义部分

一个operation只有在其同事定义了受控变量和伴随变量时才应该定义Controlled Adjoint变量。

下面分别对这几个部分进行说明。

##### Body

operation的主体是实现这个operation的Q\#代码，一个operation中也可以不包含任何主体代码，例如Pauli门和 Hadamard门等一些基本操作就是以这种方式实现的。

主体部分有body关键字和紧随其后的语句块组成。

##### Adjoint

一个operation的adjoint指出了如何实现该operation的复共轭转置，adjoint对operation不是必须的，例如基本的测量操作就没有实现其adjoint功能，operation是否应该定义adjoint取决于其是否含有Adjoint声明。

Adjoint定义由关键字Adjoint引出，后面的内容可以是几个部分中的一个：

* 关键字self指明该operation是其自身的adjoint
* 关键字auto表示由Q\#编译器根据该operation的body主体部分自动生成相应的adjoint，自动生成的adjoint是以相反的顺序应用body中的各个operation的adjoint来实现的
* 自己定义实现adjoint的语句块

以下几段代码分别对应上面的几种情况：

```
adjoint self
```

```
adjoint auto
```

```
adjoint {
    // Code for the adjoint goes here
}
```

如果一个operation的body中包含repeat循环、set语句、and/or测量操作或者调用了不支持adjoint的其他operation，那么该operation的adjoint不能使用auto关键字来实现。

如果一个operation没有body主体，但其应有adjoint定义，那么应使用adjoint self或adjoint auto来实现。

##### Controlled

一个operation的受控版本指出了该operation的量子受控版本如何实现。这一部分也是选择实现的，例如基本测量操作就没有实现controlled版本因为其是不可控的，operation是否应该定义controlled取决于其是否含有Controlled声明。

Controlled部分的定义由关键字controlled引出，其后可以跟下面几个中的某一项：

* 关键字auto表示由Q\#编译器根据该operation的body主体部分自动生成相应的controlled部分
* 一对包含了控制量子位数组符号的括号\(\)和一个实现controlled的语句块

下面几段代码展示了如何定义controlled：

```
controlled auto
```

```
controlled (controls) {
    // Code for the controlled version goes here.
    // "controls" is bound to the array of control qubits.
}
```

如果一个operation的body中包含repeat循环、测量操作或者调用了不支持controlled的其他operation，那么该operation的controlled不能使用auto关键字来实现。

如果一个operation没有body主体，但其应有controlled定义，那么应使用controlled auto来实现。

##### Controlled Adjoint

一个operation的controlled adjoint版本指出了该operation的adjoint如何实现其量子受控版本。这一部分也是选择实现的，例如基本测量操作就没有实现controlled adjoint。operation是否应该定义controlled adjoint取决于其是否同时声明了Controlled和Adjoint。

一个controlled adjoint的定义包括关键字adjoint，随后是关键字controlled，其后可以跟下面项目中的一个：

* 关键字auto指示Q\#编译器应该根据该operation的body部分生成controlled adjoint
* 一对包含了控制量子位数组符号的括号\(\)和一个实现controlled adjoint的语句块

如果一个operation的body中包含repeat循环、set语句、and/or测量操作或者调用了不支持controlled adjoint的其他operation，那么该operation的controlled adjoint不能使用auto关键字来实现。

如果一个operation没有body主体，但其应有controlled定义，那么应使用controlled auto来实现。

如果一个operation的Controlled或Adjoint的部分是通过自定义的语句块实现的，同时该operation使用auto实现了controlled adjoint，那么controlled adjoint会通过自定义的代码块实现的。

如果一个operation的同时用自定义语句块实现了Controlled和Adjoint版本，同时该operation使用auto实现了controlled adjoint，那么controlled adjoint会根据自定义的controlled版本语句块来实现。

如果一个operation没有body主体，但其应有controlled adjoint定义，那么应使用controlled adjoint auto来实现。

通过以上的描述，下面几段代码展示了如何定义operation：

```
operation X (q : Qubit) : () {
    adjoint self
    controlled auto
    adjoint controlled auto
}
```

```
///实现teleport操作
namespace Microsoft.Quantum.Samples {
    // Entangle two qubits.
    // Assumes that both qubits are in the |0> state.
    operation EPR (q1 : Qubit, q2 : Qubit) : () {
        body
        {
            H(q2);
            CNOT(q2, q1);
        }
    }

    // Teleport the quantum state of the source to the target.
    // Assumes that the target is in the |0> state.
    operation Teleport (source : Qubit, target : Qubit) : () {
        body {
            // Get a temporary for the Bell pair
            using (ancilla = Qubit[1]) {
                let temp = ancilla[0];

                // Create a Bell pair between the temporary and the target
                EPR(target, temp);

                // Do the teleportation
                CNOT(source, temp);
                H(source);
                if (M(source) == One) {
                    X(target);
                }
                if (M(temp) == One) {
                    Z(target);
                }
            }
        }
    }
}
```

#### 2.5.3 function定义

function是Q\#中的经典例程。function的定义形式与operation大体相同，使用function关键字引出，后跟function名称、输入参数、返回结果以及语句块，如下面代码所示：

```
function DotProduct(a : Double[], b : Double[]) : Double {
    if (Length(a) != Length(b) {
        fail "Arrays are not compatible";
    }
    mutable accum = 0.0;
    for (i in 0..Length(a)-1) {
        set accum = accum + a[i] * b[i];
    }
    return accum;
}
```



