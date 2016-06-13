接口LoadingCache
===============

半持久化的从key到value的映射。值是缓存自动装载，并存储在缓存中，直到被驱逐或者手工失效。

## 接口定义

```java
package com.google.common.cache

public interface LoadingCache<K, V> extends Cache<K, V>, Function<K, V> {}
```

注意 LoadingCache 实现了 Function 接口，当 LoadingCache 被作为 Function 时，cache产生的结果和调用getUnchecked()方法一致。

## 方法定义

LoadingCache 继承自 Cache 接口，自然拥有 Cache 定义的方法。此外 LoadingCache 还定义了自生的特殊方法。

### get()方法

```java
V get(K key) throws ExecutionException;
```

返回这个缓存中和key关联的value，如有必要首先装载这个值。不修改和这个缓存关联的可观察的状态直到装载完成。

如果其他 get() 或者 getUnchecked() 的调用已经正在装载这个 key 的值， 简单等待这个线程结果并返回装载好的值。注意多个线程可以并发的装载不同key的值。

通过 CacheLoader 装载的缓存将调用 CacheLoader.load()方法来装载新的值到缓存中。新装载好的值将在装载完成之后通过使用 Cache.asMap().putIfAbsent() 方法添加到缓存中。当新的值正在装载时，如果另外一个值被关联到key，则将为这个新的值发送删除通知(TBD：这里没有看懂，等后面看源码实现)。

如果和这个cache关联的cache loader被认为不会抛出受查异常， 那么建议使用 getUnchecked() 方法。

get()方法抛出ExecutionException， 如果装载值的时候抛出受查异常。如果装载时抛出非受查异常，则抛出 UncheckedExecutionException。

### getUnchecked()方法

```java
V getUnchecked(K key);
```

和get()方法不同，这个方法不会抛出受查异常，因此只能用于cache loader 不会抛出受查异常的情况。

警告：这个方法静默的将受查异常转为非受查异常，不适合用于会抛出受查异常的cache loader。在这种场合建议使用 get() 方法。


### getAll()方法

```java
ImmutableMap<K, V> getAll(Iterable<? extends K> keys) throws ExecutionException;
```

以map方式返回和key关联的值，必要时创建或者获取这些值。返回的map包含已经存储的实体和新装载的实体。key和value不会包含null。

通过 CacheLoader 装载的缓存将为所有不在缓存中的keys发起一个到CacheLoader.loadAll()的简单请求。CacheLoader.loadAll()返回的所有实体将被存储在cache中。覆盖任何之前缓存的值。这个方法将抛出异常，如果CacheLoader.loadAll()返回null，或者返回的map包含null值的key或者value，或者无法为每个请求的key返回实体。

注意：key中的重复元素(通过Object.equals()方法检测)将被忽略。

### apply()方法

```java
@Deprecated
V apply(K key);
```

提供这个方法以便满足 Function 接口的要求，请使用 get() 或者 
getUnchecked()。

### refresh()方法

```java
void refresh(K key);
```

装载key的新的value，可能是异步的。当新的值被装载时，之前的值(如果有)将继续通过 get()方法返回，除非被排除。如果新的值装载成功，它将替换缓存中之前的值。如果刷新过程中抛出异常，之前的值将被保留。而异常将被记录(使用java.util.logging.Logger)并吞掉。

如果其他线程当前正在装载这个key的值，直接返回不做任何事情。

### asMap()方法

```java
ConcurrentMap<K, V> asMap();
```

注意：虽然视图是可修改的，返回map上的任何方法不会导致实体被自动装载。

