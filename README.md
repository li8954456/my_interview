# 前言
腾讯面失败后感觉自己需要学习的地方还很多，必须要改变以前的咸鱼状态，不然今后恐怕无缘大厂。<br>
所以总结一些面试官经常回问到的问题或者自己在看书过程中觉得重要的知识点，方便自己以后回顾。<br>

# 目录
* [一、Python](#Python)
    * [1 深浅拷贝](#1-深浅拷贝)
    * [2 装饰器](#2-装饰器)
    * [3 Python协议](#3-Python协议)
* [二、数据库](#数据库)
    * [1 MySQL](#1-MySQL)
         * [1 事务](#1-事务)
         * [2 索引](#2-索引)
    * [2 Redis](#2-Redis)
         * [1 优缺点](#1-优缺点)
         * [2 持久化](#2-持久化)
         * [3 为什么Redis是单线程还那么快，怎么解决并发的？](#3-为什么Redis是单线程还那么快怎么解决并发的)
         * [4 缓存穿透、缓存击穿和缓存雪崩](#4-缓存穿透缓存击穿和缓存雪崩)
    * [3 MongoDB](#3-MongoDB)
* [三、网络](#网络)
* [四、操作系统](#操作系统)
    * [1 进程和线程](#1-进程和线程)
    * [2 I/O模型](#2-I/O模型)
* [五、高并发解决方案](#高并发解决方案)
* [六、算法相关](#算法相关)

# Python
为什么会有Python？因为我就是写Python的呗
## 1 深浅拷贝
深拷贝不用多说，完全新建一个内存地址将内容拷贝出来，修改不会影响到原来的数据，这里主要讲一下浅拷贝。<br>
在浅拷贝时候要注意，原对象中的可变元素是拷贝引用，修改会影响原有数据！<br>
什么意思呢，举个简单例子：<br>
```
a = (1, [1, 2], {1, 2}, 4)
b = list(a)
b[1].append(3)
b[2].add(3)
b.append(5)
print(a)
print(b)
```
这个时候的打印结果会是:<br>
```
(1, [1, 2, 3], {1, 2, 3}, 4)
[1, [1, 2, 3], {1, 2, 3}, 4, 5]
```
可以发现在单纯对b进行append的时候，a是不会发生改变的，但是对里面的list和set做操作时，原有数据就会发生改变。<br>
__注意：list()方法和列表生成器、字典生成器都属于浅拷贝。__<br>

## 2 装饰器
首先要知道装饰器是干什么的，我理解的装饰器就是在不干涉原有代码运行的前提下，做一些你想要做的事情，比如获取程序的运行时间、打印日志等等。
```
def do_something(func):
    def print_content():
        print('do something before')
        f = func()
        print('do something after')
        return f

    return print_content


@do_something
def hello_world():
    print('Hello world!')


if __name__ == '__main__':
    hello_world()
```
运行程序，控制台打印内容：
```
do something before
Hello world!
do something after
```
这就是一个很简单的装饰器，可以看到装饰器实际上是把被装饰的方法作为参数放到装饰器中运行，在print_content()中将参数方法返回给外层，在外层再将print_content返回给装饰器就可以了。<br>
__这里可能会感觉装饰器的写法比较像闭包，其实装饰器就是闭包，返回的是方法而不是结果。__<br>

## 3 Python协议
这个是在面试某个公司，面试官给的一道题目里接触到的。<br>
面试让我实现一个对象，创建这个对象的时候传一个带有空格的字符串例如"Hello world!"，创建的这个对象要能用for循环取出空格分割的"Hello"和"world"，用len()能得到分割后的长度。<br>
这个我当时知道意思是让我模拟列表、元组那些实现一个迭代器，虽然面试官有提醒我根据Python的协议去做，但具体怎么去实现，我当时毫无头绪，面试也就理所当然的挂了。<br>
下来之后查了一下，Python的对象是有一些自带的方法的，那个就是Python的协议。<br>
比如一个类你写了__init__就是初始化的方法，而__new__就是新建对象的时候调用。<br>
为什么list能被for循环，是因为内部实现了__iter__和__next__，而len()则是实现了__len__，这个感兴趣的可以下来看看。<br>

# 数据库
## 1 MySQL
### 1 事务
事务的基本原则：原子性、一致性、隔离性和持久性。<br>
事务的隔离级别：读未提交、读已提交、可重复读和串行化。<br>
事务的并发问题：脏读、不可重复读和幻读，事务的隔离级别就是为了解决这三个问题的。<br>
更加具体的可以看这里：[MySQL的四种事务隔离级别](https://www.cnblogs.com/wyaokai/p/10921323.html)

### 2 索引
这里说的索引指的是索引的数据结构，MySQL的索引有B+树和HASH，__重点考察B+树。__<br>
对于树这种数据结构陌生的朋友可以看：[MySQL索引-B+树（看完你就明白了）](http://www.liuzk.com/410.html)，我个人觉得讲解的非常易懂。<br>
<br>
从这里又引申出了聚簇索引和非聚簇索引，__只有InnoDB的主键索引才是聚簇索引，InnoDB中的辅助索引以及MyISAM使用的都是非聚簇索引__。<br>
具体什么时候聚簇什么又是非聚簇，上面的文章中也讲的很清楚。<br>
<br>
__什么是最左匹配原则和联合索引：__
由多个字段建立的索引就叫做联合索引或多列索引<br>
最左前缀匹配原则顾名思义，就是最左优先，在创建多列索引时，要根据业务需求，where子句中使用最频繁的一列放在最左边。<br>
最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。<br>
=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。<br>
## 2 Redis
```
首先Redis是一个key-value的内存数据库，读写都是在内存中完成的，这是背景。<br>
```
### 1 优缺点
这个问题我在面试腾讯的时候问到过，你敢信他真的就是问Redis有什么优缺点。<br>
<br>
优点：1.读写速度快 2.能够快速的进行主从备份 3.数据持久化<br>
<br>
缺点：因为是内存服务器，所以更容易受到物理内存的限制，对于数据量过大的时候难以支撑，所以使用场景有限。<br>

### 2 持久化
Redis是内存服务器，所以在服务器异常关闭的时候数据会丢失，所以需要对数据做持久化。<br>
Redis持久化的两种策略：RDB和AOF，具体的看这里：[详解Redis中两种持久化机制RDB和AOF](https://baijiahao.baidu.com/s?id=1654694618189745916)<br>

### 3 为什么Redis是单线程还那么快，怎么解决并发的？
Redis是基于内存的读写速度快，采用的模型是I/O多路复用，并且利用消息将并发访问变成了串行访问，减少了串行控制的开销。<br>

### 4 缓存穿透、缓存击穿和缓存雪崩
背景：Redis作为缓存，先请求Redis，Redis没有的数据才去请求数据库。<br>
<br>
缓存穿透：请求的数据在缓存和数据库当中都没有，这样请求会不断落在数据库上，仿佛缓存不存在一样，并发高了容易把数据库打死。<br>
解决办法一是对请求过来的数据做校验防止恶意攻击，二是数据库不存在的数据也会给一个表示没有数据的缓存，设置一个过期时间，过期了才再次请求。判断数据是否存在可以用布隆过滤器<br>
<br>
缓存击穿：某个访问量较大的热点数据过期，请求直接打到数据库上，导致数据库直接被打死。<br>
解决办法是对于热点数据设置永不过期。<br>
<br>
缓存雪崩：大量的key在某个时间点刚好同时过期，这时候所有的请求都直接打在了数据库上，导致数据库被打死。<br>
解决办法是在批量添加key的时候设置随机过期时间，避免同一时间大量的key一起失效。

## MongoDB
不知道是用mongo的企业少，还是mongo没什么好问的，目前面试还没遇到过问mongo相关的，顶多问一句有用过MongoDB这个数据库么。<br>

# 网络
几乎没遇到问的，好像记得就有一个吻了rpc相关，我这方面比较弱，没回答上来。<br>

# 操作系统
## 1 进程和线程
1.进程：在操作系统中，能够独立运行，并且作为资源分配的基本单位。它表示运行中的程序。<br>
2.线程：是进程中的一个实例，作为系统调度和分派的基本单位。是进程中的一段序列，能够完成进程中的一个功能。<br>
<br>
两者的区别和联系：<br>
```
1.同一个进程可以包含多个线程，一个进程中至少包含一个线程，一个线程只能存在于一个进程中。

2.同一个进程下的所有线程能够共享该进程下的资源

3.进程结束后，该进程下的所有线程将销毁，而一个线程的结束不会影响同一进程下的其他线程

4.线程是轻量级的进程，它的创建和销毁所需要的时间比进程小得多，所有操作系统的执行功能都是通过创建线程去完成的

5.线程在执行时是同步和互斥的，因为他们共享同一个进程下的资源。

6.在操作系统中，进程是拥有系统资源的独立单元，它可以拥有自己的资源。一般而言，线程不能拥有自己的资源，但是它能够访问其隶属进程的资源
```
例子：如果把计算机的操作系统比作一个大的工厂，那么进程就是这个工厂中的各个相互独立的车间，线程指的是车间中的流水线工人。每个车间中至少有一个工人，一个车间也也可以有多个工人，他们共享这个车间中的所有资源。也就是说，一个进程中可以有多个线程，但一个进程中至少有一个线程，他们共享这个进程下的所有资源。

## 2 I/O模型
I/O操作的四种模型：阻塞I/O、非阻塞I/O、I/O多路复用和异步I/O。 [IO模式和IO多路复用](https://www.cnblogs.com/zingp/p/6863170.html)<br>
其中I/O多路复用又有三种实现：select，poll和epoll。 [select、poll、epoll之间的区别](https://www.cnblogs.com/Anker/p/3265058.html)

# 高并发解决方案
我没有做过并发量很大的业务，只能说下自己的理解：分布式+集群+消息队列。<br>
首先说下概念，分布式是指把一个系统拆分成多个子系统来部署在不同的服务器上，集群是指把同一个业务部署在不同的机器上。<br>
__分布式__ 让业务更加细化，这样单一业务的请求相应的就会降低。<br>
__集群__ 则是通过多台服务器处理请求，提高单一业务能够承受的并发量，同时也能让系统变的高可用，其中一台机器出问题也不会影响业务。<br>
__消息队列__ 是把请求拿过来之后，将请求以队伍形式排列分发，起到削峰的作用。

# 算法相关
目前没有问到几个算法，掌握一下二分查找、快排、堆排还有深搜和广搜足矣，具体的算法自行百度。
