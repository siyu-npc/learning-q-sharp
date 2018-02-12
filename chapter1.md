# 第一章  快速入门

微软的Q\#语言是专门为量子编程打造的一款利器，它包含了开发量子程序所需要的各种工具。如果你有一定的Visual Studio使用经验，那么无论你是初学者还是经验丰富的研究人员，Q\#语言都能让你快速、高效地开发量子计算程序。

在这一章中，我们将学习如何安装和搭建Q\#语言的开发环境，并通过一个具体的程序来展示Q\#如何实现一个量子程序以及Q\#语言的一些结构和特点，同时也会讲解Q\#语言如何模拟量子计算机中的量子门完成一些量子计算任务，比如量子叠加、量子纠缠等。

## 1.1 安装Q\#开发环境

#### 1.1.1 安装前的准备

* Q\#语言的模拟器应用了高级矢量扩展指令集（AVX），Intel系列CPU中 `Sandy Bridge` 架构和之后的CPU支持这种指令集，因此要使用Q\#语言请确保你的CPU支持AVX指令集，微软也将在未来推出适用于早期CPU架构的Q\#语言开发工具。
* Q\#语言必须运行在64为Windows操作系统上。
* 使用Visual Studio 2017进行开发

如果没有安装Visual Studio 2017，可以通过以下步骤安装免费的 Community版Visual Studio 2017：

1. 访问[Visual Studio下载页面](https://www.visualstudio.com/downloads/) ，并选择Community版本进行下载（如果有钱也可以下载另外的版本）
2. 下载完毕后双击安装文件
3. **注意**：安装开始前程序会让你选择特定的开发环境和工具，记得一定要选上 `Universal Windows Platform development`** 和 **`.NET desktop development`
4. 在选择好需要的环境和工具后点击“安装”即可。

#### 1.1.2 构建Q\#语言开发环境

使用Q\#语言需要安装Q\#语言的开发工具包，安装过程也是很简单的。

1. 访问[微软Q\#语言主页](https://www.microsoft.com/en-us/quantum/development-kit)，点击左上方的 "Download Now" 按钮，此时页面将跳转至Q\#开发工具包的下载界面
2. 在下载界面的右侧，填写一些必要的信息，包括你的名字、联系方式等等，之后点击右下角的 "Download Now" 按钮，就会开始下载开发工具包。工具包是一个vsix格式的文件，大小只有1M左右。
3. 双击下载好的文件，稍等片刻Q\#语言的开发工具就安装在了Visual Studio2017中。

#### 1.1.3 验证刚刚安装的开发环境

我们使用微软为我们提供的一些例子和库对刚刚安装的环境进行检测，以验证我们是否正确安装好了Q\#语言的开发环境。

1. 克隆GitHub上微软官方提供的[Q\#语言例程](https://github.com/microsoft/quantum)
2. 使用Visual Studio打开 `QsharpLibraries.sln`  
   1. 如果此时弹出 "**Install Missing Features" **的面板，点击“安装”

3. 选择 Teleport 示例程序并运行，如果出现了类似下面这样的输出，那么说明我们的Q\#语言开发环境安装正确，你可以开始在量子世界里的挑战了！

```
        Round 0:        Sent True,      got True. 
        Teleportation successful!!
        Round 1:        Sent False,     got False. 
        Teleportation successful!!
        ...
        Round 6:        Sent True,      got True. 
        Teleportation successful!!
        Round 7:        Sent False,     got False. 
        Teleportation successful!!
```

**如果出现了与NuGet有关的错误，使用**[**NuGet package restore**](https://docs.microsoft.com/en-us/nuget/consume-packages/package-restore)**中的方法重置安装包即可.**

### 1.2 第一个Q\#语言程序

这一节的主要内容包括：

* 在Visual Studio中设置Q\#语言工程和解决方案
* Q\#语言中一个`operation`的组成
* 如何从C\#中调用Q\#语言的 `operation`
* 如何构建和运行Q\#程序

#### 1.2.1 在Q\#中创建一个贝尔态（Bell State）

现在，我们已经安装好了Q\#语言的开发环境。作为开始，我们将构建一个非常简单的程序来展示量子叠加和量子纠缠。在这个程序中，我们以一个初始态为`|0>`的量子比特为起点，对其进行一些操作，并测量其状态。

##### 1. 创建工程和解决方案

打开Visual Studio 2017，依次点击“文件--&gt;新建--&gt;项目”，在出现的新建项目对话框中，选中左侧栏目中的“`Visual C#`”，此时在对话框中部会出现许多条目，找到“`Q# Application`”并选中，设置项目名称为`Bell`，如下图所示。如果没有找到“`Q# Application`”的条目，检查对话框上部列表选择框中是否选中了“`.NET Framework 4.6.1`”。![](/Image/NewQSharpProject.png)

##### 2. 输入Q\#代码

在完成工程创建后，此时会有两个文件：`Driver.cs`和`Operation.qs`，前者是驱动Q\#程序的C\#代码，后者才是真正的Q\#代码文件。

首先将`Operation.qs`文件重命名为`Bell.qs`，Visual Studio在创建工程时会自动生成一部分代码，此时`Bell.qs`中的内容类似于这样：

```
namespace Quantum.Bell 
{
    open Microsoft.Quantum.Primitive;    
    open Microsoft.Quantum.Canon;

    operation Operation () : ()    
    {
        body
        {
        }
    }
}
```

然后将代码中的`Operation`改为`Set`，并且将其后面第一个括号中的内容改为`desired: Result, q1:Qubit`，此时`Bell.qs`中的内容应为：

```
namespace Quantum.Bell 
{
    open Microsoft.Quantum.Primitive;    
    open Microsoft.Quantum.Canon;

    operation Set (desired: Result, q1:Qubit) : ()    
    {
        body
        {
        }
    }
}
```

现在，将以下代码键入`body`后的大括号中：

```
let current = M(q1);

if (desired != current)
{
    X(q1);
}
```

`Bell.qs`中的代码为：

```
namespace Quantum.Bell 
{
    open Microsoft.Quantum.Primitive;    
    open Microsoft.Quantum.Canon;

    operation Set (desired: Result, q1:Qubit) : ()    
    {
        body
        {
            //测量量子比特状态
            let current = M(q1);
            if (desired != current)
            {
                //翻转
                x(q1);
            }
        }
    }
}
```

以上代码（`operation`）所做的工作就是设置一个已知状态的量子位。我们首先测量量子比特的状态`M(q1)`，如果与我们所期待的状态相符，那么不在进行进一步的操作，否则使用X门将此量子比特翻转。

现在我们来解释一下上面的代码。上面的代码中定义了一个Q\#语言中的 `operation`，`operation`是Q\#语言中一个基本的执行单元，它与其他编程语言如C/C++、Python中的函数，C\#和Java中的`static`函数是一样的。一个`operation`的参数是一个元祖，在`operation`名字之后的括号中指定各个参数之间用逗号分隔，`operation`的返回值与其参数形式类似，在之后的括号中指定，两个括号之间由一个冒号分隔，Q\#语言中，`operation`可以指定多个返回值，也可以将括号留空，此时该`operation`不返回任何数据，在这种情况下，一对空的括号“\(\)”就相当于C语言中的void或F\#语言中的`unit`。例如上面代码中，定义了一个名为Set的`operation`，它接受两个参数，没有返回值。一个`operation`中包含一个`body`段，在`body`段中的代码就是该`operation`功能的实现代码，`operation`中还可以包含共轭、伴随以及共轭伴随等部分，关于这些操作的详细内容，我们将在以后进行介绍。

有了上面的`operation`，我们现在来编写一个测试程序来调用这个operation并观察结果。在`Bell.qs`中添加如下代码，可以看到这也是一个`operation`，将其添加在`Set`之后。

```
operation BellTest (count : Int, initial : Result) : (Int, Int)
{
  body
  {
      mutable numOnes = 0;
      using (qubits = Qubit[1])
      {
          for (test in 1..count)
          {
              Set (initial, qubits[0]);
              let res = M (qubits[0]);
              // Count the number of ones we saw:
              if (res == One)
              {
                  set numOnes = numOnes + 1;
              }
          }
          Set(Zero, qubits[0]);
      }
      // Return number of times we saw a |0> and number of times we saw a |1>
      return (count-numOnes, numOnes);
}
```

上面的`operation BellTest`中定义了一个循环`count`次的控制结构，在每次循环中，首先将一个量子比特设置为`initial`，然后测量这个量子的状态，如果为1，将变量`numOnes`加一，所有循环结束后，我们将得到在这个过程中共有多少次测量的量子比特状态为1，多少次为0。在最后，我们把量子比特重新设置为了0，使其处于一个特定的状态。

从上面的代码中可以看出，Q\#语言使用了与C\#相似的分好、括号等来标示程序的结构，并且Q\#也拥有与C\#类似的`if`控制语句。

Q\#中只有一种方法来处理变量。默认情况下，Q\#中的变量是不可改变的，当变量已被绑定时，它们的值就不会再改变。Q\#中，关键字let用来只是绑定一个不可变的变量，同时，`operation`中的参数也是不可变的。

如果需要使用值可变的变量就需要像上面的程序中那样，使用`mutable`关键字对变量进行声明，经过`mutable`关键字声明的变量，其可以使用`set`语句重新赋值。

Q\#不是一种强类型的编程语言。在定义一个新的变量时，变量的类型由编译器根据其具体值进行推断。

`using`语句也是Q\#中特有的，它用于在一个代码段中开辟一个量子比特数组。在Q\#中，所有的量子比特都是动态分配和释放的，`using`语句在代码段的起始处分配量子比特，在代码段结束后自动释放这些量子比特。

Q\#中的`for`循环在一个范围之内进行迭代，它不同于C语言风格的for循环，当然在Q\#中实际上也不存在类似C语言中的`for`循环结构。`for`循环中， 其范围可以直接由起始和终止的值来指定，两个值之间使用`..`连接。例如`1..10`就表示1,2,3,4,5,6,7,8,9,10，默认步长为1，如果需要不同的步长值，则直接指定即可，其语法为`1..2..10`，这个范围表示的数值就是1,3,5,7,9。需要注意的是，Q\#中的范围是一个闭区间，也就是包含起始值和终止值。

Q\#中使用元组来传递多个变量，上面的`operation BellTest`的返回值就是由两个`Int`数据`(Int, Int)`组成的元组。

##### 3. C\#驱动代码

将目光转换到`Driver.cs`文件中，在创建项目时，Visual Studio会自动生成一部分C\#代码，内容如下：

```
using Microsoft.Quantum.Simulation.Core;
using Microsoft.Quantum.Simulation.Simulators;

namespace Quantum.Bell
{
    class Driver
    {
        static void Main(string[] args)
        {

        }
    }
}
```

在`Main`方法中，我们输入以下代码：

```
using (var sim = new QuantumSimulator())
{
    // Try initial values
    Result[] initials = new Result[] { Result.Zero, Result.One };
    foreach (Result initial in initials)
    {
        var res = BellTest.Run(sim, 1000, initial).Result;
        var (numZeros, numOnes) = res;
        System.Console.WriteLine($"Init:{initial,-4} 0s={numZeros,-4} 1s={numOnes,-4}");
    }
}
System.Console.WriteLine("Press any key to continue...");
System.Console.ReadKey();
```

上面的C\#代码包含4个部分：

* 构造一个量子模拟器，即代码中的`sim`
* 计算量子算法需要的参数，上面的代码中，BellTest需要的`count`为1000，`initial`是量子比特需要的初始值
* 运行量子算法程序。每一个`Q# operation`都会生成一个名字相同的C\#类，这个类拥有一个`run`方法异步地执行相应的操作，该方法之所以是异步操作，原因在于在实际的硬件设备上相应的操作是异步的，这也造成了另一个结果，那就是我们要获取`operation`的返回值时将会阻塞程序直到程序运行完成并返回结果。
* 处理`operation`返回的结果。在上面的程序中，`res`接受`operation`产生的结果。

##### 4. 构建和运行

只需点击F5键，程序将会被自动构建和运行。稍后片刻，将会得到如下的结果：

```
Init:Zero 0s=1000 1s=0
Init:One  0s=0    1s=1000
Press any key to continue...
```

##### 5. 创建量子叠加

现在有了上面的结果，是不是迫不及待想要操作一下量子比特了？首先，我们尝试在测量量子比特之前先对其进行翻转，这个操作可以使用`X`门方便地进行，将下面的代码输入`BellTest`中：

```
X(qubits[0]);
let res = M (qubits[0]);
```

此时`BellTest`代码如下：

```
operation BellTest (count : Int, initial : Result) : (Int, Int)
{
  body
  {
      mutable numOnes = 0;
      using (qubits = Qubit[1])
      {
          for (test in 1..count)
          {
              Set (initial, qubits[0]);
              X(qubits[0]);
              let res = M (qubits[0]);
              // Count the number of ones we saw:
              if (res == One)
              {
                  set numOnes = numOnes + 1;
              }
          }
          Set(Zero, qubits[0]);
      }
      // Return number of times we saw a |0> and number of times we saw a |1>
      return (count-numOnes, numOnes);
}
```

现在重新运行程序，得到下面的输出结果：

```
Init:Zero 0s=0    1s=1000
Init:One  0s=1000 1s=0
```

目前为止，我们通过上面的这些程序得到了一些结果，但这些结果都是经典的而非量子的，要想获得量子的结果，只需要将上面的X门替换为`H`门（Hadamard Gate），相比于之前`X`门对量子比特由0直接翻转为1或由1直接翻转为0，`H`门对量子比特进行半翻转。代码中相应的部分为：

```
H(qubits[0]);
let res = M (qubits[0]);
```

现在我们得到的结果如下：

```
Init:Zero 0s=484  1s=516
Init:One  0s=522  1s=478
```

在每次对一个量子比特的测量中，我们希望得到一个经典的值，但量子比特的值处于0和1之间，所以从统计意义来讲，我们有一半概率得到0一半概率得到1，这种现象就是量子叠加，上面的代码和结果也给了我们对于量子状态的初步印象。

##### 6. 创建量子纠缠

现在我们回到本程序最初要产生的`Bell State`上并创建量子纠缠。首先我们要做的就是分配两个量子比特而非当前程序中的1个。

```
using (qubits = Qubit[2])
```

两个量子比特能够使我们在测量量子状态前应用一个新的`CNOT`门：

```
Set (initial, qubits[0]);
Set (Zero, qubits[1]);

H(qubits[0]);
CNOT(qubits[0],qubits[1]);
let res = M (qubits[0]);
```

在上面的代码中，我们新增了一个`Set operation`将第一个量子比特初始化为`Zero`。

同样我们也需要在第二个量子比特被释放之前将其重置为`Zero`：

```
Set(Zero, qubits[1]);
```

现在`BellTest`代码如下所示：

```
operation BellTest (count : Int, initial: Result) : (Int,Int)
{
    body
    {
        mutable numOnes = 0;
        using (qubits = Qubit[2])
        {
            for (test in 1..count)
            {
                Set (initial, qubits[0]);
                Set (Zero, qubits[1]);

                H(qubits[0]);
                CNOT(qubits[0],qubits[1]);
                let res = M (qubits[0]);

                // Count the number of ones we saw:
                if (res == One)
                {
                    set numOnes = numOnes + 1;
                }
            }
            Set(Zero, qubits[0]);
            Set(Zero, qubits[1]);
        }
        // Return number of times we saw a |0> and number of times we saw a |1>
        return (count-numOnes, numOnes);
    }
}
```

这份代码将产生与之前代码同样的结果，然而，这里我们真正感兴趣的是在第一个量子比特被测量后，第二个量子比特将会有何反应。为此，我们对`BellTest`进行修改，向其中加入统计相关的代码：

```
    operation BellTest (count : Int, initial: Result) : (Int,Int,Int)
    {
        body
        {
            mutable numOnes = 0;
            mutable agree = 0;
            using (qubits = Qubit[2])
            {
                for (test in 1..count)
                {
                    Set (initial, qubits[0]);
                    Set (Zero, qubits[1]);

                    H(qubits[0]);
                    CNOT(qubits[0],qubits[1]);
                    let res = M (qubits[0]);

                    if (M (qubits[1]) == res) 
                    {
                        set agree = agree + 1;
                    }

                    // Count the number of ones we saw:
                    if (res == One)
                    {
                        set numOnes = numOnes + 1;
                    }
                }
            Set(Zero, qubits[0]);
            Set(Zero, qubits[1]);
            }
            // Return number of times we saw a |0> and number of times we saw a |1>
            return (count-numOnes, numOnes, agree);
        }
    }
```

在上面代码中，我们增加了一个新的返回值`agree`，他将记录第一个量子比特的测量值与第二个量子比特的测量值相等的次数。

我们相应的修改`Driver.cs`代码：

```
            using (var sim = new QuantumSimulator())
            {
                // Try initial values
                Result[] initials = new Result[] { Result.Zero, Result.One };
                foreach (Result initial in initials)
                {
                    var res = BellTest.Run(sim, 1000, initial).Result;
                    var (numZeros, numOnes, agree) = res;
                    System.Console.WriteLine(
                        $"Init:{initial,-4} 0s={numZeros,-4} 1s={numOnes,-4} agree={agree,-4}");
                }
            }
            System.Console.WriteLine("Press any key to continue...");
            System.Console.ReadKey();
```

现在我们来看一看运行结果，非常令人惊讶：

```
Init:Zero 0s=499  1s=501  agree=1000
Init:One  0s=490  1s=510  agree=1000
```

从结果看，对于第一个量子比特而言，统计结果并没有变换（50%的概率为1,50%的概率为0），但是我们对第二个量子比特的测量值总是与第一个量子比特的测量值保持一致。`CNOT`门是两个量子比特产生量子纠缠，所以两个量子比特中任意一个发生变化，另一个也将发生相同的变化，改变两个量子比特的测量顺序所得到的结果也同样如此。也就是说，第一次测量值是随机的，而第二次测量值与第一次相同，同进同退。

