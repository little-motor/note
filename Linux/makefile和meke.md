[toc]
## 1. makefile
makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为makefile就像一个Shell脚本一样，其中也可以执行操作系统的命令。makefile带来的好处就是——“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。
### 1.1 makefile规则
```
target ... : prerequisites ...
            command
            ...
            ...
```

- target可以是一个目标文件，可以是Object File，可以是执行文件，还可以是一个标签（Label）。
- prerequisites就是要生成那个target所需要的文件或是目标。
- command也就是make需要执行的命令。（任意的Shell命令）

说白一点就是说，prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。这就是Makefile的规则。也就是Makefile中最核心的内容。
### 1.2 示例
如果一个工程有3个头文件，和8个C文件，我们为了完成前面所述的那三个规则，我们的Makefile应该是下面的这个样子的
```
  edit : main.o kbd.o command.o display.o /
           insert.o search.o files.o utils.o
            cc -o edit main.o kbd.o command.o display.o /
                       insert.o search.o files.o utils.o

  main.o : main.c defs.h
            cc -c main.c
            kbd.o : kbd.c defs.h command.h
            cc -c kbd.c
  command.o : command.c defs.h command.h
          cc -c command.c
  display.o : display.c defs.h buffer.h
          cc -c display.c
  insert.o : insert.c defs.h buffer.h
          cc -c insert.c
  search.o : search.c defs.h buffer.h
          cc -c search.c
  files.o : files.c defs.h buffer.h command.h
          cc -c files.c
  utils.o : utils.c defs.h
          cc -c utils.c
  clean :
          rm edit main.o kbd.o command.o display.o /
              insert.o search.o files.o utils.o
```
## 2. make
最简单的就是直接在命令行下输入make命令，make命令会找当前目录的“GNUmakefile”、“makefile”和“Makefile”来执行，一切都是自动的。如果要删除执行文件和所有的中间目标文件，那么，只要简单地执行一下“make clean”就可以了。
make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会执行后续定义的命令。
### 2.1 make的退出码
make有三个退出码

- 0 —— 表示成功执行。
- 1 —— 如果make运行时出现任何错误，其返回1。
- 2 —— 如果你使用了make的“-q”选项，并且make使得一些目标不需要更新，那么返回2。

### 2.2 指定目标
一般来说，make的最终目标是makefile中的第一个目标，而其它目标一般是由这个目标连带出来的。这是make的默认行为。当然，一般来说，你的 makefile中的第一个目标是由许多个目标组成，你可以指示make，让其完成你所指定的目标。要达到这一目的很简单，需在make命令后直接跟目标的名字就可以完成（如“make clean”形式）。任何在makefile中的目标都可以被指定成终极目标，但是除了以“- ”打头，或是包含了“=”的目标，因为有这些字符的目标，会被解析成命令行参数或是变量。甚至没有被我们明确写出来的目标也可以成为make的终极目标，也就是说，只要make可以找到其隐含规则推导规则，那么这个隐含目标同样可以被指定成终极目标。
### 2.3 伪目标
make可以指定所有makefile中的目标，那么也包括“伪目标”，于是我们可以根据这种性质来让我们的makefile根据指定的不同的目标来完成不同的事。在Unix世界中，软件发布时，特别是GNU这种开源软件的发布时，其 makefile都包含了编译、安装、打包等功能。我们可以参照这种规则来书写我们的makefile中的目标。

```
  “all”              这个伪目标是所有目标的目标，其功能一般是编译所有的目标。

  “clean”       这个伪目标功能是删除所有被make创建的文件。

  “install”       这个伪目标功能是安装已编译好的程序，其实就是把目标执行文件拷贝到指定的目标中去。

  “print”         这个伪目标的功能是例出改变过的源文件。

  “tar”             这个伪目标功能是把源程序打包备份。也就是一个tar文件。

  “dist”           这个伪目标功能是创建一个压缩文件，一般是把tar文件压成Z文件。或是gz文件。

  “TAGS”        这个伪目标功能是更新所有的目标，以备完整地重编译使用。

  “check”和“test”    这两个伪目标一般用来测试makefile的流程。

```

参考文章：
https://blog.csdn.net/haoel/article/details/2886
https://blog.csdn.net/ruglcc/article/details/7814546