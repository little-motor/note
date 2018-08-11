 [toc]
## 1. 引言
如果想让交流220V为12V笔记本充电，需要使用电源适配器，在程序世界中，也经常会存在现有程序无法使用，需要做适当变换之后才能使用的情况。这种用于填补“现有的程序”和“所需的程序”之间差异的模式就是Adapter模式。Adapter模式也被称为Wrapper模式（包装器）
Adapter模式有以下两种

- **类适配器模式（使用继承的适配器）**
- **对象适配器模式（使用委托的适配器）**
## 2. 类适配器模式
### 2.1 示例程序
示例程序是输出（hello）和*hello*的简单程序。在Banner（有广告横幅的意思）类中，有将字符串用括号括起来的showWithParen方法和将字符串用*号括起来的showWithAster方法。假设这个Banner类是类似引言中“220V交流电”。
假设Print接口中声明了两种方法，弱化字符串显示（加括号）的printWeak方法，和强调字符串显示（加*号）的printStrong方法。假设这个Pring接口类似于笔记本“12V直流电需求”。
现在要做的事情是使用Banner类编写一个实现了Print接口的类，也就是做一个“220V交流电”转“12V直流电”的适配器，扮演适配器的角色是PrintBanner类，在该类中使用showWithParen方法实现printWeak，使用showWithAster方法实现printStrong。

 <center>|电源的比喻 | 示例程序
---------|----------|---------
 实际情况 | 交流220V | Banner类(showWithParen、showWithAster)
 变换装置 | 适配器 | PrintBanner类
需求 | 直流12V | Print接口(printWeak、printStrong)
