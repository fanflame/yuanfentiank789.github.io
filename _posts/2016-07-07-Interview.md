---
layout: post
title:  "Java面试题库"
date:   2016-07-07 1:05:00
catalog:  true
tags:
    - 面试
  
       

---

# Java 面试题库

## 一.Java基础

### 运算符

#### 1 &与&&
&和&&都可以用作逻辑与的运算符，表示逻辑与（and），当运算符两边的表达式的结果都为true时，整个运算结果才为true，否则，只要有一方为false，则结果为false。
&&还具有短路的功能，即如果第一个表达式为false，则不再计算第二个表达式，例如，对于if(str != null && !str.equals(“”))表达式，当str为null时，后面的表达式不会执行，所以不会出现NullPointerException如果将&&改为&，则会抛出NullPointerException异常。If(x==33 & ++y>0) y会增长，If(x==33 && ++y>0)不会增长
&还可以用作位运算符，当&操作符两边的表达式不是boolean类型时，&表示按位与操作，我们通常使用0x0f来与一个整数进行&运算，来获取该整数的最低4个bit位，例如，0x31 & 0x0f的结果为0x01。

#### 2 equals和==

== 物理相等，equals逻辑相等。

==操作符专门用来比较两个变量的值是否相等，也就是用于比较变量所对应的内存中所存储的数值是否相同，要比较两个基本类型的数据或两个引用变量是否相等，只能用==操作符。
　　如果一个变量指向的数据是对象类型的，那么，这时候涉及了两块内存，对象本身占用一块内存（堆内存），变量也占用一块内存，例如Objet obj = new Object();变量obj是一个内存，new Object()是另一个内存，此时，变量obj所对应的内存中存储的数值就是对象占用的那块内存的首地址。对于指向对象类型的变量，如果要比较两个变量是否指向同一个对象，即要看这两个变量所对应的内存中的数值是否相等，这时候就需要用==操作符进行比较。
　　
#### 3、Math.round(11.5)等於多少? Math.round(-11.5)等於多少?

Math类中提供了三个与取整有关的方法：ceil、floor、round，这些方法的作用与它们的英文名称的含义相对应，例如，**ceil的英文意义是天花板**，该方法就表示向上取整，所以，Math.ceil(11.3)的结果为12,Math.ceil(-11.3)的结果是-11；**floor的英文意义是地板**，该方法就表示向下取整，所以，Math.floor(11.6)的结果为11,Math.floor(-11.6)的结果是-12；最难掌握的是round方法，它表示“四舍五入”，算法为Math.floor(x+0.5)，即将原来的数字加上0.5后再向下取整，所以，Math.round(11.5)的结果为12，Math.round(-11.5)的结果为-11。
　　
　　
#### 4 请说出作用域public，private，protected，以及不写时的区别

这四个作用域的可见范围如下表所示。
说明：如果在修饰的元素上面没有写任何访问修饰符，则表示friendly。
　　
![image](/images/interview/Snip20160709_18.png)

备注：只要记住了有4种访问权限，4个访问范围，然后将全选和范围在水平和垂直方向上分别按排从小到大或从大到小的顺序排列，就很容易画出上面的图了。
　
#### 5、面向对象的特征有哪些方面

　　计算机软件系统是现实生活中的业务在计算机中的映射，而现实生活中的业务其实就是一个个对象协作的过程。面向对象编程就是按现实业务一样的方式将程序代码按一个个对象进行组织和编写，让计算机系统能够识别和理解用对象方式组织和编写的程序代码，这样就可以把现实生活中的业务对象映射到计算机系统中。
面向对象的编程语言有封装、继承 、抽象、多态等4个主要的特征。
　　
**封装**：
　　
　　封装是保证软件部件具有优良的模块性的基础，封装的目标就是要实现软件部件的“高内聚、低耦合”，防止程序相互依赖性而带来的变动影响。在面向对象的编程语言中，对象是封装的最基本单位，面向对象的封装比传统语言的封装更为清晰、更为有力。面向对象的封装就是把描述一个对象的属性和行为的代码封装在一个“模块”中，也就是一个类中，属性用变量定义，行为用方法进行定义，方法可以直接访问同一个对象中的属性。通常情况下，只要记住让变量和访问这个变量的方法放在一起，将一个类中的成员变量全部定义成私有的，只有这个类自己的方法才可以访问到这些成员变量，这就基本上实现对象的封装，就很容易找出要分配到这个类上的方法了，就基本上算是会面向对象的编程了。把握一个原则：把对同一事物进行操作的方法和相关的方法放在同一个类中，把方法和它操作的数据放在同一个类中。
　　例如，人要在黑板上画圆，这一共涉及三个对象：人、黑板、圆，画圆的方法要分配给哪个对象呢？由于画圆需要使用到圆心和半径，圆心和半径显然是圆的属性，如果将它们在类中定义成了私有的成员变量，那么，画圆的方法必须分配给圆，它才能访问到圆心和半径这两个属性，人以后只是调用圆的画圆方法、表示给圆发给消息而已，画圆这个方法不应该分配在人这个对象上，这就是面向对象的封装性，即将对象封装成一个高度自治和相对封闭的个体，对象状态（属性）由这个对象自己的行为（方法）来读取和改变。一个更便于理解的例子就是，司机将火车刹住了，刹车的动作是分配给司机，还是分配给火车，显然，应该分配给火车，因为司机自身是不可能有那么大的力气将一个火车给停下来的，只有火车自己才能完成这一动作，火车需要调用内部的离合器和刹车片等多个器件协作才能完成刹车这个动作，司机刹车的过程只是给火车发了一个消息，通知火车要执行刹车动作而已。
　　
**抽象**：

　　抽象就是找出一些事物的相似和共性之处，然后将这些事物归为一个类，这个类只考虑这些事物的相似和共性之处，并且会忽略与当前主题和目标无关的那些方面，将注意力集中在与当前目标有关的方面。例如，看到一只蚂蚁和大象，你能够想象出它们的相同之处，那就是抽象。抽象包括行为抽象和状态抽象两个方面。例如，定义一个Person类，如下：
　　class Person
　　{
　　		String name;
　　		int age;
　　}
　　人本来是很复杂的事物，有很多方面，但因为当前系统只需要了解人的姓名和年龄，所以上面定义的类中只包含姓名和年龄这两个属性，这就是一种抽像，使用抽象可以避免考虑一些与目标无关的细节。我对抽象的理解就是不要用显微镜去看一个事物的所有方面，这样涉及的内容就太多了，而是要善于划分问题的边界，当前系统需要什么，就只考虑什么。
　　
**继承**：

　　在定义和实现一个类的时候，可以在一个已经存在的类的基础之上来进行，把这个已经存在的类所定义的内容作为自己的内容，并可以加入若干新的内容，或修改原来的方法使之更适合特殊的需要，这就是继承。继承是子类自动共享父类数据和方法的机制，这是类之间的一种关系，提高了软件的可重用性和可扩展性。
　　
**多态**：

　　多态是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。因为在程序运行时才确定具体的类，这样，不用修改源程序代码，就可以让引用变量绑定到各种不同的类实现上，从而导致该引用调用的具体方法随之改变，即不修改程序代码就可以改变程序运行时所绑定的具体代码，让程序可以选择多个运行状态，这就是多态性。多态性增强了软件的灵活性和扩展性。
　　
#### 6、super.getClass()方法调用

下面程序的输出结果是多少？　　

    import java.util.Date;  
    public  class Test extends Date{  
    public static void main(String[] args) {  
        new Test().test();  
    }  
    public void test(){  
        System.out.println(super.getClass().getName());  
    }  
    }  
 
因为super并没有代表超类的一个引用的能力，只是代表调用父类的方法而已。所以，在子类的方法中，不能这样用System.out.println(super);也不能使用super.super.mathod();
 
事实上，super.getClass()是表示调用父类的方法。getClass方法来自Object类，它返回对象在运行时的类型。因为在运行时的对象类型是Test，所以this.getClass()和super.getClass()都是返回Test。
　　 

### String

#### 1 String的值传递，写出如下代码运行结果

    public static void main(String[] args) {

        String str = new String("adcdefg");
        char[] chars = {'s', 'd', 'f'};
        process(str, chars);
        System.out.println(str);
        System.out.println(chars);

    }

    private static void process(String str, char[] chars) {
        str = "fsdf";
        chars[0] = 'a';
    }

输结果应为：

    adcdefg
    adf
    
    
String虽然不是基本类型，但也用值传递，如果需要改变string的值，可改用StringBuilder代替。

[参考http://freej.blog.51cto.com/235241/168676](http://freej.blog.51cto.com/235241/168676)

#### Java面试中关于String的问题总结

参考[http://cymoft.blog.51cto.com/324099/473220/](http://cymoft.blog.51cto.com/324099/473220/)

最后测试代码如下：

        String s1 = "abc";
        String s2 = "abc";
        String s3 = new String("abc");//创建两个String对象，堆中创建一个String对象指向String常量池中的abc(如果存在就不在新建）
        String s4 = s1.intern();//从String常量池寻找一个String，和s1 equals返回true
        String s5 = s3.intern();//从String常量池寻找
        System.out.println("s1==s2:" + (s1 == s2));
        System.out.println("s1==s3:" + (s1 == s3));
        System.out.println("s1==s4:" + (s1 == s4));
        System.out.println("s1==s5:" + (s1 == s5));
        
输出结果：

    s1==s2:true
    s1==s3:false
    s1==s4:true
    s1==s5:true
    
有这样一个面试题：

        String a = "a";
        String a1 = new String("a");
        String a2 = a1.trim() + "";
        String a3 = "a" + "";
        String a4 = "a".trim() + "";
        System.out.println(a == a1);
        System.out.println(a.intern() == a1.intern());
        System.out.println(a2 == a1);
        System.out.println(a3 == a);
        System.out.println(a4 == a);
        
输出结果：

    false
    true
    false
    true
    false
    
为什么结果是这样呢？这需要从String 本身说起。
Java在运行时会维护一个字符串常量池String Pool; 在String a="a"; 首先检查字符串常量池中是否有"a"，如果有则直接返回，否则在常量池中创建一个新的；
     String a1=new String("a"); 使用new 关键字创建的对象一定在堆栈中，同样也会维护字符串常量池，因为字符串常量池中已经存在了，则不会添加新的。在JAVA中＝＝永远都是比较两个内存地址是否相同，这样因为a和a1不是同一个对象则已定返回false;
   intern()方法时返回字符串常量池中的对象，因为常量池中只存在一个“a” 则两者已定相等；
   a2.trim()+"" 则是在堆栈中创建了一个新对象，同时维护常量池；则该表达式返回false 但是字符串常量的拼接仅仅维护常量池不会在堆栈中创建新对象则"a"+""还是常量池中的“a”;

#### 3 String 和StringBuffer的区别
　　JAVA平台提供了两个类：String和StringBuffer，它们可以储存和操作字符串，即包含多个字符的字符数据。String类表示内容不可改变的字符串。而StringBuffer类表示内容可以被修改的字符串。当你知道字符数据要改变的时候你就可以使用StringBuffer。典型地，你可以使用StringBuffers来动态构造字符数据。另外，String实现了equals方法，new String(“abc”).equals(new String(“abc”)的结果为true,而StringBuffer没有实现equals方法，所以，new StringBuffer(“abc”).equals(new StringBuffer(“abc”)的结果为false。
接着要举一个具体的例子来说明，我们要把1到100的所有数字拼起来，组成一个串。

    StringBuffer sbf = new StringBuffer();    
    for(int i=0;i<100;i++)  
    {  
        sbf.append(i);  
    }

上面的代码效率很高，因为只创建了一个StringBuffer对象，而下面的代码效率很低，因为创建了101个对象。

    String str = new String();    
    for(int i=0;i<100;i++)  
    {  
        str = str + i;  
    }
    
在讲两者区别时，应把循环的次数搞成10000，然后用endTime-beginTime来比较两者执行的时间差异，最后还要讲讲StringBuilder与StringBuffer的区别。
String覆盖了equals方法和hashCode方法，而StringBuffer没有覆盖equals方法和hashCode方法，所以，将StringBuffer对象存储进Java集合类中时会出现问题。

#### 4、StringBuffer与StringBuilder的区别
  StringBuffer和StringBuilder类都表示内容可以被修改的字符串，StringBuilder是线程不安全的，运行效率高，如果一个字符串变量是在方法里面定义，这种情况只可能有一个线程访问它，不存在不安全的因素了，则用StringBuilder。如果要在类里面定义成员变量，并且这个类的实例对象会在多线程环境下使用，那么最好用StringBuffer。

### Class

1  Class.forName和ClassLoader.loadClass的比较

Class的装载分了三个阶段，loading，linking和initializing，分别定义在The Java Language Specification的12.2，12.3和12.4。
Class.forName(className)实际上是调用Class.forName(className, true, this.getClass().getClassLoader())。注意第二个参数，是指Class被loading后是不是必须被初始化。
ClassLoader.loadClass(className)实际上调用的是ClassLoader.loadClass(name, false)，第二个参数指出Class是否被link。
区别就出来了。Class.forName(className)装载的class已经被初始化，而ClassLoader.loadClass(className)装载的class还没有被link。
一般情况下，这两个方法效果一样，都能装载Class。但如果程序依赖于Class是否被初始化，就必须用Class.forName(name)了。
例如，在JDBC编程中，常看到这样的用法，Class.forName("com.mysql.jdbc.Driver")，如果换成了getClass().getClassLoader().loadClass("com.mysql.jdbc.Driver")，就不行。
为什么呢？打开com.mysql.jdbc.Driver的源代码看看，

    // Register ourselves with the DriverManager
    //
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
        throw new RuntimeException("Can't register driver!");
        }
    }
    
原来，Driver在static块中会注册自己到java.sql.DriverManager。而static块就是在Class的初始化中被执行。所以这个地方就只能用Class.forName(className)。

## Java内存

### Java内存管理

[http://www.cnblogs.com/gw811/archive/2012/10/18/2730117.html](http://www.cnblogs.com/gw811/archive/2012/10/18/2730117.html)

### Java内存回收
[http://www.cnblogs.com/gw811/archive/2012/10/19/2730258.html](http://www.cnblogs.com/gw811/archive/2012/10/19/2730258.html)

[http://www.cnblogs.com/sunniest/p/4575144.html](http://www.cnblogs.com/sunniest/p/4575144.html)

# Android

## Framework

### 1 Dalvik VM (DVM) 与Java VM (JVM)之间有哪些区别?

参考

[http://www.cnblogs.com/yejiurui/p/4859892.html](http://www.cnblogs.com/yejiurui/p/4859892.html) 

[https://www.zhihu.com/question/20207106](https://www.zhihu.com/question/20207106)