[toc]
## 1. 引言
Java与C++之间有一堵由内存动态分配和垃圾收集技术所围成的“高墙”，Java虚拟机在执行Java程序的过程中会把他所管理的内存划分为若干个不同的数据区域

![Java虚拟机运行时数据区](https://raw.githubusercontent.com/little-motor/uml/master/JVMRunTime.png)

## 2. 程序计数器(Program Counter Register)
程序计数器是一块较小的内存空间，作为当前线程执行的字节码的行号指示器，Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式实现的，每条线程都有一个独立的程序计数器。
## 3. 栈
### 3.1 Java虚拟机栈(VM Stack)
Java虚拟机栈也是线程私有的，与线程的生命周期相同，虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行时都会创建一个栈帧(Stack Frame)用于存储局部变量表、操作数栈、动态链接、方法出口等信息。
#### 3.1.1 局部变量表
局部变量表存放了编译期可知的各种基本数据类型(boolean,byte,char,short,int,float,long,double),对象引用(reference类型)和returnAddress类型。
### 3.2 本地方法栈(Native Method Stack)
与虚拟机栈所发挥的作用非常相似，本地方法栈执行Native方法。
## 4. 堆(Java Heap)
Java堆是被所有线程共享的一块内存区域，此区域的唯一目的是存放对象实例和数组，也被称为GC堆。基本都采用分代收集算法，Java堆可以被细分为：Eden空间、FromSurvivor空间、To Survivor空间等。
## 5. 方法区(Method Area)
方法区与Java堆一样，是各个线程共享的内存区域，存储虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是他却有一个别名叫做Non-Heap，目的是与Java堆区分开来。
HopSpot虚拟机设计团队选择把GC分代收集扩展至方法区，或者说使用永久代实现方法去而已，能够像管理Java堆一样管理这部分内存，能够省去专门为方法区编写内存管理代码的工作。
### 5.1 运行时常量池(Runtime Constant Pool)
运行时常量池是方法区的一部分，用于存放编译期生成的各种字面量和符号引用。运行时常量池相对于Class文件常量池的另一个重要特征是具备动态性。

