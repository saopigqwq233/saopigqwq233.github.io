---

layout:post

title:"关键字使用"

date:2024-2-21

tags:\[c\\c++\]

comments:true

author: saopigqwq233

---

# 关键字使用

## 1. const

const关键字是一个修饰符，所谓“修饰符”，就是在编译器进行编译的过程中，给编译器一些“要求”或“提示”，但修饰符本身，并不产生任何实际代码。就 const 修饰符而言，它用来告诉编译器，**被修饰的这些东西，具有“只读”的特点**。在编译的过程中，一旦我们的代码试图去改变这些东西，编译器就应该给出错误提示。  

尽可能使用const，可以帮助我们避免很多错误，提高我们的程序健壮性。

### 1.1 与变量一起使用

#### （1）const int

最基本的方式，便是const修饰变量，使之成为一个常变量----即不可以通过此变量名修改变量值。

```c++
void test1(){
    const int b=10;//b不可被修改
    b=20;//error
}
```

```c++
void test2(){
    const int b =1;
    const int* const pb=&b;//指向const int 类型的const指针，pb值不可被修改
    pb=NULL;//erro
}
```

- 需要注意的是，const与指针\*的关系,**当const在\*前**，说明该指针指向一个常变量，**当const在\*之后**，说明该指针是常指针，指针值不可修改。  
  
  ```c++
  void test3(){
      const int a =10;
      const int *pa=&a;
      pa =NULL;//ag
      *pa = 20;//error
      int b=10; 
      int* const pb = &b;
      pb = NULL;//error 
      *pb = 20;//ag
  }
  ```

- **允许把非const对象的地址赋给指向const对象的指针**。
  这样，通过指针无法改变对象值

```c++
  void test4(){
      int b=10;
      const int *pb = &b;
      *pb = 20;//error
  }
```

  同样的，常引用也无法改变非常对象值

```c++
  void test5(){
      int a=10;
      const int& b =a;
      b = 20;//error
  }
```

- 此外，非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。

#### （2）const对象赋值时的转化

- const对象地址无法赋给指向非const对象的指针
  
  ```c++
  void test6(){
      //sample1
      const int a=10;
      int *pa =&a;//error
  
      //sample2
      int b =20;
      const int *pb=&b;
      int *p=pb;//error
  }
  ```

如果用const修饰指针函数返回值，那么此返回值无法赋值给非const指针

```c++
int a=10;
const int* func1() {
    return &a;
}
void test7(){
    int *p=func1();//error
}
```

### 1.2 在类中使用const

在类中，const可以修饰成员变量和成员函数，其中，受const修饰的成员函数**不能修改成员变量值、不能调用非const成员函数**。

```c++
class Person{
private:
    char* _name;
    int _age;
    const int _sex;
public:
    Person(char* name,int age,int sex):_age(age),_sex(sex){
        _name=new char[strlen(name)+1];
        strcpy(_name,name);
    }

    int add_age(int add=1)const{
        //const类型函数不可对成员变量做修改
        _age+=add;//error
        //const类型函数不可调用非const类型成员函数
        get_sex();//error
    }
    int addd_age(int age=1){
        _age+=age;
    }
    int get_sex(){cout<<"no const "<<_sex<<endl;}
};
```

- const类型成员变量必须从1.初始化列表 2.定义时（c++11）3.和static(c++11)合用在类外 进行从初始化
  
  ```c++
  class sample1{
  public:
      const int a=1;
  };
  
  class sample2{ 
  public: 
  const int a; 
  sample2():a(1){}
   };
  
  class sample3{ 
  public: 
  const static int a;
   }; 
  const int sample3::a=1;
  ```
  
  

## 2. static

### 2.1修饰函数内部变量

无论是函数还是类中的   static修饰的变量，当变量声明为static时，空间**将在程序的生命周期内分配**。即使多次调用该函数，静态变量的空间也**只分配一次**，前一次调用中的变量值通过下一次函数调用传递。

```c++
void test8(){
 static int a=0; 
 a++;
 cout<<a<<' ';
 cout<<&a<<endl;
}
int main() 
{ 
for (int i = 0; i < 10; ++i)
     test8(); 
}
```

运行结果：

```bash
1 0x7ff74f747030
2 0x7ff74f747030
3 0x7ff74f747030
4 0x7ff74f747030
5 0x7ff74f747030
6 0x7ff74f747030
7 0x7ff74f747030
8 0x7ff74f747030
9 0x7ff74f747030
10 0x7ff74f747030

进程已结束，退出代码为 00
```

可见static修饰变量a只初始化一次，后续多次调用test8函数，使用的是同一个a。

- 不过需要注意的是，函数中的static变量**作用域只在函数内部，外部无法访问**

### 2.2修饰类中成员变量

static成员变量只会被初始化一次，在同类所有对象间共享。不同对象，不允许static变量存在多个副本

```c++
class sample4{
private:
    int a;
    static int b;
public:
    sample4():a(1){}
    void add_b(){
        b++;
    }
    int get_b(){return b;}
};
int sample4::b = 0;//static成员变量初始化方式
int main(){
    sample4 s1,s2,s3;
    s3.add_b();
    s3.add_b();
    cout<<s1.get_b()<<endl;
    return 0;
}
```

运行结果：

```bash
2

进程已结束，退出代码为 0
```

如果输出这些对象中b的地址会发现：

```c++
class sample4{
private:
    int a;
    static int b;
public:
    sample4():a(1){}
    void add_b(){
        b++;
    }
    int get_b(){return b;}
    void print_b_index(){
        cout<<&b<<endl;
    };
};
int sample4::b = 0;
int main(){
    sample4 s1,s2,s3;
    s1.print_b_index();
    s2.print_b_index();
    s3.print_b_index();
    return 0;
}
```

运行结果：

```bash
0x7ff6f43f7030
0x7ff6f43f7030
0x7ff6f43f7030

进程已结束，退出代码为 0

```

可以看出，static成员变量在多个实例对象间共享。

- 事实上，类中的成员变量不依赖类的实例化而存在
  
  ```c++
  class sample5{
  public:
      static int a;
      sample5(){
          cout<<"init class"<<endl;
      }
      ~sample5(){
          cout<<"des class"<<endl;
      }
  };
  int sample5::a=1;
  int main(){
      cout<<sample5::a<<' '<<&sample5::a<<endl;
      return 0;
  }
  ```

运行结果：

```bash
1 0x7ff6d58b3000

进程已结束，退出代码为 0
```

#### 2.2.1 static成员变量的生命周期

**static成员变量在非static对象析构后仍然存在**

```c++
class sample5{
public:
    static int a;
    sample5(){
        cout<<"init class"<<endl;
    }
    ~sample5(){
        cout<<"des class"<<endl;
    }
};
int sample5::a=222;
int main(){
    if(true) { sample5 s1, s2, s3; }//s1,s2,s3生命周期仅仅在if中

    cout<<sample5::a<<endl;
    return 0;
}
```

运行结果：

```bash
init class
init class
init class
des class
des class
des class
222

进程已结束，退出代码为 0
```

### 2.3修饰类中成员函数

- static函数不依赖于对象实例化存在

- static函数无法调用非static成员--非static的成员变量\函数可能没有实例化
  
  ```c++
  class sample6{
  public:
      static int a;
      int b;
  
      //static函数不依赖于对象实例化存在
      static void test(){
          cout<<"hello static"<<endl;
  
          //static函数无法调用非static成员--非static的成员变量\函数可能没有实例化
  //        cout<<b<<endl;//error
      }
  };
  int sample6::a=10;
  int main(){
      sample6::test();//sample6类并未实例化
      return 0;
  }
  ```
  
  运行结果：
  
  ```bash
  
  ```
