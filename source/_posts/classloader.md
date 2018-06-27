---
title: Classloader 
date: 2018-05-08 12:52:08
tags: Classloader
categories: Classloader
---


# 什么是 Java 类加载机制?

Java 虚拟机一般使用 Java 类的流程为：首先将开发者编写的 Java 源代码（.java文件）编译成 Java 字节码（.class文件），然后类加载器会读取这个 .class 文件，并转换成 java.lang.Class 的实例。有了该 Class 实例后，Java 虚拟机可以利用 newInstance 之类的方法创建其真正对象了。

ClassLoader 是 Java 提供的类加载器，绝大多数的类加载器都继承自 ClassLoader，它们被用来加载不同来源的 Class 文件。
## Class 文件有哪些来源呢?

上文提到了 ClassLoader 可以去加载多种来源的 Class，那么具体有哪些来源呢？

首先，最常见的是开发者在应用程序中编写的类，这些类位于项目目录下；

然后，有 Java 内部自带的核心类如 java.lang、java.math、java.io 等 package 内部的类，位于 $JAVA_HOME/jre/lib/ 目录下，如 java.lang.String 类就是定义在 $JAVA_HOME/jre/lib/rt.jar 文件里；

另外，还有 Java 核心扩展类，位于 $JAVA_HOME/jre/lib/ext 目录下。开发者也可以把自己编写的类打包成 jar 文件放入该目录下；

最后还有一种，是动态加载远程的 .class 文件。

既然有这么多种类的来源，那么在 Java 里，是由某一个具体的 ClassLoader 来统一加载呢？还是由多个 ClassLoader 来协作加载呢？
# 哪些 ClassLoader 负责加载上面几类 Class？

实际上，针对上面四种来源的类，分别有不同的加载器负责加载。

首先，我们来看级别最高的 Java 核心类，即$JAVA_HOME/jre/lib 里的核心 jar 文件。这些类是 Java 运行的基础类，由一个名为 BootstrapClassLoader 加载器负责加载，它也被称作 根加载器／引导加载器。注意，BootstrapClassLoader 比较特殊，它不继承 ClassLoader，而是由 JVM 内部实现；

然后，需要加载 Java 核心扩展类，即 $JAVA_HOME/jre/lib/ext 目录下的 jar 文件。这些文件由 ExtensionClassLoader 负责加载，它也被称作 扩展类加载器。当然，用户如果把自己开发的 jar 文件放在这个目录，也会被 ExtClassLoader 加载；

接下来是开发者在项目中编写的类，这些文件将由 AppClassLoader 加载器进行加载，它也被称作 系统类加载器 System ClassLoader；

最后，如果想远程加载如（本地文件／网络下载）的方式，则必须要自己自定义一个 ClassLoader，复写其中的 findClass() 方法才能得以实现。

因此能看出，Java 里提供了至少四类 ClassLoader 来分别加载不同来源的 Class。

那么，这几种 ClassLoader 是如何协作来加载一个类呢？
# 这些 ClassLoader 以何种方式来协作加载 String 类呢？

String 类是 Java 自带的最常用的一个类，现在的问题是，JVM 将以何种方式把 String class 加载进来呢？

我们来猜想下。

首先，String 类属于 Java 核心类，位于 $JAVA_HOME/jre/lib 目录下。有的朋友会马上反应过来，上文中提过了，该目录下的类会由 BootstrapClassLoader 进行加载。没错，它确实是由 BootstrapClassLoader 进行加载。但，这种回答的前提是你已经知道了 String 在 $JAVA_HOME/jre/lib 目录下。

那么，如果你并不知道 String 类究竟位于哪呢？或者我希望你去加载一个 unknown 的类呢？

有的朋友这时会说，那很简单，只要去遍历一遍所有的类，看看这个 unknown 的类位于哪里，然后再用对应的加载器去加载。

是的，思路很正确。那应该如何去遍历呢？

比如，可以先遍历用户自己写的类，如果找到了就用 AppClassLoader 去加载；否则去遍历 Java 核心类目录，找到了就用 BootstrapClassLoader 去加载，否则就去遍历 Java 扩展类库，依次类推。

这种思路方向是正确的，不过存在一个漏洞。

假如开发者自己伪造了一个 java.lang.String 类，即在项目中创建一个包java.lang，包内创建一个名为 String 的类，这完全可以做到。那如果利用上面的遍历方法，是不是这个项目中用到的 String 不是都变成了这个伪造的 java.lang.String 类吗？如何解决这个问题呢？

解决方法很简单，当查找一个类时，优先遍历最高级别的 Java 核心类，然后再去遍历 Java 核心扩展类，最后再遍历用户自定义类，而且这个遍历过程是一旦找到就立即停止遍历。

在 Java 中，这种实现方式也称作 双亲委托（Delegation Principle）。其实很简单，把 BootstrapClassLoader 想象为核心高层领导人， ExtClassLoader 想象为中层干部， AppClassLoader 想象为普通公务员。每次需要加载一个类，先获取一个系统加载器 AppClassLoader 的实例（ClassLoader.getSystemClassLoader()），然后向上级层层请求，由最上级优先去加载，如果上级觉得这些类不属于核心类，就可以下放到各子级负责人去自行加载。

如下图所示：
![](aaa.png)

# 真的是按照双亲委托方式进行类加载吗？

下面通过几个例子来验证上面的加载方式。
开发者自定义的类会被 AppClassLoader 加载吗？

在项目中创建一个名为 MusicPlayer 的类文件，内容如下：

    package classloader;
    
    public class MusicPlayer {
    	public void print() {
    		System.out.printf("Hi I'm MusicPlayer");
    	}
    }

然后来加载 MusicPlayer。

    private static void loadClass() throws ClassNotFoundException {
    Class<?> clazz = Class.forName("classloader.MusicPlayer");
    ClassLoader classLoader = clazz.getClassLoader();
    System.out.printf("ClassLoader is %s", classLoader.getClass().getSimpleName());
    }

打印结果为：

	ClassLoader is AppClassLoader

可以验证，MusicPlayer 是由 AppClassLoader 进行的加载。
验证 AppClassLoader 的双亲真的是 ExtClassLoader 和 BootstrapClassLoader 吗？

这时发现 AppClassLoader 提供了一个 getParent() 的方法，来打印看看都是什么。

	private static void printParent() throws ClassNotFoundException {
	        Class<?> clazz = Class.forName("classloader.MusicPlayer");
	        ClassLoader classLoader = clazz.getClassLoader();
	        System.out.printf("currentClassLoader is %s\n", classLoader.getClass().getSimpleName());
	
	        while (classLoader.getParent() != null) {
	            classLoader = classLoader.getParent();
	            System.out.printf("Parent is %s\n", classLoader.getClass().getSimpleName());
	        }
	}

打印结果为：

	currentClassLoader is AppClassLoader
	Parent is ExtClassLoader

首先能看到 ExtClassLoader 确实是 AppClassLoader 的双亲，不过却没有看到 BootstrapClassLoader。事实上，上文就提过， BootstrapClassLoader比较特殊，它是由 JVM 内部实现的，所以 ExtClassLoader.getParent() = null。
如果把 MusicPlayer 类挪到 $JAVA_HOME/jre/lib/ext 目录下会发生什么？

上文中说了，ExtClassLoader 会加载$JAVA_HOME/jre/lib/ext 目录下所有的 jar 文件。那来尝试下直接把 MusicPlayer 这个类放到 $JAVA_HOME/jre/lib/ext 目录下吧。

利用下面命令可以把 MusicPlayer.java 编译打包成 jar 文件，并放置到对应目录。

	javac classloader/MusicPlayer.java
	jar cvf MusicPlayer.jar classloader/MusicPlayer.class
	mv MusicPlayer.jar $JAVA_HOME/jre/lib/ext/

这时 MusicPlayer.jar 已经被放置与 $JAVA_HOME/jre/lib/ext 目录下，同时把之前的 MusicPlayer 删除，而且这一次刻意使用 AppClassLoader 来加载：
	
	private static void loadClass() throws ClassNotFoundException {
	    ClassLoader appClassLoader = ClassLoader.getSystemClassLoader(); // AppClassLoader
	    Class<?> clazz = appClassLoader.loadClass("classloader.MusicPlayer");
	    ClassLoader classLoader = clazz.getClassLoader();
	    System.out.printf("ClassLoader is %s", classLoader.getClass().getSimpleName());
	}

打印结果为：

ClassLoader is ExtClassLoader

说明即使直接用 AppClassLoader 去加载，它仍然会被 ExtClassLoader 加载到。
# 从源码角度真正理解双亲委托加载机制

上面已经通过一些例子了解了双亲委托的一些特性了，下面来看一下它的实现代码，加深理解。

打开 ClassLoader 里的 loadClass() 方法，便是需要分析的源码了。这个方法里做了下面几件事：

    1. 检查目标class是否曾经加载过，如果加载过则直接返回；
    2. 如果没加载过，把加载请求传递给 parent 加载器去加载；
    3. 如果 parent 加载器加载成功，则直接返回；
    4. 如果 parent 未加载到，则自身调用 findClass() 方法进行寻找，并把寻找结果返回。

代码如下：

	protected Class<?> loadClass(String name, boolean resolve)
	    throws ClassNotFoundException
	{
	    synchronized (getClassLoadingLock(name)) {
	        // 1. 检查是否曾加载过
	        Class<?> c = findLoadedClass(name);
	        if (c == null) {
	            long t0 = System.nanoTime();
	            try {
	                if (parent != null) {
	                	// 优先让 parent 加载器去加载
	                    c = parent.loadClass(name, false);
	                } else {
	                	// 如无 parent，表示当前是 BootstrapClassLoader，调用 native 方法去 JVM 加载
	                    c = findBootstrapClassOrNull(name);
	                }
	            } catch (ClassNotFoundException e) {
	                // ClassNotFoundException thrown if class not found
	                // from the non-null parent class loader
	            }
	
	            if (c == null) {
	            	// 如果 parent 均没有加载到目标class，调用自身的 findClass() 方法去搜索
	                long t1 = System.nanoTime();
	                c = findClass(name);
	
	                // this is the defining class loader; record the stats
	                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
	                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
	                sun.misc.PerfCounter.getFindClasses().increment();
	            }
	        }
	        if (resolve) {
	            resolveClass(c);
	        }
	        return c;
	    }
	}

	// BootstrapClassLoader 会调用 native 方法去 JVM 加载
	private native Class<?> findBootstrapClass(String name);

看完实现源码相信能够有更完整的理解。
# 类加载器最酷的一面：自定义类加载器

前面提到了 Java 自带的加载器 BootstrapClassLoader、AppClassLoader和ExtClassLoader，这些都是 Java 已经提供好的。

而真正有意思的，是 自定义类加载器，它允许我们在运行时可以从本地磁盘或网络上动态加载自定义类。这使得开发者可以动态修复某些有问题的类，热更新代码。

下面来实现一个网络类加载器，这个加载器可以从网络上动态下载 .class 文件并加载到虚拟机中使用。

后面我还会写作与 热修复／动态更新 相关的文章，这里先学习 Java 层 NetworkClassLoader 相关的原理。

1. 作为一个 NetworkClassLoader，它首先要继承 ClassLoader；
2. 然后它要实现ClassLoader内的 findClass() 方法。注意，不是loadClass()方法，因为ClassLoader提供了loadClass()（如上面的源码），它会基于双亲委托机制去搜索某个 class，直到搜索不到才会调用自身的findClass()，如果直接复写loadClass()，那还要实现双亲委托机制；
3. 在 findClass() 方法里，要从网络上下载一个 .class 文件，然后转化成 Class 对象供虚拟机使用。

具体实现代码如下：

	/**
	 * Load class from network
	 */
	public class NetworkClassLoader extends ClassLoader {
	
	    @Override
	    protected Class<?> findClass(String name) throws ClassNotFoundException {
	        byte[] classData = downloadClassData(name); // 从远程下载
	        if (classData == null) {
	            super.findClass(name); // 未找到，抛异常
	        } else {
	            return defineClass(name, classData, 0, classData.length); // convert class byte data to Class<?> object
	        }
	        return null;
	    }
	
	    private byte[] downloadClassData(String name) {
	        // 从 localhost 下载 .class 文件
	        String path = "http://localhost" + File.separatorChar + "java" + File.separatorChar + name.replace('.', File.separatorChar) + ".class"; 
	
	        try {
	            URL url = new URL(path);
	            InputStream ins = url.openStream();
	            ByteArrayOutputStream baos = new ByteArrayOutputStream();
	            int bufferSize = 4096;
	            byte[] buffer = new byte[bufferSize];
	            int bytesNumRead = 0;
	            while ((bytesNumRead = ins.read(buffer)) != -1) {
	                baos.write(buffer, 0, bytesNumRead); // 把下载的二进制数据存入 ByteArrayOutputStream
	            }
	            return baos.toByteArray();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	        return null;
	    }
	
	    public String getName() {
	        System.out.printf("Real NetworkClassLoader\n");
	        return "networkClassLoader";
	    }
	}

这个类的作用是从网络上（这里是本人的 local apache 服务器 http://localhost/java 上）目录里去下载对应的 .class 文件，并转换成 Class<?> 返回回去使用。

下面我们来利用这个 NetworkClassLoader 去加载 localhost 上的 MusicPlayer 类：



1. 首先把 MusicPlayer.class 放置于 /Library/WebServer/Documents/java （MacOS）目录下，由于 MacOS 自带 apache 服务器，这里是服务器的默认目录；


2. 执行下面一段代码：

    String className = "classloader.NetworkClass";
    NetworkClassLoader networkClassLoader = new NetworkClassLoader();
    Class<?> clazz  = networkClassLoader.loadClass(className);

3. 正常运行，加载 http://localhost/java/classloader/MusicPlayer.class成功。

可以看出 NetworkClassLoader 可以正常工作，如果读者要用的话，只要稍微修改 url 的拼接方式即可自行使用。

# 类加载器与Web容器

对于运行在 Java EE™容器中的 Web 应用来说，类加载器的实现方式与一般的 Java 应用有所不同。不同的 Web 容器的实现方式也会有所不同。以 Apache Tomcat 来说，每个Web 应用都有一个对应的类加载器实例。该类加载器也使用代理模式，所不同的是它是首先尝试去加载某个类，如果找不到再代理给父类加载器。这与一般类加载器的顺序是相反的。这是 Java Servlet 规范中的推荐做法，其目的是使得Web 应用自己的类的优先级高于 Web 容器提供的类。这种代理模式的一个例外是：Java 核心库的类是不在查找范围之内的。这也是为了保证 Java 核心库的类型安全。

绝大多数情况下，Web 应用的开发人员不需要考虑与类加载器相关的细节。下面给出几条简单的原则：

1. 每个 Web 应用自己的 Java 类文件和使用的库的 jar 包，分别放在 WEB-INF/classes和 WEB-INF/lib目录下面。

2. 多个应用共享的 Java 类文件和 jar 包，分别放在 Web 容器指定的由所有 Web 应用共享的目录下面。

3. 当出现找不到类的错误时，检查当前类的类加载器和当前线程的上下文类加载器是否正确

# 小结

类加载方式是 Java 上非常创新的一项技术，给未来的热修复技术提供了可能。本文力求通过简单的语言和合适的例子来讲解其中双亲委托机制、自定义加载器等，并开发了自定义的NetworkClassLoader。

当然，类加载是很有意思的技术，很难覆盖所有知识点，比如不同类加载器加载同一个类，得到的实例却不是同一个等等。


# 委托原则(Delegation Principle)

    如果一个类还没有被加载，类加载器会委托它的父亲加载器去加载它。

这种委托会一直延续，直到到达委托层次的最顶层，由原始的类加载器加载完成该类。下图展示了这种情况。

Systm-ClassPath classloader 加载了 MyApp.MyApp，而这个类创造了一个 java.util.Vector。假设现在 java.util.Vector 还没又被加载。因为 System-Classpath classloader 加载了 MyApp 类，它首先请求它的父亲 extension classloader 来加载这个类（java.util.Vector）。而 extension classloader 又请求 Bootstrap classloader 尝试加载。因为 java.util.Vector 是 J2SE 类，bootstrap classloader 成功加载了它。

考虑一个当略微不同的情况，如下图。

在这种情况中，MyApp 创造了一个新的用户自定义类的实例，MyClass。假设 MyClass 还没有被加载。像以往一样，当 System-Classpath classloader 接收到这个加载请求，它委托了它的父亲。最终这个委托传递到了 Bootstrap classloader。但是在 java 核心 API 里，找不到这个类。所以它的孩子加载器 Extensions classloader 尝试加载它。同样的，Extensions classloader 也没有找到它。最终，委托请求回到了 System-Classpath classloader 这里。它找到了这个类并加载成功。
# 可见性原则（Visibility principle）

    被父亲类加载器加载的类对于孩子加载器是可见的，但关系相反相反则不可见。

这说明，一个类只能看见它自己的加载器或者这个加载器的父类加载器加载的类，反过来是不可以的。比如，被 ClassX 的父亲加载器加载的类是不能看见 ClassX 的。为了更清楚的理解，让我们来看一个例子，如下图。

图中展示了四个类加载器。类加载器 A 是最顶层的加载器，B 是它的孩子。类加载器 X 和 Y 是 B 的孩子。他们各自都加载了与自己同名的类。类加载器 A 能看见 A 类，类加载器 B 能看见 A，B 类。类似的，X 能看见 A，B，X，Y 能看见 A，B，Y。但兄弟、Y 之间的类是不可见的。
# 独特性原则（Uniqueness Principle）

    当一个类加载器加载一个类时，它的孩子加载器绝不会重新加载这个类。

这是因为委托原则中，一个加载器总是会委托自己的父亲加载器加载类。当层次中的父亲加载器无法加载类的时候，孩子类加载器就会（或者尝试去）加载这个类。这样，类加载的独特性就得到了保障。当父亲和孩子加载器加载了同一个类，一个有趣的情况就出现了。你可能会想这怎么可能出现？这不是违反了独特性原则？

我们用可见性原则中的示例图来解释这个问题。我们假设没有任何类被加载到这些类加载器的层次结构中。假设 X 类被类加载器 X 加载，它强制性的用类加载器 X 加载 B 类。这可以通过像 Class.Name()这样的 API 来实现，代码如下：

	 public class X {
	
	   public X() {
	      ClassLoader cl = this.getClass().getClassLoader();
	      Class B = Class.forName(“B”, true, cl);
	   }
	}

在 X 的构造函数中，B 被显示的使用类加载器 X 加载。如果另一个被类加载器 B 加载的类需要访问 B 类，则无法实现，因为委托原则只能向父亲方向查询。如果类加载器 B 也加载了 B 类，当比较两个 B 类的实例时，如果一个实例来自于类加载器 X，一个来自于类加载器 B，则会抛出 ClassCastException 异常。



Ref
https://blog.csdn.net/briblue/article/details/54973413