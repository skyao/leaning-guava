类CacheBuilderSpec
==================

CacheBuilder 配置规格。

CacheBuilderSpec 支持从字符串中解析配置，这使得它对于命令行配置特别有用。

字符串语法是一系列的逗号分隔的key或者key-value对，每个对应到 CacheBuilder 的方法：

| 字符串 | 方法 |
|--------|--------|
|concurrencyLevel=[integer]|concurrencyLevel()|
|initialCapacity=[integer]|initialCapacity()|
|maximumSize=[long]|maximumSize()|
|maximumWeight=[long]|maximumWeight()|
|expireAfterAccess=[duration]|expireAfterAccess()|
|expireAfterWrite=[duration]|expireAfterWrite()|
|refreshAfterWrite=[duration]|refreshAfterWrite()|
|weakKeys|weakKeys()|
|softValues|softValues()|
|weakValues|weakValues()|
|recordStats|recordStats()|

随着 CacheBuilder 的演进，支持的key的集合可能增长，但是已经存在的key不会被删除。

Duration 被表示为整型，带有"d", "h", "m", 或 "s"中的其中一个符号，用来表示对应的天/days, 小时/hours, 分钟/minutes, 或者 秒/seconds。

在逗号和等号前后的空白字符将被忽略。key不可以重复。同样在一个单值中使用下列key对也是非法的：

- maximumSize 和 maximumWeight
- softValues 和 weakValues

CacheBuilderSpec 不支持配置 CacheBuilder 的有非值参数的方法。这些必须在代码中配置。

可以使用  CacheBuilder.from(CacheBuilderSpec) 或 CacheBuilder.from(String) 来从 CacheBuilderSpec 实体化CacheBuilder。
