# C++ string类简介
C++ string类存储一个字符串，并包含多种方法
## 一、初始化

- C++ string类有多种初始化方法：
- 空初始化
- 复制构造
- 子串构造（下标+长度型）
- 数组构造（接受字符数组）
- 填充构造  
```c++
void test1(){
    //空构造
    string s1;
    //拷贝构造
    string s2(s1);
    string s3 = s1;
    //数组构造
    string s4("abcdefg");
    //子串构造
    string s5(s4,3,4);
    //填充构造
    string s6(10,'a');
}
```
## 二、流插入提取操作
string类已经实现流插入和提取操作
```c++
void test2(){
    string s1("abcdefg");
    cout<<"s1:"<<s1<<endl;
    string s2;
    cin>>s2;
    cout<<"s2:"<<s2<<endl;
}
int main(){
    test2();
    return 0;
}
```
运行结果：
```shell
s1:abcdefg
asd
s2:asd
进程已结束，退出代码为 0
```
- 需要注意的是，cin默认输入到空格和换行结束
```c++
void test3(){
    string s1;
    cin>>s1;
    cout<<s1;
}
int main(){
    test3();
    return 0;
}
```
运行结果：
```shell
abc def g
abc
进程已结束，退出代码为 0
```
可以看出，如果输入的有空格，那么cin将停止读取，只保存空格之前的字符（不包含空格）  
如果想要保存整个一行的内容，需要用到方法getline  
```c++
void test4(){
    string s1;
    getline(cin,s1);
    cout<<s1<<endl;
}
int main(){
    test4();
    return 0;
}
```
运行结果：
```shell
abc def g
abc def g
进程已结束，退出代码为 0
```
## 二、迭代器
string类，以及C++其它容器，都具有迭代器  。
可以把迭代器想象是指向容器单个元素的指针，并且可以顺序得遍历容器元素。
迭代器可以分为以下4类：
- 正向迭代器
- 逆向迭代器
- 常正向迭代器
- 常逆向迭代器
### 1.正向迭代器
```c++
void test5(){
    string s1("abcdefgh");
    for(auto i = s1.begin();i!=s1.end();i++){
        cout<<*i<<'&';
    }
    cout<<endl;
    for (auto i = s1.end(); i != s1.begin() ; --i) {
        cout<<*i<<'*';
    }
    cout<<endl;
}
int main(){
    test5();
    return 0;
}
```
运行结果：
```shell
a&b&c&d&e&f&g&h&
 *h*g*f*e*d*c*b*

进程已结束，退出代码为 0
```
从上可以看出，正向迭代器从begin到end是一个左闭右开的范围，且迭代器之间最好不要写大于小于号，
因为除了string这样可能是连续存储，其它容器可能是非连续存储，大于小于无法用于判别谁先谁后
### 2.逆向迭代器
```c++
void test6(){
    string s1("abcdefgh");
    for(auto i = s1.rbegin();i!=s1.rend();i++){
        cout<<*i<<"&";
    }
    cout<<endl;
    for(auto i = s1.rend();i!=s1.rbegin();i--){
        cout<<*i<<"*";
    }
    cout<<endl;
}
int main(){
    test6();
    return 0;
}
```
运行结果：
```shell
h&g&f&e&d&c&b&a&
 *a*b*c*d*e*f*g*

进程已结束，退出代码为 0
```
同样是左闭右开
### 3.常迭代器
常迭代器类似于迭代器，只不过不能常迭代器修改指向元素
## 三、内存空间操作  
string类包含很多关于容量的方法
- size() 返回字符串存储空间大小
- length() 返回字符串长度
- resize(size_t n,char c=null) 将字符串长度变成n，但不改变存储容量，如果n大于字符串长度，那么将以字符c填充，如果未指定c，那么以null填充
- capacity() 返回存储空间大小
- reserve(size_t n=0)为字符串空间预留大小为n的空间，避免多次扩容，提高效率
- clear() 删除字符串，但保留存储容量
- empty() 测试字符串是否为空
- shrink_to_fit() 使存储容量调整至字符串大小
## 四、取值操作
string类重载了[ ]运算符，使string可以和字符数组一样使用
```c++
void test7(){
    string s1("abcdefgh");
    for (int i = 0; i < s1.length(); ++i) {
        cout<<s1[i]<<' ';
    }
    cout<<endl;
}
int main(){
    test7();
    return 0;
}
```
运行结果：
```text
a b c d e f g h

进程已结束，退出代码为 0
```
## 五、增、删、查操作
### 1.增
字符串的增加可以是连接，插入（头、中）  
连接：
#### 重载+、+=符号 
```c++
void test8(){
    string s1("hello");
    string s2 = s1 + ' ' + "world";
    cout<<s2<<endl;
    s2 += "233";
    cout<<s2<<endl;
}
int main(){
    test8();
    return 0;
}
```
运行结果：
```text
hello world
hello world233

进程已结束，退出代码为 0
```
可以实现字符串的拼接和赋值 
#### append() 
```c++
void test9(){
//    以字符串为参数
    string s1("hello");
    s1.append(" world");
    cout<<"s1:"+s1<<endl;

//    以string类为参数
    string s2;
    s2.append(s1);
    cout<<"s2:"+s2<<endl;

//    以字符串为参数，并声明长度
    string s3("hello");
    s3.append(" world",4);
    cout<<"s3:"+s3<<endl;
    
//    以string类为参数，并声明开始位置和长度
    string s4;
    s4.append(s2,6,5);
    cout<<"s4:"+s4<<endl;
//    指定拼接字符个数，拼接字符
    string s5;
    s5.append(10,'c');
    cout<<"s5"+s5<<endl;
//    以string迭代器区间为参数
    string s6;
    s6.append(++s5.begin(),--s5.end());
    cout<<"s6:"+s6<<endl;    
}
int main(){
    test9();
    return 0;
}
```
运行结果：
```text
s1:hello world
s2:hello world
s3:hello wor
s4:world
s5:cccccccccc
s6:cccccccc

进程已结束，退出代码为 0
```
#### push_back(char c)  尾插字符c
#### insert() 
我们大致能知道，  
插入的位置可以是
- 下标索引
- 迭代器  

插入内容可以是
- （n个）字符
- 字符串、字符串子串
- string类、string的迭代器区间
### 删
删除string中的一个字符或一段字符，可以是下标+长度，也可以是迭代器（区间）。
#### pop_back() 尾删
#### erase()
```c++
void test10(){
    char str[] = "hello world";
    string s1(str);
    s1.erase(3);//从下表3开始删到最后
    cout<<"s1:"+s1<<endl;

    string s2(str);
    s2.erase(3,4);//从下标3开始删4个字符
    cout<<"s2:"+s2<<endl;

    string s3(str);
    s3.erase(++s3.begin());//删掉第二个元素
    cout<<"s3:"+s3<<endl;
    s3.erase(--s3.end());//删掉最后一个元素
    cout<<"s3:"+s3<<endl;

    string s4(str);
    s4.erase(++(s4.begin()),--s4.end());//删掉左闭右开的区间
    cout<<"s4:"+s4<<endl;
}
int main(){
    test10();
    return 0;
}

```
运行结果：
```text
s1:hel
s2:helorld
s3:hllo world
s3:hllo worl
s4:hd

进程已结束，退出代码为 0
```
### 查
查找第一个出现的字符，返回下标
#### find()
```c++
void test11(){
    char str[] = "hello world";
    string s1(str);
    cout<<s1.find('e')<<endl;//从头开始找字符e

    cout<<s1.find('w',3)<<endl;//从下标3开始找

    cout<<s1.find("world",0,3)<<endl;//从下标0开始找和字符串前3个字符匹配的字符起始位置

}
int main(){
    test11();
    return 0;
}
```
运行结果：
```text
1
6
6

进程已结束，退出代码为 0
```
至于剩下的几种查找函数:
- rfind() 逆向查找
- find_first_of(str) 找到string和str中任意字符匹配的第一个字符
- find_last_of(str) 找到string和str中任意字符匹配的倒数第一个字符
- find_first_not_of(str) 找到string和str中任意字符不匹配的第一个字符
- find_last_not_of(str) 找到string和str中任意字符不匹配的倒数第一个字符

### 子串
#### substr(size_t pos=0,size_t len = npos)
pos指的是子串开始下标，npos默认指的是到字符串最后一位，若提供len = n，则从pos开始复制n个字符
到子串
## 六、兼容C部分
C语言中没有string类，C++提供了c_str()方法将string转化为C语言字符数组
```c++
void test12(){
    char str[] = "hello world";
    string s1(str);
    //printf("%s",s1);//error
    printf("%s\n",s1.c_str());
}
int main(){
    test12();
    return 0;
}
```
运行结果：
```text
hello world

进程已结束，退出代码为 0
```
