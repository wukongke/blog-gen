---
date: 2017-08-29
title: "golang笔记：游戏中排行榜的实现"
draft: false
categories:
  - golang
  - game
tags:
  - golang
  - game
thumbnailImagePosition: left
---

游戏开发中排行榜经常出现,接触过的排行榜有两种。一种是由玩家挑战排名比自己靠前的其他玩家，胜利后交换位置；另一种是根据玩家的某特性对所有玩家进行排序。第一种只涉及到两个玩家数据的变化，实现起来比较简单，因此只记录第二种情况。<br>

<!--more-->
### 需求
- 排行榜内容是有序的所有玩家信息（以等级为例）
- 玩家等级变化后，更新排行榜
- 根据玩家id获取在排行榜中的排名
- 获取排行榜中特定排名的玩家

### 思路
思路一。如果用关系型数据库的话，实现排行版比较简单，一条SQL即可。由于游戏服务器并发量大、排行榜数据变化频繁，用SQL显然不合适。<br>

思路二。用数组作为排行榜存放玩家信息并对其进行排序。有新玩家加入或玩家等级变化时，重新排序。这样的好处是获取特定排名的玩家时很快，时间复杂度仅仅是O(1)。但是获取某玩家的排名时需要遍历整个数组，时间复杂度是O(n)。有新玩家加入或玩家信息有变化时，需要重新排序，以归并排序为例，其平均时间复杂度为O(nlogn)。每次更新排行榜都需要排序，不断地排序似乎并不优雅。于是考虑定期排序，如每分钟排一次序。<br>

思路三。思路二的情况下，如果某玩家的排名上升了1000名，操作数组就要把被他超越的1000个玩家数据都向后移动。于是考虑了使用链表替代数组。但是这就带来了新的问题，通过排名获取玩家时，就要遍历链表，思路二中这个操作的时间复杂度为O(1)，现在就变成了O(n)。故不能使用链表，或者说不能直接使用普通的链表。<br>

思路四。借助Redis。网上搜索一番，似乎大家都是用Redis的zset来处理排行榜，看了一些介绍觉得靠谱。然鹅，项目中没用使用Redis，为了一个小的功能加上一个Redis似乎不太必要。幸运的是，项目中用到了ssdb，这货号称兼容Redis的api。

完美解决？呵呵。写到一半时发现ssdb文档中关于zrank有这么一句话：

>Important! This method may be extremly SLOW! May not be used in an online service.

### 大招
Redis处理zset用的是一个特殊的跳表，不妨自己实现一个一模一样的。<br>
然而 ~~不会~~ 嫌麻烦。既然Redis是开源的，[照搬过来](https://github.com/XanthusL/zset)就好了。<br>
还没搬完，只是简单实现了几个基本的需求，欢迎PR