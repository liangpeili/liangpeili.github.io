---
title: P2P与DHT的简单调研
date: 2019-01-10 18:05:09
tags: 区块链
---

# P2P 与 DHT 的简单调研

## 一、什么是哈希表？

哈希表（也叫散列表）是用来存储键值对的一种容器，英文为‘key/value pairs’。它的特点是可以放标快速的根据键(key)来获取值(value)。现实中的一个例子就是手机通信录，你可以根据某个人的名字查询到他的号码。

存储 value 的单元叫做桶(bucket)或者槽(slot)。在一个哈希表中，会包含N个桶，N通常是固定不变的。每个桶都有一个编号(index)，可以编为0~N-1。那如何确定哪条数据存入哪个桶呢？哈希表有一个映射函数f: key -> index。可以根据 key 计算出可以存入的存储位置，这个映射函数也叫做哈希函数。在想要查找某个人的联系方式时，比如`John Smith`，就可以根据哈希函数计算出他的电话号码存储位置为02，从而快速的找到 value。

![1](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/Hash_table_3_1_1_0_1_0_0_SP.svg/630px-Hash_table_3_1_1_0_1_0_0_SP.svg.png)

图1.哈希表

那有没有这种可能：不同的联系人经过哈希函数运算以后，存储到了相同的位置？这种情况叫做‘碰撞’，是存在的。因此在设计哈希函数时就需要考虑到这种情况，尽量避免。一个好的哈希函数应该满足以下条件：

1. 计算出的哈希值要足够离散；
2. 尽可能的降低碰撞；
3. 桶的数量要足够大；

![2](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d0/Hash_table_5_0_1_1_1_1_1_LL.svg/900px-Hash_table_5_0_1_1_1_1_1_LL.svg.png)

图2. 发生碰撞的情况

## 二、分布式哈希表

分布式哈希表全称是 `Distributed Hash Table`，简称DHT。分布式哈希表和哈希表类似，区别在于传统的哈希表主要用于单机软件，而分布式哈希表则主要用于分布式系统。DHT 的主要作用还是为了提供从 key 到 value 的查询服务。在一个基于DHT的分布式网络里，每个节点都存储着网络上的部分数据以及部分路由。当一份数据想要写入网络时，首先对这份数据做哈希，得到一个唯一的标识符(key)后再存入网络。由于数据是分布式存储的，修改数据也只会引起部分节点的修改，而不是全网的节点同步修改。通过这种方式实现全网的寻址和存储。

![3](https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/DHT_en.svg/1000px-DHT_en.svg.png)

DHT 只是一个理念，提出了这样的一种网络模型并解释了它对分布式存储的好处，但具体怎么实现并不是DHT的范畴。

## 三、P2P 的发展史

### 1. 第一代P2P技术

第一代P2P技术采用了中央服务器的模式：每个节点在查找文件时，需要首先连接到中央服务器，才能知道自己想要文件的地址。这种模式的缺点在于中央服务器成为整个P2P网络的最大缺点，容易发生单点故障。

第一代P2P技术的代表是 Napster。

### 2. 第二代P2P技术

第二代P2P技术采用了广播的模式。当某个节点想要查询某个文件的时候，它会向所有自己看见的节点发送请求进行询问。如果被询问的节点没有这个文件，那么它会继续发起广播。因此这种技术会引起广播风暴，极大的占用网络带宽和节点的系统资源。协议层面虽然可以设置存活时间(Time To Live, TTL)，依然解决不了广播风暴的问题。

![4](https://lh6.googleusercontent.com/trmBi0r-sOJY0WJ0TjmXaCYgo602BxNc7D5LAA1kSSByWBWMJF2DOBP64XWMe_DDV8WVUvqTSxibnPvpAKJ5oIxMAHeY3gPahp9s2BRNXUDceQzmwrjk0aWEvvx-X6z1iauTFo1X32o)

第二代P2P的典型代表是 Gnutella。

### 3. 第三代P2P技术

第三代P2P技术就是上面提到过的DHT。它既避免了第一代技术的单点故障，也没有第二代技术的广播风暴。它没有中心，可以存储海量数据。参与DHT的节点每时每刻都有新的节点上线和旧的节点下线，在这种情况下，它依然能够完成数据的高效查询。那么DHT是怎么实现的呢？

#### 哈希算法的选择

DHT 通常是直接拿业务数据的哈希值作为 key， 业务数据本身作为 value。由于DHT承载的数据量通常比较大，因为哈希函数的值域就一定要足够大才可以避免碰撞。通常DHT大多采用128bit 以上的哈希值。

#### nodeid

在DHT的分布式系统中存在着多个节点，这样的话一般会给每个节点分配唯一的 nodeid。

#### 拓扑结构

DHT 网络需要定义自己的拓扑结构，有了拓扑结构，就需要设计一种路由算法。当某个分布式系统具有自己的拓扑结构，就称之为一个`Overlay Network`，也就是‘网络之上的网络’，它的数据通信依赖于下层的互联网来实现。

由于每个节点都有 nodeid，因此在设计拓扑结构和路由算法时，它就可以只需要考虑 nodeid，而不用考虑其下层网络的属性（协议类型、ip、端口等）。

#### 路由算法

由于DHT网络中的节点可能非常多并且是动态变化的，因为不可能让每个节点记录网络上所有其他节点的信息。实际情况是：每个节点只知道少数一些节点的信息。因此需要设计一种路由算法，尽可能的利用已知节点来转发数据。路由算法确定好以后，还需要确定路由表的大小。路由表越大，就可以实现越短（跳数越少）的路由。

#### 距离算法

某些DHT 系统还会定义一种距离算法，用来计算节点之间的距离等。

#### 数据保存与提取

前面说的所有东西，都是为了这两件事（数据的保存与提取）服务的。

保存数据：
当某个节点收到了一份新的数据 value，它会先计算出这份 value 的 key，然后计算自己与这份数据之间的距离，然后再计算它所知道的节点与这个 key 的距离。全部计算完成后，如果发现自己与 key 的距离最小，那么久自己保存。要不就把它转发到自己计算出的最小距离的节点。收到这份数据的节点也做同样的处理。

提取数据：
当某个节点收到查询数据的请求(key)以后，也会计算自己以及自己知道的节点和这个 key 之间的距离。如果计算下来自己最小，然后就在本地查询有无该文件。否则就把这个请求发给最小距离的节点。收到请求的其他节点也会做同样处理。

2001年诞生了第一批DHT 洗衣，比如Chord、CAN、Tapestry 以及Pastry 等。不过最流行的还是K ademlia，Bittorrent 以及电驴等用的都是Kademlia。






[聊聊分布式散列表（DHT）的原理——以 Kademlia（Kad） 和 Chord 为例 @ 编程随想的博客](https://program-think.blogspot.com/2017/09/Introduction-DHT-Kademlia-Chord.html)

[Hash table - Wikipedia](https://en.wikipedia.org/wiki/Hash_table)

[Distributed hash table - Wikipedia](https://en.wikipedia.org/wiki/Distributed_hash_table)

[Kademlia协议原理简介 - 百度文库](https://wenku.baidu.com/view/ee91580216fc700abb68fcae.html)

[Kademlia Peer Selection · ethereum/wiki Wiki](https://github.com/ethereum/wiki/wiki/Kademlia-Peer-Selection)

