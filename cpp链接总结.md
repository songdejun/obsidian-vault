[TOC]

# C++ 中的链接（Linkage）总结

## 一、内部链接（Internal Linkage）
符号的作用域仅限于当前翻译单元（即当前源文件），在其他翻译单元中不可见。

### 1. 静态全局变量和函数
   - 使用 `static` 关键字声明的全局变量和函数。
   - **示例：**
     ```cpp
     // file1.cpp
     static int x = 10; // 仅在 file1.cpp 中可见
     static void foo() { // 仅在 file1.cpp 中可见
         // ...
     }
     ```

### 2. 匿名命名空间
   - 在匿名命名空间中定义的符号。
   - **示例：**
     ```cpp
     // file1.cpp
     namespace {
         int x = 10; // 仅在 file1.cpp 中可见
         void foo() { // 仅在 file1.cpp 中可见
             // ...
         }
     }
     ```

### 3. 常量全局变量
   - 默认情况下，`const` 修饰的全局变量具有内部链接。
   - **示例：**
     ```cpp
     // file1.cpp
     const int x = 10; // 仅在 file1.cpp 中可见
     ```
   - 如果需要外部链接，可以使用 `extern`：
     ```cpp
     extern const int x = 10; // 外部链接
     ```

### 4. 模板的静态成员变量
   - 模板的静态成员变量在实例化时具有内部链接。
   - **示例：**
     ```cpp
     template <typename T>
     class MyClass {
     public:
         static int x; // 内部链接
     };
     
     template <typename T>
     int MyClass<T>::x = 10; // 内部链接
     ```

### 5. 内联函数和变量
   - `inline` 函数和变量在多个翻译单元中共享定义，但链接器会确保只有一个实例。
   - **示例：**
     ```cpp
     // file1.cpp
     inline void foo() { // 内部链接
         // ...
     }
     ```

## 二、外部链接（External Linkage）
符号的作用域可以跨越多个翻译单元（即多个源文件），在其他翻译单元中可见。

### 1. 非静态全局变量和函数

   - 默认情况下，全局变量和函数具有外部链接。
   - **示例：**
     ```cpp
     // file1.cpp
     int x = 10; // 外部链接，其他文件可见
     void foo() { // 外部链接，其他文件可见
         // ...
     }
     ```

### 2. `extern` 声明的变量
   - 使用 `extern` 关键字声明的变量具有外部链接。
   - **示例：**
     ```cpp
     // file1.cpp
     extern int x; // 声明，外部链接
     int x = 10;   // 定义，外部链接
     
     // file2.cpp
     extern int x; // 使用 file1.cpp 中定义的 x
     ```

### 3. 非静态类成员函数
   - 类的非静态成员函数具有外部链接。
   - **示例：**
     ```cpp
     // file1.cpp
     class MyClass {
     public:
         void foo(); // 外部链接
     };
     
     void MyClass::foo() { // 外部链接
         // ...
     }
     ```

### 4. 模板的实例化
   - 模板的实例化（如函数模板、类模板）具有外部链接。
   - **示例：**
     ```cpp
     // file1.cpp
     template <typename T>
     void foo(T x) { // 外部链接
         // ...
     }
     
     template void foo<int>(int); // 显式实例化，外部链接
     ```

### 5. 内联函数和变量
   - `inline` 函数和变量在多个翻译单元中共享定义，但链接器会确保只有一个实例。
   - **示例：**
     ```cpp
     // file1.cpp
     inline void foo() { // 外部链接
         // ...
     }
     
     // file2.cpp
     inline void foo(); // 声明
     ```

### 6. 非匿名命名空间
   - 在非匿名命名空间中定义的符号具有外部链接。
   - **示例：**
     
     ```cpp
     // file1.cpp
     namespace MyNamespace {
         int x = 10; // 外部链接
         void foo() { // 外部链接
             // ...
         }
     }
     ```

## 三、注意事项
- **内部链接的符号**不会导致链接冲突，因为它们在当前翻译单元之外不可见。
- **外部链接的符号**可能导致链接冲突，如果多个翻译单元定义了相同的符号，链接器会报错。
- 避免符号冲突的方法：
  - 使用 `static` 或匿名命名空间限制符号的作用域。
  - 使用 `extern` 声明共享符号。

## 四、总结表

| 类型                 | 示例                                       | 说明                                     |
| -------------------- | ------------------------------------------ | ---------------------------------------- |
| **内部链接**         |                                            |                                          |
| 静态全局变量和函数   | `static int x; static void foo();`         | 仅在当前翻译单元中可见                   |
| 匿名命名空间         | `namespace { int x; void foo(); }`         | 仅在当前翻译单元中可见                   |
| 常量全局变量         | `const int x = 10;`                        | 默认内部链接，除非使用 `extern`          |
| 模板的静态成员变量   | `template <typename T> int MyClass<T>::x;` | 实例化时具有内部链接                     |
| 内联函数和变量       | `inline void foo();`                       | 多个翻译单元共享定义，链接器确保唯一实例 |
| **外部链接**         |                                            |                                          |
| 非静态全局变量和函数 | `int x; void foo();`                       | 默认外部链接                             |
| `extern` 声明的变量  | `extern int x;`                            | 外部链接                                 |
| 非静态类成员函数     | `void MyClass::foo();`                     | 外部链接                                 |
| 模板的实例化         | `template void foo<int>(int);`             | 外部链接                                 |
| 内联函数和变量       | `inline void foo();`                       | 多个翻译单元共享定义，链接器确保唯一实例 |
| 非匿名命名空间       | `namespace MyNamespace { int x; }`         | 外部链接                                 |
