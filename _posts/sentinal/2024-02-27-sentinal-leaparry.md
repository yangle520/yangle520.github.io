---
layout: post
title:  Sentinal 滑动时间窗口算法
date:   2024-02-27 00:00:00 +0800
categories: 算法
tag: 微服务组件
---



* content
{:toc}






针对固定时间算法会在临界点存在瞬间大流量冲击的场景，滑动时间窗口算法应运而生

### **滑动窗口原理**

它将时间窗口划分为更小的时间片段，每过一个时间片段，时间窗口就会往右滑动一格，每个时间片段都有独立的计数器。我们在计算整个时间窗口内的请求总数时会累加所有的时间片段内的计数器。时间窗口划分的越细，那么滑动窗口的滚动就越平滑，限流的统计就会越精确（下文我们具体举例说明，就会对这句话有更深刻的理解）



## Sentinel中的滑动窗口设计

### 核心类

ArrayMetric用来统计滑动窗口中的各种数据，比如已经通过的请求数，被限流的请求数、向当前窗口中增加通过的请求数等等

```java
class ArrayMetric {                  // 滑动窗口统计
    LeapArray<MetricBucket> data;    // 滑动窗口
    long pass();                     // 获取通过的请求数
    void addSuccess(int count);      // 增加通过的请求数
    // ... 其他计数
}

```

LeapArray 是滑动窗口的真正实现，包括计算请求的当前窗口重要方法

```java
abstract class LeapArray<T> {                         // 高性能数组-滑动窗口
    int windowLengthInMs;                             // 子窗口长度(ms)：500ms
    int sampleCount;                                  // 样本窗口数量(子窗口数量)
    int intervalInMs;                                 // 窗口滑动间隔时间（ms）--1000ms
    double intervalInSecond;                          // 窗口滑动间隔时间（ms）--1s
    final AtomicReferenceArray<WindowWrap<T>> array;  // 数组
}
```

WindowWrap是一个对象，真正的数据被保存到泛型T的value中，在这里使用的是MetricBucket对象

```java
class WindowWrap<T> {               // 子窗口
    final long windowLengthInMs;    // 子窗口长度(ms)：500ms
    long windowStart;               // 子窗口开始时间
    T value;                        // （ms）具体的统计数据
}
```

```java
class MetricBucket {
    final LongAdder[] counters;		// 数组 计数器
}
```

### 调用链路

 ![sentinel调用链路]({{ '/styles/images/sentinal/27688228-70f696cfddf3f1c5.webp' | prepend: site.baseurl  }})

### 限流逻辑

 ![sentinel调用链路]({{ '/styles/images/sentinal/27688228-2c51c0d8e7eb4914.webp' | prepend: site.baseurl  }})

 ![sentinel调用链路]({{ '/styles/images/sentinal/27688228-59754ca08bf8f2d7.webp' | prepend: site.baseurl  }})

### 核心代码

```java
@Spi(order = Constants.ORDER_STATISTIC_SLOT)
public class StatisticSlot extends AbstractLinkedProcessorSlot<DefaultNode> {
    // Do some checking.
    fireEntry(context, resourceWrapper, node, count, prioritized, args);

    // Request passed, add thread count and pass count.
    node.increaseThreadNum();
    node.addPassRequest(count);
    
    ......
}
```

```java
public class FlowRuleChecker {
    public boolean canPassCheck(@NonNull FlowRule rule, Context context, DefaultNode node, int acquireCount, boolean prioritized) {
        String limitApp = rule.getLimitApp();
        if (limitApp == null) {
            return true;
        }

        if (rule.isClusterMode()) {
            return passClusterCheck(rule, context, node, acquireCount, prioritized);
        }
        // 根据限流效果的不同，走直接失败、预热、允许排队
        return passLocalCheck(rule, context, node, acquireCount, prioritized);
    }
}
```

```java
public class DefaultController implements TrafficShapingController {
    @Override
    public boolean canPass(Node node, int acquireCount, boolean prioritized) {
        // 获取当前时间窗口中已经通过的请求数量
        int curCount = avgUsedTokens(node);
         /**
         * 若已经统计的数据与本次请求的数量和大于设置的阈值，返回false表示被限流
         * 若小于等于阈值，则返回true，表示检测通过
         */
        if (curCount + acquireCount > count) {
            if (prioritized && grade == RuleConstant.FLOW_GRADE_QPS) {
                long currentTime;
                long waitInMs;
                currentTime = TimeUtil.currentTimeMillis();
                waitInMs = node.tryOccupyNext(currentTime, acquireCount, count);
                if (waitInMs < OccupyTimeoutProperty.getOccupyTimeout()) {
                    node.addWaitingRequest(currentTime + waitInMs, acquireCount);
                    node.addOccupiedPass(acquireCount);
                    sleep(waitInMs);
                    // PriorityWaitException indicates that the request will pass after waiting for {@link @waitInMs}.
                    throw new PriorityWaitException(waitInMs);
                }
            }
            return false;
        }
        return true;
    }
}
```

```java
public class ArrayMetric implements Metric {
    /**
     * 循环通过窗口中的每个子窗口中通过的请求数量
     */
    @Override
    public long pass() {
        // 计算请求的当前窗口（此方法是关键）
        data.currentWindow();
        long pass = 0;
        List<MetricBucket> list = data.values();

        for (MetricBucket window : list) {
            pass += window.pass();
        }
        return pass;
    }
}
```

```java
public abstract class LeapArray<T> {
    public WindowWrap<T> currentWindow(long timeMillis) {
        if (timeMillis < 0) {
            return null;
        }
		// 获取当前时间的数组下标
        int idx = calculateTimeIdx(timeMillis);
        // 计算当前桶（子窗口）的开始时间
        long windowStart = calculateWindowStart(timeMillis);

        /**
         * 1）如果子窗口中没有值，这个时候系统刚启动，需要新建窗口（窗口长度比如500ms，窗口的起始时间和当前时间）
         * 2）如果新来的请求计算出来的起始时间和之前子窗口中的起始时间相同，表示请求间隔时间很短，在同一个窗口长度（比如500ms）内，因此这个请求也被划分到当前的这个子窗口内
         * 3）如果新来的请求计算出来的起始时间  大于  之前子窗口中的起始时间，说明一个滑动窗口长度（比如1000ms）已经过去了，
         * 这个时候滑动窗口的时间需要向前移动，因此需要重置子窗口的其实时间为 新来的请求计算出来的起始时间
         */
        while (true) {
            WindowWrap<T> old = array.get(idx);
            if (old == null) {
                /*
                 *     B0       B1      B2    NULL      B4
                 * ||_______|_______|_______|_______|_______||___
                 * 200     400     600     800     1000    1200  timestamp
                 *                             ^
                 *                          time=888
                 *            bucket is empty, so create new and update
                 *
                 * If the old bucket is absent, then we create a new bucket at {@code windowStart},
                 * then try to update circular array via a CAS operation. Only one thread can
                 * succeed to update, while other threads yield its time slice.
                 */
                WindowWrap<T> window = new WindowWrap<T>(windowLengthInMs, windowStart, newEmptyBucket(timeMillis));
                if (array.compareAndSet(idx, null, window)) {
                    // Successfully updated, return the created bucket.
                    return window;
                } else {
                    // Contention failed, the thread will yield its time slice to wait for bucket available.
                    Thread.yield();
                }
            } else if (windowStart == old.windowStart()) {
                /*
                 *     B0       B1      B2     B3      B4
                 * ||_______|_______|_______|_______|_______||___
                 * 200     400     600     800     1000    1200  timestamp
                 *                             ^
                 *                          time=888
                 *            startTime of Bucket 3: 800, so it's up-to-date
                 *
                 * If current {@code windowStart} is equal to the start timestamp of old bucket,
                 * that means the time is within the bucket, so directly return the bucket.
                 */
                return old;
            } else if (windowStart > old.windowStart()) {
                /*
                 *   (old)
                 *             B0       B1      B2    NULL      B4
                 * |_______||_______|_______|_______|_______|_______||___
                 * ...    1200     1400    1600    1800    2000    2200  timestamp
                 *                              ^
                 *                           time=1676
                 *          startTime of Bucket 2: 400, deprecated, should be reset
                 *
                 * If the start timestamp of old bucket is behind provided time, that means
                 * the bucket is deprecated. We have to reset the bucket to current {@code windowStart}.
                 * Note that the reset and clean-up operations are hard to be atomic,
                 * so we need a update lock to guarantee the correctness of bucket update.
                 *
                 * The update lock is conditional (tiny scope) and will take effect only when
                 * bucket is deprecated, so in most cases it won't lead to performance loss.
                 */
                if (updateLock.tryLock()) {
                    try {
                        // Successfully get the update lock, now we reset the bucket.
                        return resetWindowTo(old, windowStart);
                    } finally {
                        updateLock.unlock();
                    }
                } else {
                    // Contention failed, the thread will yield its time slice to wait for bucket available.
                    Thread.yield();
                }
            } else if (windowStart < old.windowStart()) {
                // Should not go through here, as the provided time is already behind.
                return new WindowWrap<T>(windowLengthInMs, windowStart, newEmptyBucket(timeMillis));
            }
        }
    }
    
}
```
