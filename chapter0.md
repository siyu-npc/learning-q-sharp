# 第0章 量子计算中的常用概念

### 什么是量子计算

在过去的几年中，出现了一系列新的计算机技术，其中量子计算可能是最需要开发者转变开发模式的技术。量子计算机是在上世纪80年代，由Richard Feynman和Yuri Manin提出的，但谈到量子计算，人么的印象往往来源于被视为物理学上的一个最大的尴尬：非凡的科学进展却对简单的系统无能为力。如你所见，早在1900~1925年间，量子力学的机理就已经被建立起来了，它是化学、凝聚态物理和从计算机芯片到LED照明再到其他技术的基石。尽管取得了很大的成就，但是利用量子力学去对一些非常简单的系统进行建模时仍然超出了人类的能力水平，这是因为即使是模拟几十个例子相互作用的系统，其所需要的计算能力也超过当今任何计算机持续计算数千年以上。

有多种方法可以理解量子力学为何难以模拟，最简单的方法就是将量子力学看做另一个问题。在量子水平上，同时存在着一系列可能的不同架构（也叫状态，量子态），不同于经典的概率论，这些可能被潜在观察到的状态可能会像水池中的波纹一样相互干扰，这种干扰阻碍了使用统计抽样来获取量子态的架构，并且，如果我们想要理解量子演进，那就必须追踪一个量子系统中所有的可能状态。

设想一个由电子组成的系统，其中电子可以出现在40个位置中的任何一个上，那么这个系统就存在$$2 ^ {40}$$种状态（每个位置上可能有也可能没有电子存在），在一个传统的计算机存储器上存储这些状态就需要130Gb的空间，对于如此大的存储当今的一些计算机还是可以满足的，但是如果将40变为41，多了一个位置造成的后果是存储空间要翻倍。如果我们继续按照常规办法这么做下去，这个增加位置的游戏马上就会进入死局，因为世界上最强大的存储设备也无法满足这样的指数增长。将电子数量扩大到数百个，那么所需要的内存甚至超过了宇宙中例子的数量，因此使用传统的计算机是无法模拟量子力学模型的。

这一问题引导着那修对量子计算有着早期愿景的人们思考这样一个问题：我们能否将这一挑战转变为一个机遇。量子力学难以模拟，那如果我们利用量子特性来建造硬件会发生什么呢？我们能否直接利用量子力学的机理来模拟相互作用的量子系统呢？我们是否可以研究在自然中不存在但遵循量子力学机制的任务呢？正事这些问题催生了量子计算。

量子计算的基础核心是是将信息存储在物质的量子态中，并使用量子门通过操控和学习给量子干涉编制程序来计算出信息。一个早期的利用对量子干涉编程来解决问题的例子是由 Peter Shor在1994年完成的，这是一个因式分解问题，对一个较大的数进行因式分解对传统计算机来说是困难的，解决因式分解问题能是现今电子商务安全基础上的许多公钥密码体制失效，例如RSA和ECDLP，到了那个时候，快速高效的量子算法被开发出来，能为我们解决许多经典计算机难以解决的难题：模拟化学、物理和材料科学中的物理系统，搜索无序数据库，求解线性方程组和机器学习等。

设计一个利用量子干涉的程序听起来像是一个艰巨的挑战，事实也确实如此，但目前已经有许多技术和工具包括微软的量子开发工具包被引入使量子程序和算法的开发更便捷。现实中有一些基本的对计算有用的策略可以被用来操控量子干涉，同时不会造成量子纠缠中的解丢失。量子编程是一种与经典编程截然不同的艺术，它需要用完全不同的工具去理解和表达量子编程思想，实际上，如果没有一般的工具来帮助开发者解决量子编程中的问题，量子程序的开发是一件不那么容易的事。

我们推出了微软量子程序开发工具包来推动正在成长的量子编程社区的发展，工具包中包含了为他们的任务、问题和解决方案解锁量子革命的工具。我们的高级程序语言Q\#旨在解决量子信息处理的挑战，它集成在一个软件栈中，使得量子算法能被编译成量子计算机的基本操作。在走进Q\#之前，回顾一下量子计算的基本原则是很有帮助的，在本书中，我们将把量子计算的基本规则作为公理，而不会详述量子力学的基础，并且我们假定读者有一定的线性代数基础（向量、矩阵等），如果你需要更深入地了解量子计算的历史和原理，请参考[https://docs.microsoft.com/zh-cn/quantum/quantum-formoreinfo?view=qsharp-preview。](https://docs.microsoft.com/zh-cn/quantum/quantum-formoreinfo?view=qsharp-preview。)

### 量子位

就像比特是经典计算中信息的基础对象，量子位是量子计算中信息的基础对象。为了理解这个对应关系，我们先来看最简单的单个量子位的情况。

#### 量子位的表示

一个比特的值可以是0或1，而一个量子位的值可以是0或1，也可以是这两个值的量子叠加，一个量子位的状态可以用一个二维的单位向量来描述，这个向量就叫做**量子态向量**，它保存了要描述一个单量子位量子系统状态所需要的全部信息。

任意一个范数为1的二维向量（无论是实数还是复数）都能表示一个量子位的状态，所以以下向量都属于量子态向量。

![](/Image/QuantumStateVector.png)

在所有量子态向量中，$$\begin{bmatrix} 1 \\  0 \end{bmatrix}$$和$$\begin{bmatrix} 0 \\  1 \end{bmatrix}$$ 有着特设的作用，这两个向量构成了描述量子状态的向量空间的基，这意味着任何一个量子态向量都可以用这两个基的和来表示，例如，向量$$\begin{bmatrix} x \\  y \end{bmatrix}$$ 可以写做$$x \begin{bmatrix} 1 \\ 0 \end{bmatrix} + y \begin{bmatrix} 0 \\  1 \end{bmatrix}$$ ，因此，这两个向量又被成为**计算基底**。

同时，我们将上述两个基所代表的量子态与经典计算中的比特位相对应，它们之间的转换关系为：

$$0\equiv \begin{bmatrix} 1 \\ 0 \end{bmatrix}\qquad 1 \equiv \begin{bmatrix} 0 \\ 1 \end{bmatrix}$$

#### 量子态的测量

我们已经知道了如何表示一个量子位，那么一个量子位的状态到底代表了什么呢？现在我们通过探讨量子态测量的概念来获取一个对量子态的认识。量子态的测量就是对量子状态的观察，根据量子力学的原理，观察一个量子会使其状态崩塌至两个基本状态$$\begin{bmatrix} 1 \\  0 \end{bmatrix}$$和$$\begin{bmatrix} 0 \\  1 \end{bmatrix}$$中的一个，当一个状态为$$\begin{bmatrix} \alpha \\  \beta \end{bmatrix}$$的量子被测量后，我们得到$$0$$的概率为$$\|\alpha\|^2$$，得到$$1$$的概率为$$\|\beta\|^2$$，并且$$\|\alpha\|^2 + \|\beta\|^2 = 1$$ 。根据上述量子态测量的性质，我们可以知道一个量子态矩阵与其符号是不相关的，这个向量与其反向量等价：$$\alpha \rightarrow -\alpha$$ , $$\beta \rightarrow -\beta$$ 。这是因为测量到$$0$$和$$1$$的概率取决于$$\alpha$$和$$\beta$$的平法的大小，在他们之前插入一个符号并不改变测量结果的概率分布。

有关测量的最后一个重要性质是测量并不会使所有量子态都发生改变。如果我们测量一个状态为$$\begin{bmatrix} 1 \\  0 \end{bmatrix}$$的量子，对应于经典计算中的比特$$0$$，那么我们得到的测量结果仍然为$$0$$，且量子本身的状态也不会改变。从这个意义上讲，如果我们只拥有经典的比特，那么我们对这些比特位进行测量也不会改变其状态，这也就意味着我们可以将经典计算的数据复制到量子计算机上，然后像在经典计算机上那样对其进行操作。

### 利用布罗兹球面可视化量子态和其转换

量子位也可以使用布罗兹球面3D化显示。布罗兹球面用三维的实值向量来描述单量子的量子态，如上所述，单个量子的状态由一个二维实值向量来描述。布罗兹球面使得一个量子的状态可视化，正因此，它在我们理解多量子状态时也有极大的作用。可视化的布罗兹球面如下所示：

![](/Image/BlochSphere.png)  
    图0.1 布罗兹球面

图中的箭头指示了量子态向量的方向，箭头的每一次变换都可以看作是一个基本轴的旋转，那么自然而然我们可以将将量子态的变换看做是一系列的旋转变换，当然使用这种思想来设和描述量子算法也是一个不小的挑战，但Q\#为我们提供了能够方便地描述这些旋转操作的语言，从而简化了我们解决问题的负担。

### 单个量子位的操作

量子计算机通过应用一组能模拟量子态向量旋转的量子门来处理数据，量子门的概念与传统计算机中的门的概念类似，如果每一个输入比特的变换都可以用有限长度电路来执行，则门集被认为是通用的。

在量子计算中，我们能够在量子位上执行的合法的变换包括单一变化和测量。共轭操作也叫复共轭转置，他对量子计算有着至关重要的作用，因为在量子反转中需要用到它。Q\#通过自动将门序列编译到它们的共轭矩阵的方式来实现这一功能，将开发者从手工编码的泥潭中解放出来，大大减轻了开发者的负担。

在经典计算中，只有四种操作将一个比特映射到另一个比特（与、或、非、异或），但在量子计算机中，变换一个量子比特状态的操作是无穷的，因此在量子计算中，不存在一个有限的基本操作集合（门集合）能完全覆盖所允许的无穷的幺正变换，这也意味着，量子计算机不可能像经典计算机那样使用有限的门操作来实现所有的量子算法，因此量子计算机不会像经典计算机一样通用。当我们谈到一个门集合对量子计算是通用的时候，我们实际要表达的意思要比完全通用弱化。对于通用性，我们要求量子计算机只在一个有限的误差内使用一个有限长的门序列来逼近某一幺正矩阵，或者说，只要任意的幺正变换在误差允许范围内能够写作一个有限长的门操作序列的乘积，那么这个门集合就是通用的，如下所示：

$$G_N G_{N-1} \cdots G_2 G_1 \approx U.$$

注意因为矩阵乘法是从右向左计算的，因此上面公式中的$$G_{N}$$实际上是最后一个应用到的门操作。更正规地说，对于误差$$\epsilon > 0$$，存在$$G_1,\ldots, G_N$$使得$$G_N\ldots G_1$$ 和 $$U$$的误差不超过$$\epsilon$$，那么我们就说集合$$G_1,\ldots, G_N$$是通用的。

现实中，这样的通用门集合是什么样的呢？对单量子门而言，这样的集合中只包含两个门：Hadamard门（H门）和T门（也叫做$$\pi/8$$门）：  
$$H=\frac{1}{\sqrt{2}}\begin{bmatrix} 1 ; 1 \\  1  ;-1  \end{bmatrix},\qquad T=\begin{bmatrix} 1  ; 0 \\  0  ; e^{i\pi/4} \end{bmatrix}.$$

但是考虑到量子就做的现实原因，一个更大的集合能够带来更多的便利，集合中其他的门可以有H和T门生成。我们将量子门分为两类：Clifford门和T门，这样分类是因为Clifford门在许多量子纠错方案中实现起来很方便，就操作和量子比特而言，他们需要很少的资源就能实现较好的容错率，然而非Clifford门的消耗就非常大了。在Q\#中，标准的单量子Clifford门包括：  
$$H=\frac{1}{\sqrt{2}}\begin{bmatrix} 1 ; 1 \\  1  ;-1  \end{bmatrix} ,\qquad S =\begin{bmatrix} 1  ; 0 \\  0  ; i \end{bmatrix}= T^2,\qquad X=\begin{bmatrix} 0  ;1 \\  1 ; 0 \end{bmatrix}= HT^4H,$$$$Y = \begin{bmatrix} 0 ; -i \\  i ; 0 \end{bmatrix}=T^2HT^4  HT^6, \qquad Z=\begin{bmatrix}1;0\\ 0;-1 \end{bmatrix}=T^4.$$

这些门中，X、Y和Z应用的非常广泛，它们也被成为Pauli操作符，这些门操作和非Clifford门一起就能组合出任意幺正变换作用与单个量子上。下面的例子展示了如何用这些基本操作构建一个幺正变换，图0.1中的三个变换对应于以下门操作序列：  
$$\begin{bmatrix} 1 \\  0 \end{bmatrix} \mapsto HZH \begin{bmatrix} 1 \\  0 \end{bmatrix} = \begin{bmatrix} 0 \\  1 \end{bmatrix}$$

前面的门操作在堆栈逻辑级别上构成了最基本的原语（逻辑级别等同与量子算法级别），但在算法级别较少地考虑基本操作会带来更大的便利，比如多使用近似函数级别的操作。幸运的是Q\#中包含有很多用于实现高层次幺正变换的方法，有了他们我们在实现更高层次的算法时就不需要将其分解为基本的Clifford和T门。

最简单的原语是单量子旋转操作。将三个单量子旋转用$$R_{x}$$、$$R_{y}$$和$$R_{z}$$表示，为了可视化旋转操作$$R_x(\theta)$$的行为，将右手大拇指指向布罗兹球面$$x$$轴的方向，然后右手旋转$$\theta/2$$弧度，相应的幺正变换为：  
$$R_z(\theta) = \begin{bmatrix} e^{-i\theta/2} ; 0\\  0 ; e^{i\theta/2} \end{bmatrix},\qquad R_x(\theta) = H R_z(\theta) H, \qquad R_y(\theta) = SHR_z(\theta)HS^\dagger.$$

正如将任意三个旋转操作组合起来就能实现完成三维空间中的任意旋转，布罗兹球面所表示的任意幺正矩阵也能写成由三个旋转操作组成的序列，特别地，对每一个幺正矩阵$$U$$都有$$\alpha,\beta,\gamma,\delta$$使得$$U= e^{i\alpha} R_x(\beta)R_z(\gamma)R_x(\delta)$$。因此$$R_x(\theta)$$和H门也可以构成一个通用的量子门集合，当然了，因为$$\theta$$可以去任意值，因此构成的量子门集合就不是离散的了，并且考虑到量子模拟的应用场景，连续的量子门对量子计算是至关重要的，特别是在量子算法的设计层面上。最终，这些操作会被编译成离散的满足误差要求的量子门序列实现这些旋转操作。

### 多量子比特

虽然单量子比特拥有一些反直观的特性，例如在某一时刻同时存在多种状态，但是如果一个量子计算机中只有单量子比特门，那它所能提供的运算能力甚至不如现在的一台小小的计算器，更不用说超级计算机了。只有增加量子数，量子计算的真正能力才能体现出来，这种能力的增长，部分原因来自与量子态向量空间的维数随量子数的增加而指数上涨。这也意味着单量子系统建模比较容易，而对50个量子的模拟就会对目前的超级计算机造成压力，每增加一个量子比特，所需要的存储空间和计算时间就会翻倍。

为什么量子态向量会成指数级的增长？这一节的目标就是在单量子态之外重新审视构建多量子态的规则，同时我们也会探讨要构建一台通用的量子计算机，需要那些门操作。Q\#中提供了一些工具是我们在理解多量子门的过程中绝对需要的，它们也会帮助我们理解为什么应用了量子效应如量子纠缠和量子干涉就能使量子计算机比传统计算机强大那么多。

#### 双量子比特的表示

单量子比特与双量子比特之间的主要区别在于双量子态向量是4维而单量子态向量是2维，这是因为双量子态的计算基底是通过单量子态的张量乘积得到的，如下图所示：  
$$\begin{align}
00 \equiv \begin{bmatrix}1 \\ 0 \end{bmatrix}\otimes \begin{bmatrix}1 \\ 0 \end{bmatrix} ;= \begin{bmatrix}1 \\ 0\\ 0\\ 0 \end{bmatrix},\qquad 01 \equiv \begin{bmatrix}1 \\ 0 \end{bmatrix}\otimes \begin{bmatrix}0 \\ 1 \end{bmatrix} = \begin{bmatrix}0 \\ 1\\ 0\\ 0 \end{bmatrix},\\
       10 \equiv \begin{bmatrix}0 \\ 1 \end{bmatrix}\otimes \begin{bmatrix}1 \\ 0 \end{bmatrix} ;= \begin{bmatrix}0 \\ 0\\ 1\\ 0 \end{bmatrix},\qquad 11 \equiv \begin{bmatrix}0 \\ 1 \end{bmatrix}\otimes \begin{bmatrix}0 \\ 1 \end{bmatrix} = \begin{bmatrix}0 \\ 0\\ 0\\ 1 \end{bmatrix}.
       \end{align}$$

容易知道，$$n$$个量子的量子态向量可以通过$$2^{n}$$维的单位向量用同样的方法进行构造。就像单量子比特一样，双量子比特的量子态向量：  
$$\begin{bmatrix} \alpha_{00} \\  \alpha_{01} \\  \alpha_{10} \\  \alpha_{11} \end{bmatrix}$$  
满足：$$|\alpha_{00}|^2+|\alpha_{01}|^2+|\alpha_{10}|^2+|\alpha_{11}|^2=1$$，并且多量子态向量保存着描述系统状态的所有信息。

如果给出两个独立的量子比特，一个的状态为$$\begin{bmatrix} \alpha \\  \beta \end{bmatrix}$$，另一个状态为$$\begin{bmatrix} \gamma \\  \delta \end{bmatrix}$$，那么这两个量子比特组成的双量子系统的状态就是：  
$$\begin{bmatrix} \alpha \\  \beta \end{bmatrix} \otimes \begin{bmatrix} \gamma \\  \delta \end{bmatrix} 
=\begin{bmatrix} \alpha \begin{bmatrix} \gamma \\  \delta \end{bmatrix} \\ \beta \begin{bmatrix}\gamma \\  \delta \end{bmatrix} \end{bmatrix}
= \begin{bmatrix} \alpha\gamma \\  \alpha\delta \\  \beta\gamma \\  \beta\delta \end{bmatrix},$$

上面的$$\otimes$$符号表示向量的张量积（克罗内克积）。需要注意的是，虽然我们总能使用两个单量子态向量来构建一个双量子系统的量子态向量，但并非所有的双量子系统状态都能表示成两个单量子态向量的张量积。例如，不存在这样两个向量$$\psi=\begin{bmatrix} \alpha \  \beta \end{bmatrix}$ and $\phi=\begin{bmatrix} \gamma \  \delta \end{bmatrix}$$，使得他们的张量积为：$$\psi\otimes \phi = \begin{bmatrix} 1/\sqrt{2} \  0 \  0 \  1/\sqrt{2} \end{bmatrix}.$$

不能用两个单量子态向量的张量积来表示的双量子态叫做“纠缠态”，并且这两个量子比特被成为纠缠的。不严格地说，双量子态不能被认为是两个单量子比特的张量积，这个状态所持有的信息也不局限与两个单量子态中的一个，而是非局部地存储于两个量子态之间的联系中，这种信息的非局部性是量子计算和传统计算之间的主要区别之一，其对许多量子协议也是必不可少的，比如量子隐形传态和量子纠错。

#### 双量子态的测量

双量子比特量子态的测量与单量子类似。测量一个状态为：  
$$\begin{bmatrix} \alpha_{00} \\ \alpha_{01} \\ \alpha_{10} \\ \alpha_{11} \end{bmatrix} $$  
的双量子系统，有$$|\alpha_{00}|^2$$的概率得到结果$$00$$，$$|\alpha_{01}|^2$$的概率得到结果$$01$$，$$|\alpha_{10}|^2$$的概率得到结果$$10$$，$$|\alpha_{11}|^2$$的概率得到结果$$11$$。在测量之后如果得到的结果是$$00$$，那么此时双量子系统的量子态已经坍塌为：  
$$ 00 \equiv \begin{bmatrix} 1 \\ 0 \\ 0 \\ 0 \end{bmatrix}.  $$

测量一个双量子系统中的某个量子的状态也是可行的，在只测量一个量子位的情况下，测量的影响是稍微不同的，因为整个系统的状态不会坍塌到计算基础状态，而是坍塌到一个子系统，或者说，在只测量一个量子位只是让其中一个子系统坍塌而不是整个双量子系统。为了理解这个现象，考虑测量如下所示状态中的第一个量子：  
$$H^{\otimes 2} \left( \begin{bmatrix}1 \\ 0 \end{bmatrix}\otimes \begin{bmatrix}1 \\ 0 \end{bmatrix} \right) = \frac{1}{2}\begin{bmatrix}1\\ 1\\ 1\\ 1\end{bmatrix}\mapsto \begin{cases}\text{outcome }=0 ; \frac{1}{\sqrt{2}}\begin{bmatrix}1\\ 1\\ 0\\ 0 \end{bmatrix}\\ \text{outcome }=1  ; \frac{1}{\sqrt{2}}\begin{bmatrix}0\\ 0\\ 1\\ 1 \end{bmatrix}\\  \end{cases}.$$

两个输出结果各占50%的概率。测量第一个或第二个量子比特状态的数学规则很简单，假设我们让$$e_{k}$$作为第$$k$$个基向量，让$$S$$表示所有第$$k$$个元素为$$1$$的向量的集合，例如我们要测量第一个量子的状态，那么$$S$$中包括$$e_2\equiv 10$$和$$e_3\equiv 11$$，如果要测量第二个量子的状态，那么$$S$$就由$$e_1\equiv 01$$和$$e_3 \equiv 11$$组成。对于量子态向量为$$\psi$$的量子，测量得到其状态为$$1$$的该率为：  
$$P(\text{outcome}=1)= \sum_{k \text{ in the set } S}\psi^\dagger e_k e_k^\dagger \psi.$$

因为测量量子比特所得的结果只能是$$0$$或者$$1$$，得到$$0$$的概率就是$$1-P(\text{outcome}=1)$$。这样的测量行为可以用数学形式表述为：  
$$ \psi \mapsto \frac{\sum_{k \text{ in the set } S} e_k e_k^\dagger \psi}{\sqrt{P(\text{outcome}=1)}}.  $$

如果我们将上面的向量$$ \psi $$看做单位向量，那么测量第一个量子得到$$1$$的概率为：  
$$ P(\text{measurement of first qubit}=1) = (\psi^\dagger e_2)(e_2^\dagger \psi)+(\psi^\dagger e_3)(e_3^\dagger \psi)=|e_2^\dagger \psi|^2+|e_3^\dagger \psi|^2.  $$

注意，这只是测量结果的两个概率的总和，10和11是所有要测量的量子位，对我们上面的例子而言，计算时是这样的：  
$$ \frac{1}{4}\left|\begin{bmatrix}0;0 ;1 ;0\end{bmatrix}\begin{bmatrix}1\\ 1\\ 1\\ 1\end{bmatrix} \right|^2+\frac{1}{4}\left|\begin{bmatrix}0 ;0 ;0 ;1\end{bmatrix}\begin{bmatrix}1\\ 1\\ 1\\ 1\end{bmatrix} \right|^2=\frac{1}{2}.  $$

同时，对应的量子态也可以写作：  
$$ \frac{\frac{e_2}{2}+\frac{e_3}{2}}{\sqrt{\frac{1}{2}}}=\frac{1}{\sqrt{2}}\begin{bmatrix} 0\\ 0\\ 1\\ 1\end{bmatrix} $$

#### 双量子比特操作

在单量子情况下，任何幺正变换都是有效的。通常来说，施加与多量子比特上的幺正变换是一个大小为$$ 2 ^ {n} \times 2 ^{n} $$的矩阵（所以它作用与大小为$$2^{n}$$的向量上）并且$$U^{-1} = U^\dagger$$。例如，CNOT（controlled-NOT）门是一个用途广泛的双量子门，它使用如下单位矩阵表示：  
$$\operatorname{CNOT} = \begin{bmatrix} 1\ 0\ 0\ 0  \\  0\ 1\ 0\ 0 \\  0\ 0\ 0\ 1 \\  0\ 0\ 1\ 0 \end{bmatrix}$$

我们也可用单量子门操作来构造双量子门。假设对一个双量子系统，我们对其第一个和第二个量子比特分别应用下面两个门：  
$$ \begin{bmatrix} a\ b\\ c\ d \end{bmatrix} \begin{bmatrix} e\ f\\ g\ h \end{bmatrix} $$

这等价于对双量子系统应用这两个门的张量积：  
$$\begin{bmatrix}
a\ b\\ c\ d
\end{bmatrix}
\otimes 
\begin{bmatrix}
e\ f\\ g\ h
\end{bmatrix}=
\begin{bmatrix}
ae\ af\ be\ bf \\
        ag\ ah\ bg\ bh \\
        ce\ cf\ de\ df \\
        cg\ ch\ dg\ dh
        \end{bmatrix}.$$

因此，我们可以使用已知的单量子门来构建双量子门，这样的例子包括：$$H \otimes H$$, $$X \otimes 1$$,和 $$X \otimes Z$$。

虽然两个单量子门的张量积可以定义一个双量子门，但所有的双量子门并非都能由单量子门构建而成，不能由两个单量子门的张量积表示的双量子门叫做纠缠门，CNOT门就是一个纠缠门的例子。

对于CNOT门的认知可以推广到任意的门。一个受控的门一般来说扮演着身份验证的角色除非一个特定的量子比特是$$1$$。我们将应用于量子比特$$x$$上的受控的幺正变换标记为$$\Lambda_x(U)$$，那么对于它有：$$\Lambda_0(U) e_{1}\otimes {\psi}=e_{1}\otimes U{\psi}$$和$$\Lambda_0(U) e_{0}\otimes {\psi}=e_{0}\otimes{\psi}$$，这里的$$e_{0}$$和$$e_{1}$$是量子态为$$0$$和$$1$$对应的单量子态基向量，例如对于下面的受控$$Z$$门，我们可以将其表示为：  
$$ \Lambda_0(Z)= \begin{bmatrix}1 0 0 0 \\ 0 1 0 0 \\ 0 0 1 0 \\ 0 0 0 -1 \end{bmatrix}=(1 \otimes H)\operatorname{CNOT}(1 \otimes H).  $$

用有效的方式构建受控的幺正变换是一个主要的挑战，实现这一点的最简单的方法是建立一个基础门操作的受控版本的数据库，并在最初的幺正变换中用受控操作替代对应的基本门操作。但这样做是相当浪费的，有一些灵巧的办法可以只替换某些门操作来达到同样的效果。在微软量子开发框架中，我们提供了原始的、受控的方法，并且也允许用户自定义一个幺正变换的受控版本。

量子门也可以受经典信息的控制。例如，一个经典的受控非门只在比特位为$$1$$时才起作用，根据这一点来看，经典的受控门可以看做代码中的一个if语句，只在某一分支上起作用。

与单量子类似，对于一个幺正矩阵，如果其可以使用一个双量子门集合中任何大小为$$4 \times 4$$量子门的向量积近似表示，那么这个双量子门集合就被认为是通用的。一个通用门集合的例子是由Hadamard门、T门和CNOT门组成的集合，通过这些门操作的向量积我们可以近似拟合出任意的作用于双量子系统的幺正矩阵。

#### 多量子系统

我们遵循在双量子系统中探索到的模型来构建多量子系统，多量子系统的状态由较小系统的张量积构建而来。例如，在量子计算机中将字符串“1011001”编码的方式为：  
$$ 1011001 \equiv \begin{bmatrix} 0 \\  1 \end{bmatrix}\otimes \begin{bmatrix} 1 \\  0 \end{bmatrix}\otimes \begin{bmatrix} 0 \\  1 \end{bmatrix}\otimes \begin{bmatrix} 0 \\  1 \end{bmatrix} \otimes \begin{bmatrix} 1 \\  0 \end{bmatrix}\otimes \begin{bmatrix} 1 \\  0 \end{bmatrix}\otimes \begin{bmatrix} 0 \\  1 \end{bmatrix}.  $$

量子门的工作方式与上面相同。例如，我们对多量子系统中的第一个量子比特应用$$X$$门，在第二个和第三个量子比特之间应用CNOT门，那么整个变换可以表示为：  
$$\begin{align} (X \otimes \operatorname{CNOT}_{12}\otimes 1 \otimes 1 \otimes 1) \begin{bmatrix} 0 \\  1 \end{bmatrix}\otimes \begin{bmatrix} 1 \\  0 \end{bmatrix}\otimes \begin{bmatrix} 0 \\  1 \end{bmatrix}\otimes \begin{bmatrix} 0 \\  1 \end{bmatrix} \otimes \begin{bmatrix} 1 \\  0 \end{bmatrix}\otimes \begin{bmatrix} 1 \\  0 \end{bmatrix}\otimes \begin{bmatrix} 0 \\  1 \end{bmatrix}\\ \qquad\qquad\equiv 0011001.  \end{align}$$

在多量子系统中，经常需要分配和释放计算机中临时的量子比特所占用的空间，这些临时的量子变量又叫做“附属（ancilla）”量子。默认情况下，新分配的量子比特都被初始化为$$e_{0}$$状态，我们进一步假定在释放之前，这个量子比特的状态重新回到$$e_{0}$$。这个假定很重要，因为如果一个附属量子在被释放时与其他量子寄存器纠缠，那么对它的释放就会破坏其他量子的状态，因此，我们才认定一个量子比特被释放时要回到最初的状态。

最后，对于双量子系统来说，构建一个通用的量子门集合需要想集合中添加新的门，但对多量子系统来说这是不必要的。H、T和CNOT门就构成了多量子系统的一个通用门集合，因为任何一个幺正变换都可以分解为一系列的双量子的旋转。因此当面对多量子系统时，我们可以继续应用双量子系统中的相关理论。

目前我们都是使用线性代数中的符号来描述多量子系统，当时当量子数量增多时，这些表述将会变得非常繁琐，例如对于一个7位长的字符串，其对应的量子态向量的维数就达到了128，因此，接下来我们将引入一个新的符号体系来描述量子系统，其能精确表达量子状态，同时使用起来也非常便捷。

### 狄拉克符号

在线性代数中，列向量符号是无处不在的，但在量子计算中，特别是处理多量子系统时，向量符号就显得笨拙而繁琐。例如，我们定义两个向量$$\psi$$和$$\phi$$，但是我们并不知道它们是列向量还是行向量，也不知道它们的大小分别是多少。除了向量的形态，使用向量符号表示哪怕是非常简单的量子态也是一件繁琐的工作。比如，我们想要表达一个$$n$$量子比特的系统状态，这些量子比特处于$$0$$状态，那么正规的表示形式为：
$$\begin{bmatrix}1 \\  0 \end{bmatrix}\otimes \cdots \otimes\begin{bmatrix}1 \\  0 \end{bmatrix}. $$

当然，对这个张量积求值是不切实际的，因为向量都位于指数级别增长的空间中，所以这个符号是使用向量符号对量子系统状态进行描述的最好的符号了。

狄拉克符号是一种新的能够满足量子计算精确需求的语言，因此我们建议读者不要把本节中的例子看作是描述量子态的严格公式，而是鼓励读者把这些看作是可以用来简明表达量子思想的建议。

在狄拉克符号中有两种向量：左矢（Bra vector）和右失（ket），之所以这样命名这两种向量，是因为将两个向量放在一起时构成一个“braket”（狄拉克符号也叫braket符号）或内积。如果$$\psi$$是一个列向量，我们可以用狄拉克符号$$|\psi \rangle$$来表示，此处$$|\cdot \rangle$$表示一个单位列向量，比如右失，类似的行向量$$\psi^\dagger$$可以表示为$$\langle \psi |$$，而符号$$\langle \psi |\psi \rangle$$就表示了向量$$\psi$$与其自身的内积，根据定义其值为$$1$$。

更一般的，如果$$\psi$$和$$\phi$$是量子态向量，他们的内积为：$$\langle \phi | \psi \rangle$$，并且测量量子态$$|{\psi}\rangle$$结果为$$|{\phi}\rangle$$的概率为$$|\langle \phi|\psi\rangle|^2$$。

下面所示是量子态$$0$$和$$1$$与狄拉克符号之间的关系：
$$ \begin{bmatrix} 1 \\  0 \end{bmatrix} = |0\rangle,\qquad \begin{bmatrix} 0 \\  1 \end{bmatrix} = |1\rangle.  $$

下面的符号通常用来描述对$$|0\rangle$$和$$|1\rangle$$应用Hadamard门所产生的状态（对应于布罗兹球面中的$$+x$$和$$-x$$方向）：
$$ \frac{1}{\sqrt{2}}\begin{bmatrix} 1 \\  1 \end{bmatrix}=H|0\rangle = |{+}\rangle,\qquad \frac{1}{\sqrt{2}}\begin{bmatrix} 1 \\  -1 \end{bmatrix} =H|1\rangle = |{-}\rangle .  $$

这些状态也可以使用狄拉克符号来表示：
$$ |+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle),\qquad |-\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle).  $$

从上面的关系可以明白为什么这些状态被称为“计算基底”，因为任何一个量子态都能通过这些基底的加和来表示，并且这些加和形式能方便的使用狄拉克符号来表达。上面的转换关系反过来也是成立的，从下面的关系中可以看出，$$|+\rangle$$和$$|-\rangle$$构成了量子太基底。

作为一个使用狄拉克符号的例子，考虑$$0$$和$$1$$的内积，其可以使用狄拉克符号表示为：
$$ |0\rangle = \frac{1}{\sqrt{2}}(|+\rangle + |-\rangle),\qquad |1\rangle = \frac{1}{\sqrt{2}}(|+\rangle - |-\rangle).  $$

上面的例子说明了$$|0\rangle$$和$$|1\rangle$$是正交向量，即$$\langle 0 |1\rangle = \langle 1 | 0\rangle =0$$，同时，根据定义可以得到：$$\langle 0 | 0 \rangle = \langle 1 | 1\rangle=1$$，说明两个计算基底是标准正交的。在下面的例子中，可以看到标准正交性质的作用，假设存在状态$$|\psi\rangle = {\frac{3}{5}} |1\rangle + {\frac{4}{5}} |0\rangle$$，因为$$\langle 1 | 0\rangle =0$$，测量结果为1的概率为：
$$\big|\langle 1 | \psi\rangle \big|^2= \left|\frac{3}{5}\langle 1 | 1\rangle +\frac{4}{5}\langle 1 |0 \rangle\right|^2=\frac{9}{25}.$$

狄拉克符号还包含一个隐式的张量积结构。这是非常重要的，因为在量子计算中，使用两个不相关的量子寄存器描述的量子态向量是这两个不相关量子态的张量积。如果你想解释量子计算，那么能够简明描述张量积结构是非常重要的，狄拉克符号体系中，两个向量$$\psi$$和$$\phi$$的张量积$$\psi \otimes \phi$$可以写为$$|\psi\rangle |\phi\rangle$$，有时也写作：$$|\psi\rangle \otimes |\phi\rangle$$，例如：
$$ \begin{bmatrix} 1 \\  0 \\  0 \\  0 \end{bmatrix}= \begin{bmatrix} 1 \\  0 \end{bmatrix} \otimes \begin{bmatrix} 1 \\  0 \end{bmatrix} = |0\rangle \otimes |0\rangle= |0\rangle |0\rangle.  $$

类似地，状态$$|p \rangle$$表示一个使用整数$$p$$的二进制编码表示的量子态，例如，我们想使用二进制编码来表示整数5，那么下面的这几个表示是等同的：
$$ |{1}\rangle|{0}\rangle|{1}\rangle = |{101}\rangle = |{5}\rangle.  $$

在上面的等式中，$$|0\rangle$$不必是一个量子比特，而是一个存储了状态为$$0$$的量子比特的寄存器，这之间的区别可以根据上下文的环境很容易判断出来，同时，这种转换在简化某些表达上是很有用处的，例如本节中的第一个例子就可以写作：
$$ \begin{bmatrix}1 \\  0 \end{bmatrix}\otimes \cdots \otimes\begin{bmatrix}1 \\  0 \end{bmatrix} = |0\rangle \otimes \cdots \otimes |0\rangle= |0\cdots 0\rangle = |0\rangle^{\otimes n} = |0\rangle.  $$

另一个使用狄拉克符号来表示量子态的例子如下所示，在每一个长度为$$n$$的位串上写一个相等的叠加态：
$$H^{\otimes n} |{0}\rangle = \frac{1}{2^{n/2}} \sum_{j=0}^{2^n-1} |{j}\rangle=|+\rangle^{\otimes n}.$$

这里你可能会想知道为什么对于$$n$$个比特，加和的范围是$$0$$到$$2^{n}-1$$。首先，注意对n个比特，可能的构型有$$2^{n}$$种，因为每个比特都有两个可选的值。同时要说明的是，在这个例子中，我们没有想$$|0\rangle^{\otimes n} = |0\rangle$$一样将$$|+\rangle^{\otimes n}$$简化为$$|+\rangle^{\otimes n}=|+\rangle$$，因为这个转换惯例通常是保留给每个初始状态为$$0$$的计算基底的，虽然$$|+\rangle^{\otimes n}=|+\rangle$$这样表达也是有意义的，但是并没有被引入量子计算的书面表达中。

除了简明的优点，狄拉克符号的另一个有点就是其是线性的。例如，对于下面向量的张量积，我们可以写为：
$$(\alpha |\psi\rangle +\beta|\phi\rangle)\otimes (\gamma |\chi\rangle + \delta |\omega\rangle)= \alpha\gamma |\psi\rangle|\chi\rangle + \alpha\delta |\psi\rangle|\omega\rangle+\beta\gamma|\phi\rangle|\chi\rangle+\beta\omega|\phi\rangle|\omega\rangle.$$

从上面的例子可以看出，将张量积用狄拉克符号表示时，就像是普通的乘法一样简单明了。

左失遵循与右失类似的惯例。例如，向量$$\langle\psi|\langle \phi|$$与状态向量$$\psi^\dagger \otimes \phi^\dagger=(\psi\otimes \phi)^\dagger$$相等。如果右失$$|\psi\rangle$$是$$\alpha |0\rangle + \beta |1\rangle$$，那么对应的左失就是$$\langle{\psi}|=|\psi\rangle^\dagger = (\langle 0|\alpha^* +\langle 1 |\beta^*)$$。举一个例子，假设我们要计算使用一个量子测量程序测量$$|\psi\rangle = \frac{3}{5} |1\rangle + \frac{4}{5} |0\rangle$$的状态为$$|+\rangle$$或$$|-\rangle$$的概率，那么所得的概率为：
$$|\langle - |\psi\rangle|^2= \left|\frac{1}{\sqrt{2}}(\langle 0| - \langle 1|)(\frac{3}{5} |1\rangle + \frac{4}{5} |0\rangle) \right|^2=\left|-\frac{3}{5\sqrt{2}} + \frac{4}{5\sqrt{2}}\right|^2=\frac{1}{50}.$$

在概率计算中出现负号是量子干涉的一种表现形式，量子干涉也是量子计算之所以优于传统计算的机制之一。

狄拉克符号体系中，最后一项值得被讨论的优点是*ketbra*，也就是外积。用狄拉克符号表示外积的形式为$$|\psi\rangle \langle \phi|$$，它也被叫做*ketbra*，因为可以看到它与内积*braket*中的符号是相反的。对于两个量子态向量$$\psi$$和$$\phi$$，它们的外积是通过矩阵乘法定义的：$$|\psi\rangle \langle \phi| = \psi \phi^\dagger$$，一个很简单的关于外积的例子如下所示:
$$|0\rangle \langle 0| = \begin{bmatrix}1\\ 0 \end{bmatrix}\begin{bmatrix}1&0 \end{bmatrix}= \begin{bmatrix}1 &0\\ 0 &0\end{bmatrix} \qquad |1\rangle \langle 1| = \begin{bmatrix}0\\ 1 \end{bmatrix}\begin{bmatrix}0&1 \end{bmatrix}= \begin{bmatrix}0 &0\\ 0 &1\end{bmatrix}.$$



