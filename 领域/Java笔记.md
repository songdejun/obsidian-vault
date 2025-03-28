# 一、Gradle
## （一）构建工具
要解释Gradle是什么，首先要搞清楚一个名词——构建工具（Build Tool）。

构建工具，顾名思义就是用于构建（Build）的工具，构建包括编译（Compile）、连接（Link）、将代码打包成可用或可执行形式等等。

如果不使用构建工具，那么对于开发者而言，下载依赖、将源文件编译成二进制代码、打包等工作都需要一步步地手动完成。但如果使用构建工具，我们只需要编写构建工具的脚本，构建工具就会自动地帮我们完成这些工作。

## （二）常用构建工具一览
java生态圈的三大构建工具：
- Ant :2000年发布，纯java语言编写，具有良好的跨平台性，用buil.xml文件来配置，需要搭配Apache lvy工具来实现网络依赖管理。 Ant是程序式的构建工具，需要自定义构建过程，优点是对于构建过程有良好的控制性。
- Maven ：2004年发布，对Ant进行了改进，用prom.xml文件来配置。但与Ant不同的是，Maven是申明式的构建工具，对目录结构有约束，不需要自定义构建过程，配置较为简单。Maven还具有生命周期，更重要的是Maven内置了依赖管理。
- Gradle ：2012年发布，Gradle结合了前两者的优点，在此基础之上做了很多改进。它具有Ant的强大和灵活，又有Maven的生命周期管理且易于使用 。 Gradle不用XML，它使用基于Groovy的专门开发的DSL，所以它的配置文件更加简洁。它跟ant一样,使用了ivy作为jar包的依赖管理工具。

## （三）用户手册
[Gradle用户手册 - Gradle8.1.1中文文档 - API参考文档 - 全栈行动派](https://doc.qzxdp.cn/gradle/8.1.1/userguide/userguide.html)

## （四）Gradle Wrapper
### 1. Gradle Wrapper
**封装 Gradle**：  每个项目可能需要不同版本的 Gradle，手动配置版本会变得繁琐，Gradle Wrapper 对 Gradle 进行封装，通过读取配置文件自动下载和配置指定版本的 Gradle。
**版本一致性**： Wrapper 确保团队中使用相同版本的 Gradle，避免版本冲突和构建问题。
**文件组成**： Wrapper 包含两个文件：`gradle-wrapper.jar` 和 `gradle-wrapper.properties`。

- `gradle-wrapper.jar`： 包含 Wrapper 的核心代码。
- `gradle-wrapper.properties`： 包含下载 Gradle 的版本信息和存放路径。
gradlew 和 gradlew.bat

**执行 Gradle 命令**： `gradlew` 和 `gradlew.bat` 分别在 Linux/macOS 和 Windows 下使用，用于执行 Gradle 命令。
**封装命令行**： Wrapper 对 Gradle 命令行进行封装，确保使用指定版本的 Gradle。

### 2. 指定下载 Gradle 版本

`gradle-wrapper.properties` 文件中的 `distributionUrl` 属性指定了要下载的 Gradle 版本和地址。

Wrapper 会自动下载并解压指定版本的 Gradle 到用户目录下的 `.gradle/wrapper/dists` 目录。

可以通过修改 `distributionUrl` 属性或手动下载并放置到指定目录来更改下载的 Gradle 版本。

### 3. `gradle-wrapper.properties` 属性解释

- `distributionUrl`： 指定下载的 Gradle 版本和地址。

- `gradleUserHome`： 指定 Gradle 本地仓库的存放路径。

- `serviceDirectory`： 指定 Gradle 服务文件的存放路径。

- `localRepository`： 指定本地 Maven 仓库的存放路径。

### 4. 总结

Gradle 是构建工具，Gradle Wrapper 对 Gradle 进行封装，使得可以为用户下载指定版本的 Gradle。

# 二、JavaWeb
## （一）概览
前后端耦合的技术路线过时

![image-20250208152248276](https://gitee.com/magus-songdejun/imgbed/raw/master/202502081522412.png)

jsp就是用后端实现动态的html的技术，现在后端只返回json数据了。

servlet就是python的wsgi（就是后端框架和web服务器交互的接口规范），对于后端程序开发来说不是该考虑的问题，专注业务即可。

现在的JavaWeb技术路线

![image-20250208152703658](https://gitee.com/magus-songdejun/imgbed/raw/master/202502081527724.png)



# 三、Java 语法
## （一）static 总结

Java 中的 `static` 关键字用于定义类的静态成员，这些成员属于类本身，而不是类的某个对象。以下是关于 Java 中 `static` 的总结：
### 1. 静态变量（类变量）
- **定义**：使用 `static` 关键字修饰的成员变量称为静态变量或类变量。
- **特点**：
  - 属于类，而不是属于类的某个对象。
  - 只有一份，不管创建多少个对象，都共用这一份静态变量。
  - 可以通过类名直接访问，无需创建对象。
- **示例**：
  ```java
  public class MyClass {
      static int count = 0;  // 静态变量
  }
  ```
### 2. 静态方法
- **定义**：使用 `static` 关键字修饰的方法称为静态方法或类方法。
- **特点**：
  - 属于类，而不是属于类的某个对象。
  - 可以直接通过类名调用，无需创建对象。
  - 静态方法不能直接访问非静态成员变量和方法，但非静态方法可以访问静态成员。
- **示例**：
  ```java
  public class MyClass {
      static void myStaticMethod() {
          // 静态方法
      }
  }
  ```
### 3. 静态代码块
- **定义**：在类中用 `static` 修饰的代码块。
- **特点**：
  - 在类被加载时执行，且只执行一次。
  - 通常用于初始化静态变量。
- **示例**：
  ```java
  public class MyClass {
      static {
          // 静态代码块
      }
  }
  ```
### 4. 静态内部类
- **定义**：在类内部定义的静态类。
- **特点**：
  - 可以访问外部类的所有静态成员，但不能直接访问非静态成员。
  - 可以在没有外部类对象的情况下创建内部类对象。
- **示例**：
  ```java
  public class OuterClass {
      static class InnerClass {
          // 静态内部类
      }
  }
  ```
### 5. 注意事项
- 静态成员通常用于类的工具方法或常量。
- 过度使用静态成员可能导致代码难以测试和维护。
- 静态方法不能被覆盖（Overridden），因为它们属于类，而不是对象。
通过以上总结，我们可以更好地理解和使用 Java 中的 `static` 关键字。

## （二）注解

### 1. 为什么要学习注解？

**框架实现原理**：理解和使用框架时，了解注解的工作原理是深入理解框架的基础。
**自定义功能**：通过自定义注解，可以实现特定功能，如自动记录日志，提高代码的可读性和逼格。
**职业发展**：掌握注解能提升编程技能，有助于在团队中脱颖而出，迈向更高的职业巅峰。

### 2. 注解是什么？

注解是一种特殊的标记，可以附加在Java的接口、类、属性、方法上。与普通注释不同，注解在运行时可以通过反射被读取并执行相应的操作。如果没有使用反射或其他机制，注解不会影响程序的运行。

- 例如：@Override注解表明一个方法重写了父类方法，它会被编译器检查是否真的覆写了相应方法。

在Spring框架中，注解会影响程序运行，因为Spring利用反射来解析和处理这些注解。

### 3. 为什么要使用注解？

**简化配置**：注解简化了配置过程，相比于XML配置，注解更易于管理和理解。
**解耦和轻量化**：注解实现了配置信息的解耦，使得每个模块可以独立配置，减少了集中式配置文件的复杂性。
**现代化开发**：注解顺应了现代软件开发中对解耦和轻量化的需求。

### 4. 注解的作用

**提供信息**：为程序元素添加元数据，提供额外的信息，增强代码的可读性。
**编译时处理**：编译器可以利用注解进行预检查，如@Override确保方法覆写的正确性。
**运行时处理**：框架和运行时环境可以解析注解，实现动态功能，如Spring的依赖注入。

# 四、Minecraft
## （一）fabric mod 开发环境配置
### 1. 下载 mod 模板
在[Template mod generator | Fabric](https://fabricmc.net/develop/template/)中填写模组名，包名，版本等信息，然后下载 mod 模板。

![image.png](https://gitee.com/magus-songdejun/imgbed/raw/master/202503060001626.png)

解压后用 IDEA 打开。

### 2. 基于 Gradle Wrapper 构建项目
挂上梯子，打开 IDEA 等待 Gradle Wrapper 配置好 Gradle 环境，等待 Gradle 下载好依赖。

> Gradle Wrapper 会读取 `./gradle/wrapper/gradle-wrapper.properties` 文件中的属性下载对应版本的 Gradle，Gradle 会读取 `./build.gradle` 下载依赖包。

你可以在设置中修改**Gradle 用户主目录**到你想将 Gradle 下载到的位置。

![image-20250306001216162](https://gitee.com/magus-songdejun/imgbed/raw/master/202503060012300.png)

从右侧侧边栏打开 Gradle 任务，双击 genSources 生成 Minecraft 反混淆源代码。

![image-20250306002341157](https://gitee.com/magus-songdejun/imgbed/raw/master/202503060023230.png)

 双击 runClient 任务就可以运行 Minecraft。
