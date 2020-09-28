# 三种限流算法：计数器算法、漏桶算法、令牌桶算法
## 1、计数器算法：
kong网关的官方限流插件使用计数器算法，它主要分为三种模式：local、redis和cluster。
- local是本地缓存，使用nginx的缓存
- redis和local限流类似，只是将缓存放到了redis上，这样可以实现分布式的限流
- cluster是数据库限流，kong支持两种数据库，PostgreSql和Cassandra，对于PostgreSql，为了提升性能使用了存储过程，而Cassandr直接采用Update语句。


**注：kong的三种限流模式都没有考虑并发的情况，当限流达到临界值（max-1）时，如果此时有多条请求同时执行get_usage，计算结果全部通过。**

<detials>
<summary>核心代码：</summary>

```java
/**
 * 简单的速率限制器
 */
public class Bucket {
    private volatile String key;
    private volatile Long start;
    private volatile Long interval;
    private volatile AtomicInteger count;
    private volatile Integer limit;
    
    public boolean request() {
        if (start == null) {
            start = new Date().getTime();
        }
        if (System.currentTimeMillis() - start <= interval) {
            if (count == null) {
                count = new AtomicInteger(0);
            }
            if (count.intValue() <= limit) {
                count.incrementAndGet();
                return true;
            } else {
                System.out.println(key + " 被拒绝访问");
                return false;
            }
        } else {
            start = new Date().getTime();
            return request();
        }
    }
}
```
</details>

## 2、指令桶算法

