---
title: "Cpp 入门"
tags: ["社团课程"]
date: 2019-10-20T21:06:47+08:00
layout: 'post'
draft: false
---

{{< blockquote author="Linus Torvalds" >}}
C++ is a horrible language. It's made more horrible by the fact that a lot of substandard programmers use it, to the point where it's much much easier to generate total and utter crap with it.
{{< /blockquote >}}


<!--more-->

为了便于讲解，本文采用由浅及深的方式组织行文主线。

本文主要采用对比 c 和 cpp 差异的方式来进行。

---

## 头文件

几乎所有的 c 语言代码都需要头文件，cpp 和 c 的头文件对比是开始编程前必须的一步。

*几乎所有的 c 代码中 `#include` 命令都可以直接复制到 cpp 中使用，但是这并不推荐*

### \#include 是什么？

```c
// module.txt
printf("%d\n", a);
a *= 3;
```

```c
// main.c
##include <stdio.h>

int main()
{
	int a = 1, b = 2;
##include "module.txt"
    printf("%d %d\n", a, b);
    return 0;
}
```

编译执行 main.c 后，输出应该是

> 1
>
> 3 2

可见，`#include` 作用是将一段文本直接引入到代码中。

对头文件使用 `#include` 时也是如此。

### 头文件名称对比


语言 | -----------|------------|-----------|-----------|-----------|-----------
----    | -----------|------------|-----------|-----------|-----------|-----------
c       | stdio.h | stdlib.h | math.h|\\|windows.h|string.h
cpp  | cstdio   | cstdlib  | cmath  |vector|windows.h|cstring

可见，对于常见的头文件，是将 c 头文件的 .h 去掉，在前面加上 c，如 <font size=5><font color=#green>c</font>stdio<font color=red>.h</font> </font> 。

尤其注意 string.h 文件，cstring 和 string 完全不同。

```c
// cstdio
...
##include <stdio.h>
...
```

---
## IOSTREAM

```cpp
##include <iostream>
using namespace std;
...
cin >> a;
cout << b;
```

上面这一段应该是大多数人最早接触的 cpp 代码。

大概讲解一下这一段内容中每行代码发生了什么。

`#include <iostream>` 引入 iostream 头文件

`using namespace std;` 使用 std 命名空间，命名空间如果有时间放在后面讲。<font color=#E0E0E0>反正也没人用得到不如不讲了。</font>

关于 cin 和 cout，我们可以构造这样一个模型：

{{<mermaid>}}
graph LR;
	input["input"]-->cin["cin"];
	cin["cin"]-->variable["variable"];
	variable["variable"]-->cout["cout"];
	cout["cout"]-->output["output"];
	style cin fill:#CFC,stroke:#333,stroke-width:0.5px
	style cout fill:#CFC,stroke:#333,stroke-width:0.5px
	style variable fill:#FFC,stroke:#333,stroke-width:0.5px
{{</mermaid>}}

cin 和 cout 仅仅是一个管道，用来让数据从中流过（data stream），这就是所谓的 iostream（input & output stream）。

`cin` 可以用来输入多个值，如 `cin >> a >> b;`

`cin` 可以隐式转换为 `bool` 。

`cout` 用法同理。

---

## 新的关键字

本篇只介绍 `new` 和 `delete` 。

###  简单介绍`malloc` 系列

`malloc` 及其内存分配系列函数（如`calloc` ）均在 stdlib.h 中，而 `new` 和`delete` 是语言本身的内存分配机制。

`malloc` 需要一个参数，`void * malloc(size_t _NumOfBytes)` ，这个参数表示所需空间的大小，并且不对这段空间进行清空，用法如`int *arr = (int *)malloc(n * sizeof(int));` 表示分配 n 个 int 的空间。

其衍生函数，`calloc` ，需要两个参数，声明形式如`void * calloc(int _NumOfElements, size_t SizePerElement)` ，用法如`int *arr = (int *)calloc(n, sizeof(int));` ，需要注意，这个函数分配后会将这一段内存置为 0。

### `new` 和`delete` 用法

`new` 用法示例如下：

```cpp
int *a = new int; // 表示让 a 指向分配的 int 类型值，值未初始化
"上面这一行等价于";
int *a = (int*)malloc(sizeof(int));

int *b = new int(); // 表示让 b 指向分配的 int 类型值，并在分配后采用默认方式初始化这个值
// 对于 int 类型，默认方式初始化意味着置为 0

int *c = new int(3); // 表示让 c 指向分配的 int 类型值，并在分配后初始化这个值为 3
// *c == 3

int *d = new int[n]; // 表示让 d 指向分配的长度为 n 的 int 数组，内存未初始化
"上面这一行等价于";
int *d = (int*)malloc(n*sizeof(int));

int *e = new int[n](); // 表示让 e 指向分配的长度为 n 的 int 数组，并采用默认方式初始化每个值
// e[0] == e[1] == e[2] == ... == e[n-1] == 0

// 特别的，有些类型分配的时候略微麻烦一点（类型系统的进阶使用）
int **a= new (int *[n]); // 长度为 n 的 int* 数组
int (**pf)()= new (int (*)()); // pf 为指向函数指针的指针，指向的函数 f 声明形式如 int f(void);
int (**pfs)()= new (int (*[n])()); // pfs 为函数指针数组
```

**无法用 new 在创建一个数组的同时初始化每个元素**

---

`delete` 用法类似。（采用`new` 举例中所用的变量）

```cpp
delete a;

delete b;

delete c;

delete []d; // 删除数组

delete []e;
```

---

### `new` 的思考

能否用`new` 在创建一个数组后初始化每个元素呢？<font color=white> placement new </font>

## 函数多态（重载）与默认参数

### 函数多态

例如，我们有

```cpp
int max(int a, int b) { return a>b?a:b;}
```

如果我们想要三个数比较该怎么做？

在 c 中，

```c
int max3i(int a, int b, int c) { int t = max(a, b); return t>c?t:c;}
```

而在 cpp 中，我们可以直接使用

```cpp
int max(int a, int b, int c) { int t = max(a, b); return t>c?t:c;}
```

cpp 可以根据参数的数量和类型来自动判断调用哪一个函数。

以至于我们可以

```cpp
int max(int a) { int t = a; return t>a?t:a;}
int max(int a, int b) { int t = max(b); return t>a?t:a;}
int max(int a, int b, int c) { int t = max(b, c); return t>a?t:a;}
int max(int a, int b, int c, int d) { int t = max(b, c, d); return t>a?t:a;}
int max(int a, int b, int c, int d, int e) { int t = max(b, c, d, e); return t>a?t:a;}
int max(int a, int b, int c, int d, int e, int f) { int t = max(b, c, d, e, f); return t>a?t:a;}
```

这时候我们可以传入 1-6 个参数来最大值，并且除了单个参数的情况，其他具有一致性。

再例如，常见的错误代码 `int a = pow(5, 2)` ，我们可以定义 pow 的多态：

```cpp
int pow(int a, int n)
{
	if(n<0) return 0;
    int res = 1;
    for(int i=0; i<n; ++i) res *= a;
    return a;
}
```

可以避免浮点误差。

---

### 默认参数

上面介绍了函数多态，这里介绍函数默认参数。

例如我们需要一个求和函数，我们预定有四个数字要被传入。

```cpp
int sum(int a, int b, int c, int d) { return a+b+c+d;}
```

但是随着时间流逝，我们有时候需要传入三个参数。

```cpp
int sum(int a, int b, int c) { return sum(a, b, c, 0);}
```

以及两个······

```cpp
int sum(int a, int b) { return sum(a, b, 0);}
```

甚至一个············

```cpp
int sum(int a) { return sum(a, 0);}
```

甚至没有参数<del>（👈这样的函数有意义吗？？？）</del>

```cpp
int sum(void) { return sum(0);}
```

这里的求和函数和上面的 max 形式很像，并且一路规约下来也很自然，只是略微冗杂了点。

能不能更简单呢？

<font size=7>能！</font>

cpp 允许函数含有默认参数。

上面的 `sum` 系列函数使用默认参数后只需要一个函数即可。

```cpp
int sum(int a=0, int b=0, int c=0, int d=0) { return a+b+c+d;}
```

这个函数的定义表示，如果没有传入对应参数，则对应参数默认置为 0。

如 `sum(1,2,3)` ，其中各参数即为`a=1, b=2, c=3, d=0` 。

在 cpp 中，各参数只能从左向右排布，因此默认参数需要从右向左赋值。

函数的默认参数只能在声明或定义中出现一次。

---

## 新的类型

### 引用

引用类型是值和指针的折中。

引用的主要用途是函数传参和创建别名。

函数传递较大的数据类型由于需要整个拷贝，因此在 c 中通常传递指针作为参数，例如：

```c
typedef struct data
{
    int a[1000000];
    int b[1000000];
}data, *pdate;

int f(pdata p)
{
    pdata->a ...
    pdata->b ...
}
```

但是频繁的指针操作并不优雅，因此我们使用引用。

```cpp
int f(data& d)
{
	d.a ...
	d.b ...
}
```

在这里我们可以像使用一个 data 类型一样使用 data& 类型。

引用和其实体共享一个对象，例如

```cpp
int a = 2;
int &b = a;
b = 3;
```

此时 a 也被改变。

### class

类是类似结构体的数据类型。

```cpp
struct A
{
    int a;
} a;

class B
{
    int b;
} b;

int main()
{
    printf("%d\n", a.a);
    printf("%d\n", b.b);
}
```

类的内容默认对外不开放。

类内可以定义函数（方法）。

```cpp
class B
{
public:
    int print();
private:
    int b;
} b;
```

public 表示类外可以访问，private 表示类外无法访问。

这时候我们可以使用 b.print() 来输出，但是不能修改 b.b 的内容。

特别的，cpp 中的 struct 也不同于 c 中的，它更像是被修改了默认访问权限的 class。

---



## 内联与宏

### 内联的好处

宏是一种功能强大的预处理工具。

但是含有隐患。

```c
##define mult(a, b) a*b
```

如果使用

```c
int x = mult(3+2,2+3);
```

如常理所想应该是 x 被赋值为 (3+2)*(2+3) == 25，但是实际上，这句在展开后会成为

```c
int x = 3+2*2+3; // x == 10
```

诚然，我们可以采用加括号的方式来封闭宏，但是总有力所不逮之处（为什么不行？思考题）。

因此，cpp 创建了内联的语法。

```
inline int mult(int a, int b) { return a*b;}
int x = mult(3+2,2+3);
```

这时候，第二行`int x = mult(3+2,2+3);` 就相当于展开成这样的三行：

```cpp
int a = 3+2;
int b = 2+3;
int x = a*b;
```

可以确保其正确性。

### 为什么还要宏？

内联如此方便、强大、安全，为什么要宏呢？

这里是宏的一些（内联做不到的）用法：

```cpp
##include <cstdio>

int test_function(void);

int main()
{
    test_function();
	printf("%s\n", __FUNCTION__);
}

int test_function(void)
{
	printf("%s\n", __FUNCTION__);
}
```

---

例如上面的 `mult` ，也可以使用宏的方式来正确处理。

```cpp
##define mult(a, b) ((a)*(b))
```

但是，它在维持原有参数结构的情况下，不能做到和内联一样安全。
