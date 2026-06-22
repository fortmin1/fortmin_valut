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
