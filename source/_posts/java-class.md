---
title: Java的Class类
date: 2017-11-9 21:35:30
tags:
    - Java
    - Class
categories:
    - Java
copyright: true
---
# Class类是什么
通过`Class`类的文档介绍可以得知以下几点：
* `Class`类的实例代表了正在运行的Java应用的类与接口
* 枚举是一种类，注解是一种接口 <!--more-->
* 每个数组也都属于一个类，这个`Class`对象被所有具有相同元素类型和维数的数组共享
``` java
int a[] = new int[3];
int b[] = new int[4];
System.out.println(a.getClass() == b.getClass()); // true
```
* 原生Java类型(`boolean`, `byte`, `char`, `short`, `int`, `long`, `float`, and `double`)以及`void`同样是`Class`对象
* `Class`类没有公有构造函数，而是由JVM自动调用类加载器的`defineClass`方法

## 获得Class对象的方法
1. 直接使用类名+.class，如`String.class`
2. 运行时通过类名获得，需要包括包名的类全名，如果找不到类会抛出`ClassNotFoundException`
``` java
String className = "java.lang.String";
Class<?> clazz = Class.forName(className);
```
3. 使用`Object`的`getClass`方法

## 使用Class<?>
`Class`类是泛型类，使用时可以指定类型`T`，对于`Class`类大多数情况使用通配符`?`来表示接受任何类型。
`Class clazz`和`Class<?> clazz`在使用上没有区别，因为编译器会自动推测，但是显式写明泛型类型可以增强代码可读性。

Bruce Eckel, Thinking in Java:
> In Java SE5, Class<?> is preferred over plain Class, even though they are equivalent and the plain Class, as you saw, doesn’t produce a compiler warning. The benefit of Class<?> is that it indicates that you aren’t just using a non-specific class reference by accident, or out of ignorance. You chose the non-specific version.

# 常用方法
## 静态方法
``` java
static Class<?> forName(String className)
static Class<?> forName(String name, boolean initialize, ClassLoader loader)
```
通过类名获得类对象，可以使用给定的类加载器。数组的类名是`[L类名;`(注意有个分号)，原生类型为`[I`(int[]),`[B`(byte[]),`[Z`(boolean[]),`[C`(char[]),`[D`(double[]),`[F`(float[]),`[J`(long[]),`[S`(short[])，可以使用`getComponentType`方法获得元素类型。
## 公有方法
### 实例
``` java
public T newInstance()
              throws InstantiationException,
                     IllegalAccessException
```
创建一个类的实例对象，返回类型是泛型类型
``` java
public boolean isInstance(Object obj)
```
相当于关键字`instanceof`的动态实现
### 方法&构造器&字段
``` java
// Method
public Method getMethod(String name,
                        Class<?>... parameterTypes)
                 throws NoSuchMethodException,
                        SecurityException
public Method getDeclaredMethod(String name,
                                Class<?>... parameterTypes)
                         throws NoSuchMethodException,
                                SecurityException
public Method[] getMethods()
                    throws SecurityException
public Method[] getDeclaredMethods()
                            throws SecurityException
```
Method结尾的方法通过指定的方法名和参数列表反射得到方法，Methods直接得到类的所有方法。带有Declared的方法可以获取到所有四种访问类型的方法，但不包含继承的方法，不带Decleard的方法只可以获得类的公有方法，包含继承而来的方法。
同时，构造函数方法和字段也有着类似的API。
``` java
// Field
Field getField(String name)
Field getDeclaredField(String name)
Field[]	getFields()
Field[]	getDeclaredFields()
// Constructor
Constructor<T> getConstructor(Class<?>... parameterTypes)
Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)
Constructor<?>[] getConstructors()
Constructor<?>[] getDeclaredConstructors()
```
#### Method类
Method类封装了类方法的一系列信息，并且可以通过`invoke`方法调用。
#### Constructor类
Constructor类封装了类构造器的一系列信息，并且可以通过`newInstance`方法调用。
#### Field类
Field类封装了类字段的一系列信息，并且可以通过`get`和`set`方法获取/设置变量值。
### 注解
``` java
<A extends Annotation> A getAnnotation(Class<A> annotationClass)
<A extends Annotation> A getDeclaredAnnotation(Class<A> annotationClass)
Annotation[] getAnnotations()
Annotation[] getDeclaredAnnotations()
<A extends Annotation> A[] getAnnotationsByType(Class<A> annotationClass)
<A extends Annotation> A[] getDeclaredAnnotationsByType(Class<A> annotationClass)
```
返回类的注解，前四个方法与前面的get方法类似，标有Declared的方法可以得到继承而来的注解，以ByType结尾的方法是用来获得可重复注解的注解数组。
### Type接口
Type是Java中所有类型的通用父接口，包括raw types(原始类型，不含类型变量的泛型声明), parameterized types(参数化类型，相当于泛型), array types(数组类型), type variables(类型变量，指`class name<T1, T2, ..., Tn>`中的`T1`，`T2`等) and primitive types(原生类型)。
这个接口只有一个方法`getTypeName`。
### 父类&接口
``` java
Class<?>[]	getInterfaces()
Class<? super T> getSuperclass()
Type[] getGenericInterfaces()
Type getGenericSuperclass()
public AnnotatedType[] getAnnotatedInterfaces()
public AnnotatedType getAnnotatedSuperclass()
```
getGeneric系列方法返回类型为Type，一般为ParameterizedType，用于父类为泛型类的情况。
getAnnotated系列方法返回类型为AnnotatedType，用于继承父类时使用了注解的情况，通过`getAnnotatedSuperclass().getAnnotations()`获得注解数组。
# 实际使用
使用Java的反射机制可以在运行时动态的实现方法，使用`Proxy.newProxyInstance()`方法来创建动态代理，通过实现`InvocationHandler`接口，将对proxy的调用转移到handler上。
以一个No-Op Proxy作为例子
``` java
public class NoOpProxy<T> implements InvocationHandler {

    private WeakReference<T> obj;

    @SuppressWarnings("unchecked")
    public T bind(T obj) {
        this.obj = new WeakReference<>(obj);
        return (T) Proxy.newProxyInstance(obj.getClass().getClassLoader(),
                obj.getClass().getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (obj == null || obj.get() == null) {
            return null;
        }

        return method.invoke(obj.get(), args);
    }
}
```
将需要的对象存在`Proxy`里的`WeakReference`中，在调用方法之前由代理来判断对象是否已被回收，若没有被回收就调用相应方法，这样在使用`WeakReference`时就不需要不断在代码里写null check，当然反射的使用会使效率降低。
# 参考
1. [Class (Java Platform SE 8 )](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)
2. [Java Reflection - Annotations](http://tutorials.jenkov.com/java-reflection/annotations.html)
3. [Chapter 4. Types, Values, and Variables/JLS 4.8 Raw Types](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.8)