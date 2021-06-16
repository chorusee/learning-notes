## Redis数据结构

Redis不是简单的string - string键值对存储，其中的值域不仅仅是简单的string，还可能是复杂的数据结构。Redis支持的数据结构包括：

+ 二进制安全的字符串（Binary-safe string）

+ 列表（List）：有序，主要是链表

+ 集合（Set）：元素不重复，无序

+ 有序集（Sorted set）：类似于集合，但是每一个元素关联一个浮点数，称为score；按score排序，有序；元素都是string类型

+ 哈希（Hash）：键值对组成的一组映射，键、值都是string类型，类似Python的字典和Java的Map

+ 位图（Bitmap）：可以将string当成比特数组处理，设置或清楚单个比特、所有比特都设为1等操作

+ HyperLogLog：用来估算集合的基的数据结构

+ 地理位置（Geospatial）：地点名，带有经纬度；可用来估算两地距离，寻找附近的人等

#### 对于键的建议

Redis的键是二进制安全的，即可以用任何的二进制序列来作键，如字符串 "foo"，或者是一个JPEG文件。空字符串也是合法的键。

+ 不要用太长的键。因为太长的键不仅占用较大的存储空间，在数据集中查找该键是也要花费较大的代价。如果手中的键太长，可以取哈希值。

+ 太短的键也不太好。短的缩写的键如 "u1000flw" 看起来有点意义不明，写成 "user:1000:followers" 可读性好很多。

+ 试着遵守一种模式。例如 "object-type:id" 就很不错，就像 "user:1000" 这样，见文知义。点 "." 和连接符 "-" 在多个单词组成的域中很常用。

+ 最大允许512M

###  底层数据结构

#### 字符串

Redis的字符串不是C语言中的字符串（以'\0'结尾的数组），它是自己构建的一种名为简单动态字符串（Simple Dynamic String，SDS）的数据结构。

```c
struct sdshdr{
     //记录buf数组中已使用字节的数量
     //等于 SDS 保存字符串的长度
     int len;
     //记录 buf 数组中未使用字节的数量
     int free;
     //字节数组，用于保存字符串
     char buf[];
}
```

![redis数组实现](/res/redis数组实现.png)

优点：

+ 常数复杂度获取字符串长度
+ 杜绝了缓冲区溢出，拼接两个字符串的时候先检查free属性判断是否有足够的空间
+ 减少修改字符串的内存重新分配次数，因为使用了空间预分配和惰性空间释放
+ 二进制安全
+ 兼容部分C字符串函数

#### 链表

```c
typedef  struct listNode{
       //前趋节点
       struct listNode *prev;
       //后继节点
       struct listNode *next;
       //节点的值
       void *value;  
}listNode;
    
typedef struct list{
     //表头节点
     listNode *head;
     //表尾节点
     listNode *tail;
     //链表所包含的节点数量
     unsigned long len;
     //节点值复制函数
     void (*free) (void *ptr);
     //节点值释放函数
     void (*free) (void *ptr);
     //节点值对比函数
     int (*match) (void *ptr,void *key);
}list;
```

Redis链表特性：

+ 双端：链表有头节点和尾节点的引用， 获取这两个节点的时间复杂度为O(1)；
+ 无环：头节点的prev指针和尾节点的next指针都指向NULL；
+ 带链表长度计数器：获取链表长度时间复杂度为O(1)；
+ 多态：链表节点通过`void *`来保存节点值

#### 字典

```c
typedef struct dictEntry{
     //键
     void *key;
     //值
     union{
          void *val;
          uint64_tu64;
          int64_ts64;
     }v;
 
     //指向下一个哈希表节点，形成链表
     struct dictEntry *next;
}dictEntry;
    
typedef struct dictht{
     //哈希表数组
     dictEntry **table;
     //哈希表大小
     unsigned long size;
     //哈希表大小掩码，用于计算索引值
     //总是等于 size-1
     unsigned long sizemask;
     //该哈希表已有节点的数量
     unsigned long used;
 
}dictht;
```

key保存键，val保存值，值可以是一个指针，或整数。

![redis哈希表](/res/redis哈希表.png)

1. hash算法

2. 解决哈希冲突：拉链法

3. 扩容和收缩：rehash步骤：

   1）基于原哈希表创建一个新的大小为已有节点数量2倍的哈希表；

   2）利用哈希算法计算哈希值，将键值对放在新的位置；

   3）所有键值对迁徙完毕后，释放原哈希表空间。

4. 触发扩容条件：

   1）服务器**没有**执行BGSAVE或BGREWRITEAOF，并且负载因子大于等于1（负载因子=used/size）；

   2）服务器**正在**执行BGSAVE或BGREWRITEAOF，并且负载因子大于等于5。

5. 渐进式rehash

   扩容和收缩不是一次性、集中式完成的，而是分多次、渐进式完成的。

   渐进式rehash期间，字典删除、查找、更新等操作可能在两个哈希表上进行，在第一个哈希表上没找到，就会去第二个哈希表上找。但新增操作肯定是在新表上进行。

#### 跳跃表

跳跃表（Skip List）是一种有序的树结构，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。跳跃表具有以下性质：

1. 由多层结构组成；
2. 每一层都是一个有序链表，排列顺序由高层到低层，每层至少包含两个链表节点，分别是前面的head节点和后面的nil节点；
3. 最底层的节点包含所有的元素；
4. 如果一个元素出现在某层，那么在该层之下的链表中也会出现（上一层元素是下一层的子集）；
5. 链表中的每个节点包含两个指针，一个指向同一层的下一个链表节点，另一个指向下一层的同一个链表节点。

![redis跳跃表](/res/redis跳跃表.png)

```c
typedef struct zskiplistNode {
     //层
     struct zskiplistLevel{
           //前进指针
           struct zskiplistNode *forward;
           //跨度
           unsigned int span;
     }level[];
 
     //后退指针
     struct zskiplistNode *backward;
     //分值
     double score;
     //成员对象
     robj *obj;
 
} zskiplistNode;
    
typedef struct zskiplist{
     //表头节点和表尾节点
     structz skiplistNode *header, *tail;
     //表中节点的数量
     unsigned long length;
     //表中层数最大的节点的层数
     int level;
 
}zskiplist;
```

![redis跳跃表实现](/res/redis跳跃表实现.png)

+ 查找

  上层大的节点相当于下层节点的索引。从最上一层开始查找，找到离查找值最近的两个节点，即是一个区间。根据这个区间往下找，一层一层缩小区间，找到则返回，没找到则返空。

+ 插入

  首先确定要插入的层数，节点要插入该层往下的所有链表。

+ 删除

  从各层中找到该节点，让后将该节点从各层链表删除即可。

#### 整数集合

整数集合（intset）是Redis用于保存整数值的集合抽象数据类型，它可以保存类型为`int16_t`，`int32_t`，`int64_t`的整数值，并保证集合中不会出现重复元素。

```c
typedef struct intset{
     //编码方式
     uint32_t encoding;
     //集合包含的元素数量
     uint32_t length;
     //保存元素的数组
     int8_t contents[];
 
}intset;
```

整数集合中的每个元素都是contents数组的一个数据项，他们按照从小到大的顺序排序，并且不包含任何重复项。

虽然contents数组声明为`int_8`类型，但实际上真正的类型由encoding来决定。

升级：当新增的元素类型比原集合元素类型长度要大时，需要对集合进行升级：

1. 根据新元素类型，扩展整数集合底层数组的大小，并为新数组分配空间；

2. 将集合中原数据都转换成新的类型，并将其放入到新数组；

3. 将新元素添加到整数集合中，保证有序。

#### 压缩列表

压缩列表（ziplist）是Redis为了节省内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型数据结构。一个压缩列表可以包含任意多各节点（entry），每个节点可以保存一个字节数组或者一个整数值。

**压缩列表并不是对数据利用某种算法进行压缩，而是将数据按照一定规则编码在一块连续的内存区域，目的是为了节省内存。**

![redis压缩列表](/res/redis压缩列表.png)

![redis压缩列表节点构成](/res/redis压缩列表节点构成.png)

+ previous_entry_length：记录压缩列表前一个节点的长度。利用这个值可以计算出前一个结点的起始位置，压缩列表可以从尾部向头部遍历。
+ encoding：节点的content的类型及长度。
+ content：用于保存节点的内容，节点内容类型和长度由encoding决定。

### 五大数据类型的实现原理

Redis每创建一个键值对，都会创建一个键对象和一个值对象，而Redis的对象是用`redisObject`结构来表示的：

```c
typedef struct redisObject{
     //类型
     unsigned type:4;
     //编码
     unsigned encoding:4;
     //指向底层数据结构的指针
     void *ptr;
     //引用计数
     int refcount;
     //记录最后一次被程序访问的时间
     unsigned lru:22;
 
}robj;
```

1. type属性

   ![redis对象type属性](/res/redis对象type属性.png)

   在Redis中键总是一个字符串对象。

2. encoding属性和*ptr指针

   对象的ptr指针指向对象的底层数据结构，而数据结构类型由encoding属性决定。

   ![redis对象encoding属性](/res/redis对象encoding属性.png)

#### 五大基本数据类型的底层实现

![redis五大基本数据类型的底层实现](/res/redis五大基本数据类型的底层实现.png)

#### 字符串的底层实现

+ int：保存整数值
+ embstr：保存短字符串
+ raw：保存长字符串

都是由SDS存储。

#### 列表的底层实现

+ 压缩列表存储，需满足：
  1. 列表保存元素个数小于512个；
  2. 每个元素长度小于64字节；

+ 链表存储

#### 哈希的底层实现

+ 压缩列表存储，需满足：

  1. 保存的元素小于512个；
  2. 每个元素长度小于64字节。

  当使用压缩链表存储时，直接将新增的键值对，键和值分别作为一个元素加入到压缩列表尾。

  ![redis哈希对象压缩列表底层实现](/res/redis哈希对象压缩列表.png)

+ 哈希表存储

  ![redis哈希对象哈希表底层实现](/res/redis哈希对象哈希表.png)

相对于字典数据结构，压缩列表用于元素个数少、元素长度小的场景。其优势在于集中存储、节省空间。

#### 集合的底层实现

集合是String类型元素的无序集合。集合和列表的区别：1. 集合中的元素无序，不能通过索引操作元素；2.集合中的元素不能重复。

+ intset，需满足：

  1. 集合对象中所有元素都是整数；
  2. 集合对象中元素数量不超过512。

  ![redis集合对象intset](/res/redis集合对象intset.png)

+ hashtable

  使用哈希表作为底层实现，哈希表中的每个键都是字符串对象，这个键就是集合中的元素；而值全部设为NULL。

  ![redis集合对象hashtable](/res/redis集合对象hashtable.png)

#### 有序集合的底层实现

+ 压缩列表，需满足：

  1. 保存的元素数量小于128；
  2. 保存的所有元素长度小于64字节。

  使用压缩列表作为底层实现，每个集合元素使用任意两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员，第二个节点保存元素的score。

  并且压缩列表内的集合元素按score从小到大的顺序排列。

  ![redis有序集合对象压缩列表](/res/redis有序集合对象压缩列表.png)

+ 跳跃表

  使用跳跃表存储，用zset结构，一个zset结构同时包含一个字典和一个跳跃表：

  ```c
  typedef struct zset{
       //跳跃表
       zskiplist *zsl;
       //字典
       dict *dice;
  } zset;
  ```

  字典：key保存元素的值，value保存元素的score。

  跳跃表：obj属性保存元素的成员，score属性保存元素的score。

  这两种数据结构会通过指针共享相同元素的成员和score，保持一致性。

  使用两种数据结构是为了能同时以较低的时间复杂度查找元素和排序。

  单独使用字典能实现查找时间复杂度O(1)，但每次执行范围操作都要排序；

  单独使用跳表虽然执行范围操作O(log n)，但查找操作由O(1)升为O(log n)。

#### 五种数据类型的使用场景

1. string
   + 计数器，如库存、访问量
   + 因为是二进制安全的，可以用来缓存图片
2. hash
   + 存放对象
3. list
   + 实现简单的消息队列
4. set
   + 字典实现，查找特别快，且数据不允许重复，用来全局去重，如判断用户名是否已被占用
   + 交、并、差操作，计算共同好友等
5. zset
   + 排行榜
