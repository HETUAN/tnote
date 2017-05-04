---
title: C# 对象的相等性与同一性
date: 2016-03-22 20:50
categories: CSharp 
tags: [CSharp]
description: CSharp 对象之间比较时的相等性与同一性
---

在C# 中两个对象进行比较时，无论是值类型之间的比较还是引用类型之间的比较，基本上都是用的 * == * 这个运算符进行运算，但是这种运算比较的仅仅是两个对象的指针是否相等而不会去比较对象中的每一个属性，当然两个变量的引用地址相同两者自然相等，但是如果两者的引用地址不同但是所有的属性值都相等这两者是否是相等的呢？所以，对于前者我们叫做 * 同一 * （同一性），而后者中的无论地址是否行等，只要对象中的属性都相等那么这叫做 * 相等 *（相等行）。 
在C#的Object这个基类中微软给我们封装了几个默认的虚方法，比如有Equals() 方法。用来比较两者是否相等，其实现方式如下：

``` cs
public class Object
{
    public virtual Boolean Equals(Object obj)
    {
        // 如果两个引用指向同一个值，他们肯定包含相同的值
        if(this == obj) return true;
        
        //假定对象不包含相同的值
        return false;
    }
}
``` 
在int,  string,float 等值类型中Equal会进行所包含的值的比较而不仅仅是比较引用是因为在值类型中 ValueType 继承了 Object 类型，而且重写了Object 类型中的Equals() 方法。

所以我们在定义对象并且要使用Equals时可以根据不同的需求进行方法的重写，重写后如果想进行同一性的比较我们可以使用Object类中提供的静态方法ReferenceEquals()进行比较其原型如下。
``` cs
public class Object{
    public static Boolean ReferenceEquals(Object objA, Object objB)
    {
        return (ojbA == objB);
    }
}
``` 

在重写Equals() 方法时我们可以参考以下过程：
1. 如果obj的参数为null就返回false，因为调用非静态的Euqlas() 方法时this所标志的当前变量显然不为null。
2. 如果this和obj实参引用同一个对象就返回true。
3. 如果this和obj引用不同 * 类型 * 的对象就返回false，一个string对象显然不等于FileStream。
4. 将类型中的每一个实例字段将this对象中的值与obj中的值进行比较（可以使用反射机制，反正在值类型的Equals() 方法重写中使用的是反射）任何字段不想等就返回false。
5. 除了字段之外还有本类的基类之间的比较，此时要使用基类中的Equals方法，如果基类的Equals()返回false就返回false否则返回true。