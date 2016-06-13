接口Cache
========

半持久化的从key到value的映射。缓存实体是通过get(Object, Callable)或者 put(Object, Object) 方法手工添加，并存储在缓存中，直到被驱逐或者手工失效。

这个接口的实现要求是线程安全(thread-safe)，并可以被多个并发线程安全访问。

## 接口定义

```java
package com.google.common.cache

public interface Cache<K, V> {}
```

## 接口方法

### getIfPresent()

```java
V getIfPresent(Object key);
```

返回在这个缓存中和key关联的值，如果这个key不存在缓存值则返回null。

> 疑问： 为什么key的类型是Object，而不是范型K？

### get()

```java
V get(K key, Callable<? extends V> valueLoader) throws ExecutionException;
```

返回在这个缓存中和key关联的值，必要时从 valueLoader 获取值。不修改和这个缓存关联的可观察的状态直到装载完成。这个方法为 传统的 "如果有缓存则返回;否则先创建再缓存并返回" 提供简单替代。

警告：在 CacheLoader.load()方法中，valueLoader **不容许**返回null;它可以返回一个非空值或者抛出异常。

当装载值时，如果抛出受查异常(checked exception)则抛出 ExecutionException; 如果抛出非受查异常则抛出 UncheckedExecutionException 。

### getAllPresent()

```java
ImmutableMap<K, V> getAllPresent(Iterable<?> keys);
```

返回在这个缓存中和key关联的所有值。返回的map将只包含已经存在于缓存的实体。

### put()

```java
void put(K key, V value);
```

关联key和value到缓存中。如果缓存之前已经有value关联到key，旧的值将被替换。

当使用"如果有缓存则返回;否则先创建再缓存并返回"模式时，推荐使用get(Object, Callable)方法。

### putAll()

```java
void put(K key, V value);
```

将指定map中的所有映射复制到缓存中。这个方法调用等同于为指定map中的每个key/value映射调用put(k, v)方法。

当操作进行的过程中，指定的map被修改，这个操作的行为是未定义的(看实现)。

### invalidate()

```java
void invalidate(Object key);
void invalidateAll(Iterable<?> keys);
void invalidateAll();
```

使缓存失效

### size()

```java
long size();
```

返回这个缓存中实体的大概数量

### stats()

```java
CacheStats stats();
```

返回这个缓存累计统计的当前快照。所有状态初始为0,并且在缓存的生存期内单调增长。

### asMap()

```java
ConcurrentMap<K, V> asMap();
```

以线程安全的map的方式返回存储在缓存中的实体的视图。对这个map的修改将直接影响缓存。

### cleanUp()

```java
void cleanUp();
```

执行任何缓存需要的待定维护操作。具体执行哪些行为是依赖实现的。

