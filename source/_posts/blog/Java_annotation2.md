---
title: "学习笔记：Java中的注解与反射(二)"

date: 2020-07-09T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Java"]
author: "Qingfeng Zhang"

---

> 之前招银网络的笔试考了动态代理和反射，然后最近的一面中又问了反射，虽然之前看过，但是忘得一干二净  
> 那道笔试题中是在动态代理中使用反射进行验证，记得不是很清楚，没做出来，还是要多写写代码。

# 自定义注解的使用

* 自定义一个注解@MyAnnotation，其中有一个String类型的参数；

```java
import java.lang.annotation.*;  
    
@Target({ElementType.METHOD})  //注解的作用目标  
@Retention(RetentionPolicy.RUNTIME)  //注解的保留策略  
@Inherited  //注解可以被继承  
@Documented  //注解包含在javadoc中  
public @interface MyAnnotation{  
    String value();  
}  
```
  
* 就不用简单的数据类型了，定义一个Person类作为数据；

```java
    public class Person {  
      
        private String name;  
        private int age;  
        private String city;  
        private int gender;  
      
        //getter、setter和toString方法  
    }  
```
  
* 假设在类UseAnnotation中的方法getPersonInfo上使用了这个自定义的注解；

```java    
    public class UseAnnotation {  
        private String test = "xsxa";  
      
        @MyAnnotation("101")  
        public void getPersonInfo(Person person){  
            System.out.println(person.toString());  
        }  
    }  
```
  
现在就是我定义了一个注解，并且把这个注解加在了我写的方法上，但是现在加这个注解并没有什么影响，注解并没有起作用，前面的笔记也说过**注解本身只是起到标记的作用，注解需要反射机制，根据注解的标记去调用注解解释器执行行为**

* 接下来就使用反射来根据的注解参数进行验证，并调用getPersonInfo方法；

```java    
import java.lang.annotation.Annotation;  
import java.lang.reflect.Field;  
import java.lang.reflect.Method;  
    
public class ReflectCallAnnotation {  
    public static void AnnotationMethod(String an,Person person) throws IllegalAccessException, InstantiationException {  
        Class use = UseAnnotation.class;  //获取使用了注解@MyAnnotation的类UseAnnotation的信息  
    
        Field[] fields = use.getDeclaredFields();  //获取UseAnnotatin类的所有属性  
        for(Field f: fields){  
            System.out.println(f.getName());  
        }  
    
        Method[] methods = use.getMethods();  //获取UseAnnotation的所有方法  
        for(Method m: methods){  
            Annotation[] annotations = m.getDeclaredAnnotations();  //获取方法的注解  
            for(Annotation a: annotations){  //遍历方法的所有注解  
                if(a instanceof MyAnnotation){  //找出自定义的注解@MyAnnotation  
                    String s = ((MyAnnotation)a).value();  //获得使用@MyAnnotation注解时传入的参数  
                    if(s.equals(an)){  //判断MyAnnotation的参数是否与传入参数an相等  
                        //到此可以确定方法m是使用了注解@MyAnnotation且与传入的参数相等  
                        try{  
                            //使用UseAnnotation实例对象调用方法m，参数是person  
                            m.invoke(use.newInstance(),person);  
                        }catch(Exception e){  
                            e.printStackTrace();  
                        }  
                    }  
                }  
            }  
        }  
    }  
}  
```
  
* 上面的代码的流程是：

1. 通过反射获取类的所有方法；
2. 在这些方法中找到使用了注解@MyAnnotation的方法；
3. 验证注解@MyAnnotation的参数和AnnotationMethod方法传入的参数是否相同；
4. 通过验证之后，使用反射执行这个使用了@MyAnnotation的方法；

* 定义好这个反射执行目标方法的方法AnnotationMethod后，编写一个测试方法来使用注解：

```java    
public class Main {  
    public static void main(String[] args) throws InstantiationException, IllegalAccessException {  
        Person person = new Person();  
        person.setName("zhang");  
        person.setAge(24);  
        person.setCity("shanghai");  
        person.setGender(1);  
        /**  
         * 通过AnnotationMethod反射执行UseAnnotation类中标识了注解@MyAnnotation的类，并验证参数  
         */  
        ReflectCallAnnotation.AnnotationMethod("101",person);  
    }  
}  
```
  
最终AnnotationMethod方法使用这个注解的参数作为验证，如果使用注解的时候参数是`"101"`，传入AnnotationMethod的参数也是`"101"`才会执行getPersonInfo方法输出person对象的信息，否则不执行；

> 所以注解还只是一个标记，需要注解实现怎样的功能是看解析注解的反射方法如何实现；
