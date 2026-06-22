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
4、make使用文件的创建时间和修改时间进行增量编译
# 伪目标
```makefile
clean:
	rm -f m.txt
	rm -f x.txt
```
1、clean没有依赖文件，要执行它必须使用make clean
2、执行时没有创建一个名为clean的文件，如果手动创建一个名为clean的文件，这个规则就不会执行
3、如果希望make不将clean视作文件，每次都执行它，可以添加一个标识，将其视为伪目标（Phony Target)
```makefile
.PHONY: clean
clean:
	rm -f m.txt
	rm -f x.txt
```
# 执行多条命令
make对每行命令都会创建一个独立的shell环境，下面是几种解决方法
```makefile
# 1.使用;分隔
cd_ok:
	pwd; cd ..; pwd;
# 2.使用换行符
cd_ok:
	pwd; \
	cd ..; \
	pwd
# 3.使用&&，惰性执行
cd_ok:
	cd .. && pwd
```
# 控制打印
```makefile
no_output:
	@echo 'not display'
	echo 'will display'
```
# 控制错误
make执行命令遇到错误会停止，使用-忽略错误并继续执行
```makefile
ignore_error:
	-rm zzz.txt
	echo 'ok'
```
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