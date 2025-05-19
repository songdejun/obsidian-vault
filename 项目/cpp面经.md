# static_cast、reinterpret_cast、const_cast 和 dynamic_cast

[C++强制类型转换运算符（static_cast、reinterpret_cast、const_cast和dynamic_cast） - C语言中文网 - Circle 阅读助手](https://c.biancheng.net/view/410.html#circle=on)

C++是一个弱类型语言，以下讨论都限于显式类型转换。

**C风格的类型转换**：**ALL IN ONE**，一句`(target_type)var`就可以实现包括：

- 自然的类型转换，例如`int`转`long`，`float`转`double`
- 指针/引用间的类型转换：
  - 不同类型指针/引用间的转换
  - 基类向派生类的转换
  - 常量指针转换到非常量指针

**示例**

```c++
int a = 10;
double b = (double)a;  // 1. 数值类型转换（安全）

const int* p1 = &a;
int* p2 = (int*)p1;    // 2. 移除 const（危险，可能导致未定义行为）

Base* base = new Derived();
Derived* derived = (Derived*)base;  // 3. 基类→派生类（不安全，无运行时检查）

int* ptr = (int*)0x1234;  // 4. 任意指针转换（危险，可能访问非法内存）
```



---

**ALL IN ONE**的反面就是不同功能和风险的类型转换都只有一个形式，既不检查安全性，出错时也难以Debug。

**CPP的类型转换**：

- `static_cast<T>()`，自然的类型转化：**数值类型转换**，**类继承中的向上转型**
- `reinterpret_cast<T>()`，C风格高危转换：**任意类型/引用转换**，**整数和指针转换**
- `const_cast<T>()`，**移除/添加`const/volatile`属性**
- `dynamic_cast<T>()`，**类继承中的向下转型**，**运行时检查**，失败返回`nullptr`（指针）或抛出**异常**（引用）

这四种类型转换方式实现了通过不同的形式体现不同的目的和功能，并在各自的领域提供安全检查功能。

**示例**

```c++
// 1. static_cast - 安全的静态类型转换
int i = 10;
double d = static_cast<double>(i);  // 数值转换

// 2. dynamic_cast - 运行时类型检查
class Base {
public:
    virtual ~Base() {}  // 必须有虚函数才能使用 dynamic_cast
};

class Derived : public Base {
public:
    void show() { std::cout << "Derived class\n"; }
};

Base* base_ptr = new Derived();
Base* base = static_cast<Base*>(base_ptr);  // 向上转型（安全）

Derived* derived = dynamic_cast<Derived*>(base);
if (derived) {
    std::cout << "dynamic_cast: 转换成功\n";
    derived->show();
} else {
    std::cout << "dynamic_cast: 转换失败\n";
}

// 3. const_cast - 常量性修改
const int x = 42;
int* px = const_cast<int*>(&x);  // 移除 const
*px = 100;  // 危险：实际是未定义行为

// 安全使用 const_cast 的例子
int y = 10;
const int* py = &y;
int* py_mutable = const_cast<int*>(py);
*py_mutable = 20;  // 安全：y 原本就是非 const

// 4. reinterpret_cast - 低级别内存重新解释
int num = 0x12345678;
char* bytes = reinterpret_cast<char*>(&num);  // 按字节访问
std::cout << "reinterpret_cast: ";
for (size_t i = 0; i < sizeof(num); ++i) {
    std::cout << std::hex << (int)bytes[i] << " ";
}

// 指针与整数转换
void* ptr = malloc(sizeof(int));
int* int_ptr = reinterpret_cast<int*>(ptr);
*int_ptr = 42;
```



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

# 虚函数
## 纯虚函数
在C++中，`虚函数 = 0`的语法用于定义**纯虚函数（Pure Virtual Function）**，它的主要作用和用法如下：

---

### **作用**
1. **定义抽象基类（Abstract Class）**  
   - 包含至少一个纯虚函数的类称为**抽象类**，它不能直接实例化（不能创建对象）。
   - 派生类必须实现（覆盖）所有纯虚函数，否则派生类也会成为抽象类。

2. **强制接口规范**  
   - 纯虚函数相当于一个“必须实现”的接口，强制派生类提供特定功能的实现。

3. **实现多态**  
   - 通过基类指针或引用调用纯虚函数时，实际执行的是派生类的实现（运行时多态）。

---

### **用法**
```cpp
class AbstractBase {
public:
    virtual void PureVirtualFunction() = 0; // 纯虚函数
};

class Derived : public AbstractBase {
public:
    void PureVirtualFunction() override { // 必须实现纯虚函数
        cout << "Implemented in Derived" << endl;
    }
};

int main() {
    // AbstractBase obj; // 错误：抽象类不能实例化
    Derived d;
    AbstractBase* ptr = &d; // 通过基类指针调用派生类实现
    ptr->PureVirtualFunction(); // 输出: "Implemented in Derived"
}
```

---

### **关键点**
1. **语法**  
   - `virtual 返回类型 函数名(参数) = 0;`  
   - 纯虚函数可以有函数体（但通常不提供，除非需要默认实现）。

2. **抽象类的特性**  
   - 不能直接创建对象，但可以定义指针或引用。
   - 可以包含其他非虚成员变量和函数。

3. **与普通虚函数的区别**  
   - 普通虚函数可以有默认实现，派生类可选择是否覆盖；纯虚函数必须被派生类实现。

---

### **典型应用场景**
- **设计模式**（如工厂模式、策略模式等）中定义接口。
- 框架或库中强制用户实现特定功能（如游戏引擎的`Render()`方法）。

```cpp
// 示例：抽象接口
class Shape {
public:
    virtual double Area() const = 0; // 纯虚函数
};

class Circle : public Shape {
public:
    double Area() const override { 
        return 3.14 * radius_ * radius_; 
    }
private:
    double radius_;
};
```

通过纯虚函数，C++实现了类似其他语言中“接口”（Interface）的功能。/