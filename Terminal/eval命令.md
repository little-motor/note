[toc]
## 1. 基本格式
```
eval command-line
```
command-line是可以在终端中输入的普通命令行。Shell会对eval进行二次扫描，然后执行。如果使用脚本构造出的命令需要被调用，那么eval的这个功能就非常有用。
## 2. 使用场景
### 2.1 从变量中构造命令行
eval常用于从变量中构造命令行，如果变量中包含了任何必须由Shell解释的字符，那就必须用到eval。命令终止符(;、|、&)、I/O重定向(<、>)以及引号都属于这类字符。例如下面的例子：
```
pipe="|"
ls $pipe wc -l
ls: -l: No such file or directory
ls: wc: No such file or directory
ls: |: No such file or directory
```
错误的原因在于pipe的值以及随后的wc -l都会被视为命令参数。Shell是在变量替换之前处理管道和I/O重定向的，因此不可能再去解释pipe中保存的管道符号。
解决方案
```
eval ls $pipe wc -l
```
Shell第一次扫描命令时，它会将pipe替换成对应的值|。然后eval会使得Shell重新扫描命令行，这时候Shell识别出了作为管道符号的|。
