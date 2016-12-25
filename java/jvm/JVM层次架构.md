### JVM

Java 虚拟机 Java 虚拟机（Java virtual machine，JVM）是运行 Java 程序必不可少的机制。**JVM实现了Java语言最重要的特征：即平台无关性。**

**原理**：编译后的 Java 程序指令并不直接在硬件系统的 CPU 上执行，而是由 JVM 执行。 JVM屏蔽了与具体平台相关的信息，使Java语言编译程序只需要生成在JVM上运行的目标字节码（.class）,就可以在多种平台上不加修改地运行。Java 虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。因此实现java平台无关性。它是 Java 程序能在多平台间进行无缝移植的可靠保证，同时也是 Java 程序的安全检验引擎（还进行安全检查）。

JVM 是编译后的 Java 程序（`.class文件`）和硬件系统之间的接口 （编译后：`javac` 是收录于 JDK 中的 Java 语言编译器。该工具可以将后缀名为`.java`的源文件编译为后缀名为`.class`的可以运行于 Java 虚拟机的字节码。）

### JVM结构图

根据《java虚拟机规范》规定，JVM的基本结构一般如下图所示：

**英文版**
![jvm-architecture-en](https://raw.githubusercontent.com/dannygod/ITArticles/master/assets/jvm_architecture_en.png)

**中文版**
![jvm-architecture-ch](https://raw.githubusercontent.com/dannygod/ITArticles/master/assets/jvm_architecture_ch.png)

**简化版**
![jvm-architecture-simple](https://raw.githubusercontent.com/dannygod/ITArticles/master/assets/jvm_architecture_simple.png)


JVM主要包括四个部分：

1. 类加载器（ClassLoader）:在JVM启动时或者在类运行时将需要的class加载到JVM中。

2. 执行引擎：负责执行`.class`文件中包含的字节码指令

3. 内存区（也叫运行时数据区）：是在JVM运行的时候操作所分配的内存区。

4. 本地方法接口：主要是调用C或C++实现的本地方法及返回结果。
