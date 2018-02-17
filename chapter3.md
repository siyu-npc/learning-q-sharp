# 第三章 量子设备和驱动管理

本章的内容将涉及：

* 一个量子算法是如何被执行的
* 量子模拟器是什么样的
* 如何用C\#语言写量子算法的驱动


### 3.1 量子开发工具包执行模型

在前面所写的程序中，我们通过将一个`QuantumSimulator`对象传递给算法程序来记性量子算法，`QuantumSimulator`对象能够完整模拟量子态向量，从而来执行我们的量子算法。

其他的机器也可以用来执行量子算法，这些机器负责提供原始量子原语的实现，包括基础的H门、CNOT门等操作和量子测量方法，以及量子比特的管理和追踪。不同的量子机器代表了不同的运行模型，每一种量子机器也可能提供这些原语的不同实现，例如，微软量子开发包中的量子计算机追踪模拟器实际上不执行任何模拟操作，它只是跟踪量子门、量子比特以及量子算法中用到的其他资源。

#### 量子机器

在未来，我们会定义更多的量子机器类型来支持其他种类的的模拟器，并支持程序在拓扑量子计算机上执行。允许算法保持不变，同时改变底层的机器实现能够使得更方便地在模拟器中开发和测试程序，最后在真正的量子计算机上运行它。

在微软量子开发包中包含两种模拟器类型，它们都定义在命名空间`Microsoft.Quantum.Simulation.Simulators`中：
- `QuantumSimulator`：完整的量子态向量模拟器
- `QCTraceSimulator`：资源追踪模拟器

#### 编写经典的驱动程序

在前面的程序中，我们编写了一个简单的C#程序来驱动`teleport`算法。一个C#驱动有4个主要的功能：

1. 构建目标机器
2. 计算量子算法所需要的参数
3. 使用模拟器运行量子算法
4. 对运行结果进行处理

下面我们分别对这些功能进行探讨。

#### 构建目标机器

Q#中的量子机器是一个常规的.NET类型的实例，它们通过自身的构造函数进行创建以及初始化。一些模拟器包括`QuantumSimulator`应用了.NET的`IDissposable`接口，因此需要使用`using`语句进行声明。

#### 计算量子算法所需要的参数

在我们的`Teleport`程序示例中，我们为量子算法计算了一些需要的相关人造数据，但是通常量子算法需要大量重要的数据，这些数据由其驱动来提供是比较方便的。

例如，在模拟化学变化时，量子算法需要一个大的分子轨道相互作用的积分表，这些数据通常存储在一个文件中，但是Q#并不能读取文件系统的数据，因此，为了将这些数据传递给量子算法，就需要驱动程序来完成数据的读取以及传递工作。

另一个驱动程序扮演关键角色的例子是变分法。在这一类算法中，一个量子态基于一些经典参数进行筹备，然后这个量子态被用来计算一些有价值的数据。这些参数基于一些爬山算法或者机器学习算法进行调整，然后运行我们的量子算法。对于这类算法，将爬山算法设计成经典的算法是最合适的，然后子驱动程序中进行调用，最后将其结果传入量子算法之中。

#### 运行量子算法

这一部分的内容是非常直观的。每一个Q#操作都会被编译进一个提供了静态`Run`方法的类中，传入这个方法的参数由经过平整的Q#操作的参数元祖给出，外加一个指向模拟器的变量作为`Run`方法的第一个参数。例如对于一个类型为`(String, (double, Double))`的元祖，它经过平整处理后的类型变为`(String, Double, Double)`。
将参数传递给`Run`方法时有一些细节需要注意：

- 数组必须包装在`Microsoft.Quantum.Simulation.Core.QArray<T>`类型中。`QArray`类型有一个能够接受任意排序集合对象`(IEnumerable<T>)`的构造函数
- 空元祖`()`在C#中使用`QVoid.Instance`来表示
- 非空元祖使用.NET的`ValueType`实例来表示
- Q#的用户自定义类型作为它们的基类来传递
- 为了将`operation`和`function`传入`Run`方法中，必须创建一个该`operation`或`function`的实例

#### 处理结果

量子算法的结果由`Run`方法返回，由于`Run`方法是异步执行的，它返回一个`Task<T>`实例，有几种方法来获取真正的执行结果，最简单的方法是使用`Task<T>`类型的`Result`属性：
```
var res = BellTest.Run(sim, 1000, initial).Result;
```
但其他的方法，如使用[`Wait`](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.wait?view=netframework-4.7.1)方法或C#中的[`await`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/await)方法也是可行的。

许多的量子算法需要大量的后期处理来提取有用的结果，例如Shor算法中的量子部分仅仅是因数分解算法的开始。在大部分情况下，将这些后期处理任务放在经典驱动中是最简单方便的，毕竟只有精短驱动能将结果汇报给用户或写入磁盘中。

#### 执行失败

当在Q#`operation`中遇到`fail`语句时一个`ExecutionFailException`会被抛出。

因为在`Run`方法中使用了`System.Task`，`fail`语句抛出的异常会被包装成为一个`System.AggregateException`，要找出产生异常的真实原因，需要在`AggregateExecption`中迭代`InnerException`，例如以下代码：
```
try
{
	using(var sim = new QuantumSimulator())
	{
		/// call your operations here...
	}
}
catch (AggregateException e)
{
	// Unwrap AggregateException to get the message from Q# fail statement.
	// Go through all inner exceptions.
	foreach (Exception inner in e.InnerExceptions)
	{
		// If the exception of type ExecutionFailException
		if (inner is ExecutionFailException failException)
		{
			// Print the message it contains
			Console.WriteLine($" {failException.Message}");
		}
	}
}
```


