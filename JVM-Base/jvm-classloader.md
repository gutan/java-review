## ClassLoader

#### 是什么？

> ClassLoader 即类加载器，将 Class 文件加载到 JVM 虚拟机中。（将 Class 的字节码形式转换成内存形式的 Class 对象，字节码可以来自于磁盘文件 *.class，也可以是 jar 包里的 *.class，也可以来自远程服务器提供的字节流，字节码的本质就是一个字节数组 []byte，它有特定的复杂的内部格式）
>

#### 如何工作？

> JVM 中内置了三个重要的 ClassLoader，分别是 BootstrapClassLoader、ExtClassLoader 和 AppClassLoader
>
> BootstrapClassLoader 负责加载 JVM 运行时核心类，这些类位于 $JAVA_HOME/lib/rt.jar 文件中，我们常用内置库 java 开头，比如 java.util.*、java.io.*、java.nio.*、java.lang.*等等。这个 ClassLoader 比较特殊，它是由 C 代码实现的，我们将它称之为「根加载器」。
>
> ExtClassLoader 负责加载 JVM 扩展类，比如 swing 系列、内置的 js 引擎、xml 解析器 等等，这些库名通常以 javax 开头，它们的 jar 包位于 $JAVA_HOME/lib/ext/*.jar 中，有很多 jar 包。
>
> AppClassLoader 也称为 SystemAppClass，加载当前应用的 classpath 的所有类。我们自己编写的代码以及使用的第三方 jar 包通常都是由它来加载的。

#### 类加载器知识点

```java
public class ClassLoaderTest {
    public static void main(String[] args) {
        ClassLoader cl = ClassLoaderTest.class.getClassLoader();
        System.out.println("ClassLoaderTest's cl = [" + cl + "]");
        System.out.println("                  cl's parent = [" + cl.getParent() + "]");
        System.out.println("                  cl's parent's parent = [" + cl.getParent().getParent() + "]");
        System.out.println("String's cl = [" + String.class.getClassLoader() + "]");
    }
}

//ClassLoaderTest's cl = [sun.misc.Launcher$AppClassLoader@18b4aac2]
//                  cl's parent = [sun.misc.Launcher$ExtClassLoader@6f94fa3e]
//                  cl's parent's parent = [null]
//String's cl = [null]
```

1. JVM 运行并不是一次性加载所需要的全部类的，它是按需加载，也就是延迟加载。程序在运行的过程中会逐渐遇到很多不认识的新类，这时候就会调用 ClassLoader 来加载这些类。加载完成后就会将 Class 对象存在 ClassLoader 里面，下次就不需要重新加载了。
2. 每个 Class 对象的内部都有一个 classLoader 字段来标识加载自己的 ClassLoader 。ClassLoader 就像一个容器，里面装了很多已经加载的 Class 对象。
3. BootstrapClassLoader、ExtClassLoader、AppClassLoader 实际是查阅相应的环境属性 `sun.boot.class.path`、`java.ext.dirs` 和 `java.class.path` 来加载资源文件。
4. sun.misc.Launcher 类是 java 的入口，在启动 java 应用的时候 JVM  首先创建 Launcher 类，创建 Launcher类的时候会初始化 ExtClassLoader、AppClassLoader类加载器，并为 ContextClassLoader 设置为刚创建的 AppClassLoader 对象。
5. 每个类加载器都有一个父加载器

> getParent()  实际上返回的就是一个 ClassLoader 对象 parent，parent 的赋值是在 ClassLoader 对象的构造方法中，它有两个情况：
>
> ```
> public abstract class ClassLoader {
> protected ClassLoader(ClassLoader parent) {
>  this(checkCreateClassLoader(), parent);
> }
> protected ClassLoader() {
>  this(checkCreateClassLoader(), getSystemClassLoader());
> }
> ...
> }
> ```
>
> 1. 由外部类创建 ClassLoader 时直接指定一个ClassLoader为parent。
>
> 2. 由 getSystemClassLoader() 方法生成，也就是在 sun.misc.Laucher 通过 getClassLoader() 获取，也就是AppClassLoader。即一个 ClassLoader 创建时如果没有指定 parent 默认就是 AppClassLoader。
>
>    ```java
>    public Launcher() {
>      ...
>      extld = Launcher.ExtClassLoader.getExtClassLoader();
>    	...
>    	this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
>    	...
>    	Thread.currentThread().setContextClassLoader(this.loader);
>    }
>    ```
>

5. ExtClassLoader 类加载器的父加载器为 BootstrapClassLoader，但通过 getParent()  获取为 null

> Bootstrap ClassLoader 是由 C/C++ 编写的，它本身是虚拟机的一部分，所以它并不是一个 JAVA 类，也就是无法在 java 代码中获取它的引用



#### 如何加载类？双亲委派

> 双亲委派：
>
> 如果一个类加载器收到了加载某个类的请求,则该类加载器并不会去加载该类,而是把这个请求委派给父类加载器,每一个层次的类加载器都是如此,因此所有的类加载请求最终都会传送到顶端的启动类加载器;只有当父类加载器在其搜索范围内无法找到所需的类,并将该结果反馈给子类加载器,子类加载器会尝试去自己加载。
>
>  原因：
>
> 如果不是同一个类加载器加载，即使是相同的class文件，也会出现判断不相同的情况，从而引发一些意想不到的情况，为了保证相同的 class 文件，在使用的时候，是相同的对象，jvm 设计的时候，采用了双亲委派的方式来加载类。

![classloader-双亲委派](https://tva1.sinaimg.cn/large/006y8mN6gy1g75u3zf54gj30k70dtaal.jpg)



1. 子类加载器先委托父类加载器加载
2. 父类加载器有自己的**加载范围**，范围内没有找到，则不加载，并返回给子类
3. 子类在收到父类无法加载的时候，才会自己去加载

#### 重要方法 loadClass()

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
    {
        // 同步上锁
        synchronized (getClassLoadingLock(name)) {
            // 先查看这个类是不是已经加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 递归，双亲委派的实现，先获取父类加载器，不为空则交给父类加载器
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    // 前面提到，bootstrap classloader的类加载器为null，通过find方法来获得
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                }

                if (c == null) {
                    // 如果还是没有获得该类，调用findClass找到类
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // jvm统计
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            // 连接类
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

> 实现步骤：
>
> 1. 执行 findLoadedClass(String) 检测 class 是不是已经加载。
> 2. 执行父加载器 loadClass 方法。如果父加载器为 null，则 jvm 内置的加载器即通过 Bootstrap ClassLoader 加载。这也解释了 ExtClassLoader.parent 为 null, 仍然以 Bootstrap ClassLoader作为父加载器。
> 3. 如果向上委托父加载器没有加载成功，则通过 findClass(String) 查找。

自定义一个 ClassLoader，建议覆盖 `findClass()` 方法，而不要直接改写 `loadClass()` 方法

#### 自定义 ClassLoader

> 实现步骤
>
> 1. 编写一个类继承自 ClassLoader 抽象类。
> 2. 覆写它的 `findClass()` 方法。
> 3. 在 `findClass()` 方法中调用 `defineClass()` 。

方法 defineClass() 作用：将 class 二进制内容转换成 Class 对象，如果不符合要求的会抛出各种异常。

有很多字节码加密技术就是依靠定制 ClassLoader 来实现的。先使用工具对字节码文件进行加密，运行时使用定制的 ClassLoader 先解密文件内容再加载这些解密后的字节码

#### ContextClassLoader

 sun.misc.Laucher 对象构造函数初始化，Thread.currentThread() 设置为 AppClassLoader，即程序启动时的 main 线程的 contextClassLoader 就是 AppClassLoader。其次线程的 contextClassLoader 默认是从父线程那里继承（Thread 对象构造函数复制父线程 contextClassLoader），故可以做到跨线程共享同一个 contextClassLoader。父子线程之间会自动传递 contextClassLoader，所以共享起来将是自动化的。

#### 破坏“双亲委派”

> 因为在某些情况下父类加载器需要委托子类加载器去加载 class 文件。受到加载范围的限制，父类加载器无法加载到需要的文件，以 Driver 接口为例，由于 Driver 接口定义在jdk当中的，而其实现由各个数据库的服务商来提供，比如 mysql 的就写了`MySQL Connector`，那么问题就来了，DriverManager（也由 jdk 提供）要加载各个实现 Driver 接口的实现类，然后进行管理，但是 DriverManager 由启动类加载器加载，只能记载JAVA_HOME 的 lib 下文件，而其实现是由服务商提供的，由系统类加载器加载，这个时候就需要启动类加载器来委托子类来加载 Driver 实现，从而破坏了双亲委派，这里仅仅是举了破坏双亲委派的其中一个情况。

跟着关键源码分析（最好打开源码跟着跳进去）：

```java
//main 函数部分代码
static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
static final String DB_URL = "jdbc:mysql://localhost:3306/demo";
//JDBC 4.0 drivers 不需要 Class.forName(JDBC_DRIVER) https://stackoverflow.com/questions/28220724/class-fornamejdbc-driver-no-longer-needed
//Class.forName(JDBC_DRIVER)
conn = DriverManager.getConnection(DB_URL, USER, PASS);

//DriverManager
public class DriverManager {
  static {
    	...
      loadInitialDrivers();
    	...
  }
  private static void loadInitialDrivers() {
  	...
    ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
    Iterator<Driver> driversIterator = loadedDrivers.iterator();
    while(driversIterator.hasNext()) {
      driversIterator.next();
    }
    ...
  }

}
//ServiceLoader
public static <S> ServiceLoader<S> load(Class<S> service) {
  //上文提到 contextClassLoader 默认加载器 AppClassLoader
  ClassLoader cl = Thread.currentThread().getContextClassLoader();
  return ServiceLoader.load(service, cl);
}

public static <S> ServiceLoader<S> load(Class<S> service,
                                        ClassLoader loader){
    return new ServiceLoader<>(service, loader);
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}

public void reload() {
    providers.clear();
    lookupIterator = new LazyIterator(service, loader);
}
//ServiceLoader 私有内部类 LazyIterator
private class LazyIterator implements Iterator<S>{
    // ServiceLoader 的 iterator() 方法最后调用的是这个迭代器里的next
    public S next() {
        if (acc == null) {
            return nextService();
        } else {
            PrivilegedAction<S> action = new PrivilegedAction<S>() {
                public S run() { return nextService(); }
            };
            return AccessController.doPrivileged(action, acc);
        }
    }
    
    private S nextService() {
        if (!hasNextService())
            throw new NoSuchElementException();
        String cn = nextName;
        nextName = null;
        Class<?> c = null;
        // 根据名字来加载类
        try {
            c = Class.forName(cn, false, loader);
        } catch (ClassNotFoundException x) {
            fail(service,
                 "Provider " + cn + " not found");
        }
        if (!service.isAssignableFrom(c)) {
            fail(service,
                 "Provider " + cn  + " not a subtype");
        }
        try {
            S p = service.cast(c.newInstance());
            providers.put(cn, p);
            return p;
        } catch (Throwable x) {
            fail(service,
                 "Provider " + cn + " could not be instantiated",
                 x);
        }
        throw new Error();          // This cannot happen
    }
    
    public boolean hasNext() {
        if (acc == null) {
            return hasNextService();
        } else {
            PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
                public Boolean run() { return hasNextService(); }
            };
            return AccessController.doPrivileged(action, acc);
        }
    }
    
    private boolean hasNextService() {
        if (nextName != null) {
            return true;
        }
        if (configs == null) {
            try {
                // 在classpath下查找META-INF/services/java.sql.Driver名字的文件夹
                // private static final String PREFIX = "META-INF/services/";
                String fullName = PREFIX + service.getName();
                if (loader == null)
                    configs = ClassLoader.getSystemResources(fullName);
                else
                    configs = loader.getResources(fullName);
            } catch (IOException x) {
                fail(service, "Error locating configuration files", x);
            }
        }
        while ((pending == null) || !pending.hasNext()) {
            if (!configs.hasMoreElements()) {
                return false;
            }
            pending = parse(service, configs.nextElement());
        }
        nextName = pending.next();
        return true;
    }

}
```

在 JDBC4.0 之前，连接数据库的时候，通常会用 `Class.forName("com.mysql.jdbc.Driver")` 先加载数据库相关的驱动，然后再进行获取连接等的操作。而 JDBC4.0 之后不需要 `Class.forName` 来加载驱动，直接获取连接即可，这里使用了 Jav a的 [SPI](java-spi.md) 扩展机制来实现。

[Java API 官方文档](https://docs.oracle.com/javase/6/docs/api/java/sql/DriverManager.html)

> The `DriverManager` methods `getConnection` and `getDrivers` have been enhanced to support the Java Standard Edition [Service Provider](https://docs.oracle.com/javase/6/docs/technotes/guides/jar/jar.html#Service Provider) mechanism. JDBC 4.0 Drivers must include the file `META-INF/services/java.sql.Driver`. This file contains the name of the JDBC drivers implementation of `java.sql.Driver`. For example, to load the `my.sql.Driver` class, the `META-INF/services/java.sql.Driver` file would contain the entry:
>
> ```
>  my.sql.Driver
> ```
>
> Applications no longer need to explictly load JDBC drivers using `Class.forName()`. Existing programs which currently load JDBC drivers using `Class.forName()` will continue to work without modification.

#### Class.forName()

```
//Class 静态方法 forName 作用是通过调用者的加载器加载指定类
A a = (A)Class.forName(“pacage.A”).newInstance();
效果等同
A a = new A()；
```

注意：

1. newInstance( ) 是一个方法，而 new 是一个关键字；
2. Class.newInstance( ) 使用有局限，因为它生成对象只能调用无参的构造函数，而使用 new 关键字生成对象没有这个限制；
3. Class.forName() 返回 Class 对象，Class.forName().newInstance() 返回的具体对象。

## 巨人肩膀

[老大难的 Java ClassLoader 再不理解就老了](https://juejin.im/post/5c04892351882516e70dcc9b)，掘金，2018

[一看你就懂，超详细java中的ClassLoader详解](https://blog.csdn.net/briblue/article/details/54973413)，CSDN，2017

[【JVM】浅谈双亲委派和破坏双亲委派](https://www.cnblogs.com/joemsu/p/9310226.html)，博客园，2017

[Class.forName()用法详解](https://blog.csdn.net/Kaiwii/article/details/7405761)，CSDN，2012