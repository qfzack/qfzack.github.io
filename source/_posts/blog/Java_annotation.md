---
title: "学习笔记：Java中的注解与反射(一)"

date: 2020-03-09T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Java"]
author: "Qingfeng Zhang"
---


# 一、注解

## 1.注解的概念

注解是Java提供的一种原程序中的元素关联任何信息和任何元数据的途径和方法。

## 2.Java中的常见注解

### 2.1 JDK自带的注解

  * `@Override`：当对接口中的方法进行实现时，需要覆盖；
  * `@Deprecated`：表示方法已经过时不再使用；
  * `@Suppvisewarnings`：如果想要使用已标注`@Deprecated`的方法，可以加上注解`@Suppvisewarnings("deprecated")`;

## 3.注解的分类

**按照运行机制分类：**

  * 源码注解：注解只在源码中存在，编译成.class文件就不存在了；
  * 编译时注解：在源码和.class文件中都存在，如上述的JDK自带注解；
  * 运行时注解：在运行阶段还会起作用，升值会影响运行时逻辑，如`@Autowired`；

**按照来源分类：**

  * 来自JDK的注解；
  * 来自第三方的注解；
  * 自定义的注解；

**元注解：**

  * 注解的注解；

## 4.自定义注解

### 4.1 自定义注解的语法要求

使用`@interface`关键字定义注解，以下是定义注解的示例，类定义的上面是**元注解** ：  
第一行`@Target`标明注解的作用域是方法和类，可以使用的参数有：构造方法的声明，字段声明，局部变量的声明，方法声明，包声明，参数声明，类接口；  
第二行`@Retention`表明注解的生命周期，如在源码显示编译时丢弃，编译时记录到calss运行时忽略，运行时存在可通过反射读取；  
第三行`@Inherited`表明允许子类继承，但是不能继承接口和方法的，只能继承类的注解；  
第四行`@Documented`表明生成javadoc时会包含注解；

```java
@Target({ElementType.METHOD, ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Inherited  
@Documented  
public @interface Description{  
    String desc();  //成员以无参无异常的方式声明；  
    
    String author();  
    
    int age() default 18;  //可以使用default为成员指定默认值；  
}  
```
  
注解类中成员的类型是受限的，合法的类型包括原始类型和String,Class,Annotation,Enumeration；  
如果注解只有一个成员，则成员名必须取名为`value()`，在使用时可以忽略成员名和赋值号`=`；  
注解类可以没有成员，没有成员的注解称为标识注解；

### 4.2 自定义注解的使用

使用注解的语法：

```java
@<注解名>(<成员名1>=<成员值1>,<成员名2>=<成员值2>,...)  
```
  
举个栗子如下：

```java
@Description(desc="IAmEyeColor",author="Zack",age=18)  
public String eyeColor(){  
    return "Blue";  
}  
```
  
其中`@Description`是上面自定义的注解；

### 4.3 注解的解析

解析注解是通过反射获取类、函数或成员上的**运行时** 注解信息，从而实现动态控制程序运行的逻辑；  
**找到类上的注解：**

```java
Class c = Class.forName("com.zhangqf.test.Child");  //加载一个类；  
boolean isExist = c.isAnnotationPresent(Description.class);  
if(isExist){  
    Description d = (Description)c.getAnnotation(Description.class);  
    System.out.println(d.value());  
}  
```
  
**找到方法上的注解：**

```java
Method[] ms = c.getMethods();  
for(Method m: ms){  
    boolean isMExist = m.isAnnotationPresent(Description.class);  
    if(isMExist){  
        Description d = (Description)m.getAnnotation(Description.class);  
        System.out.println(d.value());  
    }  
}  
```
  
**注解本身只是起到标记的作用，注解需要反射机制，根据注解的标记去调用注解解释器执行行为。**

# 二、反射

## 1.Class类的使用

“在面向对象的世界里，万事万物皆对象”，在Java中静态的成员和基本数据类型不是对象(有封装类)，而每一个类都是java.lang.Class的实例对象;

  * **使用反射实例化一个类有三种方式:**

```java
Food food = new Food();  //Food是一个自定义的类；  
//方式1：类.class  
Class c1 = Food.class;    
//方式2：对象.getClass()  
Class c2 = food.getClass();  
//方式3：Class.forName("name")  
Class c3 = Class.forName("com.zhangqf.Food");  
    
//通过类的类型创建该类的实例对象  
//方式一：使用newInstance()方法  
Food food = (Food)c3.newInstance();  
//方式二：先调用构造器，再使用newInstance()方法  
Object food2 = c3.constructor().newInstance();  
```
  
以上的c1，c2，c3表示了food类的类类型(class type)，且一个类只能是Class类的一个实例对象，所以c1，c2，c3相等；

  * **获取类的信息，包括类的成员函数和成员变量：**

```java    
public class ClassUtil{  
    public static void getClassMessage(Object obj){  
        //先获取对象的类的类类型  
        Class c = obj.getClass();  //传递的是哪个子类的对象，c就是该子类的类类型  
        //获取类名称  
        System.out.println(c.getName());  
    
        //成员方法是Method的对象，getMethods()获取类的public方法，包括父类继承来的  
        //getDeclaredMethods()是获取该类自己声明的方法，不问权限  
        Method[] ms = c.getMethods();  //c.getDeclaredMethods();  
        //获取对象中方法的返回值类型的类类型、方法名称和参数类型  
        for(int i=0;i<ms.length;i++>){  
            Class returnType = ms[i].getReturnType();  //方法返回值的类类型  
            System.out.println(returnType.getName());  //返回值类类型的名称  
            System.out.println(ms[i].getName());  //方法的名称  
            Class[] paramType = ms[i].getParameterTypes();  //得到参数列表的类型的类类型，如int.class  
            for(Class class1: paramType){  
                System.out.println(class1.getName());  
            }  
        }  
    
        //获取类对象中方法的成员变量信息  
        //在java.lang.reflect.Field中封装了关于成员变量的操作  
        //getFields()获取所有public的成员变量的信息；  
        Field[] fs = c.getDeclaredFields();  //获取该类声明的所有成员变量信息；  
        for(Fiels field: fs){  
            Class fieldType = field.getType();  //获取成员变量类型的类类型  
            String typeName = fieldType.getName();  //获取成员变量的名称  
        }  
    
        //获取对象的构造函数的信息，构造函数也是对象  
        //在java.lang.Constructor中封装了构造函数的信息  
        //getConstructors()获取所有的public的构造函数  
        Constructor[] cs = c.getDeclaredConstructors();  //获取所有的构造方法，构造函数均是自己声明的；  
        for(Constructor constructor: cs){  
            System.out.println(constructor.getName());  
            //获取构造函数的参数列表，得到的是参数列表的类类型  
            Class[] paramTypes = constructor.getParameterTypes();  
        }  
    }  
}  
```
  
## 2.方法的反射执行

通过方法的名称和方法的参数列表才能唯一决定某个方法；  
举个栗子，有以下的类定义，类中有两个同名的方法：

```java    
Class A{  
    public void print(int a,int b){  
        System.out.println(a+","+b);  
    }  
    public void print(String s1,String s2){  
        System.out.println(s1+s2);  
    }  
}  
```
  
如果要获取`print(int a,int b)`方法，首先要获取类的类类型，从而获取类的信息：

```java
//获取A类的类类型  
A a1 = new A();  
Class c = a1.getClass();  
    
//获取方法、名称和参数列表  
try{  
    //参数可以写为("print",new Class[]{int.class,int.class})  
    Method m = c.getMethod("print",int.class,int.class);  //方法可能不存在，因此需要处理异常，  
    
    //方法的反射操作是使用对象m来进行方法的调用与a1.print()效果相同  
    //与方法的返回值相同，若没有则为null  
    Object o = m.invoke(a1,123,456);  //参数可以写为(a1,new Object[]{123,456})  
}catch(Exception e){  
    e.printStackTrace();  
}  
```
  
## 3.动态类加载

`new`创建对象是静态加载类，在编译的时候就需要加载所有可能用到的类，比如在`main`方法中写到了类的使用，就必须保证该类已被加载，否则报错，如：

```java    
Class Office{  
    public static void main(String[] args){  
        if("Word".equals(args[0])){  
            Word w = new Word();  
            w.start();  
        }  
        if("Excel".equals(args[0])){  
            Excel e = new Excel();  
            e.start();  
        }  
    }  
}  
```
  
如果Word类或Excel类未定义或未加载，则会报错，如果只想使用其中一个类，静态加载类无法做到；  
通过**动态加载类** 可以解决这个问题：

```java    
class Office{  
    public static void main(String[] args){  
        try{  
            //动态加载类，在运行时刻加载  
            Class c = Class.forName(args[0]);  
            //通过类类型创建对象  
            OfficeAble oa = (OfficeAble)c.newInstance();  //需要另外定义OfficeAble  
            oa.start()  
        }catch{  
            e.printStackTrace();  
        }  
    }  
}  
```

定义`OfficeAble`为一个接口：

```java
interface OfficeAble{  
    public void start();  
}  
```
  
此时如果想使用`Word`类，只需要让`Word`类实现`OfficeAble`接口，定义如下：

```java
class Word implements OfficeAble{  
    public void start(){  
        System.out.println("word...start...");  
    }  
}  
```
  
## 4.通过反射认识泛型的本质

集合的泛型是为了防止错误输入，如：  
`ArrayList list1 = new ArrayList()`  
可以放入任何类型的数据，但是  
`ArrayList<String> list2 = new ArrayList<String>()`  
就只能放入字符串String类型的数据，但是**list1.class()与list2.class()相等**
，而反射的操作都是在编译之后的操作，这说明编译之后集合的泛型是**去泛型化的**
，因此Java中集合的泛型只在编译阶段有效，为了防止错误输入，绕过编译之后就无效了；  
可以**通过方法的反射来绕过编译** ：

```java
Class c = list2.getClass();  
try{  
    Method m = c.getMethod("add",Object.class);  
    c.invoke(list2,10);  //list2本来只能放字符串  
}catch(Exception e){  
    e.printStackTrace();  
}  
```
  
上面的例子中可以通过反射绕过泛型向list2中添加整数类型的值，但是for-each遍历会有类型转换错误；  
总之记住：反射、Class、Method和Field的操作都是绕过编译在运行时执行的。
