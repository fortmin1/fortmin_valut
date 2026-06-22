# 规则
Makefile由若干条规则（Rule）构成，每一条规则指出一个目标文件（Target），若干依赖文件（prerequisites）,以及生成目标文件的命令。
```makefile
# 目标文件: 依赖文件1 依赖文件2
m.txt: a.txt b.txt
	cat a.txt b.txt > m.txt
```
1、makefile会默认执行第一条规则，其它规则的顺序无关紧要，make执行时会自动判断依赖。
2、命令必须以Tab开头。
3、make会打印出执行的每一条命令。
# 伪目标
# 隐式规则

# 变量
## 定义
```makefile
OBJS = hello.o main.o
TARGET = world.out

$(TARGET): $(OBJS)
	cc -o $(TARGET) $(OBJS)

clean:
	rm -f *.o $(TARGET)
```
## wlidcard、patsubst
```makefile
# $(wildcard *.c) 列出当前目录下的所有 .c 文件: hello.c main.c
# 用函数 patsubst 进行模式替换得到: hello.o main.o
OBJS = $(patsubst %.c,%.o,$(wildcard *.c))
TARGET = world.out

$(TARGET): $(OBJS)
	cc -o $(TARGET) $(OBJS)

clean:
	rm -f *.o $(TARGET)
```
## 内置变量
我们还可以用变量`$(CC)`替换命令`cc`：

```makefile
$(TARGET): $(OBJS)
	$(CC) -o $(TARGET) $(OBJS)
```

没有定义变量`CC`也可以引用它，因为它是`make`的内置变量（Builtin Variables），表示C编译器的名字，默认值是`cc`，我们也可以修改它，例如使用交叉编译时，指定编译器：

```makefile
CC = riscv64-linux-gnu-gcc
```
## 自动变量
在`Makefile`中，经常可以看到`$@`、`$<`这样的变量，这种变量称为自动变量（Automatic Variable），它们在一个规则中自动指向某个值。
例如，`$@`表示目标文件，`$^`表示所有依赖文件，因此，我们可以这么写：
```makefile
world.out: hello.o main.o
	cc -o $@ $^
```
在没有歧义时可以写`$@`，也可以写`$(@)`，有歧义时必须用括号，例如`$(@D)`。
为了更好地调试，我们还可以把变量打印出来：
```makefile
world.out: hello.o main.o
	@echo '$$@ = $@' # 变量 $@ 表示target
	@echo '$$< = $<' # 变量 $< 表示第一个依赖项
	@echo '$$^ = $^' # 变量 $^ 表示所有依赖项
	cc -o $@ $^
```
执行结果输出如下：
```plain
$@ = world.out
$< = hello.o
$^ = hello.o main.o
cc -o world.out hello.o main.o
```