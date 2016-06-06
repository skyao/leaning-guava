LongAddable实现
==============

## 接口LongAddable

抽象接口，容许并发的添加long。

```java
package com.google.common.cache;

interface LongAddable {
    void increment();
    void add(long x);
    long sum();
}
```

只定了几个简单方法。

## LongAddables工厂方法类

LongAddables类提供工厂方法create()来创建一个LongAddable实现类：

```java
public static LongAddable create() {
	return SUPPLIER.get();
}
```

SUPPLIER的实现很特别：

```java
private static final Supplier<LongAddable> SUPPLIER;

static {
    Supplier<LongAddable> supplier;
    try {
      //尝试初始化类 LongAdder
      //这里有可能失败
      new LongAdder(); // trigger static initialization of the LongAdder class, which may fail
      //如果 LongAdder 初始化成功，就用LongAdder的方案
      supplier = new Supplier<LongAddable>() {
        @Override
        public LongAddable get() {
          return new LongAdder();
        }
      };
    } catch (Throwable t) { // we really want to catch *everything*
      //如果失败了就使用纯java的方案
      supplier = new Supplier<LongAddable>() {
        @Override
        public LongAddable get() {
          return new PureJavaLongAddable();
        }
      };
    }
    SUPPLIER = supplier;
}
```

### 类PureJavaLongAddable

纯java的方案，基于AtomicLong：

```java
    private static final class PureJavaLongAddable extends AtomicLong implements LongAddable {
    @Override
    public void increment() {
      getAndIncrement();
    }
    ......
}
```

### 类LongAdder

```java
final class LongAdder extends Striped64 implements Serializable, LongAddable {}
```

按照LongAdder javadoc的说明：

一个或者多个变量一起组成一个初始化为0的 long 总数。当跨线程竞争更新(方法add())时，多个变量的集合可以动态的增长来减少竞争。方法 sum() (等级于 longValue)返回总数， 合并用于组成sum的变量。

这个类通常比 AtomicLong 更适合多线程更新总数，用于收集统计的总数，而不适合细粒度的同步控制。在低更新竞争下，两个类有类似的表现，但是在高竞争下，这个类的吞吐量有很大的提高，代价是更多的空间消耗。

> 注意：jsr166e， 这个类预计将被纳入java.util.concurrent.atomic

