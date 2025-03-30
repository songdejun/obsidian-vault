# static_cast、reinterpret_cast、const_cast 和 dynamic_cast

C++ 引入了四种功能不同的强制类型转换运算符以进行强制类型转换：static_cast、reinterpret_cast、const_cast 和 dynamic_cast。

强制类型转换是有一定风险的，有的转换并不一定安全，如把整型数值转换成[指针](https://c.biancheng.net/c/80/)，把基类指针转换成派生类指针，把一种函数指针转换成另一种函数指针，把[常量指针](https://c.biancheng.net/view/1ltv8ex.html)转换成非常量指针等。C++ 引入新的强制类型转换机制，主要是为了克服C语言强制类型转换的以下三个缺点。

1. 不能体现转换的风险和功能不同
   例如，将 int 强制转换成 double 是没有风险的，而将常量指针转换成非常量指针，将基类指针转换成派生类指针都是高风险的，而且后两者带来的风险不同（即可能引发不同种类的错误），C语言的强制类型转换形式对这些不同并不加以区分。

2. 将多态基类指针转换成派生类指针时不检查安全性，即无法判断转换后的指针是否确实指向一个派生类对象。
   ```c++
   class Base {
       public:
           virtual void print() { std::cout << "Base" << std::endl; }
       };
       
   class Derived : public Base {
   public:
       void print() override { std::cout << "Derived" << a << std::endl; }
   };
       
   int main() {
       Base* b = new Base();
       Derived* d = (Derived*)b; // C语言的强制类型转换
       d->print(); // 输出: Base
   }
   ```
3. 比较难Debug，出错的地方不会指出是类型转换错误

具体四种cast介绍：[C++强制类型转换运算符（static_cast、reinterpret_cast、const_cast和dynamic_cast） - C语言中文网 - Circle 阅读助手](https://c.biancheng.net/view/410.html#circle=on)

# 智能指针

类型：

- shared_ptr<T>()
- unique_ptr<T>()
- weak_ptr<T>()

share是线程安全的，因为引用计数是原子变量。但是又循环引用问题，用weak_ptr解决。

```c++
struct Boo;
struct Foo {
    std::weak_ptr<Boo> boo;
    ~Foo(){
        cout << "foo is free." << endl;
    }
};
struct Boo {
    std::shared_ptr<Foo> foo;
    ~Boo(){
        cout << "boo is free." << endl;
    }
};

int main() {
    auto f_ptr = std::make_shared<Foo>();
    auto b_ptr = std::make_shared<Boo>();
    auto b_wptr = weak_ptr(b_ptr);

    f_ptr->boo = b_wptr;
    b_ptr->foo = f_ptr;
    // 输出
    // boo is free.
    // foo is free.

    auto f_ptr = std::make_shared<Foo>();
    auto b_ptr = std::make_shared<Boo>();

    f_ptr->boo = b_ptr;
    b_ptr->foo = f_ptr;
    // 没有输出
}
```

unique_ptr是所有权系统，默认移动语义。

[C++面试八股文：什么是智能指针？ - 知乎](https://zhuanlan.zhihu.com/p/638292065)