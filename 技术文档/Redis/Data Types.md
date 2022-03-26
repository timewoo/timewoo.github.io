# Redis Data Types
## Redis数据存储类型
redis不只是存储简单的key-value类型的数据，同时支持多种不同的数据类型。在传统key-value的存储结构中，主要是存储string类型的key-value，在redis中
value不仅仅支持简单的string类型，同时支持复杂数据结构。redis的value主要支持下面的几种数据类型：  
- 二进制安全的string类型
- List类型，根据插入元素的顺序排序的字符串集合，基本上是链表的数据结构
- Set类型，唯一的，没有排序的string类型集合
- Sorted Set类型，和Set的类型相似，但是每一个string元素都关联了一个float类型的数字，称为sort。元素根据sort值进行排序，因此Sort Set可以获取范围
内的元素，比如前10个或后10个元素  
- Hash类型，key和value都是string类型的map，类似Ruby或者Python的hash
- Bit数组类型，类似BitMap，可以使用特殊的命令来像处理Bit数组一样处理string值，比如可以设置或者清空单个bit，计算所有设置为1的bit数，找到第一个设置
或者未设置的bit位等
- HyperLogLogs，是一种概率的数据结构，用来估计集合的基数，即集合中的不重复数据的个数。在输入元素的数量或体积非常大时，计算基数所需要的空间是固定的、并且
是很小的。
- Streams，Redis 5.0出现的一种数据类型，是一种仅附加(append-only)的类似键值对的集合