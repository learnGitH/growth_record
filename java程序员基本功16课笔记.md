一、数组与内存控制

1、声明一个数组的过程中，是如何分配内存的？

2、Java数组的初始化一共有哪几种方式？

3、基本数据类型数组和引用类型数组之间，在初始化时的内存分配机制有什么区别？



1、数组的初始化方式

java的数组是静态的，因为一旦初始化之后，数组的长度是不可以改变的。

初始化：数组对象的元素分配内存空间，并为每个元素赋初始值。

静态初始化：初始化时由程序员显式指定每个数组元素的初始值，由系统决定数组长度。

动态初始化：初始化时程序员只指定数组长度，由系统为数组元素分配初始值。

2、初始化理解

对象本身需要进行初始化，对象的引用变量是不需要进行初始化的

基本数据类型数组的对象存储的是该元素的值，引用类型数组对象存储的是另一个对象的地址



二、对象与内存控制

1、实例变量属于Java对象

2、类变量属于类本身

3、实例变量的初始化细节

4、类变量的初始化细节

5、子类构造器调用父类构造器

6、避免在构造器中访问子类的实例变量

7、避免在构造器中调用被子类重写的方法

8、Java继承对成员变量和方法的区别

9、父、子实例的实例变量的内存分配机制

10、父、子类的类变量的内存分配

11、final修饰符的作用

12、系统对哪些final变量执行“宏替换”

13、final方法注意点

14、使用final修饰被匿名、局部内部类访问的局部变量



2.1 实例变量和类变量

java程序主要有两种类型的变量：成员变量和局部变量

局部变量：形参、方法内的局部变量、代码块的局部变量（作用时间很短暂，他们都被存储在方法的栈内存中）。

成员变量：在类上顶定义的变量（如果通过static修饰，则为静态变量或类变量。如果没有static修饰则为实例变量，static可以修饰成员变量、方法、内部类、初始化块、内部枚举类）。

实例变量的初始化时机：**定义实例变量时指定的初始值、初始化块中为实例变量指定初始值、构造器中为实例变量指定的初始值，三者的作用完全类似**，都用于对实例变量指定初始值。经过编译器处理之后，它们对应的赋值语句都被合并到构造器中。在合并过程中，定义变量语句转换得到的赋值语句、初始化块里的语句转换得到的赋值语句，总是位于构造器的所有语句之前；合并后，两种赋值语句的顺序保持它们在源代码中的顺序。

类变量的初始化时机：定义类变量时指定初始值，静态初始化块中对类变量指定初始值。

 2.2 父类构造器

super/this

不要在父类的构造器中调用被子类重写方法

如果父类构造器调用了被子类重写的方法，且通过子类构造器来创建子类对象，调用（不管是显式还是隐式）了整个父类构造器，就会导致子类重写方法在子类构造器的所有代码之前被执行，从而导致子类的重写方法访问不到子类的实例变量的情形。

2.3 父子实例的内存控制

当通过引用变量调用方法时，方法的行为总是表现出它们实际类型的行为；但如果通过这些变量来访问它们所指对象的实例变量，这些实例变量的值总是表现出声明这些变量所用类型的行为。由此可见，Java继承在处理成员变量和方法时是有区别的。

如果子类没有重写父类的方法，那么编译器就会将父类的方法直接迁移到子类中。

如果子类重写父类的方法，就意味着子类里定义的方法彻底覆盖了父类里的同名方法，系统不会再将父类的方法迁移到子类中。

因为继承成员变量和继承方法之间存在这样的差别，所以对于一个引用类型的变量而言，当通过该变量访问所引用的对象的实例变量时，该实例变量的值取决于声明该变量时类型，当通过该变量来调用它所引用的对象的方法时，该方法行为取决于它所实际引用的对象的类型。

2.4 final修饰符

final可以修饰变量，被final修饰的变量被赋初始值之后，不能对它重新赋值。

被final修饰的实例变量赋值的时机（本质都是在构造器中赋初始值）：

```
定义final实例变量时指定初始值
在非静态初始化块中为final实例变量指定初始值
在构造器中为final实例变量指定初始值
```

被final修饰的类变量赋值的时机（本质都是在静态初始化块完成的）:

```
定义final类变量时指定初始值
在静态初始化中为final类变量指定初始值
```

如果被final修饰的变量定义时指定初始值，那么在编译期间就会对使用该变量进行“宏替换“”,如果是通过代码块或者构造器则不会。

```
final static String str = "Java";
System.out.println(str + str == "JavaJava") //在编译期就会编程（"Java"+"Java" == "JavaJava"）
```

final可以修饰方法，被final修饰的方法不能被重写。

final可以修饰类，被final修饰的类不能派生子类。



三、常见Java集合的实现细节

Set/Map

HashSet/HashMap

TreeSet/TreeMap

3.1 Set和Map的关系

Set集合继承体系图：

<img src="C:\Users\hp\AppData\Roaming\Typora\typora-user-images\image-20230127201804862.png" alt="image-20230127201804862" style="zoom: 50%;" />

Map集合继承体系图：

<img src="C:\Users\hp\AppData\Roaming\Typora\typora-user-images\image-20230127201841218.png" alt="image-20230127201841218" style="zoom:50%;" />

set本质也是map,其实就是对应于map中的key集合

TreeMap和TreeSet

TreeMap本质就是一棵红黑树，TreeMap的每个Entry就是该红黑树的一个节点。TreeSet本质也是通过TreeMap实现的。

3.2 Map和List

3.3 ArrayList和LinkedList

List的三个主要的实现类：ArrayList、Vector、LinkedList。

Vector子类：Stack(Stack只是在Vector增加了5个方法)

Deque:ArrayDeque(双端队列，既有队列的功能也有栈的功能)

Vector是线程安全的，ArrayList是线程不安全的，如果想要线程安全，可以通过Java中Collections提供的工具类的synchronizedList方法即可将一个普通ArrayList包装成线程安全的ArrayList。

ArrayList和LinkedList的实现差异：

List代表一种线性表的数据结构，ArrayList则是一种顺序存储的线性表。ArrayList底层采用数组来保存每个集合元素，LinkedList则是一种链式存储的线性表。其本质就是一个双向链表，但它不仅实现了List接口，还实现了Deque接口。也就是说LinkedList即可以当成双向链表使用，也可以当成队列使用，还可以当成栈来使用（Deque代表栓动队列，既有队列的特征，也具有栈的特征）。

当程序把LinkedList当成双端队列、栈使用，调用addFirst(E e)、addLast(E e)、getFirst(E e)、getLast(E e)、offer(E e)、offerFirtst()、offerLast()等方法来操作集合元素时LinkedList可以快速定位需要操作的元素，因此LinkedList总是具有较好的性能表现。

3.4 Iterator迭代器

```
for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
	//获得对应的元素
    String ele = it.next();
}

public boolean hasNext() {
   return cursor != size;
}
```

四、Java的内存回收

Java引用的功能和意义

Java引用与内存回收之间的关系

Java对象在内存中的不同状态

软引用的作用和使用软引用的注意点

弱引用的作用和使用弱引用的注意点

虚引用和使用虚引用的注意点

Java内存泄漏的原因

Java内存泄漏和C++内存泄漏的差别

Java垃圾回收机制的基本算法

堆内存的分代回收

Young代、Old代和Permanent代各自存储的对象

Young代、Old代和Permanent代的特定及适用的回收算法

常见垃圾回收机制对堆内存的回收细节

设计开发中内存管理的小技巧

问题：

（1）JVM在何时决定回收一个Java对象所占据的内存？

（2）JVM会不会漏掉回收某些Java对象，使之造成内存泄漏？

（3）JVM回收Java对象所占用内存的实现细节？

（4）JVM能否对不同Java对象占用的内存区分对待、回收？

（5）常见垃圾回收机制的实现细节是怎样的？

4.1 Java引用的种类

当一个对象在堆内存中运行时，根据它在对应有向图中的状态，可以把它所处的状态分成如下3种：

可达状态：当一个对象被创建后，有一个以上的引用变量引用它。

可恢复状态：没有引用变量引用，如果此时系统调用finalize方法重新让一个以上引用变量引用该对象，则这个对象会再次变为可达状态；否则该对象将进入不可达状态。

不可达状态：没有引用指向该对象，并且已调用finalize方法依然没有使该对象变成可达。

<img src="C:\Users\hp\AppData\Roaming\Typora\typora-user-images\image-20230129162101045.png" alt="image-20230129162101045" style="zoom:50%;" />

​	一个对象可以被一个方法局部变量所引用，也可以被其他类的类变量引用，或者被其他对象的实例变量所引用。当某个对象被其他类的类变量引用时，只有该类被销毁后，该对象才会进入可恢复状态。当某个对象被其他对象的实例变量引用时，只有当引用该对象的对象被销毁或变成不可达状态后，该对象才会进入不可达状态。

java语言对象的4类引用：

强引用：这是Java程序中最常见的引用方式，程序创建一个对象，并把这个对象赋给一个引用变量，这个引用变量就是强引用。由于JVM肯定不会回收强引用所引用的Java对象，因此强引用时造成Java内存泄漏的主要原因之一。

软引用（SoftReference）：当系统内存充足的时候强引用和软应用没有什么本质的区别，只有当内存资源紧张的时候，软引用所引用的Java对象将会被垃圾回收，软引用通过SoftReference来创建

```
public class SoftReferenceTest {

    public static void main(String[] args) {
    	//软引用
        SoftReference<Person>[] people = new SoftReference[10000];
        //强引用
        //Person[] person = new Person[10000];
        for (int i = 0; i < people.length; i++) {
            people[i] = new SoftReference<Person>(new Person("名字" + i, (i + 1) * 4 % 100));
        }
        System.out.println(people[2].get());
        System.out.println(people[4].get());
        //通知系统进行垃圾回收
        System.gc();
        System.runFinalization();
        //垃圾回收机制运行之后，SoftReference数组里的元素保持不变
        System.out.println(people[2].get());
        System.out.println(people[4].get());
    }

}
```

弱引用(WeekReference)：弱引用与软引用有点相似，区别在于弱引用所引用对象的生存期更短。弱引用通过WeekReference类实现，弱引用和软引用很像，但弱引用的引用级别更低。对于只有弱引用的对象而言，当系统垃圾回收机制运行时，不管系统内存是否足够，总会回收该对象所占用的内存。当然，并不是说当一个对象只有弱引用时，它就会立即被回收-正如那些失去引用的对象一样，必须等到系统垃圾回收机制运行时才会被回收。

```
public class WeekReferenceTest {

    public static void main(String[] args) throws Exception{
        //创建一个字符串对象
        String str = new String("疯狂Java讲义");
        //创建一个弱引用，让次弱引用引用到"疯狂Java讲义"字符串
        WeakReference<String> wr = new WeakReference<String>(str);
        //切断str引用和"疯狂Java讲义"字符串之间的引用
        str = null;
        System.out.println(wr.get());
        //强制垃圾回收
        System.gc();
        System.runFinalization();
        //再次取出弱引用所引用的对象
        System.out.println(wr.get());
    }

}
```

虚引用（PhantomReference）：虚引用通过PhantomRefrence类实现，它完全类似于没有引用。虚引用对对像本身没有太大影响，对象甚至感觉不到虚引用的存在。如果一个对象只有一个虚引用那它和没有引用的效果大致相同。虚引用主要用于跟踪对象被垃圾回收的状态，虚引用不能单独使用，虚引用必须和引用队列（ReferenceQueue)联合使用。

```
public class PhantomReferenceTest {

    public static void main(String[] args) {
        //创建一个字符串对象
        String str = new String("疯狂Java讲义");
        //创建一个引用队列
        ReferenceQueue<String> rq = new ReferenceQueue<String>();
        //创建一个虚引用，让此虚引用引用到“疯狂Java讲义”字符串
        PhantomReference<String> pr = new PhantomReference<String>(str,rq);
        //切断str引用和“疯狂Java讲义”字符串之间的引用
        str = null;
        //试图取出虚引用所引用的对象
        //程序并不能通过虚引用访问被引用的对象，所以此输出为null
        System.out.println(pr.get());
        //强制垃圾回收
        System.gc();
        System.runFinalization();
        System.out.println(rq.poll() == pr);
    }

}
```

4.2Java的内存泄漏

​	程序运行过程中会不断地分配内存空间，那些不再使用的内存空间应该即时回收它们，从而保证系统可以再次使用这些内存，如果无用内存没有被回收回来，那就是内存泄漏。

<img src="C:\Users\hp\AppData\Roaming\Typora\typora-user-images\image-20230130112515419.png" alt="image-20230130112515419" style="zoom:50%;" />

4.3 垃圾回收机制

垃圾回收机制主要完成下面两件事情：

跟踪并监控每个Java对象，当某个对象处于不可达状态时，回收该对象所占用的内存；

清理内存分配、回收过程中产出的内存碎片。‘

复制算法：将内存划分为两部分，每次只只使用一部分，回收时将存活下来的对象复制到空的一部分

标记算法：标记-清除-压缩

串行回收器

并行回收器

并行压缩回收器

并发标识-清理（Mark-Sweep）回收器CMS

4.4 内存管理的小技巧

（1）尽量使用直接量，例如字符串或者Byte、Integer等，不要用new，而是直接String a = "test"

（2）使用StringBuilder和StringBuffer进行字符串连接

（3）尽早释放无用对象的引用：obj = null;

（4）尽量少用静态变量

（5）避免在经常调用的方法、循环中创建Java对象

（6）缓存经常使用的对象

（7）缓存经常使用的对象（数据库连接池），可以使用HashMap进行缓存；或者使用某些开源的缓存项目；

（8）尽量不要使用finalize方法

（9）考虑使用SoftReference

五、表达式中的陷阱

Java字符串的特点；

String、StringBuilder、StringBuffer；

表达式类型自动提示的陷阱；

复合赋值运算符隐含的类型转换；

输入法导致的陷阱；

必须使用合法的注释字符；

慎用字符的Unicode转义形式；

泛型中原始类型变量的赋值；

原始类型带来的擦除；

Java不支持泛型数组；

正则表达式中点号（.）匹配任意字符；

不要调用线程对象的run方法；

静态同步方法的同步监视器是类；

多线程执行环境的线程安全问题。

5.1关于字符串的陷阱

Java程序中创建对象的常见方式有如下4种：

（1）通过new调用构造器创建Java对象。

（2）通过Class对象的newInstance()方法调用构造器创建Java对象。

（3）通过Java的反序列化机制从IO流中恢复Java对象。

（4）通过Java对象提供的clone()方法复制一个新的Java对象。

除此之外，对于字符串以及Byte、Short、Integer、Long、Character、Float、Double和Bolean这些基本类型的包装类，Java还允许以直接量的方式来创建Java对象，例如如下语句：

String str = "abc";

Integer in = 5;

除此之外，也可通过简单的算法表达式、连接运算来创建Java对象，例如如下语句：

String str2 = "abc" + "xyz";

Long price = 23 +12;

不可变的字符串：一旦创建该对象就不可变

字符串比较：==、equals、compareTo

5.2 表达式类型的陷阱

表示式中的会像高类型的靠拢，注意类型转换问题

5.3 输入法导致的陷阱

如果在编译Java程序时编译提示形如“非法字符：\xxxxx”的错误提示，那么就可断定该Java程序包含“全角字符”，逐个删除它们即可。

5.4 注释的字符必须合法

5.5 转义字符的陷阱

Java提供3种方式来表示字符:

直接使用单引号括起来的字符值，如'a'

使用转义字符，如'\n'；

使用Unicode转义字符，如’\u0062‘

慎用Unicode转义字符，会引起一些编译错误的问题

5.6 泛型可能引起的错误

5.7 正则表达式的陷阱

5.8 多线程的陷阱

创建线程的方式：

（1）继承Thread类来创建线程类，重写run()方法作为多线程执行体；

（2）实现Runnable接口来创建多线程类，重写run()方法作为线程执行体

（3）实现Callable接口来创建线程类，重写call()方法作为线程执行体

不要认为所有放在静态初始化块中的代码就一定是类初始化操作，静态初始化块中启动的新线程的run()方法代码只是新线程的线程执行体，并不是类初始化操作。类似地，不要认为所有放在非静态初始化块中的代码一定是对象初始化操作，非静态初始化块中启动的新线程run()方法代码只是新线程的线程执行体，并不是对象初始化操作。

六、流程控制的陷阱

switch语句中的default语句；

switch语句中break语句的作用;

switch语句允许的表达式；

流程控制中的标签；

if语句else的隐含条件；

空语句导致的隐藏错误；

尽量不要省略循环体的花括号；

分号导致的空语句；

尽量避免改变循环计数器的值。

6.1 switch语句陷阱

default:只有前面不执行时才会执行。

break的重要性:每条语句后面都有一条break语句，这个break语句极其重要，用于终止当前分支的执行体。如果没有则不管条件是否匹配都会继续执行后面的语句，直到遇到break。

switch表达式：

byte:字节整型

short:短整型

int:整型

char:字符型

enum:枚举型

结论：beak的重要性，default的执行时机，switch接受的表达式

6.2 标签引起的陷阱

6.3 if语句的陷阱

注意分号问题，注意花括号问题

6.4 循环体的花括号

省略循环体花括号产生的一些问题

6.5 for循环的陷阱

6.6 foreach循环的循环计数器

七、面向对象的陷阱

instanceof运算符的陷阱；

构造器不能使用void声明返回值；

构造器只是初始化对象；

避免无限递归的构造器；

区分重载的方法；

private方法不能被重写；

包访问权限的方法也可能无法重写；

非静态内部类的构造器必须传入外部类的实例；

非静态内部类不能拥有静态成员；

static关键字的作用；

静态内部类不能访问外部类的非静态成员；

native方法跨平台时可能有问题；

7.1 instanceof运算符的陷阱

instanceof运算符前面操作树的编译时类型必须是如下3种情况：

（1）要么与后面的类相同；

（2）要么是后面类的父类；

（3）要么是后面类型的子类；

String str = null;	str instanceof String  = false

7.2 构造器的陷阱

​	构造器是Java每个类都会提供的一个“特殊方法”。构造器负责对Java对象执行初始化操作，不管是定义实例变量时指定的初始值，还是载非特殊静态初始化块中所做的操作，实际都会被提取到构造器中被执行。

以下两种方式创建的Java对象无需使用构造器：

（1）使用反序列化的方式恢复Java对象；

（2）使用clone方法复制Java对象。

通过反序列化恢复出来的对象和原来的对象具有完全相同的实例变量值，但系统中将会产生两个不同的对象。

通过clone()方法复制出来的对象和原来对象具有完全相同的实例变量值，但系统中将会产生两个不同的对象。

7.3 持有当前类的实例

7.4 到底调用哪个重载的方法

7.5 方法重写的陷阱

7.6 非静态内部类的陷阱

​		非静态内部类并没有无参构造器，它的构造器需要一个外部类的参数。这符合非静态内部类的规则；非静态内部类必须寄生在外部类的实例中，没有外部类的对象，就不可能产生非静态内部类的对象。因此，非静态内部类不可能有无参的构造器--即使系统为非静态内部类提供一个默认的构造器，这个默认的构造器也需要一个外部类形参。	

​		系统在编译阶段总会为非静态内部类的构造器增加一个参数，非静态内部类的构造器的第一个形参总是外部类，因此调用非静态内部类的构造器时必须传入一个外部类对象作为参数，否则程序将会引发运行时异常。

7.7 static关键字

7.8 native方法的陷阱

```
public class NativeTest {
	public native void info();
}
```

native方法通常需要借助C语言来完成，即需要使用C语言为Java方法提供实现。其步骤如下：

（1）用javah编译上面NativeTest生成的class文件，将产生一个.h文件。

（2）写一个.cpp文件实现native方法，其中需要包含第（1）步产生的.h文件（.h文件中又包含了JDK带的jni.h文件）。

（3）将第（2）步的.cpp文件编译成动态链接库文件。

（4）在Java中用System的loadLibrary方法或Runtime的loadLibrary()方法加载第（3）步产生的动态链接库文件，就可以在Java程序中调用这个native（）方法了。

八、异常捕捉的陷阱

catch块代码里的return;语句时，是否还会执行finally块？-----------会

如果系统执行时遇到System.exit(0);或Runtime.getRuntime.exit(0);时是否还会执行对应的finally块？

使用finally正确地关闭资源；

finally遇到return的处理；

finally遇到System.exit()的处理；

catch块只能先捕捉子类异常；

catch块不能代替流程控制；

只有catch可能抛出的异常；

catch块内应做实际的修复；

只能声明抛出所实现方法允许声明抛出异常的交集。

8.1 正确关闭资源的方式

```
//使用finally来关闭资源
finally{
	if(oos != null) {
		try{
			oos.close();
		}catch(Exception ex) {
			ex.printStackTrace();
		}
	}
}
```

使用finally块来关闭物理资源，保证关闭操作总是会被执行；

关闭每个资源之前首先保证引用该资源的引用变量不为null;

为每个物理资源使用单独try...catch来关闭资源，保证关闭资源时引发的异常不会影响其他资源的关闭；

8.2 finally块的陷阱

当执行System.exit(0)后finally块是不会被执行的；

执行System.exit(0)将停止当前线程和所有其他当场死亡的线程。finally块并不能让已经停止的线程继续执行。

当System.exit(0)被调用时，虚拟机退出前要执行两项清理工作：

（1）执行系统中注册的所有关闭的钩子；Runtime.getRuntime().addShutdownHook(new Thread({}));

（2）如果程序调用了System.runFinalizerOnExit(true);那么JVM会对所有还未结束的对象调用Finalizer

当try和finally都有return语句时，只会执行finally的return语句。

8.3 catch的用法

进行异常捕获时，一定要记住先捕获小的异常，再捕获大的异常。

Runtime异常:RuntimeException类及其子类的实例被称为Runtime异常，无需显示声明抛出，只要程序有需要，即可以在任何有需要的地方使用try...catch块来捕获Runtime异常。

Checked异常：不是RuntimeException类及其子类的实例被称为Checked异常。如果一个catch子句试图捕获一个类型为XxxException的Checked异常，那么它对应的try子句必须可能抛出XxxException或其子类的异常，否则编译器将提示该程序具有编译错误--但在所有Checked异常中，Exception是一个异类，无论try块是怎样的代码，catch(Exception ex)总是正确的。

8.4 继承得到的异常

九、线性表

