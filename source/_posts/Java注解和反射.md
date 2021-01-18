---
title: Java注解和反射
date: 2021-01-15 21:52:40
tags:
---

## 注解

注解`annotation`：JDK5.0引入

注释`comment`

## 内置注解

```
@Override 重写
@deprecated 不推荐使用
@SuppressWarnings 抑制警告
```

## 元注解

元注解：负责注解其他注解的注解，Java定义了4个次奥准的meta-annotation类型，用来提供对注解的说明。

```
@Target 表示注解的使用范围
@Retention 表示需要使用什么级别的注解 source < class < runtime
@Document 说明该注解被包含在javadoc中
@inherited 说明子类可以继承父类的注解
```

@Target

```
package java.lang.annotation;

public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

## 自定义注解

```
package annotation;

import java.lang.annotation.*;

/**
 * @author luu
 */
@Target(value = {ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
@Retention(value = RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface MyAnnotation {

    String name() default "";
    
    int age() default -1;
    
    String[] address();

}
```

如果注解中只有一个值并命名为value，可以在使用注解的时候省略value，直接写赋值的内容。

## 反射机制

`reflection`反射：允许程序在执行期间接住Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。

### 获取Class的几种方法

```
package reflection;

/**
 * @author: luu
 * @date: 2021-01-15 23:30
 **/
public class GetClass {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> clazz1 = Class.forName("reflection.GetClass");
        Class clazz2 = GetClass.class;
        Class clazz3 = new GetClass().getClass();
        Class clazz4 = Integer.TYPE;

        System.out.println(clazz1.hashCode());
        System.out.println(clazz2.hashCode());
        System.out.println(clazz3.hashCode());
        System.out.println(clazz4);
    }
}
```

## 所有类型的Class

```
package reflection;

import java.lang.annotation.ElementType;
import java.util.LinkedList;
import java.util.List;

/**
 * @author: luu
 * @date: 2021-01-15 23:46
 **/
public class AllClassType {

    public static void main(String[] args) {
        List<Class<?>> classes = new LinkedList<>();
        classes.add(Object.class);
        classes.add(Runnable.class);
        classes.add(String[].class);
        classes.add(int[][].class);
        classes.add(Override.class);
        classes.add(ElementType.class);
        classes.add(void.class);
        classes.add(Integer.class);
        classes.add(int.class);
        classes.add(Class.class);

        for (Class<?> aClass : classes) {
            System.out.println(aClass);
        }
    }

}

// output:
class java.lang.Object
interface java.lang.Runnable
class [Ljava.lang.String;
class [[I
interface java.lang.Override
class java.lang.annotation.ElementType
void
class java.lang.Integer
int
class java.lang.Class
```

## 类加载内存分析

Java内存

+ 堆
+ 栈
+ 方法区： 包含了class和static变量
+ 静态方法去
+ 程序计数器

> 类加载过程：加载、链接、初始化
>
> + 加载：将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后生成一个代码这个类的java.lang.Class对象。
> + 链接：将Java类的二进制代码合并到JVM的运行状态之中的过程。
>   + 验证：确保加载的类信息符合JVM规范，没有安全问题。
>   + 准备：正式为类变量分配内存并设置类变量的默认初始值，这些内存将在方法区进行分配。
>   + 解析：逊尼基常量池内的符号引用替换为直接引用的过程。
> + 初始化：
>   + 执行类构造器方法。
>   + 当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。
>   + 虚拟机报账一个类的方法在多线程环境中被正确的加锁和同步。

## 类加载器

+ `BootstrapClassloader`根类加载器，JVM自带类加载器，负责加载Java平台核心库类，无法直接获取。

+ `ExtensionClassloader`扩展类加载器，负责jre/lib/ext目录下的jar包或-D java.ext.dirs知道目录下的jar包装入工作库

+ `AppClassLoader`系统类就加载器，负责java -classpath或

  -D java.class.path所指定的目录下的类与jar包装入工作，最常见的类加载器。

+ 自定义类加载器

```
package reflection;

/**
 * @author: luu
 * @date: 2021-01-16 19:27
 **/
public class GetClassloader {

    public static void main(String[] args) {
        ClassLoader systemClassLoader = GetClassloader.class.getClassLoader();
        System.out.println(systemClassLoader);

        ClassLoader parent = systemClassLoader.getParent();
        System.out.println(parent);

        ClassLoader bootStrapClassloader = parent.getParent();
        System.out.println(bootStrapClassloader);
    }

}
```

## 获取类的信息

```
package reflection;

import reflection.bean.Person;

/**
 * @author: luu
 * @date: 2021-01-16 20:15
 **/
public class GetClassInfo {

    public static void main(String[] args) {
        Class<Person> clazz = Person.class;
        System.out.println(clazz.getName());
        System.out.println(clazz.getCanonicalName());
        System.out.println(clazz.getSimpleName());
        System.out.println(clazz.getClassLoader());
        System.out.println(clazz.getInterfaces());
        System.out.println(clazz.getDeclaredAnnotations());
        System.out.println(clazz.getConstructors());
        System.out.println(clazz.getDeclaredAnnotations());
        System.out.println(clazz.getFields());
        System.out.println(clazz.getDeclaredFields());
        System.out.println(clazz.getMethods());
        System.out.println(clazz.getDeclaredMethods());
    }

}
```

## 通过反射动态创建对象

```
package reflection;

import reflection.bean.Person;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Date;

/**
 * @author: luu
 * @date: 2021-01-16 20:48
 **/
public class GetInstance {

    public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {

        Class clazz = Person.class;
        // 通用newInstance方法
        Person person1 = (Person) clazz.newInstance();
        System.out.println(person1);
        // 通过构造方法
        Constructor personConstructor = clazz.getConstructor(Long.TYPE, String.class, Integer.TYPE, Date.class, Date.class);
        Person person2 = (Person) personConstructor.newInstance(1L, "luu", 30, new Date(), new Date());
        System.out.println(person2);
        // 修改字段
        Field name = clazz.getDeclaredField("name");
        name.setAccessible(true);
        name.set(person2, "luu -> lu");
        name.setAccessible(false);
        System.out.println(person2);
        // 调用方法
        Method setName = clazz.getMethod("setName", String.class);
        setName.invoke(person2, "lu -> luu");
        System.out.println(person2);
    }

}

```

## 反射操作泛型

为了通过反射操作这些类，Java新增了`ParameterizedType` `GenericArrayType` `TypeVariable` `wildcardType`集中类型来代表不能被归一到Class类中的类型但是又和原始类型齐名的类型。

+ `ParameterizedType`：表示一种参数化类型，比如`Collection<String>`
+ `GenericArrayType`：表示一种元素类型是参数化类型或者类型变量的数组类型
+ `TypeVariable`：是各种类型变量的公共父接口
+ `wildcardType`：代表一种通配符类型表达式

```
package reflection;

import reflection.bean.Person;

import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.Arrays;
import java.util.List;
import java.util.Map;

/**
 * @author: luu
 * @date: 2021-01-16 21:16
 **/
public class GetGenericsByReflection<T> {

    public void getInfo(T t) {
        System.out.println(t);
    }

    public void genericsParams(Map<String, Person> map, List<Person> list) {

    }

    public static void main(String[] args) throws NoSuchMethodException {
        Class clazz = GetGenericsByReflection.class;
        Method method = clazz.getMethod("genericsParams", Map.class, List.class);
        Type[] genericParameterTypes = method.getGenericParameterTypes();
        for (Type genericParameterType : genericParameterTypes) {
            if (genericParameterType instanceof ParameterizedType) {
                Type[] actualTypeArguments = ((ParameterizedType) genericParameterType).getActualTypeArguments();
                for (Type actualTypeArgument : actualTypeArguments) {
                    System.out.println(actualTypeArgument);
                }
            }
        }
    }

}
```

并不能动态获取到运行时的类型，还是需要手动传入Class类来为运行时提供类型信息。只能够获取到编译期间的泛型信息。

```
import java.util.Arrays;

/**
 * @author: luu
 * @date: 2021-01-16 21:16
 **/
public class GetGenericsByReflection<T> {

    public void getInfo(T t) {
        System.out.println(t);
    }

    public static void main(String[] args) throws NoSuchMethodException {
        GetGenericsByReflection<String> stringGetGenericsByReflection = new GetGenericsByReflection<>();
        Class clazz = stringGetGenericsByReflection.getClass();
        Method getInfo = clazz.getMethod("getInfo", Object.class);
        Type[] genericParameterTypes = getInfo.getGenericParameterTypes();
        System.out.println(Arrays.toString(genericParameterTypes));
    }

}
```

## 反射操作注解

```
package reflection;

import java.lang.annotation.*;
import java.lang.reflect.Field;

/**
 * @author: luu
 * @date: 2021-01-16 22:10
 **/
public class GetAnnotationByReflection {

    public static void main(String[] args) throws IllegalAccessException {
        Person person = new Person();
        Person person1 = GetAnnotationByReflection.annotationHandler(person, Person.class);
        System.out.println(person1);
    }

    public static <T> T annotationHandler(T t, Class<T> tClass) throws IllegalAccessException {
        ClazzValue clazzValue = tClass.getAnnotation(ClazzValue.class);
        String instanceName = clazzValue.value();
        System.out.println("handler " + instanceName + " now");
        Field[] fields = tClass.getDeclaredFields();
        for (Field field : fields) {
            if (field.isAnnotationPresent(ClazzField.class)) {
                ClazzField fieldAnnotation = field.getAnnotation(ClazzField.class);
                String value = fieldAnnotation.value();
                field.setAccessible(true);
                field.set(t, value);
                field.setAccessible(false);
            }
        }
        return t;
    }

}

@ClazzValue("person")
class Person {
    @ClazzField(value = "luu")
    private String name;
    @ClazzField(value = "30")
    private String age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

/**
 * @author luu
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@interface ClazzValue {
    String value();
}

/**
 * @author luu
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@interface ClazzField {
    String value();
}
```

