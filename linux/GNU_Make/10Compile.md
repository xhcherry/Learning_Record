# Compile

## 编译过程
### 预处理

>示例
```makefile
cpp_srcs := $(shell find src -name *.cpp)
pp_files := $(patsubst src/%.cpp,src/%.i,$(cpp_srcs))

src/%.i : src/%.cpp
	@g++ -E $^ -o $@

preprocess : $(pp_files)

clean :
	@rm -f src/*.i

debug :
	@echo $(pp_files)

.PHONY : debug preprocess clean
```

### 编译成汇编语言
>示例
```makefile
cpp_srcs := $(shell find src -name *.cpp)
as_files := $(patsubst src/%.cpp,src/%.s,$(cpp_srcs))

src/%.s : src/%.cpp
	@g++ -S $^ -o $@

assemble : $(as_files)

clean :
	@rm -f src/*.s

debug :
	@echo $(as_files)

.PHONY : debug assemble clean
```

### 编译成目标文件
>示例
```makefile
cpp_srcs := $(shell find src -name *.cpp)
cpp_objs := $(patsubst src/%.cpp,objs/%.o,$(cpp_srcs))

objs/%.o : src/%.cpp
	@mkdir -p $(dir $@)
	@g++ -c $^ -o $@

objects : $(cpp_objs)

clean :
	@rm -rf objs src/*.o

debug :
	@echo $(as_files)

.PHONY : debug objects clean
```

### 链接可执行文件
```makefile
cpp_srcs := $(shell find src -name *.cpp)
cpp_objs := $(patsubst src/%.cpp,objs/%.o,$(cpp_srcs))

objs/%.o : src/%.cpp
	@mkdir -p $(dir $@)
	@g++ -c $^ -o $@

workspace/exec : $(cpp_objs)
	@mkdir workspace/exec
	@g++ $^ -o $@

run : workspace
	@./$<

clean :
	@rm -rf objs workspace/exec

debug :
	@echo $(as_files)

.PHONY : debug run clean
```

## 编译选项
>编译选项
- `-m64`: 指定编译为 64 位应用程序
- `-std=`: 指定编译标准，例如：-std=c++11、-std=c++14
- `-g`: 包含调试信息
- `-w`: 不显示警告
- `-O`: 优化等级，通常使用：-O3
- `-I`: 加在头文件路径前
- `fPIC`: (Position-Independent Code), 产生的没有绝对地址，全部使用相对地址，代码可以被加载到内存的任意位置，且可以正确的执行。这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的

>链接选项
- `-l`: 加在库名前面
- `-L`: 加在库路径前面
- `-Wl,<选项>`: 将逗号分隔的 <选项> 传递给链接器
- `-rpath=`: "运行" 的时候，去找的目录。运行的时候，要找 .so 文件，会从这个选项里指定的地方去找

## Implicit Rules
- CC: Program for compiling C programs; default cc
- CXX: Program for compiling C++ programs; default g++
- CFLAGS: Extra flags to give to the C compiler
- CXXFLAGS: Extra flags to give to the C++ compiler
- CPPFLAGS: Extra flags to give to the C preprocessor
- LDFLAGS: Extra flags to give to compilers when they are supposed to invoke the linker

## 编译带头文件的程序
>add.hpp
```c++
#ifndef ADD_HPP
#define ADD_HPP
int add(int a, int b);

#endif // ADD_HPP
```
>add.cpp
```c++
int add(int a, int b)
{
    return a+b;
}
```
>minus.hpp
```c++
#ifndef MINUS_HPP
#define MINUS_HPP
int minus(int a, int b);

#endif // MINUS_HPP
```
>minus.cpp
```c++
int minus(int a, int b)
{
    return a-b;
}
```
>main.cpp
```c++
#include <stdio.h>
#include "add.hpp"
#include "minus.hpp"

int main()
{
    int a=10; int b=5;
    int res = add(a, b);
    printf("a + b = %d\n", res);
    res = minus(a, b);
    printf("a - b = %d\n", res);

    return 0;
}
```

>Makefile
```makefile
cpp_srcs := $(shell find src -name *.cpp)
cpp_objs := $(patsubst src/%.cpp,objs/%.o,$(cpp_srcs))

# 你的头文件所在文件夹路径（建议绝对路径）
include_paths := 
I_flag        := $(include_paths:%=-I%)


objs/%.o : src/%.cpp
	@mkdir -p $(dir $@)
	@g++ -c $^ -o $@ $(I_flag)

workspace/exec : $(cpp_objs)
	@mkdir -p $(dir $@)
	@g++ $^ -o $@ 

run : workspace/exec
	@./$<

debug :
	@echo $(I_flag)

clean :
	@rm -rf objs

.PHONY : debug run
```
