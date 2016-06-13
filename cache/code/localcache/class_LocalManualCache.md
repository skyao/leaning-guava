类LocalManualCache
==================

类LocalManualCache是类LocalCache的内部类。

## 类定义

```java
class LocalCache {
      static class LocalManualCache<K, V> implements Cache<K, V>, Serializable {}
}
```

> 注： LocalManualCache直接实现了Cache，而不是从AbstractCache继承，有些奇怪： 那AbstractCache写来给谁用呢？

## 构造函数

```java
LocalLoadingCache(CacheBuilder<? super K, ? super V> builder,
    CacheLoader<? super K, V> loader) {
  super(new LocalCache<K, V>(builder, checkNotNull(loader)));
}
```

## 方法实现

### 简单委托给localcache的方法

以下方法都是将实现简单委托给localcache的同名方法：

- getIfPresent()
- getAllPresent()
- put()
- putAll()
- invalidateAll
- cleanUp()

代码典型如下：

```java
@Override
@Nullable
public V getIfPresent(Object key) {
	// 直接委托给 localCache 的 getIfPresent()方法。
	return localCache.getIfPresent(key);
}
```

### 稍作变化再委托给localcache的方法

以下方法是稍作变化再委托给localcache的方法，有些只是方法名不同而已：

- get()
- invalidate()
- invalidateAll()

具体如下：

```java
@Override
public V get(K key, final Callable<? extends V> valueLoader) throws ExecutionException {
  checkNotNull(valueLoader);
  // 委托给localCache的get(key, CacheLoader)方法
  return localCache.get(key, new CacheLoader<Object, V>() {
    @Override
    public V load(Object key) throws Exception {
      // 通过一个内部匿名类将Callable转为CacheLoader
      return valueLoader.call();
    }
  });
}

@Override
public void invalidate(Object key) {
  checkNotNull(key);
  //委托给localCache的remove方法
  localCache.remove(key);
}

@Override
public void invalidateAll() {
  localCache.clear();
}

public long size() {
  return localCache.longSize();
}
```

### asMap()方法

```java
@Override
public ConcurrentMap<K, V> asMap() {
  // 直接把localcache这个map返回了
  return localCache;
}
```

### stats()方法

```java
@Override
public CacheStats stats() {
  SimpleStatsCounter aggregator = new SimpleStatsCounter();
  // 先加上localCache的globalStatsCounter
  aggregator.incrementBy(localCache.globalStatsCounter);
  for (Segment<K, V> segment : localCache.segments) {
    // 再将各个segment的statsCounter累加起来
    aggregator.incrementBy(segment.statsCounter);
  }
  //最后得到当前的快照
  return aggregator.snapshot();
}
```

### writeReplace()方法

```java
Object writeReplace() {
  return new ManualSerializationProxy<K, V>(localCache);
}
```

TBD：再细看。
