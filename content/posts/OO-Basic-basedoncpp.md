---
title: "面向对象入门"
tags: ["社团课程"]
date: 2019-11-11T15:42:01+00:00
draft: false
---

{{< blockquote author="Tom Cargill" >}}
If you think C++ is not overly complicated, just what is a protected abstract virtual base pure virtual private destructor and when was the last time you needed one?
{{< /blockquote >}}


<!--more-->

本文通篇将会构造一个类，并以此类的构造过程为讲解顺序。

本文需求的先修知识：cpp iostream (`std::cin,std::cout` & `std::endl`)，cpp class defination, c struct, c pointer and cpp keywords (`const, new, delete, class, public, private` [`protected, pure, virtual`])

<!-- toc -->

---

## 编程目标

在命令行窗口编程时，我们有时候会希望输出一些特定的格式，如：（每行前面有一个空格）

```
 Hello,
 World!
```

像这样的，基于字符构造的，有特定格式、内容的整体，我们姑且称之为 “字符图像”。
我们可以给字符图像加上边框：

```
.-------.
| Hello,|
| World!|
'-------'
```

更多地，可能会希望支持两个这样的字符图拼接，形成新的图像。

```
.-------.
| Hello,| 你好，
| World!| 世界！
'-------'
```

以及希望能够修改边框的样式等等······

---

## 面向过程可以吗？

如我们所学过的 C/Cpp， 我们可能会想到这样做：

```cpp
int main()
{
    const char *content[] = {"Hello,", "World!"};
    int left_padding = 1;
    for(int i=0; i<2; ++i)
    {
        for(int j=0; j< left_padding; ++j) printf(" ");
        printf("%s", content[i]);
        printf("\n");
    }
}
```

运行可以得到输出：

```
 Hello,
 World!
```

但是很显然，这样无法（很难）对多个字符图进行操作。

## C 语言也能面向对象吗？

我们需要将一个图视为一个对象，一个物体，才能方便地管理、输出两个（及以上）的图。

### C 语言面向对象

*如果下面的例程编译错误，请修改文件后缀为 cpp 或添加编译选项 -std=c99 或 -std=c11*

我们用 C 语言来试试看：

```c
struct tagPicture
{
	char **m_content;
	int m_padding_left, m_padding_top;
	int m_height, m_width;
};

typedef struct tagPicture Picture;
```

在这个结构体 `struct tagPicture` （或记作`Picture`）中，我们使用 `m_content` 来保存图像内容，`m_height, m_width` 表示图像的宽与高，`m_padding_left` 和`m_padding_top` 表示图像左侧和上侧内边界大小。

怎么创建一个 `Picture` 呢？
在 C 语言中，我们需要定义一个函数来创建该类型：

```c
// 此处需要库文件 stdlib.h 和 string.h
Picture getPicture(const char **_ArrayOfCharArray, int _Lines)
{
	Picture t;
	t.m_content = (char **) malloc(_Lines *sizeof(char *));
	t.m_padding_left = t.m_padding_top = 0;
	t.m_height = _Lines;
	t.m_width = 0;
	for(int i=0; i<_Lines; ++i)
	{
		int n = _ArrayOfCharArray[i] ? strlen(_ArrayOfCharArray[i]) : 0;
		if( t.m_width < n ) t.m_width = n;
	}
	for(int i=0; i<_Lines; ++i)
	{
		t.m_content[i] = (char *) malloc((t.m_width+1) * sizeof(char));
		strcpy(t.m_content[i], _ArrayOfCharArray[i]);
	}
	return t;
}
```

还需要设定一个输出用的函数：

```c
// 此处需要库文件 stdio.h
void printPicture(Picture _pic)
{
	for(int i=0; i<_pic.m_height; ++i)
	{
		if(!(i < _pic.m_padding_top))
		{
			for(int j=0; j<_pic.m_width; ++j)
			{
				if(j < _pic.m_padding_left) printf(" ");
				else if(_pic.m_content[j-_pic.m_padding_left])
					printf("%c", _pic.m_content[i-_pic.m_padding_top][j-_pic.m_padding_left]);
				else break;
			}
		}
		if(i+1 != _pic.m_height) printf("\n");
	}
}
```

接下来在主函数调用试试：

```c
int main()
{
	const char *content[] = {"Hello,", "World!"};
	Picture a = getPicture(content, 2);
//  a.m_padding_left = 1;
	printPicture(a);
}
```

编译运行后可以得到这样的输出：

```
Hello,
World!
```

我们在主函数中将 `a.m_padding_left` 设置为 1，则会看到：

```
 Hello
 World
```

### 有何缺陷？

首先，我们的这个图像单个输出、存储都没有问题，但是如果想要拼接，尤其是横向拼接，就很难做，其次，进行其他复杂操作时，不能较容易地存储结构信息。

### C语言课后习题

+ 课后习题 1 : 经过上面的示例，我们可以发现纵向拼接很容易，在此处不做详解，将 C 语言面向对象实现图像的纵向拼接这一任务留作习题。
+ 课后习题 2 : 纵向拼接实现很轻松，横向拼接相对来说难度略大，但聪明的读者一定能解决这个问题，请用 C 语言面向对象实现图像的横向拼接。
+ 课后习题 3 : 请用 C 语言面向对象实现给图像加边框，边框样式需要作为函数参数，如可以指定用 '\*' 来画边框。
+ 课后习题 4 : 请用 C 语言面向对象实现图像修改边框样式，例如将 '*' 样式的边框改为 '+' 样式。

## Cpp 面向对象

Cpp 的 class、struct 与 C 中的 struct 相似。

|                        | C struct | Cpp struct | Cpp class |
| :--------------------: | :------: | :--------: | :-------: |
|     Plain Old Data     |   true   |   false    |   false   |
| default access rights  |    /     |   public   |  private  |
| member function enable |  false   |    true    |   true    |

简单用法在此不做赘述。
不妨先实现上面程序的内容，让我们看看能否更加优雅。

### 构造和输出功能

我们姑且以此结构构造程序：

```
src
 |-- main.cpp
 |-- Picture.h
 '-- Picture.cpp
```

首先得定义这个类，并声明其方法。

```cpp
#ifndef __PICTURE__INC
#define __PICTURE__INC

class Picture
{
public:
	Picture(const char **_Strs, int _Lines);
	void print(void);
private:
	char **m_content;
	int m_width, m_height;
	int m_padding_left, m_padding_top;
};

#endif
```

接下来我们在主函数内直接定义 Picture 的实例 pic，并以此初始化字符串数组初始化这个实例。

此后，我们调用其输出方法。

```cpp
//main.cpp
#include "picture.h"

const char *initstrs[] = {"Hello,", "World!"};

int main()
{
	Picture pic(initstrs, 2);
	pic.print();
}
```

最后，我们实现这两个函数。

```cpp
// picture.cpp
#include "picture.h"
#include <cstdio>
#include <cstdlib>
#include <cstring>


Picture::Picture(const char **_Strs, int _Lines)
{
	m_content = (char **) malloc(_Lines *sizeof(char *));
	m_padding_left = m_padding_top = 0;
	m_height = _Lines;
	m_width = 0;
	for(int i=0; i<_Lines; ++i)
	{
		int n = _Strs[i] ? strlen(_Strs[i]) : 0;
		if( m_width < n ) m_width = n;
	}
	for(int i=0; i<_Lines; ++i)
	{
		m_content[i] = (char *) malloc((m_width+1) * sizeof(char));
		strcpy(m_content[i], _Strs[i]);
	}
}

void Picture::print(void)
{
	for(int i=0; i<m_height; ++i)
	{
		if(!(i < m_padding_top))
		{
			for(int j=0; j<m_width; ++j)
			{
				if(j < m_padding_left) printf(" ");
				else if(m_content[j-m_padding_left])
					printf("%c", m_content[i-m_padding_top][j-m_padding_left]);
				else break;
			}
		}
		if(i+1 != m_height) printf("\n");
	}
}
```

### 管理内存

对于 Picture 来说，管理内存比较容易，只需要给其添加一个析构函数即可。

```cpp
// picture.h
@@ -5,6 +5,7 @@
 {
 public:
        Picture(const char **_Strs, int _Lines);
+       ~Picture(void);
        void print(void);
 private:
        char **m_content;
```

```cpp
@@ -23,6 +23,13 @@
        }
 }

+Picture::~Picture(void)
+{
+       for(int i=0; i<m_height; ++i)
+               free(m_content[i]);
+       free(m_content);
+}
+
 void Picture::print(void)
 {
        for(int i=0; i<m_height; ++i)
```

### 怎么纵向连接？俯瞰继承关系

首先我们要知道，纵向连接后的结果也应该是一个 Picture，并能够继续纵向连接。

#### 我们应该怎样设计呢？

这里引入一个概念：继承

如下，是图形之间的继承关系。
我们可以认为，如果 A 指向 B ，那么 B 属于 A。

*这里涉及到面向对象设计的基本理念，目前主流思路有两种：is-a，has-a <del>和 like-a</del>。*

{{<mermaid>}}
graph TB;
	Shape["Shape"]-->Polygen["Polygen"];
	Shape["Shape"]-->Ellipse["Ellipse"];
	Polygen["Polygen"]-->Rectange["Rectange"];
	Ellipse["Ellipse"]-->Circle["Circle"];
	Rectange["Rectange"]-->Square["Square"];
{{</mermaid>}}

在我们这个例子中，我们有理由认为，Picture 是所有类型的基础。
我们可以想当然如此构造：

{{<mermaid>}}
graph TB;
	Picture["Picture"] --> vCatPicture["vCatPicture"];
	Picture["Picture"] --> hCatPicture["hCatPicture"];
{{</mermaid>}}

这时候出现了一个问题：我们连接两个图像的时候，应该使用类型本身呢，还是使用其指针呢？
例如上面的图形例子，Rectangle 类型（通常）可以无损转化为 Square 类型，但是 Rectangle 类型的指针不能转换为 Square 类型的指针。相反的，Square 类型（通常）不能无损转换为 Rectangle 类型，但是 Square * 类型却可以实现到 Rectangle * 的转换。

指针类型并不方便，而值类型（及引用类型）并不能支持这样的转换。

如果采用指针类型的话，我们需要定制这样的框架：

{{<mermaid>}}
graph TB;
	Picture["Picture"]-->BasicPicture["BasicPicture"];
	Picture["Picture"]-->vCatPicture["vCatPicture"];
	Picture["Picture"]-->hCatPicture["hCatPicture"];
	Picture["Picture"]-->FramedPicture["FramedPicture"];
{{</mermaid>}}

其中，`BasicPicture` 表示纯文本的图，`vCatPicture` 表示经过了一次垂直连接，`hCatPicture` 表示经过一次水平连接，`FramedPicture` 表示加框架的图。
在具体使用时，我们使用 Picture * 来表示所有可能的类型。

而值类型则不同，值类型只能从上游向下游转换（从亲代到子代），因此只能是这两种情况之一（或其组合形式）。

{{<mermaid>}}
graph TB;
	A["A"]-->D["D"];
	B["B"]-->D["D"];
	C["C"]-->D["D"];
	a["a"]==>b["b"];
	b==>c["c"];
	c==>d["d"];
{{</mermaid>}}

显然，这两种构造都无法用 is-a 来解释，用 has-a 解释也很勉强。因此，这是一种不合理的构造方式。

**因此，我们采用指针来进行运算，采用共同基类来实现**。

---

#### 我们需要哪些类型呢？

`BasicPicture` ：保存实际的图像（字符串）内容，及其宽度与高度。
`vCatPicture` ，`hCatPicture` ：保存其所连接的两图像指针
`FramedPicture` ：保存一个图像指针及其边框内容。

基于这些类型，我们可以容易地实现图像连接，图像解除连接，图像加边框，图像去边框以及修改内容。

---

#### 我们怎样管理内存？

之前的例子里，我们采用构造函数分配内存，析构函数释放内存的方式来处理内存问题。

但是，如下所示的，在断点处，`FramedPicture b` 的内容物（a）已经被析构，因此输出 b 可能是一个非法操作。

```cpp
FramedPicture b;
{
    BasicPicture a = ...;
    b = enframe(a);
}
// breakpoint
```

经过一系列思考，我们可以推理出引用计数的必要性。

### 类型的设计

#### Picture 类

经过思考，我们设计这样一个 `Picture` 类。

```cpp
class Picture
{
public:
	Picture(void);
	Picture(const char* const *, int);
	Picture(const Picture&);
	~Picture(void);

	Picture& operator=(const Picture&);
	void reframe(int p, char c);
	void reframe(char c, char v, char h);
	Picture split(int id);
private:
	PictureEntity *ap ;
	Picture(PictureEntity* p);

	int width(void) const;
	int height(void) const;
	std::ostream& display(std::ostream&, int, int) const;
};
```

最上面的三个构造函数，分别有以下作用：

```cpp
Picture::Picture(void); // 构造一个空的 Picture 对象
Picture::Picture(const char* const *, int); // 根据字符串指针和其行数构造一个 Picture
Picture::Picture(const Picture&); // 拷贝构造函数，例如
```

`reframe` 函数用来重设框架格式。
`private` 内容则是对外不可见的具体实现。
`PictureEntity *ap` 是一个指针，这里的 `PictureEntity` 即为上面设计中的所有类型的基类 。
剩余的函数姑且不考虑。

#### PictureEntity 基类

这个类型，作为所有行为的具体呈现者，并且是一个基类，我们如此定义：

```cpp
class PictureEntity
{
public:
	PictureEntity(void);
	virtual ~PictureEntity(void) = 0;
	virtual void incuse(void) = 0;
	virtual int decuse(void) = 0;
	friend class Picture;
protected:
	int use;
	virtual int width(void) const = 0;
	virtual int height(void) const = 0;
	virtual ostream& display(ostream&, int, int) const = 0;
	ostream& fillEmpty(ostream&, int, int, char _ch = ' ') const;
	static int max(int a, int b);
	virtual void reframe(int pos, char c);
	virtual void reframe(char corner, char vlimit, char hlimit);
	virtual Picture split(int id) = 0;
}
```

首先看 `public:` 部分：
继承中的基类必须拥有虚析构函数，在这里，为了防止创建 `PictureEntity` 的实例，我们将其析构函数设置为纯虚析构函数。
其次，其拥有两个对引用进行处理的函数，且均为纯虚函数。

其次看 `protected` ：
`use` 为引用计数器。
其他的函数作用如其名所示。

这个类无法被实例化。

#### BasicPicture

```cpp
class BasicPicture : public PictureEntity
{
public:
	BasicPicture(const char* const*, int);
	~BasicPicture(void);
	void incuse(void);
	int decuse(void);

	int width(void) const;
	int height(void) const;
	ostream& display(ostream&, int, int)const;

	Picture split(int id);
private:
	char **m_data;
	int m_size;
	int *m_width;
	int m_maxwidth;
};
```

这个类型看起来更像是我们在[前面](#构造和输出功能) 里面实现的类型。其用法也与之相似。

#### FramedPicture

```cpp
class FramedPicture : public PictureEntity
{
public:
	FramedPicture(const Picture&, char c, char v, char h);
	FramedPicture(const Picture&, char ct, char cb, char v, char h);
	FramedPicture(const Picture&, char ct, char cb, char t, char b, char l, char r);
	FramedPicture(const Picture&, char c1, char c2, char c3, char c4, char t, char b, char l, char r);
	~FramedPicture(void);
	void incuse(void);
	int decuse(void);

	void reframe(int pos, char c);
	void reframe(char corner, char v, char h);

	int width(void) const;
	int height(void) const;
	ostream& display(ostream&, int y, int mw) const;

	Picture split(int id);

	friend Picture enframe(const Picture&);
private:
	Picture m_insidePic;
	char c1, c2, c3, c4;
	char l, r, t, b;
};
```

这个类型 `public:` 部分有大量构造函数，其余部分与上面的`BasicPicture` 相似。

#### VcatPicture

```cpp
class VcatPicture : public PictureEntity
{
public:
	VcatPicture(const Picture&, const Picture&);
	~VcatPicture(void);
	void incuse(void);
	int decuse(void);

	int width(void)const ;
	int height(void)const ;
	ostream& display(ostream&, int y, int mw) const;
	Picture split(int id);

	friend Picture operator&(const Picture&, const Picture&);
private:
	Picture top, bottom;
};
```

这个类表示 `top` 和 `bottom` 纵向连接。

#### HcatPicture

```cpp
class HcatPicture :public PictureEntity
{
public:
	HcatPicture(const Picture&, const Picture&);
	~HcatPicture(void);
	void incuse(void);
	int decuse(void);

	int width(void)const ;
	int height(void)const ;
	ostream& display(ostream&, int y, int mw) const;
	Picture split(int id);

	friend Picture operator|(const Picture&, const Picture&);
private:
	Picture left, right;
};
```

---

## 总述及效果展示

通过上面从[继承关系设计](#怎么纵向连接？俯瞰继承关系) 到 [类型的具体设计](#类型的设计) ，我们基本有了程序的框架。

在经过具体实现这些类型后，我们就得到了完整的[代码](/resources/archive/CharacterPicture.zip)。

### Cpp 课后习题

+ 课后习题 1 : 经过上面的示例，我们可以发现一个给定的图可以唯一确定一个（系列性的）输出动作，而 `std::string` 也可以唯一确定一个输出动作，请将一个给定的图转换为 `std::string` 。
+ 课后习题 2 : 请设计一个类，可以独立保存一张图及其结构信息。
+ 课后习题 3 : 习题 2 自然不难，能否将这个类的 `std::cin` 和`std::cout` 的对应操作重载，使其能够被保存和读取。
+ 课后习题 4 : 如果将`'*'` 视为黑色，`' '` 视为白色，那么任何一张只有黑白两色的图可以唯一确定一个字符图，请设计一个方法，能够根据一张图片（`.jpg` 或`.png`）生成一个字符图。（可以利用第三方库读入图片）
+ 课后习题 5 : 相信对于聪明的读者们，习题 4 比较容易。请在习题 4 的基础上，支持字符图的旋转、平移等图形操作。
+ 课后习题 6 : 其实根据字符密度，[纯字符方式](/resources/text/thirdeye.txt)可以表现黑白灰色彩（单通道八位），请在习题 4 的基础上，生成一个可以生成黑白灰色彩的图。（如 `' '` 表示纯黑，`'■'` 表示纯白，`'*'` 表示一种灰色）
