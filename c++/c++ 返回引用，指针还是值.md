# C++：返回引用，指针还是值

自c语言中的指针以来，内存的管理与使用一直是程序员的痛处。不恰当的指针使用常常会导致严重的内存错误，而随意地返回安全的对象和值却间接地导致了程序运行效率地下降

在C++中引入了引用(Reference)来化解指针所带来的难题，但是这也是建立在正确使用的基础之上的

> 引用在C++中可以看作是自解引用的指针，两者的性质相同，在汇编代码上并无异处

## 返回值

```cpp
int &func() {
    int a = 1;
    return a;
}
```

对于这个函数，可以看出我们的本意是想返回a的引用值，但是对于函数来是，其是动态生成的，在完成调用后其中的局部变量值就会在栈中被销毁，无法形成长久的引用关系，一般在编译的时候就会为我们抛出这样一个警告：

```text
main.cc:4:10: warning: reference to stack memory associated with local variable 'a' returned [-Wreturn-stack-address]
  return a;
         ^
1 warning generated.
```

因此我们可以得到这样一个结论

> 当返回局部变量时我们一定要返回值，同时较小的结构也可以用过值来进行返回

## 返回指针

指针在C中是常用的一种返回手段，能直接返回结构对应的地址

但是在C++中除了数组之外，一般直接用引用来替代指针的作用

## 返回值优化

> 既然对于值我们可以任意返回而不易出错，问什么我们还要设计出返回指针或者是引用的形式呢?

这里给出一个相对于普通变量来说更大的结构

```cpp
struct Foo {
  int i;
  // 假设这里有许许多多你看不见的属性
}
```

当我们调用这样一个函数时，其可返回类型为Foo的值：

```cpp
Foo f() {
    Foo a;
    a.i = 101;
    return a;
}
```

由于Foo很大，当我们使用f()的返回值，比如Foo foo = f();的时候，按照常理，f()的返回值会被赋予foo，这个过程可能会发生拷贝，导致性能下降。这时对返回结果进行优化就显得尤为重要

### 普通引用

```cpp
string &func(const string &s1, const string &s2) {
    return s1 + s2;
}
```

对于引用来说其在调用或返回时不会再用一个新值来对原先的值进行拷贝，而是直接使用原先的值进行操作，并在返回时直接返回内存中这个值的引用，并不会发生拷贝，从而一定程度上提升了程序的性能

同时需要需要了解的是这种引用称为是左值引用

> 所谓左值一般可以认为是变量，是能够随意被赋值并使用的值，也是只能放在等号左边的值

因此对于一个非局部变量的值，我们可以使用引用来对其进行返回

### const引用

使用引用固然能在函数返回时提高程序的性能，但是如果我们想要返回一个较大结构的临时变量呢?是不是只能返回值呢?

显然结果并不是这样的，我们可以使用const关键字对引用进行修饰从而延长被引用对象的生命周期，比如使用`const Foo &foo = f()`就是正确的

> Only local const references prolong the lifespan

这里引用下大佬博客上总结的两点

1. 引用必须是静态的，也就是必须用const来修饰这个引用。这是因为函数的返回值属于右值，也就是（rvalue）。普通的引用是左值引用，也就是lvalue reference。左值引用不能指向右值引用，只有const的左值引用才能用来指向右值
2. 本来f()的返回值是一个临时的变量，在它调用结束后，就应该销毁了。可是通过像这样const Foo& foo = f();，把临时的返回值赋值给一个const左值引用，f()返回值并不会立即销毁。这等于是在const引用的作用域内，延长了f()返回值的存活时间

> 所谓右值就是等号右边的值，一般为函数的返回值或者是明确的数值，右值只能在等号右边，并不能作为左值并被赋值

### 右值引用

右值引用是C++11中新加入的内容，是一个尤为重要的概念，其主是要用来解决C++98/03中遇到的两个问题，第一个问题就是临时对象非必要的昂贵的拷贝操作，第二个问题是在模板函数中如何按照参数的实际类型进行转发。通过引入右值引用，很好的解决了这两个问题，改进了程序性能

相比于`&`是对左值进行绑定，`&&`就是对右值进行一个绑定，我们可以这样完成绑定

```cpp
// 左值引用
int i = 100;
int &a = i;

// 右值引用
int &&b = 100;
```

右值可以看作是一种特殊的匿名变量，通过右值引用的声明，右值又“重获新生”，其生命周期与右值引用类型变量的生命周期一样长，只要该变量还活着，该右值临时量将会一直存活下去

同时右值引用可以看作是加了const的左值引用，其自身是一个左值，能够被任意修改，而其本质却是临时变量的右值，所以左值引用可以指向一个右值引用，而右值引用去不能被另外一个右值引用所指向

```cpp
int &&i = 100;

// 正确做法
i++;
int tmp = 200; i = tmp;

int &left = i;

// 错误做法
int &&j = 200;
j = i;
```

## 参考链接

[从4行代码看右值引用](https://www.cnblogs.com/qicosmos/p/4283455.html)
[C++：关于函数返回的一件小事——是返回值还是返回引用？](https://marvinsblog.net/post/2018-08-07-cplusplus-return-value-or-reference/)
[Does a const reference class member prolong the life of a temporary?](https://stackoverflow.com/questions/2784262/does-a-const-reference-class-member-prolong-the-life-of-a-temporary)
