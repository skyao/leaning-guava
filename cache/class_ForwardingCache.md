类ForwardingCache
================

转发所有方法调用到另外一个缓存的缓存实现。

子类可以覆盖一个或者更多方法来修改底层缓存的行为，参考 装饰模式(decorator pattern)。

有两个抽象类 ForwardingCache 和 ForwardingLoadingCache，然后给出了两个默认实现 ForwardingCache.SimpleForwardingCache 和 ForwardingLoadingCache.SimpleForwardingLoadingCache。

> 注： 暂时看不到适合使用的场景。







