---
title: C语言结构体大小分析
author: saopigqwq233
date: 2022-04-05
---

# C语言结构体大小分析

## 一，基本类型

C语言自带的数据类型大小如下

| 数据类型        | 大小（字节） |
| ----------- |:------:|
| char        | 1      |
| short       | 2      |
| int         | 4      |
| long        | 4或8    |
| float       | 4      |
| double      | 8      |
| long double | 16     |

## 二，自定义类型---struct

C语言除了以上这些基本类型，还支持用户自己定义数据类型

类似于一下形式：

```c
struct Student{
    char name[10];//学生姓名
    int age;//学生年龄
};
```

这里自定义了一个**struct Student** 类型的数据类型，包含有**字符数组**和**整形**两种内容。

## 三，自定义结构体大小分析

### 1.错误示范

如果直接按照结构体内的成员类型相加，那么如上**struct Student**类型的大小应该是

*10+4=14* 

似乎没什么问题，但是，当我们用以下代码做测试时，却发现出了问题

```c
#include"stdio.h"
#include"stdlib.h"
struct Student{
    char name[10];//学生姓名
    int age;//学生年龄
};
int main()
{
    printf("%d\n",sizeof(struct Student));//测试struct Student大小
    system("pause");
    return 0;
}
```

运行结果
![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405100545240-1927080210.png)


**可以发现，结构体大小并非结构体成员大小简单相加**

为什么会出现这样的结果？这是因为C语言中存在一种行为叫结构体内存对齐。

### 2.内存对齐

结构体成员在内存存放时，编译器会在结构体成员之间添加填充字节，以保证结构体成员的对齐要求。

#### 2.1对齐规则

##### 1)结构体第一个成员永远放在0偏移处

在C语言中，结构体第一个成员的地址和整个结构体的初始地址是相同的，也就是说，结构体的第一个成员始终位于结构体的初始地址处。

可以用以下代码证明：

```c
#include"stdio.h"
#include"stdlib.h"
struct student {
    char name[20];
    int age;
    float score;
};
int main()
{
    struct student stu;
    printf("%p\n%p\n",&(stu),&(stu.name));
    system("pause");
    return 0;
}
```

运行结果如下（不同设备上运行的数值可能不相同，但是一台设备上两行的数据相同）:
![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405100628390-1894499373.png)

可以看到，结构体变量的地址和结构体变量第一个成员的地址是相同的。

##### 2）从第二个成员开始，以后的每个成员的地址距离都要对齐到某个对齐数的整数倍处，这个对齐数是：*（成员自身大小和默认对齐数）的较小值*

这句话是什么意思呢，让我们先看一个例子：

```c
#include"stdio.h"
struct S
{
    char a;
    int b;
    char c;
    long long d;
}s;//创建结构体变量s
int main()
{
    printf("结构体大小:%d\n",sizeof(struct S));
    printf("各个成员的地址:\n");
    printf("%p char a\n",&(s.a));
    printf("%p int b\n",&(s.b));
    printf("%p char c\n",&(s.c));
    printf("%p long long d",&(s.d));
    return 0;
}
```

运行结果如下：

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405100652045-1637426794.png)

需要注意的是，%p是以16进制的格式进行输出，最后一个long long 型数据d的大小为8字节，则其结束地址应该是7d057

因此，结构体大小：7d057<sub>(16)</sub>-7d039<sub>(16)</sub>=18<sub>(16)</sub>=24<sub>(10)</sub>

**接下来我们将以excel表格代表内存空间，分析每个成员在内存的分布** 

***其中，D列的0到23代表每个地址距离起始地址的偏移量***

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405100723794-283498635.png)


@1 首先是 **char a**,已知结构体第一个成员永远放在0偏移处，且char 只占1字节，那么a在内存的分布暂时是这样的

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405100732352-1483753861.png)


@2 接下来是**int b**,第二个成员要对齐到对齐数的整数倍，也就是说，它的起始地址的偏移量必须是对齐数的整数倍。

**对齐数**：1）数据类型自身的对齐数：char型数据自身对齐值为1字节，short型数据为2字节，int/float型为4字节，double型为8字节。

2）默认对齐数：VS2013默认对齐数为8，或#pragma pack (value)时的指定对齐值value。

3）数据成员、结构体的有效对齐数：自身对齐值和指定对齐值中较小者，即有效对齐值=min{自身对齐值，当前指定的pack值}。

**需要注意的是gcc无默认对齐数*

由于我使用的是gcc编译器，则**int b**的对齐数是其自身对齐值**4**,如果使用的是Visual Studio这个IDE，那么其对齐数为**min(4,8)=4**。

因此，编译器将会在成员**char a**后再填补3个字节，使**int b**对齐，内存分布如下：

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101954193-9080476.png)


@3 接下来是**char c**，其对齐数是**min(1,8)=1**，大小是1字节，那么，直接接在偏移量为8的地方

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101201589-1293016426.png)


@4 最后一个成员是**long long d**,对齐数是8，大小8字节，则需要对齐到8*2=16这个偏移量的地址，并在char c占用内存后填补7个字节，内存分布如下：

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101207316-438579325.png)


##### 3）结构体大小是所有成员对齐数中最大对齐数的整数倍

当最后一个结构体成员存放后，如果结构体大小不是所有成员的对齐数中最大对齐数的整数倍，那么会在结构体最后的成员后补字节。

```c
#include"stdio.h"
struct S{
    int a;
    char c;
}s;
int main()
{
    printf("%d struct S\n",sizeof(struct S));
    printf("%p int a\n",&(s.a));
    printf("%p char c\n",&(s.c));
    return 0;
}
return 0;
}
```

接下来我们分析结构体成员是如何分布在内存中的

@1 首先是**int a**，对齐数为4，大小是4个字节，由于是第一个成员，直接放在0偏移处

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101232618-781907198.png)


@2 其次是**char c**，对齐数为1，大小是一个字节，则可以存放在偏移量为4的地方

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101237924-1692565266.png)


@3 我们发现，如果结构体到这里就分配结束，那么结构体大小应该为5，但是实际情况却是结构体大小为8。实际上结构体也要进行内存对齐。

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101604784-1231494932.png)

此结构体中**int a和char c**的对齐数分别为**4和1**，结构体对齐数是成员对齐数中的最大对齐数，则此结构体对齐数大小**MAX(4,1)=4**，那么，就需要在char c后填补字节到结构体大小为8.

##### 4）嵌套结构体中子结构体对齐到子结构体自己成员的最大对齐数的整数倍

@1 ****offsetof(type, member-designator)**** 求偏移量宏

        此库宏需要包含头文件“stddef.h”，会生成一个类型为size_t的整形数，代表该成员在内存中距离起始地址的偏移量。

        实例可参考：[C 库宏 – offsetof() | 菜鸟教程 (runoob.com)](https://www.runoob.com/cprogramming/c-macro-offsetof.html)[]()

      

话不多说，上代码：

```c
#include"stdio.h"
#include"stddef.h"
struct stu{
    int name;
    double grades;
};
struct team{
    char name[6];
    struct stu Students;
    int num_stu;
};
int main()
{
    struct team Team;
    printf("%d struct stu\n",sizeof(struct stu));
    printf("%d struct team\n",sizeof(struct team));
    printf("%p char name[10]\n"
    "%p struct stu Students\n"
    "%p int num_stu\n",(Team.name),&(Team.Students),&(Team.num_stu));
    printf("%d struct stu Students\n%d int num_stu\n",offsetof(struct team,Students),offsetof(struct team,num_stu));
    return 0;
}
```

运行结果如下：

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101620522-1759266773.png)


@2 根据上面三个规律，可以得出**struct stu**的大小是16，接下来我们看看**struct team**的成员是怎么分布的

@3 首先是**char name[6]**，第一个成员直接占用6字节，我知道你们都懂😊

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101713423-1149171922.png)

@4 然后来到了我们的重头戏，**struct stu**,首先，这个子结构体成员最大的对齐数是**8(double)**,所以，需要对齐到偏移量为8的地方，符合**offsetof(struct team,Students)** 返回值为8的结果。因此，还需要在第一个成员后补两个字节，子结构体大小为16，占用16个字节
![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101719871-895842435.png)


@5 最后一个成员**int num_stu**,占用4个字节，对齐数为4，24=4\*6，所以对齐到24偏移量处。最后一个成员放进去后，结构体大小为28，不等于**结构体对齐数（成员的最大对齐数）8**的整数倍，所以，需要再最后一个成员后补上4个字节，让结构体大小为32=8\*4.

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101754407-313883693.png)


#### 2.2对齐数的设置

编译时的默认对齐数可以通过**\#pragma pack(n)** 来设置，它会影响结构体中每个成员的有效对齐值，有效对齐值是编译器的对齐数和成员大小中较小的那个。例如，如果编译器的对齐数是4，而结构体中有一个double类型的成员（占8字节），那么该成员的有效对齐值是4，而不是8。

上代码：

@1 首先我们看看不带#pragma pack(n)的：

```c
#include"stdio.h"
#include"stdlib.h"
struct stu{
	int age;
	double grades;
};
int main()
{
	printf("%d\n",sizeof(struct stu));
	return 0;
}

```

先分析一下，**int age**占用四个字节，**double grades**对齐数为8，则需要补4个字节到int age，double grades自己占用8个字节，所以结构体大小为16个字节，看看运行结果吧：

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101809345-896098190.png)


果然如此，那加上**prama pack(4)** 呢。

@2 带**pragma pack(4)** 

上代码(*其实只是多了一个#pragma pack(4)而已😏*):

```c
#pragma pack(4)
#include"stdio.h"
#include"stdlib.h"
struct stu{
	int age;
	double grades;
};
int main()
{
	printf("%d\n",sizeof(struct stu));
	return 0;
}
```

运行结果：

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101824030-1584896892.png)


现在编译器对齐数为4。首先，**int age**占用4个字节，然后，**double grades** 对齐数=MIN(自己大小8，编译器对齐数4)=4，所以不用在age后补字节，grades直接从偏移量为4的地方开始。两个成员放完后，结构体大小为12，结构体对齐数为MAX（成员们的对齐数）=4，12=4*3，则无需补字节。

![](https://img2023.cnblogs.com/blog/2951836/202304/2951836-20230405101832052-854446439.png)


---

*感谢你阅读我的博客，如果你对我的内容有任何的意见、建议或者问题，欢迎在评论区留言，我会尽快回复。如果你发现了我的错误或者疏漏，也请不吝指正，我会及时修改。希望我的博客能对你有所帮助，也期待与你的交流和分享。*
