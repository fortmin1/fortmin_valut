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
## patsubst
