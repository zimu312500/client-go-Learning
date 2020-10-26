# client-go-Learning

## 问题记录
### 1.Informer 中为什么需要引入 Resync 机制？
https://github.com/cloudnativeto/sig-k8s-source-code/issues/11

### 2.Indexer
- Indexer---> Indexers["byUsr"] = ByUserFunc //byUser为索引函数名,如果在Kubernetes中如命名空间的索引函数名如MetaNamespaceKeyFunc
- IndexFunc --> IndexFunc(Obj) (String[]) 根据对象利用索引函数生成对象键数组
- Indices --> Indices["byUser"] = Index //Index为数据存储对象
- Index ---> Index["对象键"] = 对象Set

### 3.workqueue
- 延迟队列是依靠waitingLoop实现延迟的。一直循环？不是的，如果有元素会等到最早元素的时间，如果没有元素则一直等待。
```
  // 如果有序队列中没有元素，那就不用等一段时间了，也就是永久等下去
        // 如果有序队列中有元素，那就用第一个元素指定的时间减去当前时间作为等待时间，逻辑挺简单
        // 有序队列是用时间排序的，后面的元素需要等待的时间更长，所以先处理排序靠前面的元素
        nextReadyAt := never
        if waitingForQueue.Len() > 0 {
            entry := waitingForQueue.Peek().(*waitFor)
            nextReadyAt = q.clock.After(entry.readyAt.Sub(now))
        }
        // 进入各种等待
        select {
        // 有退出信号么？
        case <-q.stopCh:
            return
        // 定时器，没过一段时间没有任何数据，那就再执行一次大循环，从理论上讲这个没用，但是这个具备容错能力，避免BUG死等
        case <-q.heartbeat.C():
        // 这个就是有序队列里面需要等待时间信号了，时间到就会有信号
        case <-nextReadyAt:
        // 这里是从chan中获取元素的，AddAfter()放入chan中的元素
        case waitEntry := <-q.waitingForAddCh:
            // 如果时间已经过了就直接放入通用队列，没过就插入到有序队列
            if waitEntry.readyAt.After(q.clock.Now()) {
                insert(waitingForQueue, waitingEntryByData, waitEntry)
            } else {
                q.Add(waitEntry.data)
            }
            // 下面的代码看似有点多，目的就是把chan中的元素一口气全部取干净，注意用了default意味着chan中没有数据就会立刻停止
            drained := false
            for !drained {
                select {
                case waitEntry := <-q.waitingForAddCh:
                    if waitEntry.readyAt.After(q.clock.Now()) {
                        insert(waitingForQueue, waitingEntryByData, waitEntry)
                    } else {
                        q.Add(waitEntry.data)
                    }
                default:
                    drained = true
                }
            }
        }
    }
```
https://blog.csdn.net/weixin_42663840/article/details/81482553#%E5%BB%B6%E6%97%B6%E9%98%9F%E5%88%97
- 令牌桶算法
