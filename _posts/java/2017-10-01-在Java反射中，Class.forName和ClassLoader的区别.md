---
layout: post
title: "在Java反射中，Class.forName和ClassLoader的区别"
description: 在Java反射中，Class.forName和ClassLoader的区别
category: Java
---

来源[在Java反射中，Class.forName和ClassLoader的区别](https://www.toutiao.com/a6628740867957981700/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1543384943&app=news_article&utm_source=weixin&iid=51177022387&utm_medium=toutiao_ios&group_id=6628740867957981700)

# 解释

在java中Class.forName()和ClassLoader都可以对类进行加载。ClassLoader就是遵循双亲委派模型最终调用启动类加载器的类加载器，实现的功能是“通过一个类的全限定名来获取描述此类的二进制字节流”，获取到二进制流后放到JVM中。Class.forName()方法实际上也是调用的CLassLoader来实现的。

Class.forName(String className)；这个方法的源码是：

```java
@CallerSensitive
public static Class<?> forName(String className)
            throws ClassNotFoundException {
    Class<?> caller = Reflection.getCallerClass();
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
}
```


最后调用的方法是forName0这个方法，在这个forName0方法中的第二个参数被默认设置为了true，这个参数代表是否对加载的类进行初始化，设置为true时会类进行初始化，代表会执行类中的静态代码块，以及对静态变量的赋值等操作。

也可以调用Class.forName(String name, boolean initialize,ClassLoader loader)方法来手动选择在加载类的时候是否要对类进行初始化。Class.forName(String name, boolean initialize,ClassLoader loader)的源码如下：

```java
    /**
     * Returns the {@code Class} object associated with the class or
     * interface with the given string name, using the given class loader.
     * Given the fully qualified name for a class or interface (in the same
     * format returned by {@code getName}) this method attempts to
     * locate, load, and link the class or interface.  The specified class
     * loader is used to load the class or interface.  If the parameter
     * {@code loader} is null, the class is loaded through the bootstrap
     * class loader.  The class is initialized only if the
     * {@code initialize} parameter is {@code true} and if it has
     * not been initialized earlier.
     *
     * <p> If {@code name} denotes a primitive type or void, an attempt
     * will be made to locate a user-defined class in the unnamed package whose
     * name is {@code name}. Therefore, this method cannot be used to
     * obtain any of the {@code Class} objects representing primitive
     * types or void.
     *
     * <p> If {@code name} denotes an array class, the component type of
     * the array class is loaded but not initialized.
     *
     * <p> For example, in an instance method the expression:
     *
     * <blockquote>
     *  {@code Class.forName("Foo")}
     * </blockquote>
     *
     * is equivalent to:
     *
     * <blockquote>
     *  {@code Class.forName("Foo", true, this.getClass().getClassLoader())}
     * </blockquote>
     *
     * Note that this method throws errors related to loading, linking or
     * initializing as specified in Sections 12.2, 12.3 and 12.4 of <em>The
     * Java Language Specification</em>.
     * Note that this method does not check whether the requested class
     * is accessible to its caller.
     *
     * <p> If the {@code loader} is {@code null}, and a security
     * manager is present, and the caller's class loader is not null, then this
     * method calls the security manager's {@code checkPermission} method
     * with a {@code RuntimePermission("getClassLoader")} permission to
     * ensure it's ok to access the bootstrap class loader.
     *
     * @param name       fully qualified name of the desired class
     * @param initialize if {@code true} the class will be initialized.
     *                   See Section 12.4 of <em>The Java Language Specification</em>.
     * @param loader     class loader from which the class must be loaded
     * @return           class object representing the desired class
     *
     * @exception LinkageError if the linkage fails
     * @exception ExceptionInInitializerError if the initialization provoked
     *            by this method fails
     * @exception ClassNotFoundException if the class cannot be located by
     *            the specified class loader
     *
     * @see       java.lang.Class#forName(String)
     * @see       java.lang.ClassLoader
     * @since     1.2
     */
    @CallerSensitive
    public static Class<?> forName(String name, boolean initialize,
                                   ClassLoader loader)
        throws ClassNotFoundException
    {
        Class<?> caller = null;
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            // Reflective call to get caller class is only needed if a security manager
            // is present.  Avoid the overhead of making this call otherwise.
            caller = Reflection.getCallerClass();
            if (sun.misc.VM.isSystemDomainLoader(loader)) {
                ClassLoader ccl = ClassLoader.getClassLoader(caller);
                if (!sun.misc.VM.isSystemDomainLoader(ccl)) {
                    sm.checkPermission(
                        SecurityConstants.GET_CLASSLOADER_PERMISSION);
                }
            }
        }
        return forName0(name, initialize, loader, caller);
    }
```

源码中的注释只摘取了一部分，其中对参数initialize的描述是：if {@code true} the class will be initialized.意思就是说：如果参数为true，则加载的类将会被初始化。

# 举例

下面还是举例来说明结果吧：

一个含有静态代码块、静态变量、赋值给静态变量的静态方法的类


```java
public class ClassForName {
 //静态代码块
 static {
 System.out.println("执行了静态代码块");
 }
 //静态变量
 private static String staticFiled = staticMethod();
 //赋值静态变量的静态方法
 public static String staticMethod(){
 System.out.println("执行了静态方法");
 return "给静态字段赋值了";
 }
}
```

测试方法：


```java
public class MyTest {
 @Test
 public void test44(){
 try {
 Class.forName("com.test.mytest.ClassForName");
 System.out.println("#########分割符(上面是Class.forName的加载过程，下面是ClassLoader的加载过程)##########");
 ClassLoader.getSystemClassLoader().loadClass("com.test.mytest.ClassForName");
 } catch (ClassNotFoundException e) {
 e.printStackTrace();
 }
 }
}
```

运行结果：


```
执行了静态代码块
执行了静态方法
#########分割符(上面是Class.forName的加载过程，下面是ClassLoader的加载过程)##########
根据运行结果得出Class.forName加载类是将类进了初始化，而ClassLoader的loadClass并没有对类进行初始化，只是把类加载到了虚拟机中。
```


# 应用场景

在我们熟悉的Spring框架中的IOC的实现就是使用的ClassLoader。

而在我们使用JDBC时通常是使用Class.forName()方法来加载数据库连接驱动。这是因为在JDBC规范中明确要求Driver(数据库驱动)类必须向DriverManager注册自己。

以MySQL的驱动为例解释：


```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver { 
 // ~ Static fields/initializers 
 // --------------------------------------------- 
 // 
 // Register ourselves with the DriverManager 
 // 
 static { 
 try { 
 java.sql.DriverManager.registerDriver(new Driver()); 
 } catch (SQLException E) { 
 throw new RuntimeException("Can't register driver!"); 
 } 
 } 
 // ~ Constructors 
 // ----------------------------------------------------------- 
 /** 
 * Construct a new driver and register it with DriverManager 
 * 
 * @throws SQLException 
 * if a database error occurs. 
 */ 
 public Driver() throws SQLException { 
 // Required for Class.forName().newInstance() 
 } 
}
```

我们看到Driver注册到DriverManager中的操作写在了静态代码块中，这就是为什么在写JDBC时使用Class.forName()的原因了。
