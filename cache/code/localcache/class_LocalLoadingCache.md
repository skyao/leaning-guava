类LocalLoadingCache
===================

类LocalLoadingCache是类LocalCache的内部类。

## 类定义

```java
class LocalCache {
    static class LocalLoadingCache<K, V> extends LocalManualCache<K, V> implements LoadingCache<K, V> {
    }
}
```

> 注： LocalLoadingCachejicheng继承LocalManualCache，实现LoadingCache，又没有用到AbstractLoadingCache继承，还是那个问题： 那AbstractLoadingCache写来给谁用呢？

## 构造函数和属性

```java
LocalLoadingCache(CacheBuilder<? super K, ? super V> builder,
    CacheLoader<? super K, V> loader) {
  super(new LocalCache<K, V>(builder, checkNotNull(loader)));
}
```

### 简单委托给localcache的方法

以下方法都是将实现简单委托给localcache的同名方法：

- refresh()
- getAll()

代码典型如下：

```java
@Override
public void refresh(K key) {
  localCache.refresh(key);
}
```

### 稍作变化再委托给localcache的方法

以下方法是稍作变化再委托给localcache的方法，有些只是方法名不同而已：

- get()


具体如下：

```java
@Override
public V get(K key) throws ExecutionException {
  return localCache.getOrLoad(key);
}

@Override
public final V apply(K key) {
  return getUnchecked(key);
}
```

### getUnchecked()方法

```java
@Override
public V getUnchecked(K key) {
  try {
  	// 调用get()方法
    return get(key);
  } catch (ExecutionException e) {
  	// 如果抛出受查异常，就包装为 UncheckedExecutionException 再抛出去
    throw new UncheckedExecutionException(e.getCause());
  }
}

// UncheckedExecutionException是一个RuntimeException
public class UncheckedExecutionException extends RuntimeException {}
```

### writeReplace()方法

```java
@Override
Object writeReplace() {
  return new LoadingSerializationProxy<K, V>(localCache);
}
```



