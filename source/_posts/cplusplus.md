---
title: 蓝桥网课C++进阶实验
categories: C++
tags: C++
---

## 封装

### 初识类与对象

#### 类的有关函数
|概念|描述|
|---|---|
|构造函数|类的构造函数是一种特殊的函数，在**创建**一个新的对象时自动调用|
|析构函数|类的析构函数也是一种特殊的函数，在**删除**所创建的对象时自动调用|
|拷贝构造函数|拷贝构造函数，是一种特殊的构造函数，它在创建对象时，是使用同一类中**之前创建的对象来初始化新创建的对象**|
|友元函数|友元函数可以访问类的 **private 和 protected** 成员|
|内联函数|通过内联函数，编译器试图在调用函数的地方**扩展函数体中的代码**|
|类成员函数|类的成员函数是指那些把定义和原型**写在类定义内部的函数**，就像类定义中的其他变量一样|
|类访问修饰符|类成员可以被定义为 public、private 或 protected。默认情况下是定义为 private|
|this 指针|每个对象都有一个特殊的指针 this，它指向对象本身|
|静态成员|类的数据成员和成员函数都可以被声明为静态的|

#### 类的定义
e.g.
``` cpp
#include <iostream>
#include <string>
using namespace std;

// class 类关键字、Student 类名
class Student
{
// 访问限制符 - 公有属性
public:
    Student() {}    // 构造函数
    ~Student() {}    // 析构函数

    // 成员函数
    void setName(string name) { this->name = name; }
    string getName() const { return name; };
    void setAge(int age) { this->age = age; }
    int getAge() const { return age; }
// 访问限制符 - 私有属性
private:
    // 数据成员
    string name;
    int age;
};
```
对于上述定义需要注意几点：
+ 可以没有解析和析构函数，系统自动生成
+ 除了private和public还有protected
+ private的数据成员，不能够直接访问，需要使用成员函数访问（比如这里的getAge()）

#### 实例化对象
实例化有**栈上和堆上**两种：
``` cpp
int main()
{
    // 栈上实例化
    Student stu1;
    stu1.setName("jake");
    stu1.setAge(15);
    cout << "My name is " << stu1.getName() << ", I'm " << stu1.getAge() << " years old." << endl;

    // 堆上实例化
    Student *stu2 = new Student;
    // 访问
    stu2->setName("Siri");
    stu2->setAge(5);
    cout << "My name is " << stu2->getName() << ", I'm " << stu2->getAge() << " years old." << endl;
    // 释放内存
    delete stu2;
    stu2 = nullptr;

    return 0;
}
```
对于两种实例化，需要注意下面几点：
+ 栈上实例化的对象，超出定义域的资源会被自动回收
+ 堆上实例化的对象，需要手动回收(delete)
+ delete释放堆上资源后，需要置空指针
+ 栈上对象使用"."访问，堆上对象使用"->"访问

#### string类
string类在namespace std中，常用操作如下：
|操作|解释|
|---|---|
|s.empty()|判断是否为空|
|s.size()|同s.length()|
|s1 + s2|拼接|
|s.at(i)|同s[i]|
|s.c_str()|返回char*，同s.data()，注意返回的指针不能操作，只读|
|stoi()|转int，类似的还有stol()，stoll()|
|stof()|字符串转 float, 还有 stod()，stold()|
|s.find(substr, begin)|从begin位置开始查找子串|
|s.rfind(substr)|反向查找子串|

#### 内联函数
在函数定义之前加上inline关键字，标识编译器在编译的时候，在每个调用了这个函数的地方都编译一遍这个函数，从而节省运行时调用函数的时间。
一般来说，内联函数是比较简单的函数，如果函数比较复杂，编译器会自动取消内联。

##### 类内联函数
在类内，与上述一致，也可以加入inline关键字。
即使没有inline，编译器也会自动尝试对简单的成员函数作为内联函数编译。

#### 类的定义与实例化
##### 类内定义
也就是在之前的例子中，在类里边定义的函数。
##### 类外定义
e.g.
在同一个文件中：
``` cpp
#include <iostream>
#include <string>
using namespace std;

class Student
{
public:
    Student();
    ~Student();
    void setName(string name);
    string getName() const;
    void setAge(int age);
    int getAge() const;
private:
    string name;
    int age;
};

Student::Student()
{
}

Student::~Student()
{
}

void Student::setName(string name)
{
    this->name = name;
}

string Student::getName() const
{
    return name;
};


void Student::setAge(int age)
{
    this->age = age;
}

int Student::getAge() const
{
    return age;
}


int main()
{
    Student stu;
    stu.setName("jake");
    stu.setAge(15);
    cout << "My name is " << stu.getName() << ", I'm " << stu.getAge() << " years old." << endl;

    return 0;
}
```
注意这里getName()和getAge()两个函数后面都加了const，意思是这两个函数**不允许修改类数据成员的值**。

除此之外，函数还可以定义在不同文件中。其中，类定义可以放在.h文件中，类内函数定义可以放在.cpp文件中，然后在其他文件中，通过`include<xxx.h>`就可以调用这个类：
``` cpp
// Student.h 文件
#ifndef __STUDENT__
#define __STUDENT__
#include <iostream>
#include <string>
using namespace std;

class Student
{
public:
    Student();
    ~Student();
    void setName(string name);
    string getName() const;
    void setAge(int age);
    int getAge() const;
private:
    string name;
    int age;
};

#endif // __STUDENT__
```

``` cpp
// Student.cpp 文件
#include "Student.h"    // 记得添加类头文件

Student::Student()
{
}

Student::~Student()
{
}

void Student::setName(string name)
{
    this->name = name;
}

string Student::getName() const
{
    return name;
};


void Student::setAge(int age)
{
    this->age = age;
}

int Student::getAge() const
{
    return age;
}
```

``` cpp
// main.cpp 文件
#include <iostream>
#include "Student.h"

int main()
{
    Student stu;
    stu.setName("jake");
    stu.setAge(15);
    cout << "My name is " << stu.getName() << ", I'm " << stu.getAge() << " years old." << endl;
    return 0;
}
```
编译时，需要使用如下命令：
``` shell
g++ main.cpp Student.cpp -o main
```

### 类的封装
官方定义：封装是将数据和处理数据的程序组合起来，仅对外公开接口，达到信息隐藏的功能。
类封装以后，用户就只能通过给定的接口操作private数据，且不知道数据处理过程。举一个例子，用户不知道芯片内部结构，只对引脚操作。

#### 类的访问权限
之前说过，类的关键字有public，protected，private：
+ public用户可以直接访问
+ protected用户需要继承类后访问
+ private用户不能直接访问
（类的继承：一个类在已经存在的类（基类）基础上，添加自己的函数与数据。如，动物类有eat，sleep函数，还可以派送一个狗类，继承自动物类并添加bark函数）

### 对象的生离死别
一个对象从实例化到销毁的历程为：申请内存->初始化列表->构造函数->参与计算->析构函数->释放内存
#### 内存分区
|内存区|用途|
|---|---|
|栈|函数参数、局部变量|
|堆|程序员分配、释放|
|全局|全局和静态变量|
|常量|常量|
|代码|逻辑代码的二进制|
**栈与堆对比：**栈由编译器自动分配回收，空间比较小，申请效率快；堆由程序员分配（C：malloc-free；C++：new-delete），空间很大，申请比较慢；
#### 构造函数
函数与类同名，可以有多个重载形式，如：
``` cpp
#include <iostream>
#include <string>
using namespace std;

class Teacher
{
public:
    // 1. 无参构造函数
    Teacher() {}
    // 2. 有参构造函数
    Teacher(string name, int age) {
        this->name = name;
        this->age = age;
    }
    // 3. 有参构造函数--全部有默认参数-->默认构造函数
    /*Teacher(string name = "jake", int age = 15) {
        this->name = name;
        this->age = age;
    }*/
private:
    string name;
    int age;
};
```
注意：
+ 根据重载函数的规则，例如示例代码 2 中有参构造函数与缺省构造函数不能同时使用，因为参数的个数和类型都是相同的。
+ 示例代码 2 中无参构造函数与缺省构造函数也不能同时使用，因为编译器无法识别是使用无参构造函数还是使用缺省函数。
+ 如果实例化对象不需要传递参数的构造函数统称默认构造函数。

注：C++函数重载需要形式参数不同
#### explicit关键字
explicit关键字用于类内构造函数之前，表示不允许隐式转换，具体可以看下面的示例代码：
``` cpp
#include <iostream>
#include <string>
using namespace std;

class A
{
public:
    // 默认 implicit（隐式转换）
    A(int) { }      // 转换构造函数
    A(int, int) { } // 转换构造函数 (C++11)
    operator bool() const { return true; }
};

class B
{
public:
    // 申明构造函数使用显示声明，不能隐式转换
    explicit B(int) { }
    explicit B(int, int) { }
    explicit operator bool() const { return true; }
};

int main()
{
    A a1 = 1;      // OK赋值初始化选择 A::A(int)
    A a2(2);       // OK：直接初始化选择 A::A(int)
    A a3 {4, 5};   // OK：直接列表初始化选择 A::A(int, int)
    A a4 = {4, 5}; // OK赋值列表初始化选择 A::A(int, int)
    A a5 = (A)1;   // OK：显式转型进行 static_cast
    if (a1) ;      // OK：A::operator bool()
    bool na1 = a1; // OK赋值初始化选择 A::operator bool()
    bool na2 = static_cast<bool>(a1); // OK：static_cast 进行直接初始化

//  B b1 = 1;      // err赋值初始化不考虑 B::B(int)
    B b2(2);       // OK：直接初始化选择 B::B(int)
    B b3 {4, 5};   // OK：直接列表初始化选择 B::B(int, int)
//  B b4 = {4, 5}; // err赋值列表初始化不考虑 B::B(int,int)
    B b5 = (B)1;   // OK：显式转型进行 static_cast
    if (b2) ;      // OK：B::operator bool()
//  bool nb1 = b2; // err赋值初始化不考虑 B::operator bool()
    bool nb2 = static_cast<bool>(b2); // OK：static_cast 进行直接初始化
}
```
class A没有explicit，所以可以使用`A a1 = 1`，`A a4 = {4,5}`这样的初始化方式，而class B可以。
#### 拷贝构造函数
语法`类名(const 类名 &变量)`
用法如下：
``` cpp
#include <iostream>
#include <string>
using namespace std;

class Teacher
{
public:
    // 1. 无参构造函数
    Teacher() {}
    // 2. 有参构造函数
    Teacher(string name, int age) {
        this->name = name;
        this->age = age;
    }
    // 3. 拷贝构造函数
    Teacher(const Teacher &tea) {
        this->name = tea.name;
        this->age = tea.age;
    }
private:
    string name;
    int age;
};

int main()
{
    // 执行默认构造函数
    Teacher tea1;
    // 执行拷贝构造函数
    Teacher tea2 = tea1;
    Teacher tea3 = Teacher(tea1);

    return 0;
}
```
#### 初始化列表
有的时候类中会有const常量，是不允许被函数修改的，所以有了初始化列表这个方法，在构造函数之前执行，const常量必须要用初始化列表初始化。语法为`类名() 数据成员(参数), ... {}`如：
``` cpp
#include <iostream>
using namespace std;

class Circle
{
public:
    Circle() : Pi(3.14) {}
    // 错误：Circle() { Pi = 3.14; }
private:
    const double Pi;
};
```
在这里,Pi作为const double常量，只能够用初始化列表执行。
#### 析构函数
销毁对象的时候调用：
``` cpp
#include <iostream>
using namespace std;

class Student
{
public:
    Student() : buffer(new char[16]) {}
    ~Student() {
        // 析构函数释放内存
        delete [] buffer;
        buffer = nullptr;
    }
private:
    char *buffer;
};
```
### 对象与对象数组
提示：为了方便课程讲解，示例代码使用类内定义的方式实现，如果自己动手做实验的时候希望能够使用分文件类外定义的方式来编写代码。
本节教程使用class Point实例：
``` cpp
#include <iostream>
using namespace std;

class Point
{
public:
    // 使用带参数默认构造函数，并使用初始化列表初始化 x，y
    Point(double x = 0, double y = 0) : x(x), y(y) {
        //cout << "Point(double x = 0, double y = 0)" << endl;
        cout << "Point(double x = " << x << ", double y = " << y << ")" << endl;
    }
    // 拷贝构造函数
    Point(const Point & p) {
        //cout << "Point(const Point &p)" << endl;
        // 打印点的值
        cout << "Point(const Point &p:(" << p.x << ", " << p.y << ")" << endl;
        this->x = p.x;
        this->y = p.y;
    }
    // 析构函数，由于没有申请内存，析构函数中不需要做什么
    ~Point() {
        //cout << "~Point()" << endl;
        cout << "~Point():(" << x << ", " << y << ")" << endl;
    }
    // x, y 绑定的成成员函数
    void setPoint(const Point &p) {
        this->x = p.x;
        this->y = p.y;
    }
    void setPoint(double x, double y) {
        this->x = x;
        this->y = y;
    }
    void setX(double x) { this->x = x; }
    void setY(double y) { this->y = y; }
    double getX() { return x; }
    double getY() { return y; }
private:
    double x;
    double y;
};
```
#### 栈上实例化
和普通数据类型一样，直接按照数组栈上申请方式来就行：
``` cpp
// 栈上实例化
void stackInstantiation()
{
    // 实例化对象数组
    Point point[3];

    // 对象数组操作
    cout << "p[0]: (" << point[0].getX() << ", " << point[0].getY() << ")" << endl;
    cout << "p[1]: (" << point[1].getX() << ", " << point[1].getY() << ")" << endl;
    cout << "p[2]: (" << point[2].getX() << ", " << point[2].getY() << ")" << endl;
    point[0].setPoint(3, 4);
    cout << "p[0]: (" << point[0].getX() << ", " << point[0].getY() << ")" << endl;
}

int main()
{
    stackInstantiation();

    return 0;
}
```
在运行stackInstantiation()后，虽然函数中没有调用析构函数，但是由于数组是申请在栈上面的，所以会自动执行析构函数。
#### 堆上实例化
数据量较大建议在堆上实例化，注意需要手动释放内存（语法为`delete [] point`）：
``` cpp
int main()
{
       // 堆上实例化对象数组
    Point *point = new Point[3];

    cout << "p[0]: (" << point[0].getX() << ", " << point[0].getY() << ")" << endl;
    cout << "p[1]: (" << point[1].getX() << ", " << point[1].getY() << ")" << endl;
    cout << "p[2]: (" << point[2].getX() << ", " << point[2].getY() << ")" << endl;
    point[0].setPoint(3, 4);
    cout << "p[0]: (" << point[0].getX() << ", " << point[0].getY() << ")" << endl;

    // 释放内存
    delete [] point;
    point = nullptr;
    return 0;
}
```
除了`array[x].function()`这样的访问方式外，还可以使用`(array+x)->function()`指针的方式访问。
#### 对象成员
构造一个class Line，由两个点组成：
``` cpp
class Line
{
public:
    Line(const Point & pA, const Point &pB) : pointA(pA), pointB(pB) {
        cout << "Line(const Point & pA, const Point &pB)" << endl;
    }
    Line(double aX, double aY, double bX, double bY) : pointA(aX, aY), pointB(bX, bY) {
        cout << "Line(double aX, double aY, double bX, double bY)" << endl;
    }
    ~Line() {
        cout << "~Line()" << endl;
    }
private:
    Point pointA;
    Point pointB;
};

int main()
{
    // 实例化
    Line *line = new Line(1, 2, 3, 5);

    // 释放内存
    delete line;
    line = nullptr;
    return 0;
}
```
实例化Line时，会先实例化两个Point；而析构Line时，Point的析构函数会在后面执行。
如果使用析构函数的第一个重载（Point*参数传入），如下所示：
``` cpp
int main()
{
    // 实例化
    Line *line = new Line(Point(1, 2), Point(3, 5));

    // 释放内存
    delete line;
    line = nullptr;
    return 0;
}
```
在这种情况下，传入的Point（1, 2）,Point(3, 5)会被实例化为常数，然后再初始化完成以后自动销毁。
### 深拷贝与浅拷贝
+ 浅拷贝：对象A的数据成员赋值给对象B的数据成员
+ 深拷贝：对象B先为数据成员申请和A中数据成员一样大的内存，然后再赋值
+ 
#### 浅拷贝
浅拷贝示例：
``` cpp
#include <iostream>
using namespace std;

class Array
{
public:
    // 构造函数
    Array(int count) {
        this->count = count;
        cout << "Array()" << endl;
    }
    // 拷贝构造函数
    Array(const Array & arr) {
        // 浅拷贝
        this->count = arr.count;
        cout << "Array(const Array & arr)" << endl;
    }
    // 析构函数
    ~Array() {
        cout << "~Array(): " << count << endl;
    }
    void setCount(int count) { this->count = count; }
    int getCount() { return count; }
private:
    int count;
};

// 栈上实例化
void stackInstantiation()
{
    Array arr1(5);
    // 调用拷贝构造函数
    Array arr2 = arr1;
    arr2.setCount(5);
    Array arr3(arr2);
}

int main()
{
    stackInstantiation();
    return 0;
}
```

但是，浅拷贝有时候会出现错误，比如：
``` cpp
#include <iostream>
using namespace std;

class Array
{
public:
    // 构造函数
    Array(int count) {
        this->count = count;
        cout << "Array()" << endl;
    }
    // 拷贝构造函数
    Array(const Array & arr) {
        // 浅拷贝
        this->count = arr.count;
        cout << "Array(const Array & arr)" << endl;
    }
    // 析构函数
    ~Array() {
        cout << "~Array(): " << count << endl;
    }
    void setCount(int count) { this->count = count; }
    int getCount() { return count; }
private:
    int count;
};

// 栈上实例化
void stackInstantiation()
{
    Array arr1(5);
    // 调用拷贝构造函数
    Array arr2 = arr1;
    arr2.setCount(5);
    Array arr3(arr2);
}

int main()
{
    stackInstantiation();
    return 0;
}
```
在这段代码里，Array有int*类型的成员Arr，在拷贝的时候仅仅拷贝了Arr指针，而析构函数中会释放这个指针。这样，在析构arr1、arr2的时候，就会把同一片内存释放两次。
如果我们不去写这个浅拷贝构造函数，而是让编译器自动生成会怎么样呢？答案是编译器自动生成的依旧是这样，只会进行浅拷贝。
所以，遇到对象成员有指针等情况，一定要**手动写一个深拷贝构造函数**。
#### 深拷贝
``` cpp
Array(const Array & arr) {
    // 浅拷贝
    this->count = arr.count;
    // 深拷贝正确用法
    this->Arr = new int[count];
    // for (int i = 0; i < count; i++) {
    //     this->Arr[i] = arr.Arr[i];
    // }
    memcpy(Arr, arr.Arr, count*sizeof(int));

    cout << "Array(const Array & arr)" << endl;
}
```