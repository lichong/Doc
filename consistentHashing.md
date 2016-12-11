# 一致性哈希 #
一致性哈希主要用于分布式系统中的哈希，比如redis保存数据时需要根据哈希计算数据保存在哪一个节点上。

## 普通哈希算法 ##
普通的哈希算法一般是直接将key值进行哈希，然后对所有节点进行取模，计算应该落在哪个节点上，并将数据保存在这个节点上。

对于没有节点变化的场景下这种哈希没有问题，只要所有节点都正常工作，每次有请求过来的时候都可以到正确的节点上进行保存和提取数据。但是一旦出现节点故障或者需要扩充节点，这种哈希算法对于数据的迁移是巨大的，基本上所有的数据都会迁移，因为新的节点加入后是，key值哈希后的取模基本上都会变化。

## 一致性哈希算法 ##
### 简单一致性哈希 ###
一致性哈希就是为了解决这种节点有新增或者删除情况下的哈希问题。诚然对于哈希算法来说，有节点变化必然会带来数据的迁移，那我们考虑的就应该是如何能将迁移的量降低到最小。一致性哈希在这个问题上给出了一种解决方法。将数据分散到各个节点上，还是先将key进行哈希计算，但是计算的结果并不是直接取模落到某一个节点上，而是将所有节点围成一个环（环需要满足单调性），这样在key值哈希以后就会出现在这个环上的某一点，寻找比这个点大的最小节点，这个key就落在这个节点上。如果没有比这个节点大的，那就直接落在第一个节点上即可。

这样在有新增节点时只需要将原有某一个节点下的部分数据迁移到新节点上即可。其他数据依然不变。

而节点删除时同样只需要将原有的节点数据迁移到比删除节点大一点的节点上即可。

### 优化一致性哈希 ###
简单的一致性哈希可以解决普通哈希算法的大量数据迁移问题，但是会出现另外一个问题，那就是数据不均衡问题，一旦某一段哈希值对应的数据比较多就会出现某一个节点比较忙，而其他节点比较闲的情况，为了解决这个情况，一致性哈希算法引入了一个虚拟节点的概念，在整个哈希环上，每个实体节点会虚拟出一定数量的虚拟节点，这样就可以针对简单一致性哈希中的数据均衡问题进行处理，对于某一段来说基本上会散步在各个节点上，尽可能的做到数据分散。

## 总结 ##
一致性哈希算法满足了一下几个条件：
均衡性
单调性
分散性
负载

参考[百度百科](http://baike.baidu.com/link?url=QwVdW9ZvXDrw7kIX5AeO-FaIcnTi8UIz5Qz7fVB-S2VCiqOiGTYLtIi-Mw-zIV14u0DbqqmHHBiCpo5qvm0WsK)
