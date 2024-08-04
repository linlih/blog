---
title: "【数据结构】跳表"
description: "data_struct_skiplist"
keywords: "data_struct_skiplist"

date: 2024-08-04T23:50:35+08:00
lastmod: 2024-08-04T23:50:35+08:00

categories:
 -
tags:
  -
  -

# Post's origin author name
#author:
# Post's origin link URL
#link:
# Image source link that will use in open graph and twitter card
#imgs:
# Expand content on the home page
#expand: true
# It's means that will redirecting to external links
#extlink:
# Disabled comment plugins in this post
#comment:
# enable: false
# Disable table of content int this post
# Notice: By default will automatic build table of content 
# with h2-h4 title in post and without other settings
#toc: false
# Absolute link for visit
#url: "data_struct_skiplist.html"
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

# 跳表

跳表是有序链表，为了加快查找速度，建立多层的索引链表。

使用的例子是：
```shell
header --------------------------> 6  
header -----------> 3 -----------> 6 -> 7 ------> 9 
header -> 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10
```

结合代码说明查找、删除、插入的过程：

跳表中每个节点的结构：
```go
type Skipnode struct {
	Key        uint32      // 用于排序对比
	Val        interface{} // 实际节点存储的内容
	LevelEntry []*Skipnode // 存储每一层指向下一个节点的指针
	Level      int         // 当前节点的创建高度，在删除的时候会用到
}
```

查找算法：
```shell
2[header]-------------------------->[6]  
1 header -----------> 3 ----------->[6]->[7]------> 9 
0 header -> 1 -> 2 -> 3 -> 4 -> 5 -> 6 ->[7]-> 8 -> 9 -> 10
```
假设要查找的是8，查找路径对应[]标注的数值。从高度最高的地方开始查找，也就是上图中的从第2层开始查找，从header开始，然后到节点6，这个时候6的下一个节点是空，就会进入下一层循环。这个时候已经在第1层节点6的位置了，直接比较第1层对的7节点，满足小于8，这个时候判断节点7的下一个节点9，大于8了。这个时候就进入下一层，也就是第0层进行查找，第0层的7下一个节点就是8了，不满足Key<searchKey，跳出循环。

当循环结束的时候，currentNode最终是在第0层的7节点上，判断7的下一个节点是否是要查找的值即可。
```go
func (s *Skiplist) Search(searchKey uint32) (interface{}, error) {
	currentNode := s.Header
	for i := s.Level - 1; i >= 0; i-- {
		for currentNode.LevelEntry[i] != nil && currentNode.LevelEntry[i].Key < searchKey {
			currentNode = currentNode.LevelEntry[i] // 指向下一个节点
		}
	}

	currentNode = currentNode.LevelEntry[0] // 指向完整链表的下一个元素
	if currentNode != nil && currentNode.Key == searchKey {
		return currentNode.Val, nil
	}
	return nil, errors.New("not Found")
}
```
删除算法：
```shell
2[header]--------------------------> 6  
1 header ----------->[3]-----------> 6 -> 7 ------> 9 
0 header -> 1 -> 2 -> 3 -> 4 ->[5]-> 6 -> 7 -> 8 -> 9 -> 10
```
以删除节点6为例子，删除过程首先要记录删除节点的前置节点信息，也就是这里的updateList数组，在上图中[]标注的节点，就是最终updateList数组中存放的节点，下表index则表示所在的层。

前面的过程和查找过程是类似的，增加了说更新updateList节点指向要删除节点的下一个节点即可。而且，如果要删除的节点是最高层的最后一个节点，比如这里删除的是6，那么相应的就要减小当前跳表的高度。

删除结果：
```shell
1 header -----------> 3 -----------> 7 ------> 9
0 header -> 1 -> 2 -> 3 -> 4 -> 5 -> 7 -> 8 -> 9 -> 10
```

```go
func (s *Skiplist) Delete(searchKey uint32) error {
	updateList := make([]*Skipnode, s.MaxLevel)
	currentNode := s.Header

	for i := s.Header.Level - 1; i >= 0; i-- {
		for currentNode.LevelEntry[i] != nil && currentNode.LevelEntry[i].Key < searchKey {
			currentNode = currentNode.LevelEntry[i]
		}
		updateList[i] = currentNode
	}

	currentNode = currentNode.LevelEntry[0]
	if currentNode.Key == searchKey {
		// 这里使用是currentNode.Level可以减少一定的循环次数，如果用s.Level也是可以的
		for i := 0; i < currentNode.Level; i++ {
			updateList[i].LevelEntry[i] = currentNode.LevelEntry[i]
		}
		for s.Level > 1 && s.Header.LevelEntry[s.Level-1] == nil {
			s.Level--
		}
		currentNode = nil
		return nil
	}
	return errors.New("not found")
}
```

插入算法：

假设现在的跳表结构为，要插入的数据是6，并且假设要插入节点的高度为3，对应代码中的newLevel
```shell
1 header ----------->[3]-----------> 7 ------> 9
0 header -> 1 -> 2 -> 3 -> 4 ->[5]-> 7 -> 8 -> 9 -> 10
```

根据上面的删除算法的过程，我们可以推论得到updateList为上图中[]标注的节点。

找到插入的位置之后，如果当前节点高于现有跳表的高度，增加updateList的节点，在这里也就是增加了第三层的header节点。然后创建新的节点，更新每一层的链表指向即可。

插入结果：

```shell
2[header]--------------------------> 6  
1 header ----------->[3]-----------> 6 -> 7 ------> 9 
0 header -> 1 -> 2 -> 3 -> 4 ->[5]-> 6 -> 7 -> 8 -> 9 -> 10
```

```go
func (s *Skiplist) Insert(searchKey uint32, value interface{}) {
	updateList := make([]*Skipnode, s.MaxLevel)
	currentNode := s.Header

	// 找到插入的位置
	for i := s.Header.Level - 1; i >= 0; i-- {
		for currentNode.LevelEntry[i] != nil && currentNode.LevelEntry[i].Key < searchKey {
			currentNode = currentNode.LevelEntry[i]
		}
		updateList[i] = currentNode // 记录每一层的最后满足的节点
	}
	currentNode = currentNode.LevelEntry[0]

	// 节点已经存在则更新Val
	if currentNode != nil && currentNode.Key == searchKey {
		currentNode.Val = value
	} else {
		newLevel := s.RandomLevel() // 生成随机的level高度，高度范围是：[0, MaxLevel-1]
		if newLevel > s.Level { // 如果高度超过了现在的高度
			for i := s.Level + 1; i <= newLevel; i++ { // 超过当前高度的第一个节点都是s.Header
				updateList[i-1] = s.Header
			}
			s.Level = newLevel
			s.Header.Level = newLevel
		}
		newNode := NewNode(searchKey, value, newLevel, s.MaxLevel)
		// 更新链表指向
		for i := 0; i <= newLevel-1; i++ {
			newNode.LevelEntry[i] = updateList[i].LevelEntry[i]
			updateList[i].LevelEntry[i] = newNode
		}
	}
}
```


代码参考自：
https://github.com/kkdai/skiplist/blob/master/skiplist.go

这个实现代码有错误，具体错误是在删除Delete函数中，if中的第二个for循环，对currentNode.Level进行--操作，是没有必要的，因为currentNode节点是要删除的，让Golang自动回收其内存。所以这里的本意应该是说如果该删除的节点是最高层的唯一链表节点，那么当前跳表的高度减1.
```go
if currentNode.Key == searchKey {
    for i := 0; i <= currentNode.Level-1; i++ {
        if updateList[i].Forward[i] != nil && updateList[i].Forward[i].Key != currentNode.Key {
            break
        }
        updateList[i].Forward[i] = currentNode.Forward[i]
    }

    for currentNode.Level > 1 && b.Header.Forward[currentNode.Level] == nil {
        currentNode.Level--
    }

    //free(currentNode)  //no need for Golang because GC
    currentNode = nil
    return nil
}
```






