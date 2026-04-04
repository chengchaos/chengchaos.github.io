---
title: ReflectASM 用法
key: 2020-11-23
tags: java reflectasm
---

ReflectASM 是一个很小的 java 类库，它仅仅有 5 个类，但是却提供了非常高性能的属性操作、方法调用、构造方法调用，它在底层使用了 asm [https://www.ibm.com/developerworks/cn/java/j-lo-asm30/index.htm](https://www.ibm.com/developerworks/cn/java/j-lo-asm30/index.html) 动态构建出字节码，这相比于反射，直接方法的调用性能高出很多。

<!--more-->

## 使用

pom.xml

```xml
        <!-- https://mvnrepository.com/artifact/com.esotericsoftware/reflectasm -->
        <dependency>
            <groupId>com.esotericsoftware</groupId>
            <artifactId>reflectasm</artifactId>
            <version>1.11.9</version>
        </dependency>
```

以 MethodAccess 为例。

```java

import com.esotericsoftware.reflectasm.MethodAccess;
import org.junit.Test;

public class ReflectasmTest {

    static class SomeClass {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }

    @Test
    public void methodAccessTest() {

        SomeClass someObject = new SomeClass();
        MethodAccess methodAccess = MethodAccess.get(SomeClass.class);
        methodAccess.invoke(someObject, "setName", "abc");

        String name = someObject.getName();
        System.out.println("name => "+ name);
    }
}
```

通过 `get` 方法得到某个类的加强类，直接调用类上的 `setName` 完成方法的调用，主要的过程便是在 `get` 方法中，它通过 asm 生成 SomeClass 的代理类，实现了 MethodAccess 的 `invoke` 方法，方法的内容是生成 SomeClass 的所有方法的调用 index，这样可以通过指定方法名称的方式调用类上的方法。直接调用类上方法的速度肯定要快于反射调用了。

## 复制类

应用中用到从一个类到另外一个类的 copy 使用 ReflectASM 性能会好很多，具体的方式如下：

其中对于耗时的 get 操作增加了缓存，代理类生成一次就够了。

```java
import com.esotericsoftware.reflectasm.MethodAccess;
import com.google.common.collect.Maps;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Arrays;
import java.util.concurrent.ConcurrentMap;

public class ReflectAsmManager {
    private static final Logger logger = LoggerFactory.getLogger(ReflectAsmManager.class);
    private static final ConcurrentMap<Class<?>, MethodAccess> localCache = Maps.newConcurrentMap();

    public static MethodAccess get(Class<?> clazz) {
        if (localCache.containsKey(clazz)) {
            return localCache.get(clazz);
        }

        MethodAccess methodAccess = MethodAccess.get(clazz);
        localCache.putIfAbsent(clazz, methodAccess);
        return methodAccess;
    }

    /**
     *
     * @param from F
     * @param to T
     * @param <F> From type
     * @param <T> To type
     */
    public static <F, T> void copyProperties(final F from, final T to) {

        Class<?> fromClass = from.getClass();
        final MethodAccess fma = get(fromClass);
        final MethodAccess tma = get(to.getClass());
        Arrays.stream(fromClass.getDeclaredFields())
                .forEach(field -> {
                    String capitalizeName = StringUtils.capitalize(field.getName());
                    try {
                        Object value = fma.invoke(from, "get" + capitalizeName, null);
                        tma.invoke(to, "set" + capitalizeName, value);
                    } catch (Exception e) {
                        logger.warn("ex => ", e);
                    }
                });

    }
}

```

参考：

- [ReflectASM详解](https://www.jianshu.com/p/ca7bdf8b7718)

EOF

---

Power by TeXt.
