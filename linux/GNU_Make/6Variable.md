# Makefile 的变量
变量在声明时需要给予初值，而在使用时，需要给在变量名前加上 `$` 符号，并用小括号 `()` 把变量给包括起来。

## 变量的定义

```makefile
cpp := src/main.cpp 
obj := objs/main.o
```
## 变量的引用
- 可以用 `()` 或 `{}`
```makefile
cpp := src/main.cpp 
obj := objs/main.o

$(obj) : ${cpp}
	@g++ -c $(cpp) -o $(obj)

compile : $(obj)
```

## 预定义变量
- `$@`: 目标(target)的完整名称
- `$<`: 第一个依赖文件（prerequisties）的名称
- `$^`: 所有的依赖文件（prerequisties），以空格分开，不包含重复的依赖文件
```make
cpp := src/main.cpp 
obj := objs/main.o

$(obj) : ${cpp}
	@g++ -c $< -o $@
	@echo $^

compile : $(obj)
.PHONY : compile
```