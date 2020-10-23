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
- 延迟队列是依靠waitingLoop实现延迟的。一直循环？
https://blog.csdn.net/weixin_42663840/article/details/81482553#%E5%BB%B6%E6%97%B6%E9%98%9F%E5%88%97
- 令牌桶算法
