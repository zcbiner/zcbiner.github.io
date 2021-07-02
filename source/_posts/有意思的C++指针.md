---
title: 有意思的C++指针
date: 2018-06-17 20:46:00
tags: C++
---

在学习C++的过程中，经常搞错相关的指针问题，本文记录下自己容易搞混的知识点。

### 指针常量与常量指针

`const`关键字用来修饰常量，那么`const`用在指针上有什么需要注意的点呢？

如果`const`右侧紧跟着的是指针，则为常量指针，紧跟着常量就是指针常量。

直接用代码说明：

```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 1;
    int b = 2;
    // const修饰指针，p1为常量指针。
    // p1指向可以改，指向的值不能改。
    const int* p1 = &a;
    //*p1 = 3; //报错
    p1 = &b;
    cout << "*p1=" << *p1 << endl;

    // const修饰常量，p2为指针常量。
    // p2指向不可以改，指向的值可以改。
    int* const p2 = &a;
    *p2 = 3;
    //p2 = &b; // 报错
    cout << "*p2=" << *p2 << endl;

    // 既修饰指针又修饰常量，p3指向和指向的值都不能改
    const int* const p3 = &a;
    //*p3 = 3;// 报错
    //p3 = &b;// 报错
    return 0;
}
```



### 指针与函数

使用指针当函数参数，可以直接修改int，short等基本类型，Java就做不到。本质上是`int* a`,` int* b`分别代表a，b的地址，通过地址修改指向的值导致原数据被修改。而`swapData(int a, int b)`方法，a，b为形参，实际上是拷贝了一份数据到函数内部操作。

```cpp
#include <iostream>
using namespace std;
// 无法改变a，b的值
void swapData(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
}
// 可以改变a，b的值
void swapData2(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
int main() {
    int a = 1;
    int b = 2;
    swapData(a, b);
    cout << "a=" << a << ";b=" << b << endl;
    swapData2(&a, &b);
    cout << "a=" << a << ";b=" << b << endl;
    return 0;
}
```

