类LocalCache
============

guava cache中真正实现cache功能的类，这个类才是精华所在。

典型如，这个类有4900多行！

```java
class LocalCache<K, V> extends AbstractMap<K, V> implements ConcurrentMap<K, V> {
}
```

