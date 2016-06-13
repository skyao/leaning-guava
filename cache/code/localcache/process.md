流程
======

## get()

```java
V getOrLoad(K key) throws ExecutionException {
	//使用defaultLoader，调用get
	return get(key, defaultLoader);
}

V get(K key, CacheLoader<? super K, V> loader) throws ExecutionException {
	//计算key的hash值
    int hash = hash(checkNotNull(key));
    //根据hash值得到对应的segment
    //然后在这个segment中执行get方法
    return segmentFor(hash).get(key, hash, loader);
}
```


### hash算法

```java
int hash(@Nullable Object key) {
    int h = keyEquivalence.hash(key);
    return rehash(h);
}
```

Equivalenced的hash()实现，最终还是需要看子类的doHash()算法：

```java
public abstract class Equivalence<T> {
  public final int hash(@Nullable T t) {
    if (t == null) {
      return 0;
    }
    return doHash(t);
  }
}

protected abstract int doHash(T t);
```

- Equals： Equals的实现是取对象的hashCode()

    ```java
    static final class Equals extends Equivalence<Object> implements Serializable {
        @Override
        protected int doHash(Object o) {
          return o.hashCode();
        }
    }
    ```

- Identity： Identity的实现是 System.identityHashCode(o)

    ```java
    static final class Identity extends Equivalence<Object> implements Serializable {
    @Override
    protected int doHash(Object o) {
      return System.identityHashCode(o);
    }
    ```



