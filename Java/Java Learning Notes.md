# **Java学习笔记**

## 1 面向对象

### 	1.1 抽象过程

1. 万物皆为对象。你可以抽取待求解问题的任何概念化构件，将其表示为程序中的对象。
2. 程序是对象的集合，它们通过发送信息来告知彼此所要做的。
3. 每个对象都有自己的由其他对象所构成的存储。
4. 每个对象都拥有其类型。
5. 某一特定类型的所有对象都可以接收同样的信息。

### 1.2 每个对象都提供服务

​		当正在试图开发或者理解一个程序设计时，最好的方法之一就是将对象想象为“服务提供者”。程序本身向用户提供服务，它将通过调用其他对象提供的服务来实现目的。我们的目标就是去创建(或者在现有代码库中寻找)能够提供理想的服务来解决问题的一系列对象。

​		将对象看作是服务提供者还有一个附带的好处：它有助于提高对象的内聚性。<u>高内聚</u>是软件设计的基本质量要求之一：这意味着一个软件构件的各个方面“组合”得很好。

>  	在良好的面向对象设计中，每个对象都可以很好地完成一项任务，但是它并不试图做更多的事。

### 	1.3 访问控制

​		访问控制存在的第一个原因就是让`class`外部的程序无法触及不该触及的部分——这些部分对`class`的内部操作来说是必须的，但并不是用户用于解决特定问题所需的接口的一部分。

​		访问控制存在的第二个原因就是允许`class`设计者可以改变`class`内部的工作方式而不用担心会影响到提供服务的调用。				   

​		Java用三个关键字在`class`内部设定边界：==public==、==private==、==protected==。这些访问指定词（access specifier）决定了紧跟其后被定义的东西能被谁使用。

| public                           | private                                                  | protected                                                    |
| -------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| 紧随其后的元素对任何人都是可用的 | 除类型创建者和类型的内部方法之外的任何人都不能访问的元素 | 与**private**相当，差别在于继承的类可以访问**protected**成员，但不能访问**private**成员 |

​		Java还有一种默认的访问权限，当没有使用前面提到的任何访问指定词（access specifier）时，他将发挥作用。这种权限通常被称为==包访问权限==，因为在这种权限下，类可以访问在同一个包（库构件）中的其他类的成员，但在包之外，这些成员如同指定了**private**一样。

### 	1.4 复用的实现

> ​	代码复用是面向对象程序设计语言所提供的最了不起的优点之一。

​		最简单地复用某个类的方法就是直接使用该类的一个对象，此外也可以将这个类的一个对象置于某个新的类中。我们称其为“创建一个成员对象”。新的类可以由任意数量、任意类型的其他对象以任意可以实现新的类中想要的功能的方式所组成。这种概念被称作`组合（composition）`，如果组合是动态发生的，那么它通常被称为`聚合（aggregation）`。组合经常被视为“has-a”（拥有）关系，就像我们常说的“汽车拥有引擎”一样。

​		组合带来了极大的灵活性。新类的成员对象通常被声明为**private**，使得使用这个新类的外部程序不能访问它。

**由于继承在面向对象程序设计中如此重要，所以它常常被高度强调。实际上，在建立新类时，应该首先考虑组合，因为它更加简单灵活。**

### 	1.5 继承

​		以现有的类为基础，复制它，通过添加和修改这个副本来创建新的类，即`继承`。当继承现有的类时，也就创造了新的类型。这个新的类型不仅包含现有类型的所有成员（尽管**private**成员被隐藏了，并且不可访问），更重要的是它复制了基类的接口，这同时也意味着子类与父类具有相同的类型。

​		有两种方法可以使基类与导出类产生差异。

​		第一种方法非常直接：直接在导出类中添加新方法。这些新方法并不是基带接口的一部分。这意味着基带无法满足你的所有需求，因此必须添加更多的方法；

​		第二种也是更重要的一种使导出类和基带之间产生差异的方法是改变现有基类的方法的行为，这被称之为==覆盖（override）==那个方法。想要覆盖某个方法，可以直接在导出类中创建该方法的新定义。

### 	1.6 多态

​		多态是同一个行为具有多个不同表现形式或形态的能力，换句话说，多态就是同一个接口，使用不同的实例而执行不同操作。如图所示：

![转自菜鸟教程](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/java_polymorphism_from_runoob.png)

多态存在的三个必要条件：

- 继承
- 重写（override）

- 父类引用指向子类对象：**Parent p = new Child();**

将子类看作它的父类的过程称为**向上转型**。

### 	**1.7 单根继承结构**

​		在OOP中，自C++面世以来有一个非常瞩目的问题：是否所有的类最终都继承自单一的基类。在Java中（事实上还包括除C++以外的所有的OOP语言），答案是yes，这个终极基类的名字就是**Obejct**。

​		单根继承结构使垃圾回收器的实现变得容易得多，而垃圾回收器正是Java相对C++的重要改进之一。

## 2 一切都是对象

### 	2.1 用引用操控对象

​		在Java中，一切都被视为对象，但操控的标识符实际上是对象的一个“引用”。举个例子：

```java
String s = "asdf";
//在这里，标识符s是对字符串对象“asdf”的引用
String s;
//在这里，我们创建了String引用，但并未关联一个对象，因此，对s采用String的方法是错误的
/* 注释：这个关联很容易让人联想到C语言中指针指向，这样可能可以更好理解	*/
```

实际上，通常用`new`来创建新的对象，如下。

```java
String s = new String("asdf");
```

### 	2.2 存储

Java将数据分别存储在五个地方：

1. **寄存器**。这是最快的存储区，它位于处理器内部，但是寄存器的数量极其有限，所以寄存器根据需求进行分配。

2. **栈（stack）**。在一些书籍上，也被翻译成堆栈，它位于通用RAM（随机访问存储器）中，通过栈指针获得支持，栈指针向下移动，则分配新的内存；若向上移动，则释放哪些内存。这是一种快速有效的分配存储方法，仅次于寄存器，但在创建程序时，Java系统必须知道存储在堆栈内所有项的确切生命周期，以便上下移动栈指针。

   > 在Java中，八大基本类型以及对象的引用存储在栈中。

3. **堆（heap）**。一种通用的内存池（也位于RAM区），==用于存放所有的Java对象==。不同于栈的好处是：编译器不需要知道存储的数据在堆中的存活时间，因此，在堆里分配存储有很大的灵活性。当需要一个对象时，只需用`new`写一行简单的代码，当执行这行代码时，会自动在堆里进行存储分配。当然，为了这种灵活性必须付出代价：用堆进行存储分配和清理可能比栈进行存储分配需要更多时间。
4. **常量存储**。常量值通常直接存放在程序代码内部，这样做是安全的，因为它们不会被改变。有时，在嵌入式系统中，常量本身会和其他部分隔离开来，这种情况下，可以将其放在ROM（只读存储器）中。
5. **非RAM存储**。如果数据完全存活于程序之外，那么它可以不受程序的任何控制，在程序没有运行时也可以存在。其中两个例子是`流对象`和`持久化对象`。在流对象中，对象转化为字节流，通常发往另一台机器。在“持久化对象”中对象被存储在磁盘上。

### 2.3 基本类型

​		Java将对象存储在`堆`里，因此用`new`创建一个对象——特别是小的、简单的变量，往往不是很有效（对象需要标识符来引用，而标识符存储在`栈`中，对于很小的、简单的对象，如果再给它创建一个标识符存于`栈`中，不如直接将其存于`栈`），因此，对于一系列类型，将它们特殊对待，不用`new`来创建变量，而是直接在`栈`中创建一个非引用的“自动”变量，这个变量直接存储“值”。

| 基本类型    | 大小    | 最小值    | 最大值         | 包装器类型    | 默认值         |
| :---------- | ------- | --------- | -------------- | ------------- | -------------- |
| **boolean** | --      | --        | --             | **Boolean**   | false          |
| **char**    | 2 Bytes | Unicode 0 | Unicode 2^16-1 | **Character** | ‘\u0000’(null) |
| **byte**    | 1 Byte  | -128      | +127           | **Byte**      | (byte)0        |
| **short**   | 2 Bytes | -2^15     | +2^15-1        | **Short**     | (short)0       |
| **int**     | 4 Bytes | -2^31     | +2^31-1        | **Integer**   | 0              |
| **long**    | 8 Bytes | -2^63     | +2^63-1        | **Long**      | 0L             |
| **float**   | 4 Bytes | IEEE754   | IEEE754        | **Float**     | 0.0L           |
| **double**  | 8 Bytes | IEEE754   | IEEE754        | **Double**    | 0.0d           |
| **void**    | --      | --        | --             | **Void**      |                |

> 当类的某个成员是基本数据类型，即使没有初始化，Java也会确保它获得一个默认值。
>
> 注意：这个默认的初始化并不适用于“局部”变量（即并非某个类的字段）

**boolean**类型所占存储空间的大小没有明确指定，仅定义为能够取字面量**true**或**false**。

>  高精度数字

Java提供了两个用于高精度计算的类：BigInteger和BigDecimal。

这两个类包含的方法，提供的操作与对基本类型所能执行的操作相似，但必须以方法调用的方式取代运算符方式来实现。

BigInterger支持任意精度的整数；BigDecimal支持任意精度的浮点数。

### 	**2.4 数组**

​		虽然Java借鉴了C++中许多概念，但Java并不像C++那样，直接以内存块来作为数组。Java确保数组会被初始化，而且不能在它的范围之外被访问。这种范围检查，是以每个数组上少量的内存开销及运行时的下标检查为代价的。

​		当创建了一个数组，实际上就是创建了一个引用数组，并且每个引用都会自动被初始化为**null**，一旦Java看到了null，就知道这个引用还没有指向某个对象。

### 	**2.5 对象的作用域**

> 记那么多，自己以后都懒得翻了QAQ，以后只记一些比较难的，自己没怎么理解的了

```java
{
  String s = new String("Hello, world!");
}
```

​		引用s在作用域终点就消失了，然后，s指向的**String**对象仍占据着内存空间，虽然已经没法访问它了。这引发了一个有趣的问题：如果Java让对象继续存在，靠什么才能防止这些对象填满内存空间，进而阻塞程序呢？答案是Java有一个`垃圾回收器`，用来监视用`new`创建的所有对象，并辨别那些不会再被引用的对象，随后释放这些对象的内存空间，以便其他新的对象使用。

> 事实上，当我看到这里，想起了上面在单根继承结构中提到的：所有的类最终都继承自单一的基类，单根继承结构使垃圾回收器的实现变得容易得多。也许所有的对象都具有最终基类（Object）的一个字段，而这个字段指明了这个对象是否还能再被引用，从而判断它是不是垃圾？哈哈，这只是我的胡思乱想罢了。

### 	**2.6 构建一个Java程序**

```java
import java.util.*;

public class HelloDate {
  public static void main(String[] args){
    System.out.printIn("Hello, it's: ");
    System.out.printIn(new Date());
  }
}
```

> 关于Java的文档，可以直接去Oracle的官网下载，在此贴上链接https://www.oracle.com/java/technologies/javase-jdk8-doc-downloads.html



## 3 积累的知识点

### 	3.1 关于equals与==的区别

- 如果是基本类型比较，那么只能用==来比较，不能用equals
- 对于基本类型的包装类型，比如Boolean、Character、Byte、Short、Integer、Long、Float、Double等引用变量，==用于比较地址，即是否引用相同，而equals用于比较内容。如下
- 在默认情况下，类继承Object的equals方法，比较对象的引用是否相同，因此如果想要通过equals来对比类的值是否相同，应当在类中重写equals方法。例如：在String类的定义中，重写了equals方法的定义，从而使得比较的是值而不是地址

```java
public class TestEquals { 
	public static void main(String[] args) { 
		Integer n1 = new Integer(30); 
		Integer n2 = new Integer(30); 
		System.out.println(n1 == n2);//false ， 
		System.out.println(n1.equals(n2));//true
	} 
}
```

### 3.2 关于异常的分类

> 摘自https://www.cnblogs.com/qlqwjy/p/7816290.html

​		Java从==Throwable==直接派生出==Exception==和==Error==。其中==Exception==是可以抛出的基本类型，在Java类库、方法以及运行时故障中都可能抛出==Exception型==异常。==Exception==表示可以恢复的异常，是编译器可以捕捉到的；==Error==表示编译时和系统错误，表示系统在运行期间出现了严重的错误，属于不可恢复的错误，由于这属于JVM层次的严重错误，因此这种错误会导致程序终止执行。==Exception==又分为检查异常和运行时异常。异常类的结构层次图如下：

![img](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/316892-20151218125019552-659007001.png)

​		而按照编译器检查方式划分，异常又可以分为检查型异常（CheckedException）和非检查型异常 （UncheckedException）。Error和RuntimeException合起来称为UncheckedException，之所以这么称呼，是因为编译器不检查方法是否处理或者抛出这两种类型的异常，因此编译期间出现这种类型的异常也不会报错，默认由虚拟机提供处理方式。除了Error 和RuntimeException这两种类型的异常外，其它的异常都称为Checked异常。

​		对于==RuntimeException==即使我们在一个方法上throws声明可能抛出异常，编译器也会检测到它是运行时异常而不要求我们在主方法中必须处理；对于检查型异常我们throws声明之后也必须处理才可以编译通过。

例如：

![image-20210805213635442](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/image-20210805213635442.png)

```java
public class MyTest {

    public static void test1() throws NumberFormatException {
        String str = "11,22,33,fg,44,55,66";
        String[] split = str.split(",");
        for (String s : split) {
            System.out.println(Long.parseLong(s));
        }
    }
		// 抛出的是NumberFormatException,通过继承链可看出其为非检查型异常
    public static void main(String[] args) {
        MyTest.test1();
    }
}
```

![image-20210805213828520](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/image-20210805213828520.png)

```java
public class MyTest {

    public static void test1() throws SQLException {
        String str = "11,22,33,fg,44,55,66";
        String[] split = str.split(",");
        for (String s : split) {
            System.out.println(Long.parseLong(s));
        }
    }
		// 抛出的是SQLException,通过继承链可看出其为检查型异常
    public static void main(String[] args) {
      	//Unhandled exception type SQLException
      	//2 quick fixes available:
      	//Add throws declaration
      	//Surround with try/catch
        //!MyTest.test1();
    }
}
```

对于checked类型异常，我们要么对它进行处理，要么在方法头使用throws进行说明，将其抛出至更高一层的方法中。

> Tips: 如果在try和finally块中都执行了return语句，最终返回的将是finally中的return值。

### 3.3 关于数组转List

​		数组转List通常使用Arrays.asList(E... a)方法，但需要注意的是，这个函数返回的是继承自AbstractList的内部类，因此，并没有显示ArrayList中的添加和删除元素的方法。因此，需要将返回的AbstractList作为参数传递给ArrayList构造：

```java
List<T> list = new ArrayList<T>(Arrays.asList(T... a))
```

> T必须作为类，因此，对于int[] arr这样的基本类型数组，不能直接通过上面的方法将数组转换为List，但如果是Integer[] arr这种包装类，就可以直接使用上述方法。

对于上面提到的基本类型数组转容器，我们也有相应的办法：

```java
// 方法1
int[] ints = {1,2,3};
List<Integer> list = new ArrayList<Integer>(ints.length);
for(int i: ints)	list.add(i);

// 方法2，java8+
int[] ints = {1,2,3};
List<Integer> list = Arrays.stream(ints).boxed().collect(Collectors.toList());
```

