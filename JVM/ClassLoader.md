- ## ClassLoader 
    - ### 作用:
        负责加载.class文件,.class文件在文件开头有特定的文件标识,将class文件字节码内容加载到内存中,并将这些内容转化成方法区中的运行时数据结构并且ClassLoader只负责.class文件的加载,至于它是否可以运行,则由Execution Engine决定 
    - ### 分类
        - #### BootstrapClassLoader(启动类加载器) ：
            最顶层的加载类，由C++ 实现，负责加载 ==%JAVA_HOME%/lib==目录下的jar包和类或者或被 ==Xbootclasspath==参数指定的路径中的所有类。
        - #### ExtensionClassLoader(扩展类加载器) ：
            主要负责加载目录 %JRE_HOME%/lib/ext`目录下的jar包和类，或被 ==java.ext.dirs== 系统变量所指定的路径下的jar包
        - #### AppClassLoader(应用程序类加载器):
            面向我们用户的加载器，负责加载==当前应用classpath==下的所有jar包和类。
        - #### 用户自定义加载器
            Java.Lang.ClassLoader的子类,用户可以定制类的加载方式 
        ![image](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1583238914996&di=af4b583c07d7fa68caa8708192273e9c&imgtype=jpg&src=http%3A%2F%2Fimg1.imgtn.bdimg.com%2Fit%2Fu%3D1057950551%2C2474137492%26fm%3D214%26gp%3D0.jpg)
    - ### 双亲委派机制
        即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。加载的时候，==首先会把该请求委派该父类加载器的 loadClass() 处理==，因此所有的请求最终都应该传送到顶层的启动类加载器 BootstrapClassLoader 中。当父类加载器无法处理时，才由自己来处理。当父类加载器为null时，会使用启动类加载器 BootstrapClassLoader 作为父类加载器。 
    ![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/classloader_WPS%E5%9B%BE%E7%89%87.png)
        - #### 好处:
            - 保证了Java程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类）
            - 保证了 Java 的核心 API 不被篡改。如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 java.lang.Object 类的话，那么程序运行的时候，系统就会出现多个不同的 Object 类

        - 源码

```java
private final ClassLoader parent; 
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查请求的类是否已经被加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {//父加载器不为空，调用父加载器loadClass()方法处理
                        c = parent.loadClass(name, false);
                    } else {//父加载器为空，使用启动类加载器 BootstrapClassLoader 加载
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                   //抛出异常说明父类加载器无法完成加载请求
                }

                if (c == null) {
                    long t1 = System.nanoTime();
                    //自己尝试加载
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
```
### 如何打破双亲委派机制:
- 自定义加载器
    - 只需自定义一个加载器类,继承ClassLoader类并重写loadClass和findClass方法

##### 参考博客

[https://snailclimb.gitee.io/javaguide/#/docs/java/jvm/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8](https://snailclimb.gitee.io/javaguide/#/docs/java/jvm/类加载器)