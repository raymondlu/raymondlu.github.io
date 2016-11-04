---
layout: post
title:  "对于C++11中lambda函数的一点理解（下篇）"
date:   2016-11-04 21:00
categories: jekyll
permalink: /archivers/lambda-in-c-plus-plus-2
published: true
---
&emsp;&emsp;要解释清楚编译器为了这神奇的lambda表达式在背后都做了什么工作，我们不得不提到一个可能对很多人来说有点陌生的东西——函数对象。首先我必须承认我也是最近因为要了解lambda才接触到的。函数对象简单的说就是重载了括号操作符的对象，请看下面C++代码中的Functor这个类。在main函数里，首先我们以局部变量s为构造函数参数生成了Functor的实例doTrivialStuff。doTrivialStuff因为括号操作符被重载了，它可以像函数一样使用doTrivialStuff(a,b)。仿函数对象和函数最大的不同在于，仿函数有成员变量，也就是说可以有状态，而函数是没有状态的。
~~~cpp
#include <iostream>
using namespace std;

class Functor{
    int _s;
public:
    Functor(int s):_s(s){
        
    }
    
    int operator ()(int x,int y){
        return x+y*_s;
    }
};

int main() {
    int a=3;
    int b=4;
    int s=5;
    Functor doTrivialStuff(s);
    int result=doTrivialStuff(a,b);
    cout<<"result:"<<result<<"\n";
    cout<<"s:"<<s<<"\n";
    return 0;
}
~~~
对于实现同样的功能，lambda函数实现：
~~~cpp
#include <iostream>
using namespace std;

int main() {
    int a=3;
    int b=4;
    int s=5;
    auto doTrivialStuff=[s](int x,int y)->int{
        return x+y*s;
    };
    int result=doTrivialStuff(a,b);
    cout<<"result:"<<result<<"\n";
    return 0;
}
~~~
对比仿函数对象和lambda函数，我们发现其实他们二者是类似的。其实在编译器背后C++11标准就是按仿函数对象的方式来实现lambda函数的，理解了仿函数对象也就能理解lambda函数了。实际上你可以简单的把lambda函数理解成带状态的函数，而状态来自捕捉列表。