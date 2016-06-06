类CacheStats
============

缓存的性能统计。

这个类的实例是不可变的(immutable)。

缓存统计依照下列规则增长：

- 当缓存查找遭遇存在的缓存实体时，hitCount 增加
- 当缓存查找第一次遭遇丢失的缓存实体时，新的实体被装载

	- 在成功装载实体后，missCount 和 loadSuccessCount 增加， 而总的装载时间(以纳秒为单位)被增加到 totalLoadTime
	- 当装载实体时抛出异常，missCount 和 loadExceptionCount 增加，而总的装载时间(以纳秒为单位)被增加到 totalLoadTime
	- 遭遇缓存实体丢失而正在转载中的缓存查找将等待装载完成(成功或者不成功)，然后再增加 missCount

- 当实体被从缓存中移除时，evictionCount 增加
- 当缓存实体失效或者手工删除时，没有状态修改
- 在通过asMap得到的缓存视图做操作，没有状态修改

查找被特别定义为下面方法的调用：

- LoadingCache.get(Object)
- LoadingCache.getUnchecked(Object)
- Cache.get(Object, Callable)
- LoadingCache.getAll(Iterable)

## 实现

### 属性定义

以下属性都被定义为final，在初始化后就不可修改：

```java
//命中数量
private final long hitCount;
//不命中数量
private final long missCount;
//装载成功数量
private final long loadSuccessCount;
//装载失败数量
private final long loadExceptionCount;
//总装载时间
private final long totalLoadTime;
//移除数量
private final long evictionCount;
```

额外的requestCount()方法用来获取requestCount，requestCount属性被定义为 "hitCount + missCount":

```java
public long requestCount() {
return hitCount + missCount;
}
```

额外的loadCount()方法用来获取loadCount，loadCount属性被定义为 "loadSuccessCount + loadExceptionCount":

```java
public long loadCount() {
	return loadSuccessCount + loadExceptionCount;
}
```

### 方法定义

#### hitRate()方法

hitRate()方法用来计算命中率：

```java
public double hitRate() {
	long requestCount = requestCount();
	return (requestCount == 0) ? 1.0 : (double) hitCount / requestCount;
}
```

命中率被定义为"hitCount / requestCount"，特殊情况当requestCount=0时视为 "1.0"。

#### missRate()方法

missRate()方法用来计算不命中率：

```java
public double hitRate() {
	long requestCount = requestCount();
	return (requestCount == 0) ? 0.0 : (double) missCount / requestCount;
}
```

命中率被定义为"missCount / requestCount"，特殊情况当requestCount=0时视为 "0.0"。

#### loadExceptionRate()方法

loadExceptionRate()方法用来计算装载异常率：

```java
public double loadExceptionRate() {
    long totalLoadCount = loadSuccessCount + loadExceptionCount;
    return (totalLoadCount == 0)
        ? 0.0
        : (double) loadExceptionCount / totalLoadCount;
  }
```

命中率被定义为"loadExceptionCount / totalLoadCount"，特殊情况当totalLoadCount=0时视为 "0.0"。

#### averageLoadPenalty()方法

averageLoadPenalty()方法用来计算平均装载代价：

```java
public double averageLoadPenalty() {
    long totalLoadCount = loadSuccessCount + loadExceptionCount;
    return (totalLoadCount == 0)
        ? 0.0
        : (double) totalLoadTime / totalLoadCount;
}
```

平均装载代价被定义为"totalLoadTime / totalLoadCount"，特殊情况当totalLoadCount=0时视为 "0.0"。

#### plus()方法

plus()方法将两次统计结果加起来，得到一个新的实例

#### minus()方法

minus()方法计算两次统计结果的差值，得到一个新的实例。如果差值为负数的情况，设置为0.
