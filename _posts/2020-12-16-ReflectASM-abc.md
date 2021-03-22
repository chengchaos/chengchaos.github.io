---
title: ReflectASM 介绍
key: 2020-12-29
tags: ReflectASM
---


ReflectASM 是一个很小的 Java 库，它通过字节码生成的方式实现了读写字段、调用方法或构造实例等反射操作。由于 ReflectASM 是通过生成字节码的方式而非依赖于 Java 的反射，所以它的执行速度会更快，并且这种方式在操作基本类型的字段时可以避免装箱操作。

<!--more-->

## ReflectASM 的应用

下面通过几个简单的例子演示如何使用 ReflectASM。首先在使用 ReflectASM 的 API 之前，需要引入它的依赖：

```java
  com.esotericsoftware
  reflectasm
  1.11.9
```

maven

```xml
        <dependency>
            <groupId>com.esotericsoftware</groupId>
            <artifactId>reflectasm</artifactId>
            <version>1.11.9</version>
        </dependency>
```

## 读写字段

通过 FieldAccess 可以读写类的字段：

```java
SomeClass someObject = // ...
FieldAccess access = FieldAccess.get(SomeClass.class);
access.set(someObject, "name", "Huey");
String name = (String) access.get(someObject, "name");
assertEquals(name, "Huey");
```

## 调用方法

通过 MethodAccess 可以调用对象实例的方法：

```java
SomeClass someObject = // ...
MethodAccess access = MethodAccess.get(SomeClass.class);
access.invoke(someObject, "setName", "Huey");
String name = (String) access.invoke(someObject, "getName");
assertEquals(name, "Huey");
```

## 构造实例

通过 ConstructorAccess 可以构造对象实例：

```java
ConstructorAccess access = ConstructorAccess.get(SomeClass.class);
SomeClass someObject = access.newInstance();
// do something with someObject.
```

## 索引

相比通过名称来访问成员，使用索引的方式会更快。如果需要重复地访问同一个成员，那么通过索引来访问该成员效率更高：

```java
String[] names = // ...
SomeClass someObject = // ...
MethodAccess access = MethodAccess.get(SomeClass.class);
int addNameIndex = access.getIndex("addName");
for (String name : names) {
    access.invoke(someObject, addNameIndex, name);
}
```

## 遍历字段

遍历一个类的所有字段：

```java
FieldAccess access = FieldAccess.get(SomeClass.class);
for(int i = 0, n = access.getFieldCount(); i < n; i++) {
    access.set(instanceObject, i, valueToPut);              
}
```

## 复制类

应用中用到从一个类到另外一个类的copy使用ReflectASM性能会好很多，具体的方式如下：
其中对于耗时的get操作增加了缓存，代理类生成一次就够了

```java
public class ReflectAsmManager {
    private static final ConcurrentMap<Class, MethodAccess> localCache = Maps.newConcurrentMap();

    public static MethodAccess get(Class clazz) {
        if(localCache.containsKey(clazz)) {
            return localCache.get(clazz);
        }

        MethodAccess methodAccess = MethodAccess.get(clazz);
        localCache.putIfAbsent(clazz, methodAccess);
        return methodAccess;
    }

    public static <F,T> void copyProperties(F from, T to) {
        MethodAccess fromMethodAccess = get(from.getClass());
        MethodAccess toMethodAccess = get(to.getClass());
        Field[] fromDeclaredFields = from.getClass().getDeclaredFields();
        for(Field field : fromDeclaredFields) {
            String name = field.getName();
            try {
                Object value = fromMethodAccess.invoke(from,  "get" + StringUtils.capitalize(name), null);
                toMethodAccess.invoke(to, "set" + StringUtils.capitalize(name), value);
            } catch (Exception e) {
                // 设置异常，可能会没有对应字段，忽略
            }
        }

    }
}
```

## 访问控制权限

需要注意的是，对于任何的 public 成员，ReflectASM 都可以访问；在同一个类加载器，ReflectASM 也可以访问protected 或不加修饰符的成员；ReflectASM 无法访问 private 成员。

参考：

- [ReflectASM 初体验：高性能的 Java 反射类库](https://www.5axxw.com/wenku/qh/1070195j.html)
- [ReflectASM详解](https://www.jianshu.com/p/ca7bdf8b7718)
EOF

---

Power by TeXt.
