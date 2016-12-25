### JVM ClassLoader

ClassLoader，即Java类加载器，主要作用是将class加载到JVM内，同时它还要考虑class由谁来加载。

工作方式(时机)：

1. 隐式：运行过程中，碰到`new`方式生成对象时，隐式调用classLoader到JVM

2. 显式：通过`Class.forname()`动态加载

了解类加载机制的好处：

1. **按需加载**: JVM启动时不能确定我要加载哪些东西，或者有些类非常大，我只希望用到它时再加载，并非一次性加载所有的class，所以这时候了解了加载机制就可以按需加载了。

2. **类隔离**: 比如web容器中部署多个应用，应用之间互相可能会有冲突，所以希望尽量隔离，这里可能就要分析各个应用加载的资源和加载顺序之间的冲突，针对这些冲突再自己定些规则，让它们能够愉快地玩耍。

3. **资源回收**: 如果你不了解java是如何加载资源的，又怎么理解java是如何回收资源的？

### 机制

一般说到java的类加载机制，都要说到“双亲委派模型”（英文叫“parent”）。使用这种机制，可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。JVM根据 `类名+包名+ClassLoader实例ID` 来判定两个类是否相同，是否已经加载过（所以这里可以略微扩展下，可以通过创建不同的classloader实例来实现类的热部署）。

#### 双亲委派模型

该模型要求除了顶层的`BootstrapClassLoader`启动类加载器外，其余的类加载器都应当有自己的**父类加载器**。子类加载器和父类加载器**不是以继承（Inheritance）的关系**来实现，而是通过**组合（Composition）关系**来复用父加载器的代码。每个类加载器都有自己的命名空间（由该加载器及所有父类加载器所加载的类组成，在同一个命名空间中，不会出现类的完整名字（包括类的包名）相同的两个类；在不同的命名空间中，有可能会出现类的完整名字（包括类的包名）相同的两个类

##### 工作过程

1. 当前 `ClassLoader` 首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类。

> 每个类加载器都有自己的加载缓存，当一个类被加载了以后就会放入缓存，等下次加载的时候就可以直接返回了。

2. 当前 `ClassLoader` 的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到 `BootstrapClassLoader` 。

3. 当所有的父类加载器都没有加载的时候，再由当前 `ClassLoader` 加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回。

##### 具体层次结构(父子关系)

**英文版**

![jvm-classloader-en](https://raw.githubusercontent.com/dannygod/ITArticles/master/assets/jvm_classloader_en.png)

**中文版**

![jvm-classloader-ch](https://raw.githubusercontent.com/dannygod/ITArticles/master/assets/jvm_classloader_ch.png)

**Note: 如果想形象地看到java的类加载顺序，可以在运行java的时候加个启动参数，即`java –verbose`**

+ **BootStrapClassLoader**: 它是最顶层的类加载器，是由C++编写而成, 已经内嵌到JVM中了。在JVM启动时会初始化该ClassLoader，它主要用来读取Java的核心类库JRE/lib/rt.jar中所有的class文件，这个jar文件中包含了java规范定义的所有接口及实现。

+ **ExtensionClassLoader**: 它是用来读取Java的一些扩展类库，如读取JRE/lib/ext/*.jar中的包等**（这里要注意，有些版本的是没有ext这个目录的）**。

+ **SystemClassLoader/AppClassLoader**: 它是用来读取CLASSPATH下指定的所有jar包或目录的类文件，一般情况下这个就是程序中默认的类加载器。

+ **UserDefinedClassLoader/CustomClassLoader**: 它是用户自定义编写的，它用来读取指定类文件 。基于自定义的ClassLoader可用于加载非Classpath中（如从网络上下载的jar或二进制）的jar及目录、还可以在加载前对class文件优一些动作，如解密、编码等。

此处需要具体说明的，**`ExtClassLoader`的父类加载器是`null`**，只不过在默认的`ClassLoader`的`loadClass`方法中，当parent为`null`时，是交给`BootStrapClassLoader`来处理的，而且`ExtClassLoader`没有重写默认的`loadClass`方法，所以，`ExtClassLoader`也会调用`BootStrapLoader`类加载器来加载，这就导致“BootStrapClassLoader具备了ExtClassLoader父类加载器的功能”。

通过下面来理解:

```java
public static void test() {
  ClassLoader appClassLoader = ClassLoader.getSystemClassLoader();
  System.out.println("系统类装载器:" + appClassLoader);
  ClassLoader extensionClassLoader = appClassLoader.getParent();
  System.out.println("系统类装载器的父类加载器——扩展类加载器:" + extensionClassLoader);
  ClassLoader bootstrapClassLoader = extensionClassLoader.getParent();
  System.out.println("扩展类加载器的父类加载器——引导类加载器:" + bootstrapClassLoader);
}
```

##### 层次结构的好处

主要是为了**安全性**，避免用户自己编写的类动态替换 Java 的一些核心类，比如`String类`，同时也避免了重复加载，因为 JVM 中区分不同类，不仅仅是根据类名，相同的 `.class` 文件被不同的 ClassLoader 加载就是不同的两个类，如果相互转型的话会抛`java.lang.ClassCaseException`

##### ClassLoader的3个重要方法

查看`ClassLoader`的源码可以发现三个重要的方法:

+ **loadClass**: `Classloader`加载类的入口，此方法负责加载指定名字的类，实现方法为先从已经加载的类中寻找，如没有则继续从父ClassLoader中寻找，如仍然没找到，则从BootstrapClassLoader中寻找，最后再调用`findClass`方法来寻找，如要改变类的加载顺序，则可覆盖此方法，如加载顺序相同，则可通过覆盖findClass来做特殊的处理，例如解密、固定路径寻找等，当通过整个寻找类的过程仍然未获取到Class对象时，则抛出`ClassNotFoundException`。如类需要resolve，则调用`resolveClass`进行链接。

+ **findClass**: 此方法直接抛出`ClassNotFoundException`，因此需要通过覆盖`loadClass`或此方法来以自定义的方式加载相应的类。

+ **defineClass**: 此方法负责将二进制的字节码转换为Class对象，这个方法对于自定义加载类而言非常重要，如二进制的字节码的格式不符合JVM Class文件的格式，抛出`ClassFormatError`；如需要生成的类名和二进制字节码中的不同，则抛出`NoClassDefFoundError`；如需要加载的class是受保护的、采用不同签名的或类名是以java.开头的，则抛出`SecurityException`；如需加载的class在此ClassLoader中已加载，则抛出`LinkageError`。

