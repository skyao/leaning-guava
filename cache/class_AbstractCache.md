类AbstractCache和AbstractLoadingCache
===============

# 类AbstractCache

AbstractCache提供一个 Cache 接口的骨架实现来减少实现这个接口需要的工作量。

为了实现cache，程序员仅仅需要继承这个类并提供 put() 和 getIfPresent() 方法的实现，其他的方法将基于他们实现:

- getAllPresent() 基于 getIfPresent()
- putAll() 基于 put()
- invalidateAll() 基于 invalidate()
- cleanUp()是空操作

所有其他方法都抛出 UnsupportedOperationException。

## 类定义

```java
public abstract class AbstractCache<K, V> implements Cache<K, V> {}
```

## 实现了的方法

### getAllPresent()

```java
public ImmutableMap<K, V> getAllPresent(Iterable<?> keys) {
    Map<K, V> result = Maps.newLinkedHashMap();
    //游历传入的所有的key
    for (Object key : keys) {
      //如果结果集合中不包含这个key
      //相当于在这里做了key的去重
      if (!result.containsKey(key)) {
        @SuppressWarnings("unchecked")
        K castKey = (K) key;
        //调用getIfPresent()方法获取单个key的value
        V value = getIfPresent(key);
        //如果value存在
        if (value != null) {
          //加入结果集合
          result.put(castKey, value);
        }
      }
    }
    //最后返回结果的不可变形式
    return ImmutableMap.copyOf(result);
}
```

### putAll()

```java
public void putAll(Map<? extends K, ? extends V> m){
	for (Map.Entry<? extends K, ? extends V> entry : m.entrySet()) {
    	//简单游历+逐个put
		put(entry.getKey(), entry.getValue());
	}
}
```

## 内部类

### StatsCounter接口和SimpleStatsCounter类

用来cache的实现计数器，以便返回Cache接口要求的CacheStatus。

```java
public static final class SimpleStatsCounter implements StatsCounter {
    private final LongAddable hitCount = LongAddables.create();
	......
}
```

实现很简单，定义了若干个record方法，然后各个计数器响应更新。

比较有意思的是计数器的实现，不是long + 同步或者AtomicLong，而是guava自己的LongAddable。

> 注： LongAddable的实现后面章节。

# 类AbstractLoadingCache

```java
public abstract class AbstractLoadingCache<K, V>
    extends AbstractCache<K, V> implements LoadingCache<K, V> {
}
```

继承自AbstractLoadingCache，并实现LoadingCache接口。

## 类方法

### getUnchecked()

```java
public V getUnchecked(K key) {
    try {
      //调用get()方法
      return get(key);
    } catch (ExecutionException e) {
      //如果get()方法抛出受查异常 ExecutionException
      //则抛弃这个 ExecutionException， 替换成UncheckedExecutionException
      throw new UncheckedExecutionException(e.getCause());
    }
}
```

### getUnchecked()

```java
public final V apply(K key) {
	return getUnchecked(key);
}
```

直接转给getUnchecked()方法了。

