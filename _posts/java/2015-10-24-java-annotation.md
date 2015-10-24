---
layout: post
title: java Annotation
category: java
tags: java
---

# 1.什么是java标注

注解相当于一种标记，在程序中加了注解就等于为程序打上了某种标记，没加，则等于没有某种标记，以后，javac编译器，开发工具和其他程序可以用反射来了解你的类及各种元素上有无何种标记，看你有什么标记，就去干相应的事。标记可以加在包，类，字段，方法，方法的参数以及局部变量上。

# 2.自定义注解及其应用

自定义注解及使用的步骤为:

1. 定义一个最简单的注解

	```
	public @interface MyAnnotation {
	    //......
	}
	```

2. 把注解加在某个类上：

	```
	@MyAnnotation
	public class AnnotationTest{
	    //......
	}
	```
	
下面是一个完整的案例：<br/>先定义一个简单的注解，将在其他自定义注解中使用之：

```
package com.company;

/**
 * 定义一个注解，模拟注解中添加注解属性
 *
 * @author millions
 *
 */
public @interface MetaAnnotation {
    String birthday();
}
```

然后定义一个枚举：

```
package com.company;

/**
 * Created by millions on 15/10/24.
 *
 */
public enum Gender {
    MAN(1){
        @Override
        public String getExpressValue() {
            return "男人";
        }
    },WOMEN(2) {
        @Override
        public String getExpressValue() {
            return "女人";
        }
    };

    private final int value;

    private Gender(int value) {
        this.value = value;
    }

    public static Gender valueOf(int value) {
        switch (value) {
            case 1:
                return MAN;
            default:
                return null;
        }
    }

    public int value() {
        return this.value;
    }

    abstract public String getExpressValue();

}

```

<br/>
>这里插播一句关于枚举的认识，由于之前是搞c/c++的，对枚举的认识就是简单的一种值的集合，来增强代码的可读性。然而java中的枚举似乎更复杂。我觉得java的枚举可以认为是一种特殊的class，enum中的一个个枚举值其实就是一个一个的static内部类，在上面这个gender内中，首先定义了field `int value`  这样就可以以类似`enum Gender{MAN=1,WOMEN=2}`的方式来定义若干个枚举值。c++中使用枚举时枚举可以当做int，而java中需要定义一个类似于value之类的值，去返回成员变量的值，同时还需要有一个valueOf一类的函数从int获取相应的枚举。最后，java中的枚举还可以定义一些抽象的方法，在一个一个static的内部类中实现这些方法。

然后我们定义一个比较复杂的注解，使用了之前只有一个birthday属性的简单注解

```
package com.company;


import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;


//Java中提供了四种元注解，专门负责注解其他的注解，分别如下

//@Retention元注解，表示需要在什么级别保存该注释信息（生命周期）。可选的RetentionPoicy参数包括：
//RetentionPolicy.SOURCE: 停留在java源文件，编译器被丢掉
//RetentionPolicy.CLASS：停留在class文件中，但会被VM丢弃（默认）
//RetentionPolicy.RUNTIME：内存中的字节码，VM将在运行时也保留注解，因此可以通过反射机制读取注解的信息

//@Target元注解，默认值为任何元素，表示该注解用于什么地方。可用的ElementType参数包括
//ElementType.CONSTRUCTOR: 构造器声明
//ElementType.FIELD: 成员变量、对象、属性（包括enum实例）
//ElementType.LOCAL_VARIABLE: 局部变量声明
//ElementType.METHOD: 方法声明
//ElementType.PACKAGE: 包声明
//ElementType.PARAMETER: 参数声明
//ElementType.TYPE: 类、接口（包括注解类型)或enum声明

//@Documented将注解包含在JavaDoc中

//@Inheried允许子类继承父类中的注解


@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface MyAnnotation {
    //为注解添加属性
    String color();
    String value() default "我是陈喆"; //为属性提供默认值
    int[] array() default {1, 2, 3};
    Gender gender() default Gender.MAN; //添加一个枚举
    MetaAnnotation metaAnnotation() default @MetaAnnotation(birthday="我的出身日期为1989-08-17");
}
```


最后我们来测试一下这个注解：

```java
package com.company;


//调用注解并赋值
@MyAnnotation(metaAnnotation=@MetaAnnotation(birthday = "我的出身日期为1988-2-18"),color="red", array={23, 26})
public class Main {

    public static void main(String[] args) {
        //检查类AnnotationTest是否含有@MyAnnotation注解
        if(Main.class.isAnnotationPresent(MyAnnotation.class)){
            //若存在就获取注解
            MyAnnotation annotation=(MyAnnotation)Main.class.getAnnotation(MyAnnotation.class);
            System.out.println(annotation);
            //获取注解属性
            System.out.println(annotation.color());
            System.out.println(annotation.value());
            //数组
            int[] arrs=annotation.array();
            for(int arr:arrs){
                System.out.println(arr);
            }
            //枚举
            Gender gender=annotation.gender();
            System.out.println("性别为："+gender);
            System.out.println("中文名"+gender.getExpressValue());
            System.out.println("value为"+gender.value());

            //获取注解属性
            MetaAnnotation meta=annotation.metaAnnotation();
            System.out.println(meta.birthday());
        }
    }
}
```

这样就可以凭借反射获取注解的相关信息了 <br/>
输出的结果:

```
@com.company.MyAnnotation(gender=MAN, metaAnnotation=@com.company.MetaAnnotation(birthday=我的出身日期为1988-2-18), value=我是陈喆, array=[23, 26], color=red)
red
我是陈喆
23
26
性别为：MAN
中文名男人
value为1
我的出身日期为1988-2-18
```