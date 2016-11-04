---
layout: post
title:  "对于C++11中lambda函数的一点理解（上篇）"
date:   2016-11-04 20:00
categories: jekyll
permalink: /archivers/lambda-in-c-plus-plus-1
published: true
---
&emsp;&emsp;作为一个C++程序员，在日常的编程中，我们时不时会到这样的窘境：在一个方法里，某些代码逻辑会出现这不同的地方，假如要单独为它们写个方法嘛会显得有点琐碎，不写嘛同样的代码又会出现在多个地方，违反了程序员编码的第一大铁律：Don't Repeat Yourself（不要重复你自己）。但是同样的情况对于脚本程序员来却不是什么烦恼，他们会好不犹豫的把这些代码逻辑封装到一个局部函数里。比如下面的这段lua代码：

~~~lua
function main()
    local a=3
    local b=4
    local s=5
    local function doTrivialStuff(x,y)
        return x+y*s
    end
    local result=doTrivialStuff(a,b)
    print('result:'..result)
end
~~~

我们在main这个函数里嵌套定义了一个名字叫做doTrivialStuff的局部函数，并且在随后调用了它。这种在函数内部在嵌套定义另一个函数的方式在之前的C++标准里是没办法支持的，不过多亏C++11标准里引入了一个叫lambda的东西，现在我们C++程序员也可以这么做了。


##lambda函数的组成和捕捉列表
&emsp;&emsp;lambda表达式，也可以叫做匿名函数[(wiki)](https://en.wikipedia.org/wiki/Lambda_expression)，并不是什么新鲜玩意，而且恰恰相反，它是一个比所有编程语言还要老的古董[(wiki)](https://en.wikipedia.org/wiki/Lambda_calculus#Definition)。在C++11里，它的组成可以表示成如下的样子：

~~~cpp
[捕捉列表](参数列表)->返回类型{函数体}
~~~

我们在C++里用lambda实现一下与上面lua里相同的功能：

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

同样的功能，C++代码显得稍微复杂一点，毕竟它不是解释执行的弱类型语言。从lambda表达式的构成和使用方式，这大概也是最不像C++的C++特性了。中括号“［］”是lambda的引出符号，编译器根据它来确定接下来的代码是lambda函数。我们在大括号“｛｝”里来实现lambda函数的逻辑。实际上对于lambda函数来说只有这两个是必须的，其余的组成部分按照具体要实现的逻辑可省略的。所以最简单的一个lambda函数是这样的：

~~~cpp
[]{};
~~~

当然这个最简单的lambada函数其实什么事情也没做。在读上面的C++代码的时候，大概你对中括号“[]”的这个s变量有点小疑惑。在lambda函数的组成里，“[]”出现的东西叫做捕捉列表。你可以这样理解只有在捕捉列表里被“捕捉”到外部局部变量才可以在lambda的大括号“{}”里使用。假如你细心的话，你会发现开头的lua代码里并不用这样做，这个涉及到另外一个叫做“闭包”的概念，这里就不展开了。在C++11里lambda捕捉父函数作用域里的变量的方式有两种，一种是按值来捕捉，比如就是上面的代码里出现的这种方式：

~~~cpp
[s](int x,int y)->int{return x+y*s;}
~~~

所谓按值捕捉，这个和函数调用按值传参是一样的，lambda函数内部对捕捉变量的修改不会反映到外面来。另外一种方式就是按引用捕捉，只需在被捕捉变量前加上“&”，比如：

~~~cpp
[&s](int x,int y)->int{return x+y*s;}
~~~

这个和函数调用时按引用传参一样，lambda函数内对对捕捉变量的修改会反映到外面来。比如执行下面的代码：

~~~cpp
int main() {
    int a=3;
    int b=4;
    int s=5;
    auto doTrivialStuff=[&s](int x,int y)->int{
        s++;
        return x+y*s;
    };
    int result=doTrivialStuff(a,b);
    cout<<"result:"<<result<<"\n";
    cout<<"s:"<<s<<"\n";
    return 0;
}
~~~

输出为：

~~~cpp
result:27
s:6
~~~

假如你写的lambda函数要捕捉父函数里出现的所有局部变量，那么在捕捉列表里把这些变量一个一个写上去也蛮累的，你可以在“[]”里填上“＝”表示按值捕捉父函数里所有的局部变量，或者填上“&”表示按引用捕捉，就像下面这样：

~~~cpp
[=](int x,int y)->int{return x+y*s;}
[&](int x,int y)->int{return x+y*s;}
~~~

当然你还会遇到这样的情况，父函数中的局部变量除了a按引用捕捉外，其余的按值捕捉：

~~~cpp
[=,&a](){...}
~~~

同样的父函数中除a按值捕捉外，其余的按引用捕捉，可以这样：

~~~cpp
[&,a](){...}
~~~

至于怎样决定一个变量是按值捕捉还是按引用捕捉，这个和你在设计方法时决定参数是按值传递还是按引用传递是一样的。按值捕捉就意味着值拷贝带来的开销，假如这个开销过大那么你就要考虑是否要改成按引用捕捉。不过按引用捕捉则要更加小心点，需要考虑lambda函数和父函数对该捕捉值的互相影响。


##结语
&emsp;&emsp;lambda表达式的引进，很好的解决了我在开头时提到的窘境，但是它实在太不像C++了，这让第一次接触它的C++程序员感到有点摸不着头脑。为了支持lambda，编译器为我们做了很多的事情，下篇我将尝试解释一下编译器到底是这样做的。
