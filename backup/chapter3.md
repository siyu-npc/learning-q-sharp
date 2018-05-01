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

Q#中的量子机器是一个常规的.NET类型的实例，它们通过自身的构造函数进行创建以及初始化。一些模拟器包括`QuantumSimulator`实现了.NET的`IDisposable`接口，因此需要使用`using`语句进行声明。

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

### 量子计算机模拟器

微软量子开发包提供了一个全状态的量子计算机模拟器，使用它你可以在自己的计算机上运行和调试使用Q#语言开发的程序。

该模拟器通过类`QuantumSimulator`引入，使用模拟器时，只需要定义一个该类的实例并将这个实例传入`Run`方法的第一个参数中即可，如下代码所示：

```
using (var sim = new QuantumSimulator())
{
	var res = myOperation.Run(sim).Result;
	///...
}
```

#### IDisposable

类`QuantumSimulator`实现了接口`IDissposable`接口，因此一个`QuantumSimulator`的实例不再被使用时，应调用方法`Dispose`。使用`QuantumSimulator`最好的方法是使用`using`语句，就像上面的例子中那样，这样可以不用显式调用`Dispose`方法，也减少了出错的概率。

#### Seed

类`QuantumSimulator`使用了一个随机数生成器来模拟量子力学中的随机性。为了方便调试，有时候需要得到一个确定的结果，此时可以通过向`QuantumSimulator`的构造函数中传入一个seed来实现，这个seed传给了参数`randomNumberGeneratorSeed`：

```
using (var sim = new QuantumSimulator(randomNumberGeneratorSeed: 42))
{
	var res = myOperationTest.Run(sim).Result;
	///...
}
```

#### Thread

类`QuantumSimulator`使用[OpenMP](http://www.openmp.org/)实现所需要的线性代数的并行计算。默认情况下，OpenMP使用所有能用的硬件线程来进行计算，这也意味着有着较少数量量子比特的程序运行起来会比较慢，因为所需要的协调工作相比真正的工作占据了大部分资源。这个问题可以通过将环境变量`OPM_NUM_THREADS`设置为一个较小的数字来解决。一个常用的策略是，1个线程对应至多4个量子比特是较好的，然后每增加一个量子比特，就增加一个线程，当然了，你也可以根据自己的算法进行更好地调整。


### 量子轨迹模拟器

微软量子轨迹模拟器并不是靠真实地模拟量子计算机的状态来运行程序，因此轨迹模拟器可以运行一个使用了成百上千个量子比特的量子程序。这一点有两个好处：
1. 方便调试量子程序中的经典代码
2. 方便估算在量子计算机上运行一个量子程序所需要的资源

#### 提供测量输出结果的概率

在量子算法中主要有两种测量，第一类通常其辅助作用，这样的情况通常是用户知道测量输出结果的概率，此时用户可以使用`Microsoft.Quantum.Primitive`命名空间中的`AssertProb`来表达这一信息，如下所示：

```
operation Teleportation (source : Qubit, target : Qubit) : () {
	body {
		using (ancillaRegister = Qubit[1]) {
			let ancilla = ancillaRegister[0];

			H(ancilla);
			CNOT(ancilla, target);

			CNOT(source, ancilla);
			H(source);

			AssertProb([PauliZ], [source], Zero, 0.5, "Outcomes must be equally likely", 1e-5);
			AssertProb([PauliZ], [ancilla], Zero, 0.5, "Outcomes must be equally likely", 1e-5);

			if (M(source) == One)  { Z(target); X(source); }
			if (M(ancilla) == One) { X(target); X(ancilla); }
		}
	}
}
```

当轨迹模拟器运行`AssertProb`时，它将记录在`source`和`ancilla`上测量`PauliZ`得到`Zero`结果的概率为$$0.5$$，在这之后，当模拟器运行`M`时，它就会查询所记录的输出概率，并且`M`返回`Zero`和`One`的概率为$$0.5$$。当同样的代码在一个跟踪着相同量子状态的模拟器中运行时，这个模拟器会检查`AssertProb`中提供的概率是否正确。

#### 使用量子轨迹模拟器运行程序

量子轨迹模拟器可以被视为另一种目标机器，在这上面所使用的C#驱动程序与其他模拟器类似，如下所示：

```
using Microsoft.Quantum.Simulation.Core;
using Microsoft.Quantum.Simulation.Simulators;
using Microsoft.Quantum.Simulation.Simulators.QCTraceSimulators;

namespace Quantum.MyProgram
{
	class Driver
	{
		static void Main(string[] args)
		{
			QCTraceSimulator sim = new QCTraceSimulator();
			var res = MyQuantumProgram.Run().Result;
			System.Console.WriteLine("Press any key to continue...");
			System.Console.ReadKey();
		}
	}
}
```

注意，如果代码中有至少一个测量没有使用`AssertProb`或`ForceMeasure`标记，模拟器将会抛出`UnconstrainMeasurementException`异常。

除了运行量子程序，轨迹模拟器还有5个组件用于检测程序中的bug和进行量子程序资源估算，它们是：
- 明确的输入检查器（Distinct Inputs Checker）
- 非法的量子比特使用检查器（Invalidated Qubits Use Checker）
- 基本操作计数器（Primitive Operations Counter）
- 量子线路深度计数器（Circuit Depth Counter）
- 量子线路宽度计数器（Circuit Width Counter）

每一个组件都可以在`QCTraceSimulatorConfiguration`中设置合适的标志进行激活。下面的部分我们将对这些部分进行详细介绍。

#### 明确的输入检查器

`Distinct Inputs Checker`是轨迹模拟器的一部分，它用来检测代码中可能存在的bug。考虑以下Q#代码：
```
operation DoBoth( q1 : Qubit, q2 : Qubit, op1 : (Qubit=>()), op2 : (Qubit=>()) ) : () {
	body {
		op1(q1);
		op2(q2);
	}
}
```

当用户看到这段代码时，他们会假定参数中的`op1`和`op2`被调用的顺序是没有影响的，因为`q1`和`q2`是两个不同的量子比特，并且作用于不同量子比特上的操作也是明确的。现在来看下面的例子，例子中使用了上面定义的操作：
```
operation DisctinctQubitCaptured2Test () : () {
	body {
		using( q = Qubit[3] ) {
			let op1 = CNOT(_,q[1]);
			let op2 = CNOT(q[1],_);
			DoBoth( q[0],q[2],op1,op2);
		}
	}
}
```

现在`op1`和`op2`都通过局部应用来获取，并且共享一个量子比特，当用户调用`DoBoth`时，运行结果将依赖于`op1`和`op2`在`DoBoth`中出现的顺序，这绝对不会是用户希望发生的。`Distinct Inputs Checker`能够检测这样的情况并抛出`DisctinctInputsCheckerException`。

##### 在C#程序中使用`Distinct Inputs Checker`

下面的代码展示了如何在C#代码中使用配置了`Distinct Inputs Checker`的轨迹模拟器。
```
using Microsoft.Quantum.Simulation.Core;
using Microsoft.Quantum.Simulation.Simulators;
using Microsoft.Quantum.Simulation.Simulators.QCTraceSimulators;

namespace Quantum.MyProgram
{
	class Driver
	{
		static void Main(string[] args)
		{
			var traceSimCfg = new QCTraceSimulatorConfiguration();
			traceSimCfg.useDistinctInputsChecker = true; //enables distinct inputs checker
			QCTraceSimulator sim = new QCTraceSimulator(traceSimCfg);
			var res = MyQuantumProgram.Run().Result;
			System.Console.WriteLine("Press any key to continue...");
			System.Console.ReadKey();
		}
	}
}
```

上面的代码中，类`QCTraceSimulatorConfiguration`存储轨迹模拟器的配置，并且能够作为`QCTraceSimulator`构造函数的参数。当`useDistinctInputsChecker`被置为`true`时，`Distinct Inputs Checker`将被激活。

##### 非法量子比特使用检测器

`Invalidated Qubits Use Checker`是轨迹模拟器的一部分，它被用来检测代码中潜在的bug。考虑下面的Q#代码：
```
operation UseReleasedQubitTest () : () {
	body {
		mutable q = new Qubit[1];
		using( ans = Qubit[1] ) {
			set q[0]= ans[0];
		}
		H(q[0]);
	}
}
```

在上面的代码中，当`H`被应用到`q[0]`时，它指向了一个已经被释放的量子比特，这将导致未定义行为。当`Invalidated Qubits Use Checker`被激活时，它将抛出`InvalidatedQubitsUseCheckerException`异常。

##### 在C#程序中使用`Invalidated Qubits Use Checker`

下面的代码展示了如何在C#程序中使用配置了`Invalidated Qubits Use Checker`的轨迹模拟器。

```
using Microsoft.Quantum.Simulation.Core;
using Microsoft.Quantum.Simulation.Simulators;
using Microsoft.Quantum.Simulation.Simulators.QCTraceSimulators;

namespace Quantum.MyProgram
{
	class Driver
	{
		static void Main(string[] args)
		{
			var traceSimCfg = new QCTraceSimulatorConfiguration();
			traceSimCfg.useInvalidatedQubitsUseChecker = true; // enables useInvalidatedQubitsUseChecker
			QCTraceSimulator sim = new QCTraceSimulator(traceSimCfg);
			var res = MyQuantumProgram.Run().Result;
			System.Console.WriteLine("Press any key to continue...");
			System.Console.ReadKey();
		}
	}
}
```

上面的代码中，类`QCTraceSimulatorConfiguration`存储轨迹模拟器的配置，并且能够作为`QCTraceSimulator`构造函数的参数。当`useInvalidatedQubitsUseChecker`被置为`true`时，`Invalidated Qubits Use Checker`将被激活。

##### 基本操作计数器（Primitive Operations Counter)

`Primitive Operations Counter`是轨迹模拟器的一部分。它对量子程序中每一个`operation`执行的基本操作进行计数，统计的结果被保存在一个操作调用图的边缘上。现在让我们来看看使用一个`CCNOT`操作需要多少的`T`门。我们首先定义一个名为`CCNOTDriver`的操作：
```
open Microsoft.Quantum.Primitive;
operation CCNOTDriver() : () {
	body {
		using( qubits = Qubit[3] ) {
			CCNOT(qubits[0],qubits[1],qubits[2]);
			T(qubits[0]);
		} 
	}
}
```

为了计算`CCNOT`操作和`CCNOTDriver`操作中到底需要多少个`T`门，我们使用下面的C#代码：
```
// using Microsoft.Quantum.Simulation.Simulators.QCTraceSimulators;
// using System.Diagnostics;
var config = new QCTraceSimulatorConfiguration();
config.usePrimitiveOperationsCounter = true;
var sim = new QCTraceSimulator(config);
var res = CCNOTDriver.Run(sim).Result;

double tCountAll = sim.GetMetric<CCNOTDriver>(PrimitiveOperationsGroupsNames.T);
double tCount = sim.GetMetric<Primitive.CCNOT, CCNOTDriver>(PrimitiveOperationsGroupsNames.T);
```

上面代码中，第一部分运行`CCNOTDriver`，第二部分我们使用`QCTraceSimulator.GetMetric`方法来获得`CCNOTDriver`中运行`T`门的次数。上面程序的输出结果为`CCNOT`执行了7此`T`门，而`CCNOTDriver`执行了8此`T`门。

当使用两个类型参数来调用`GetMetric`时，它返回与给定调用图边缘相关联的数值，在上面的例子中，`Primitive.CCNOT`在`CCNOTDriver`中被调用，因此调用图中包含边缘`<Primitive.CCNOT, CCNOTDriver>`。

如果要获取`CNOT`门被使用的次数，那么我们可以使用以下代码：
```
double cxCount = sim.GetMetric<Primitive.CCNOT, CCNOTDriver>(PrimitiveOperationsGroupsNames.CX);
```

最后，我们可以使用以下代码将所有的统计结果输出为CSV格式：
```
double cxCount = sim.GetMetric<Primitive.CCNOT, CCNOTDriver>(PrimitiveOperationsGroupsNames.CX);
```

##### 深度计数器（Depth Counter）

深度计数器是轨迹模拟器的一部分，它用来采集量子程序中调用的每一个操作的深度。默认情况下，`T`门的深度为1，其他所有门的深度为0，这也意味着默认情况下只有`T`门的深度才会被计算，同时用户也可以自行设定每一个基本操作的深度。下面的Q#和C#代码用来演示深度计数器。

```
open Microsoft.Quantum.Primitive;
operation CCNOTDriver() : () {
	body {
		using( qubits = Qubit[3] ) {
			CCNOT(qubits[0],qubits[1],qubits[2]);
			T(qubits[0]);
		} 
	}
}
```

下面是相应的C#驱动代码：
```
using Microsoft.Quantum.Simulation.Simulators.QCTraceSimulators;
using System.Diagnostics;
var config = new QCTraceSimulatorConfiguration();
config.useDepthCounter = true;
var sim = new QCTraceSimulator(config);
var res = CCNOTDriver.Run(sim).Result;

double tDepth = sim.GetMetric<Primitive.CCNOT, CCNOTDriver>(DepthCounter.Metrics.Depth);
double tDepthAll = sim.GetMetric<CCNOTDriver>(DepthCounter.Metrics.Depth);
```

上面程序中第一部分运行了`CCNOTDriver`操作，第二部分我们使用`QCTraceSimulator.GetMetric`方法来获取`CCNOT`和`CCNOTDriver`中`T`的深度：
```
double tDepth = sim.GetMetric<Primitive.CCNOT, CCNOTDriver>(DepthCounter.Metrics.Depth);
double tDepthAll = sim.GetMetric<CCNOTDriver>(DepthCounter.Metrics.Depth);
```

最后，我们可以使用下面的代码将`Depth Counter`采集的数据用CSV的格式输出：
```
string csvSummary = sim.ToCSV()[MetricCalculatorsNames.depthCounter];
```

##### 宽度计数器（Width Counter）

宽度计数器对每个操作中分配的和借用的量子比特进行计数，一些操作还会分配额外的量子比特，例如受控的`X`门和受控的`T`门。下面我们来统计实现一个多量子受控`X`门所需要额外分配的量子比特的数目。

```
open Microsoft.Quantum.Primitive;
operation MultiControlledXDriver( numberOfQubits : Int ) : () {
	body {
		using( qubits = Qubit[numberOfQubits] ) {
			(Controlled X)(qubits[ 1 .. numberOfQubits - 1] ,qubits[0]);
		} 
	}
}
```

下面是用来进行量子比特数量统计的C#代码，它展示了一个作用域5个量子比特的多量子受控`X`门还需要分配2个辅助量子比特，并且它的输入宽度为5.

```
var config = new QCTraceSimulatorConfiguration();
config.useWidthCounter = true;
var sim = new QCTraceSimulator(config);
int totalNumberOfQubits = 5;
var res = MultiControlledXDriver.Run(sim, totalNumberOfQubits).Result;

double allocatedQubits = 
sim.GetMetric<Primitive.X, MultiControlledXDriver>(
		WidthCounter.Metrics.ExtraWidth,
		functor: OperationFunctor.Controlled); 

double inputWidth =
sim.GetMetric<Primitive.X, MultiControlledXDriver>(
		WidthCounter.Metrics.InputWidth,
		functor: OperationFunctor.Controlled);
```

上面程序中第一部分运行了`MultiControlledXDriver`，第二部分使用`QCTraceSimulator.GetMetric`方法來获取受控的`X`门操作中分配的量子比特数和输入其中的量子数量。最后，我们可以使用以下代码得到CSV格式的输出结果：

```
string csvSummary = sim.ToCSV()[MetricCalculatorsNames.widthCounter];
```
