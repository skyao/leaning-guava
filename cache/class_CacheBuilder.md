类CacheBuilder
==============

## 说明

> 注: 以下内容翻译自 类CacheBuilder 的javadoc

LoadingCache 和 Cache 实例的构建器，缓存有下列特性的任意组合：

- 自动装载到缓存
- 当超过最大数量时，LRU(least-recently-used/最近最少使用)移除
- 基于时间的超时机制，从最后一次访问或者最后一次写入开始测量
- key自动用弱引用(weak reference)包裹
- value自动用弱引用(weak reference)或者软(soft reference)引用包裹
- 移除通知（或者其他删除方式）
- 缓存访问统计

这些特性是可选的。可以使用他们中的所有或者都不来创建缓存。CacheBuilder默认创建的缓存不执行任何形式的移除。

使用案例：

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
    .maximumSize(10000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .removalListener(MY_LISTENER)
    .build(
       new CacheLoader<Key, Graph>() {
         public Graph load(Key key) throws AnyException {
           return createExpensiveGraph(key);
         }
       });
```

或者等价的：

```java
//实际开发中这些将来自命令行或者配置文件
String spec = "maximumSize=10000,expireAfterWrite=10m";

LoadingCache<Key, Graph> graphs = CacheBuilder.from(spec)
   .removalListener(MY_LISTENER)
   .build(
       new CacheLoader<Key, Graph>() {
         public Graph load(Key key) throws AnyException {
           return createExpensiveGraph(key);
         }
       });
```

返回的cache实现为hash table，和 ConcurrentHashMap 有类似的性能。它实现了 LoadingCache 和 Cache 接口的所有可选操作。asMap 视图(和集合视图)是"弱一致性迭代器(weakly consistent iterator)"。这意味着他们可以安全的并发使用，但是如果其他线程在迭代器创建之后修改缓存，哪些修改会影响到迭代器是未定义的。这些迭代器绝不会抛出 ConcurrentModificationException。

注意：默认，返回的cache使用相等比较(equals()方法)来检测key和value的相等性。但是，如果指定了 weakKeys(), cache将为key使用同一性(==)比较. 类似的，如果指定了 weakValues() 或者 softValues(), cache 为value使用同一性比较。

> 注: 相等比较是调用对象的equals()方法, 同一性比较是使用运算符==，直接比较对象的句柄。

当 maximumSize, maximumWeight, expireAfterWrite, expireAfterAccess, weakKeys, weakValues, 或者 softValues中的任何一个有要求时，实体将被从缓存中自动移除.

如果 maximumSize 或者 maximumWeight 被要求，实体可能在每次缓存修改时被移除。

如果要求 expireAfterWrite 或者 expireAfterAccess ，实体可能在每次缓存修改， 间歇的缓存访问，或者调用 Cache.cleanUp() 时被移除。过期的实体可能被 Cache.size() 计算，但是对于读写操作是绝对不可见。

如果要求 weakKeys, weakValues 或 softValues，这使得缓存中的key或者value可以被GC回收。key或者value被回收的实体可能在每次缓存修改，间歇的缓存访问，或者调用 Cache.cleanUp() 时从缓存中移除。过期的实体可能被 Cache.size() 计算，但是对于读写操作是绝对不可见。

某些缓存配置会导致周期性维护任务的增加， 这些任务在写操作或者没有写操作时在间歇的读操作期间被执行。返回的缓存的 Cache.cleanUp()方法也将执行维护操作，但是对于高吞吐量的缓存没有必要调用这个方法。只有构建时带有 removalListener, expireAfterWrite, expireAfterAccess, weakKeys, weakValues, 或 softValues 的缓存会执行周期性维护。

CacheBuilder 生成的缓存是可序列化的，而反序列化后的缓存保持所有的原始缓存的配置属性。注意序列化形式不包含缓存内容，仅有配置。

## 类定义

```java
package com.google.common.cache;

public final class CacheBuilder<K, V> {}
```

- 范型`<K>` 这个builder创建的所有缓存的key类型
- 范型`<V>` 这个builder创建的所有缓存的value类型

## 类方法

### 创建builder

#### newBuilder()方法

```java
public static CacheBuilder<Object, Object> newBuilder() {
	return new CacheBuilder<Object, Object>();
}
```

newBuilder()方法使用默认设置构建一个新的 CacheBuilder 实例，包括 strong key， strong value，没有任何类型的自动移除。

> 注: 为什么这里类型是Object，而不是范型？

#### from()方法

```java
public static CacheBuilder<Object, Object> from(CacheBuilderSpec spec) {
	return spec.toCacheBuilder().lenientParsing();
}

public static CacheBuilder<Object, Object> from(String spec) {
	return from(CacheBuilderSpec.parse(spec));
}
```

from()方法使用指定的设置来构建 CacheBuilder 实例。其中from(String spec)方法特别适合用于命令行或配置。

### 指定key和value比较的方式

#### keyEquivalence()方法

为key的比较设置自定义的相等策略：

```java
CacheBuilder<K, V> keyEquivalence(Equivalence<Object> equivalence) {
    checkState(keyEquivalence == null, "key equivalence was already set to %s", keyEquivalence);
    keyEquivalence = checkNotNull(equivalence);
    return this;
}
```

> 注：从代码上看，只容许设置一次equivalence，否则报错。

默认，如果指定weakKeys()，缓存将使用 Equivalence.identity 来检测key的相等性，否则使用Equivalence.equals。

具体实现的代码：

```java
Equivalence<Object> getKeyEquivalence() {
	//检查keyEquivalence
    //如果没有设置就使用getKeyStrength().defaultEquivalence()
	return MoreObjects.firstNonNull(keyEquivalence, getKeyStrength().defaultEquivalence());
}

Strength getKeyStrength() {
	//如果没有设置keyStrength
    //就取Strength.STRONG
    return MoreObjects.firstNonNull(keyStrength, Strength.STRONG);
  }

enum Strength {
	STRONG {
        Equivalence<Object> defaultEquivalence() {
        	//STRONG默认Equivalence.equals()
        	return Equivalence.equals();
    }
    WEAK {
      Equivalence<Object> defaultEquivalence() {
        //WEAK默认Equivalence.identity()
        return Equivalence.identity();
      }
    };
```

### valueEquivalence()方法

为value的比较设置自定义的相等策略：

```java
CacheBuilder<K, V> valueEquivalence(Equivalence<Object> equivalence) {
    checkState(valueEquivalence == null,
        "value equivalence was already set to %s", valueEquivalence);
    this.valueEquivalence = checkNotNull(equivalence);
    return this;
}
```

> 注：从代码上看，只容许设置一次equivalence，否则报错。

默认，当指定 weakValues() 或者 softValues()时缓存使用 Equivalence.identity()来检测值的相等性，其他情况使用Equivalence.equals()。

### 指定缓存的基本配置

#### initialCapacity()方法

为内部的hash table设置最小的总大小，默认16。

例如，如果初始容量为 60, 而 concurrency level 是8，那么就会创建8个片段，每个片段有一个大小为8的hash table。在构建时刻提供一个足够的估算可以避免稍后昂贵的resize操作的需要，但是设置过大会浪费内存。

#### concurrencyLevel()方法

设置并发等级，指导更新操作期间容许的并发度。为了实现并发更新而不竞争，table内部进行分区。

理想情况，应该选择一个可以容纳同时并发修改table的线程数的值。使用一个明显过高的值会浪费空间和时间，而一个明显过低的值会导致线程竞争。

默认为4.

当前实现使用 concurrency level 来创建一个固定数量的 hashtable segment，每个segment由自己的写锁管理。片段锁在每次显式写时获取一次，而在每次缓存计算时获取两次(一次在装载新值之前，一次在装载完成之后)。很多内部的缓存管理在segment粒度上执行。例如，当要求有移除算法时，访问队列和写队列在每个segment上保持。

#### maximumSize()方法

指定缓存可以容纳的实体的最大数量。

注意：缓存可能在这个限制被超过之前移除实体。当缓存大小增长到接近最大值时，缓存移除最近不被再使用的实体。例如，缓存因为实体最近没有被使用或者不是很经常使用而移除这个实体。

当大小设置为0时，元素将在装载进入缓存之后被立即移除。测试中这个很有用，或者在不修改代码的情况下让缓存临时失效。

这个特性不能和 maximumWeight 同时使用。

#### maximumWeight()方法

指定缓存可以容纳的实体的最大重量。

使用 weigher()方法指定的 Weigher 来检测重量，而且使用这个方法要求在调用 build() 方法之前对应的调用 weigher() 方法。

注意：缓存可能在这个限制被超过之前移除实体。当缓存大小增长到接近最大值时，缓存移除最近不被再使用的实体。例如，缓存因为实体最近没有被使用或者不是很经常使用而移除这个实体。

当大小设置为0时，元素将在装载进入缓存之后被立即移除。测试中这个很有用，或者在不修改代码的情况下让缓存临时失效。

注意：重量(指maximumWeight()方法的weight参数)仅仅用于检测缓存是否超过容量。对于选择哪些实体移除没有影响。

#### weigher()

指定用于检测实体重量的 weigher。当决定哪个实体被移除时，实体重量被考虑到，而使用这个方法需要在调用build()之前对应调用maximumWeight(long)方法。当实体被插入到缓存时计算并记录重量，而之后在缓存实体的生存期内不变。

当实体的重量为0时，这个实体将不被基于大小的排除算法考虑(当然它还是可能被其他算法排除)。

非常重要：不再作为CacheBuilder实例返回this，这个方法返回 CacheBuilder<K1, V1> 。从这点上，无论是原始引用还是返回的引用都可以用来用来完成配置和构建缓存，但是只有"范型"的这个才是类型安全的。也就是说，它可以正确的防止你构建key或者value的类型和已经提供的 weigher 不兼容的缓存，而 CacheBuilder 类型无法做到这点。

如果没有设置 weigher，则默认使用 OneWeigher.INSTANCE，将每个实体的重量简单返回1.

#### setKeyStrength()方法

指定缓存中的每个key(不是value)应该包裹为 WeakReference (默认，使用strong reference)。

警告： 当这个方法被使用时，得到的缓存将使用 identity comparison (==)来检测key的相等性。

key已经被GC的实体可能还被 Cache.size() 计数，但是对读或者写操作绝对不可见。

#### setValueStrength()和softValues()方法

指定缓存中的每个value(不是key)应该包裹为 WeakReference 或者 SoftReference (默认，使用strong reference)。

#### expireAfterWrite()方法

指定在实体创建或者最后一次覆盖值后，一旦固定期限达到，应该从缓存中自动删除。

当 duration 为0时，这个方法类似于 maximumSize(0)，忽略其他指定的最大size或者weight。这可以用于测试，或者在不修改代码的情况下临时让缓存失效。

过期的实体可能还被 Cache.size() 计数，但是对读或者写操作绝对不可见。

#### expireAfterAccess()方法

指定在实体创建，最后一次覆盖值，或者最后一次访问后，一旦固定期限达到，应该从缓存中自动删除。

访问时间会被所有缓存读和写操作(包括Cache.asMap().get(Object)和Cache.asMap().put(K, V))重置，但是不包括 Cache.asMap() 的集合视图上的操作。

#### refreshAfterWrite()方法

指定在实体创建或者最后一次覆盖值后，一旦固定周期达到，活动实体有资格自动刷新。刷新的语义在方法 LoadingCache.refresh() 中指定，而且也是通过调用 CacheLoader.reload() 来执行的。

因为默认 CacheLoader.reload() 的实现是同步的，推荐这个方法的用户用一个异步实现覆盖 CacheLoader.reload() 方法，否则刷新将在不相关的缓存读和写操作期间执行。

目前自动刷新被实现为当实体的第一个陈腐的请求发生执行。这个触发刷新的请求将阻塞调用 CacheLoader.reload() 并立即返回新的值，如果返回的future是完成的，否则返回旧的值。

注意：在刷新期间抛出的异常都被记录日志然后吞掉。

注意：refreshAfterWrite()需要使用 LoadingCache。

#### removalListener()方法

指定监听器实例，缓存应该在每次实体因为任何 RemovalCause 原因被删除时通知。这个builder创建的每个缓存将调用这个监听器。

警告：在调用这个方法后，不要在继续使用 this cache builder引用，而是使用这个方法返回的引用。在运行开始，这些指向同一个实例，但是只有返回的引用拥有正确的范型类型来保证类型安全。

警告：监听器抛出的任何异常都将不会传播到缓存的用户，仅仅通过 Logger来记录日志。

#### recordStats()方法

开启在缓存操作期间的 CacheStats 累积。不调用这个方法则 Cache.stats() 将为所有统计返回0.注意记录统计需要为每次操作执行记录，而这将带来性能损失。

注意：在版本12.0之前，统计收集是自动开启的。

### 构建缓存实例的方法

#### build(CacheLoader)方法

构建一个缓存，要不返回已经装载的值，要不通过提供的 CacheLoader 自动计算或者获取值。如果其他线程正在装载这个key的值，简单等待这个线程完成并返回它装载的值。注意多个线程可以并发的装载不同的key。

这个方法不会改变在这个 CacheBuilder 实例的状态，因此可以再次调用来创建多个独立的缓存。

#### build()方法

构建一个缓存，当key被要求时不自动装载。

考虑使用 build(CacheLoader)，如果可能实现 CacheLoader。

这个方法不会改变在这个 CacheBuilder 实例的状态，因此可以再次调用来创建多个独立的缓存。


