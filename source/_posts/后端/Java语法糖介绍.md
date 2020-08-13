title: Java语法糖介绍
tags: [Java, 语法糖]
date: 2020-08-13
categories: 后端
---

# 概述
1. 什么是语法糖
> 语法糖（Syntactic sugar），也译为糖衣语法，是由英国计算机科学家彼得·兰丁发明的一个术语，指计算机语言中添加的某种语法，这种语法对语言的功能没有影响，但是更方便程序员使用。

2. 能够带来的好处
> 语法糖让程序更加简洁，有更高的可读性

3. 有哪些语法糖
   1. 自动拆箱、装箱
   2. 泛型擦除
   3. 不定长参数
   4. 迭代器
   5. 枚举
   6. switch支持枚举和字符串
   7. 内部类
   8. try-with-resources
   9. lambda

<!-- more -->
# 自动拆箱、装箱
1. Java是面向对象编程（万物皆对象）
2. 对象即需要new出来的，但是想想基本数据类型（int/double/boolean...）,并不需要去new
3. why?
4. 为了方便去使用，Java对这些基本数据类型提供了装箱类型
```java
public class Demo {
    public static void main(String[] args) {
        // 自动装箱
        // 相当于代码：Integer i = Integer.valueOf(1);
        Integer i = 1;

        // 自动拆箱
        // 相当于代码：int j = i.intValue();
        int j = i;

        System.out.println(i + j);
    }
}

// # 编译
// javac Demo.java
// # 查看字节码
// javap -c Demo

Compiled from "Demo.java"
public class Demo extends java.lang.Object{
public Demo();
  Code:
   0:   aload_0
   1:   invokespecial   #1; //Method java/lang/Object."<init>":()V
   4:   return

public static void main(java.lang.String[]);
  Code:
   0:   iconst_1
   1:   invokestatic    #2; //Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
   4:   astore_1
   5:   aload_1
   6:   invokevirtual   #3; //Method java/lang/Integer.intValue:()I
   9:   istore_2
   10:  getstatic       #4; //Field java/lang/System.out:Ljava/io/PrintStream;
   13:  aload_1
   14:  invokevirtual   #3; //Method java/lang/Integer.intValue:()I
   17:  iload_2
   18:  iadd
   19:  inv// okevirtual   #5; //Method java/io/PrintStream.println:(I)V
   22:  return

}
```

5. 思考
   1. Integer的装箱类型能否用==比较？
      1. Integer类型的valueof方法，-128~127区间采用缓存，这部分值是可以用==比较的，区间外的是重新创建的对象，用==比较的只是引用
   2. ==、equals的区别？
   3. 为何阿里规范中心POJO以及RPC的入参、返回值的类属性定义都要用用包装数据？
      1. 有时候，接收到的值可能事null类型，如果用的是基本类型，则会在拆箱的过程中发生NPE

# 泛型擦除
1. Java的泛型，其实是 `伪泛型` ，泛型仅仅存在于编码期间，供编译器进行类型检测，编译后，会被擦除。
2. 运行时，通过类型强制转换来实现的。
3. 泛型是JDK1.5之后引入
```java
// // 1. 源码
import java.util.*;

public class Demo {
    public static void main(String[] args) {
        List<String> a = new ArrayList<String>();
        // 并不能编译通过
        // a.add(111);
        a.add("test");
        String temp = a.get(0);
        System.out.println(temp);
    }
}

2. 通过jd-gui反编译后得到
import java.util.ArrayList;

public class Demo {
  public static void main(String[] paramArrayOfString) {
    ArrayList arrayList = new ArrayList();

    arrayList.add("test");
    // 通过类型强制转换来获取List中的对象，这里已经由编码期的泛型来保证了类型安全
    String str = (String)arrayList.get(0);
    System.out.println(str);
  }
}

// 3. 如何证明泛型在编译时被擦除，运行时并不需要这个
import java.util.*;

public class Demo {
    public static void main(String[] args) {
        List<Integer> a = new ArrayList<Integer>();
        List<String> b = new ArrayList<String>();

        // 并不能编译通过
        // a.addAll(b);

        Class c1 = a.getClass();  
        Class c2 = b.getClass();   
        // 打印:
        // class java.util.ArrayList
        // true
        // 运行时获取a和b的类型
        System.out.println(c1);  
        System.out.println(c1 == c2);  
    }
}
```

# 枚举
1. 枚举其实就是一个Java类，继承了 `java.lang.Enum` ，并且其本身是不允许被继承的
2. 用 `enum` 修饰的
```java
// 枚举源码
public enum Demo {
    ENUM_A,
    ENUM_B
}

// 2. 编译后，用javap查看字节码文件
// javac Demo.java
// javap -c Demo
Compiled from "Demo.java"
// 此处可以看出来枚举类，其实就是一个普普通通的Java类
// 首先，它是被final修饰的，所有不能被继承
// 其次，默认继承了java.lang.Enum，并包含泛型，泛型类型为其本身
public final class Demo extends java.lang.Enum<Demo> {
  // 有两个静态定义的枚举值
  public static final Demo ENUM_A;
  public static final Demo ENUM_B;
    
  public static Demo[] values();
    Code:
       0: getstatic     #1                  // Field $VALUES:[LDemo;
       3: invokevirtual #2                  // Method "[LDemo;".clone:()Ljava/lang/Object;
       6: checkcast     #3                  // class "[LDemo;"
       9: areturn

  public static Demo valueOf(java.lang.String);
    Code:
       0: ldc           #4                  // class Demo
       2: aload_0
       3: invokestatic  #5                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;

       6: checkcast     #4                  // class Demo
       9: areturn

  static {};
    Code:
       // new一个对象，类型事Demo
       0: new           #4                  // class Demo
       3: dup
       // 将常量压入栈
       4: ldc           #7                  // String ENUM_A
       // 定义变量0
       6: iconst_0
       // 调用构造器，注意入参是两个(上面入栈的两个)
       7: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      // 赋值
      10: putstatic     #9                  // Field ENUM_A:LDemo;
      13: new           #4                  // class Demo
      16: dup
      17: ldc           #10                 // String ENUM_B
      19: iconst_1
      20: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      23: putstatic     #11                 // Field ENUM_B:LDemo;
      26: iconst_2
      // 创建数组，类型是o
      27: anewarray     #4                  // class Demo
      // 压栈
      30: dup
      // 第0位
      31: iconst_0
      // 取值
      32: getstatic     #9                  // Field ENUM_A:LDemo;
      // 存入对应的数组元素中
      35: aastore
      36: dup
      37: iconst_1
      38: getstatic     #11                 // Field ENUM_B:LDemo;
      41: aastore
      42: putstatic     #1                  // Field $VALUES:[LDemo;
      45: return
}
```

# try-with-resources
1. 以前在写访问数据库、流编程，通常需要将资源关闭
2. 以前使用try-catch-finally的写法，在finally中将资源关闭
3. 这时候就要注意很多细节上的问题
   1. 资源是否打开，是否存在
   2. 关闭的过程中如何处理异常
   3. finally中不能写return
4. 自从JDK7之后，支持 `try-with-resources` 的写法，下面简单介绍一下两种写法的区别：
```java
// 1. try-catch-finally写法：
public static void copy(String src) {
    InputStream in = null;
    try {
        System.out.println(in);
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (in != null) {
            try {
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

// 2. try-with-resources写法：
public static void copy(String src) {
    try (InputStream in = new FileInputStream(src)) {
        System.out.println(in);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
5. 很明显，第二种写法较第一种，清晰、明了。
6. 下面进行源码分析：
7. 查看JDK7版本中的 `FileInputStream` 源码
```java
// 1. FileInputStream源码
public class FileInputStream extends InputStream { 
    // ...
    public void close() throws IOException {
        synchronized (closeLock) {
            if (closed) {
                return;
            }
            closed = true;
        }
        if (channel != null) {
            channel.close();
        }

        fd.closeAll(new Closeable() {
            public void close() throws IOException {
                close0();
            }
        });
    }
}   
// 2. FileInputStream父类InputStream源码
public abstract class InputStream implements Closeable {
    // ...
    public void close() throws IOException {}
}
// 3. InputStream实现的接口Closeable的源码
public interface Closeable extends AutoCloseable {
    public void close() throws IOException;
}
// 4. AutoCloseable的源码
/**
 * ...
 * @since 1.7
 */
public interface AutoCloseable {
    void close() throws Exception;
}

```
8. 可见最终是实现了一个JDK7才提供的接口 `AutoCloseable` 
9. 我们再去查看一下`try-with-resources` 写法的字节码文件
```java
// javap -c Demo
Compiled from "Demo.java"
public class Demo {
  public Demo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void copy(java.lang.String);
    Code:
       0: new           #2                  // class java/io/FileInputStream
       3: dup
       4: aload_0
       5: invokespecial #3                  // Method java/io/FileInputStream."<init>":(Ljava/lang/String;)V
       8: astore_1
       9: aconst_null
      10: astore_2
      11: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
      14: aload_1
      15: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
      18: aload_1
      19: ifnull        89
      22: aload_2
      23: ifnull        42
      26: aload_1
      27: invokevirtual #6                  // Method java/io/InputStream.close:()V
      30: goto          89
      33: astore_3
      34: aload_2
      35: aload_3
      36: invokevirtual #8                  // Method java/lang/Throwable.addSuppressed:(Ljava/lang/Throwable;)V
      39: goto          89
      42: aload_1
      43: invokevirtual #6                  // Method java/io/InputStream.close:()V
      46: goto          89
      49: astore_3
      50: aload_3
      51: astore_2
      52: aload_3
      53: athrow
      54: astore        4
      56: aload_1
      57: ifnull        86
      60: aload_2
      61: ifnull        82
      64: aload_1
      65: invokevirtual #6                  // Method java/io/InputStream.close:()V
      68: goto          86
      71: astore        5
      73: aload_2
      74: aload         5
      76: invokevirtual #8                  // Method java/lang/Throwable.addSuppressed:(Ljava/lang/Throwable;)V
      79: goto          86
      82: aload_1
      83: invokevirtual #6                  // Method java/io/InputStream.close:()V
      86: aload         4
      88: athrow
      89: goto          97
      92: astore_1
      93: aload_1
      94: invokevirtual #10                 // Method java/io/IOException.printStackTrace:()V
      97: return
    Exception table:
       from    to  target type
          26    30    33   Class java/lang/Throwable
          11    18    49   Class java/lang/Throwable
          11    18    54   any
          64    68    71   Class java/lang/Throwable
          49    56    54   any
           0    89    92   Class java/io/IOException
}
```

10. 思考
   1. 问题一
      1. 在第一种方案中，如果try块中抛出了异常，该异常时可以被正常抛出记录的
      1. 如果在finally中产生异常，然后抛出，那该异常也可以被正常记录
      1. 如果try中先出现异常，finally中也会被正常执行，但是如果finally中也出现了异常并抛出，try中的异常还能被捕获么？
11. 关于异常
	1. jdk7之前，如果不做任何处理，try中的异常就会被忽略掉，无法捕获
	2. jdk7之后，引入了“可被抑制”的异常，finally中产生的异常，可以获取到try中的异常
	3. jdk7之前，需要实现自己的异常类
	4. jdk7之后，已经对Throwable类进行了修改以支持这种情况。在java7中为Throwable类增加addSuppressed方法。当一个异常被抛出的时候，可能有其他异常因为该异常而被抑制住，从而无法正常抛出。这时可以通过addSuppressed方法把这些被抑制的方法记录下来。被抑制的异常会出现在抛出的异常的堆栈信息中，也可以通过getSuppressed方法来获取这些异常。这样做的好处是不会丢失任何异常，方便开发人员进行调试。
	5. 可以查看一下上面例子反编译后的源码：
	```java
	import java.io.*;

	public class Demo {
	   public Demo() {
	   }
	   public static void copy(String s) {
		   FileInputStream fileinputstream;
		   Throwable throwable;
		   fileinputstream = new FileInputStream(s);
		   throwable = null;
		   try {
			   System.out.println(fileinputstream);
		   }
		   catch(Throwable throwable2) {
			   throwable = throwable2;
			   throw throwable2;
		   }
		   if(fileinputstream != null)
			   if(throwable != null)
				   try {
					   fileinputstream.close();
				   } catch(Throwable throwable1) {
					   // 这里就是将catch中的异常，放在可能阻断异常抛出的异常中，然后统一在后面抛出异常，这样throwable1就在throwable的异常栈中了
					   throwable.addSuppressed(throwable1);
				   }
			   else
				   fileinputstream.close();
		   // 这一块，源码是finally块，由于反编译工具的问题，导致看到break语句
		   // 其实Java中的break也是支持跳出的指定label的
		   break MISSING_BLOCK_LABEL_97;
		   Exception exception;
		   exception;
		   if(fileinputstream != null)
			   if(throwable != null)
				   try {
					   fileinputstream.close();
				   } catch(Throwable throwable3) {
					   throwable.addSuppressed(throwable3);
				   }
			   else
				   fileinputstream.close();
		   throw exception;
		   IOException ioexception;
		   ioexception;
		   ioexception.printStackTrace();
	   }
	}
	```
