---
title: "Java学习小记"

date: 2020-11-05T05:00:00Z
# thumbnail: "images/photo/R0006712.JPG"
categories: ["Java"]
author: "Qingfeng Zhang"
---

# 1\. 对java中泛型的理解

刚开始学习java，发现java建立的数组不能够增加或是删除元素，非常的不方便，而java中的list可以，在Java中新建一个整型list可以通过：

```java
List<Integer> l = new ArrayList<>()  
```
  
得到，可以指定类型为Integer或String等，也可以不指定；  
**问题：首先是这个类型的指定不理解，其次这像是一个建立对象实例化的语法，但是怎么前面是list，后面却是ArrayList** ；  
**对于第一个问题** ，查了资料发现这是**泛型** ，那泛型有什么作用？关于这个看了一些资料进行总结：  
java程序开发是面向对象，所以会有各种各样类型的对象，而对象需要用容器来装，像数组、ArrayList、HashMap都是容器；  
对于整型数组，一开始就指定了所装的对象类型：

```java
int[] L;  
```
  
对于集合类型的容器，如ArrayList和HashMap，可以装入任何类型的对象，因此这些容器设计为装Object类型的对象，即根类，但是在取出对象的时候需要进行强制类型转换，转换成实际的类型，这样很不安全，因此就需要泛型，在这个问题里，泛型可以指定容器装入对象的类型,以我的理解（当然这只是一个片面且不一定正确的理解），如果建立一个整型list：

```java
List<Integer> result = new ArrayList<>()  
```
  
那么这个result只能添加整型数据，并且如果将Integer换成String后只能添加字符串，如果运行`result.add(1)`会报错；  
关于上面的Integer和int又是一个问题了，简单来说Integer是java为int提供的封装类，即Integer是一个对象，而int只是一个基本数据类型，**基本类型不能作为泛型的参数**
，泛型指定的类型必须是**引用数据类型**(类，接口，数组)，因此不能用**基本数据类型**(内置数据类型)int；  
比如，如果定义了一个类phone，指定list泛型的数据类型为phone，那么list就只能装phone对象：

```java
List<phone> P = new ArrayList<>();  
P.add(new phone("xiaomi"));  
```
  
泛型的应用场景有泛型类，泛型接口，泛型方法；

**对于第二个问题**
，这里的List是一个接口，而ArrayList是对List接口实现的类，因此ArrayList的方法包含List的方法，但是接口是不能实例化的，这里是一个接口类型的引用指向了一个实现给接口的对象；  
举一个例子，先定义一个接口Talking，里面的方法say默认为静态方法，需要重载，然后定义一个people类来实现Talking接口，在people类里对say方法进行重载，然后用people实例化一个对象P，并声明一个Talking接口类型的引用指向这个对象：

```java
interface Talking{  
    void say();  
}  
    
class People implements Talking{  
    public void say(){  
        System.out.println("Hello!");  
    }  
}  
    
public class interface_test{  
    public static void main(String[] args){  
        Talking P = new People();  
        P.say();  
    }  
}  
```
  
实际中遇到过的问题：当我想用一个整数数组来初始化HashSet，错误的代码为：

```java    
Set<Integer> set = new HashSet<>(Arrays.asList(nums));  
```
  
问题在于这里的nums是一个int数组，是基本类型，`Arrays.asList(nums)`的数据类型为`java.util.Arrays$ArrayList`，因此不能用来初始化一个HashSet，对于ArrayList也是一样，如果想要把基本类型int的数组转换为HashSet或是ArrayList，可以通过：

```java
Set<Integer> set = Arrays.stream(A).boxed().collect(Collectors.toSet());  
```
  
或是循环逐个添加；

# 2\. Collections接口与Map接口

最近在做题的时候遇到了一个小问题，在判断一堆数里是否存在一个值时，会使用ArrayList的contains方法，导致的问题就是会超时，找了一圈才发现用HashSet的contains方法更好（注意ArrayList可以按索引删除），然后在定义HashSet对象的时候又蒙了，这些数据结构的接口与类之间到底是什么关系，于是就想整理一下；  
Collections是一个基本的集合接口，List、Set、Queue接口继承了Collections接口；List是一个有序的Collections，因此能够控制元素插入的位置，并且通过索引来访问元素，允许元素重复；Set接口与Collections完全相同，只是Set存储一组无序的对象，且不保存重复的元素；List按索引查找的效率高，插入和删除的效率低，因为有序会对导致其他元素的位置改变，而Set检索的效率低，删除和插入的效率高，因为Set无序；  
1.对于**List** 接口的实现类：**ArrayList** 、LinkedList；  
2.对于**Set** 接口的实现类：**HashSet** 、TreeSet、LinkedHashSet；  
3.Queue接口可以使用LinkedList类；  
Map也是一个接口，Map中的每一个元素都包含键对象和值对象，**Map** 接口的实现类有：HashMap、HashTable、TreeMap；  
每个接口都包含一些方法，每个实现类都实现了其接口的方法，并且都具有自身的特点，因此，对于不同的问题应该选择合适的类建立对象；

# 3\. Lambda表达式

Lambda表达式是Java 8的一个重要的新特性，也可称为闭包，Lambda表达式允许把函数作为一个方法的参数，主要还是因为遇到了这样一个问题：  
有一个经典的问题：找出一些数种最小的k个数，这里不详解这个问题的解题方法，直接排序是可以的，但是对于数据量巨大的情况不合适，其中有一个方法就是维护一个容量为k的大根堆，大根堆会将最大的数放在树的根节点，每次遇到比根节点小的数就更新根节点为小的数，而Java种可以使用PriorityQueue来实现大根堆或小根堆，且默认是小根堆，如果要使用大根堆则要自定义比较器，如下：

```java    
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(k,new Comparator<Integer>(){  
    @Override  
    public int compare(Integer o1,Integer o2){  
        return o2-o1;  
    }  
});  
```
  
当然这样写没问题，但是也可以使用Lambda表达式来实现：

```java
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(k,(o1,o2)->{return o2-o1;})  
```
  
非常简洁，因为这个小问题，所以我想看下Lambda表达式；  
**Lambda表达式的语法：**  
`(parameters) -> expression`  
或者：  
`(parameters) -> {statements;}`  
Lambda的参数不需要声明类型，编译器会自动返回表达式的值，或是使用`{}`指明需要返回的值，如：  
`(a, b) -> a*b;`  
`(a, b) -> {return a*b};`  
**Lambda表达式的变量：**  
1.lambda表达式只能引用标记final的外层局部变量，不能在表达式内修改外部的变量值；  
2.如果引用的变量没有声明为final，该变量之后不能修改值（隐性final）；  
3.lambda表达式内不能使用与局部变量同名的参数；  
对于一个接口，如果只包含一个抽象方法，那么它就是一个**函数式接口**
，对于函数式接口可以通过Lambda表达式来创建该接口的对象，下面是一个从内部类到Lambda表达式的例子：

```java
public class test{  
    //1.静态内部类  
    static class cat1 implements Animal{  
        @Override  
        public void call(){  
            System.out.println("Hello!");  
        }  
    }  
    
    public static void main(String[] args){  
        Animal a = new cat1();  //静态内部类的使用  
        a.call();  
    
        //2.局部内部类  
        class cat2 implements Animal{  
            @Override  
            public void call(){  
                System.out.println("Hello!");  
            }  
        }  
        a = new cat2();  //局部内部类的使用  
        a.call();  
    
        //3.匿名内部类，没有类名称，借助接口或父类  
        a = new Animal(){  
            @Override  
            public void call(){  
                System.out.println("Hello!");  
            }  
        };  
        a.call();  
    
        //4.Lambda表达式  
        a = () -> {  
            System.out.println("Hello!");  
        };  
        a.call();  
    
    }  
}  
interface Animal{  
    void call();  
}  
```
  
# 4\. HashMap与HashCode：

刚刚遇到了一个小问题：使用HashMap时想使用数组作为key，但是内容相同的数组并不能通过containsKey()查找出来，因此想知道HashMap是怎么判断key相等的？还有哪些数据类型作为key可以直接通过内容判断是否存在对应的key？哪些又不行？  
简单来说，JVM每new一个Object，都会根据其hashcode将这个Object放到Hash表中，查找的时候直接依据它的hashcode查找以提高效率，对于hashcode有以下三个约定：

  1. 在一个应用程序运行期间，假设一个对象的equals方法做比較所用到的信息没有被改动的话。则对该对象调用hashCode方法多次，它必须始终如一地返回同一个整数。
  2. 假设两个对象依据equals(Object o)方法是相等的，则调用这两个对象中任一对象的hashCode方法必须产生同样的整数结果。
  3. 假设两个对象依据equals(Object o)方法是不相等的。则调用这两个对象中任一个对象的hashCode方法。不要求产生不同的整数结果。但假设能不同，则可能提高散列表的性能。

注意两个hashcode相同的对象不一定相等；  
HashMap中比较两个key是否相等时会使用hashcode()和equals()方法，即先比较key的hashcode()，相等的话再比较equals()，所以如果要在HashMap放入自定义的key类，一般需要重写该类的hashcode()和equals()(Object)方法，以保证相同的比较逻辑；  
int[]对象的hashcode方法是直接使用Object类的，计算hashcode的结果与引用有关，因此两个内容相同但是引用不同的数组有不同的hashcode；  
String的hashcode是由内容计算的，因此内容相等的字符串hashcode也相同，但是也存在内容不同hashcode相同的情况；  
`==`符号对于基本数据类型是比较值，对于引用数据类型是比较引用(地址)；  
`equals()`在Object中比较的是引用（没有被重写的话则是比较引用），不用于基本数据类型的比较，**在String类中被重写，比较的是值** ；  
（对于基本数据类型的包装类，`==`比较的是地址，`equals()`比较的是值）  
综上可以知道，当使用基本数据类型的包装类作为HashMap的key时，每次比较(计算hashcode()和equals())的是变量的值，对于引用类型，如果没有重写hashcode()和equals()的话，默认比较引用(地址)；

# 5\. 笔记碎片

## 001 列表List的常用方法：

Java集合有两个体系：Collection和Map；  
Collection主要有三个接口：List(列表)、Set(集)、Queue(队列)；  
List中主要有ArrayList和LinkedList两个实现类；  
List集合是有序的，ArrayList底层通过数组实现，可以动态扩容，而LinkedList底层通过链表实现，也可以不断增加节点实现扩容；  
在这里记录一下**List的常用方法：**

```java    
List<Integer> l = new ArrayList<>();  //初始化list  
    
l.add(12);  //向list添加元素  
l.set(0,13);  //修改指定位置的元素值  
l.get(0);  //获取指定位置的元素值  
l.isEmpty();  //判断是否为空  
l.remove(0);  //删除指定位置的值  
l.clear();  //清空list  
```
  
关于remove()方法一直有疑惑，怎么知道里面的参数是索引还是元素值？网上的一些博客很好地“避免”了这个问题，因为看到的所有List的泛型都是String，几乎不谈Integer。其实这个也很容易理解，举个例子就知道了：

```java
List<Integer> l = new ArrayList<>();  
    
l.add(9);  //[9]  
l.add(8);  //[9,8]  
l.add(7);  //[9,8,7]  
    
l.remove(0);  //[8,7]  
l.remove((Integer)7);  //[8]  
```
  
这样就一目了然了，因为List内的元素是Integer类型，只有将整数转为Integer包装类型，才能按照元素值删除，否则就是将整数当作索引；

## 002 字符串String的基本方法与字符类型Char：

在Java中，char是一个基本数据类型，在定义的时候用的是单引号`''`，但是String是一个类，属于引用数据类型，定义的时候用的是双引号`""`;

  * String的常用方法： 

```java
String name = "zhangqingfeng";  
char[] L = name.toCharArray();  //转换为字符数组；  
String[] S = name.split("n");  //按*字符串*"n"来分割字符串；  
System.out.println(name.length());  //字符串的长度；  
System.out.println(name.charAt(2));  //获取索引为1的字符，返回char类型；  
System.out.println(name.indexOf("q"));  //获取*字符串*"q"的位置；  
System.out.println(name.subString(0,5));  //获取字符串的一个子串；  
System.out.println(name.contains("z"));  //判断字符串中是否包含另一个字符串；  
System.out.println(name.replaceAll("z","Z"));  //替换字符串中的所有指定子串；  
System.out.println(name.replaceFirst("z","Z")); //替换字符串中第一次出现的子串；  
```

经常将长度为1的字符串和单个字符弄混，其实这两个是不一样；

  * 将char转换成String：
    
```java
String S = String.valueOf('z');  //效率比较高；  
String S = "" + 'z';  //简单但是效率低；  
```

将长度为1的String转为char直接使用类型转换`(char)"z"`就行；

## 003 list排序与初始化：

对一个**二维数组** M按照**第一列进行排序** ：

```java
Arrays.sort(M,Comparator.comparingInt(x -> x[0]));  
```
  
**创建一个list并用变量l和r初始化** ，可以使用：

```java
List<Integer> L = new ArrayList<>();  
```
  
先创建一个list然后`L.add(l);``L.add(r);`,这样比较麻烦，可以更简单点：

```java
List<Integer> L = new ArrayList<>(Arrays.asList(l,r))  
```
  
或是：

```java
List<Integer> L = Arrays.asList(l,r)  
```
  
## 004 关于二维数组与二维list：

有个题目要求返回一个二维的数组`int[][] result`，但是数组不方便，只好用list，一开始是`List<List<Integer>>
result`，即先建立一个二维的list，但最后想转换为二维数组时不知道怎么转;  
其实可以建立一个存放一维数组的list，即`List<int[]> result`，其存放的一维数组长度为2，这样的话就可以方便的将其转换为二维数组：

```java
result.toArray(new int[result.size()][2]);  
```
  
里面的参数可以理解为构造一个int类型与result维度一致的空数组。

## 005 字符串与字符(数组)的转换

**字符串- >字符串(字符)数组：**

```java
String name = "zhang,qing,feng";  
String[] arr1 = name.split(",");  //转换成字符串数组  
char[] arr2 = name.toCharArray(name);  //转换成字符数组  
```
  
**字符数组- >字符串：**

```java    
char[] arr = ['z','h','a','n','g'];  
String name1 = new String(arr);  //方法一  
String name2 = String.valueOf(arr);  //方法二  
```
  
**字符串- >字符**

```java    
String s = "z";  
char ch = name.charAt(0);  
```
  
**字符- >字符串**

```java
char ch = 'z';  
String s1 = Character.toString(ch);  //方法一  
String s2 = String.valueOf(ch);  //方法二  
String s3 = ch+"";  //方法三  
```
  
## 006 自定义排序方法对list进行排序：

在NO.937中有一种奇怪的字符串比较方式，还要按照这种方式排序；  
对于数组的排序是：

```java
Arrays.sort(arr);  
```
  
对于集合的排序是：

```java
Collections.sort(arr2);  
```
  
后者有更复杂的排序，可以对Comparator接口的compare方法进行重写，如：

```java
Collections.sort(l1,new Comparator<String>(){  
    public int compare(String s1, String s2){  
        int i=s1.indexOf(" "), j=s2.indexOf(" ");  
        if(s1.substring(i).compareTo(s2.substring(j))==0){  
            return s1.substring(0,i).compareTo(s2.substring(0,j));  
        }  
        return s1.substring(i).compareTo(s2.substring(j));  
    }  
});  
```
  
根据compare返回值判断s1和s2的大小关系，如果返回正数表示s1大，负数表示s2大，0表示相等；

## 007 关于Java中list与int[]互转的坑：

以下代码是建立一个整数list并将其转换为int[],结果报错说

```java    
List<Integer> l = new ArrayList<>();  
l.add(1);  
l.add(3);  
l.add(4);  
System.out.println(l.toArray(new int[l.size()]));  
```
  
结果报错说没有`toArray(int[])`方法，但是换成String却可以，问题在于后面的`int[]`，要写成`Integer[]`;  
对于一个`List<int[]> L`要将其转为`int[][]`只要`L.toArray(new
int[L.size()][])`即可，所以问题就在于不能用基本数据类型，换成包装类就行；  
**将list 转为int[]:**  
`int[] arr = list.stream().mapToInt(Integer::valueOf).toArray();`  
**将int[]转为list :**  
`List<Integer> list =
Arrays.Stream(arr).boxed().collect(Collectors.toList());`  
**将int[]转为Integer[]**  
`Integer[] I = Arrays.stream(arr).boxed().toArray(Integer[]::new);`  
**将Integer[]转为int[]**  
`int[] arr = Arrays.Stream(I).mapToInt(Integer::valueOf).toArray();`

## 008 Java中的栈与队列：

### 栈的实现与方法：

栈可以通过Java本身的集合类型Stack实现：

```java
Stack stack1 = new Stack();  
Stack<Integer> stack2 = new Stack<>();  
```
  
**栈的常用方法如下：**

```java    
stack.empty()  //判断栈是否为空  
stack.peek()  //获取栈顶值（不出栈）  
stack.push(value)  //入栈  
stack.pop()  //出栈  
```
  
### 队列的实现与方法：

Java给出了队列的接口Queue，但是没有具体的实现类，可以使用LinkedList实现类，

```java    
Queue<Integer> que = new LinkedList<>();  
```

**队列的常用方法：**

```java    
que.isEmpty()  //判断队列是否为空  
    
// 获取队头元素  
que.peek()  //队空返回null  
que.element()  //队空抛出异常  
    
// 元素入队  
que.offer(value)  //队满返回false  
que.add(value)  //队满抛出异常  
que.put(value)  //队满阻塞  
    
// 元素出队  
que.poll()  //队空返回null  
que.remove()  //队空抛出异常  
que.take()  //队空阻塞  
```
  
此外，队列还包括双端队列，双端队可以使用LinkedList实现类和ArrayDeque实现类；LinkedList是大小可变的链表双端队列，允许null值，ArrayDeque是大小可变的数组双端队列，不允许null值：

```java
Deque<Integer> deq = new LinkedList<>();  
Deque<Integer> deq = new ArrayDeque<>(4);  
```
  
**双端队列的常用方法：**

```java
deq.peekFirst()  //获取队头元素  
deq.peekLast()  //获取队尾元素  
    
deq.offerFirst()  //从队头入队  
deq.offerLast()  //从队尾入队  
    
deq.pollFirst()  //队头元素出队  
deq.pollLast()  //队尾元素出队  
```
  
还可以使用一般队列的方法，此外队列还有BlockingQueue，以上的所列出的只是常用的，不是全部；
