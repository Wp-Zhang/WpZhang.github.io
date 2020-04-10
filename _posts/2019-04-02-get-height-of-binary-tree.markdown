---
layout:     post
title:      利用广度优先搜索求二叉树高度
subtitle:   "算法思考"
date:       2019-04-02 08:51:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Algorithm
---
## 题目：利用广度优先搜索求二叉树高度

---

### 定义二叉树

```python

class TreeNode:

	"""

	树节点

	"""

    def __init__(self, data, left, right):

        self.data = data

        self.left = left

        self.right = right



class BinTree:

	"""

	树

	"""

    def __init__(self,data=0):

        self.root = TreeNode(data,None,None)

```



### 方法一

&nbsp; &nbsp; &nbsp; &nbsp; 在检测到当层遍历完成时在节点队列中加入一个分隔节点用于分隔  

  

**Python代码：**

```python

def getHightByBFS1(tree):

    queue = [tree.root,TreeNode("#",None,None)]

    hight = 0

    while queue:

        node = queue.pop(0)

        if node.data == "#":

            hight += 1

            if len(queue)>0:

                queue.append(TreeNode("#",None,None))

        if node.left:

            queue.append(node.left)

        if node.right:

            queue.append(node.right)

    return hight

```



### 方法二

&nbsp; &nbsp; &nbsp; &nbsp; 用两个遍历分别存储下一层的节点数m2和当前层的节点数m1，每次遍历一个节点将m2减1，若该节点有子节点则m1加1，并将子节点加入到节点队列中。当m1减为零时说明当前层遍历完毕，于是高度加1，将m2赋给m1后将m2置0。  

  

**Python代码**：

```python

def getHightByBFS2(tree):

    queue = [tree.root]

    hight = 0

    count1 = 1

    count2 = 0

    while queue:

        node = queue.pop(0)

        if node.left:

            queue.append(node.left)

            count2 += 1

        if node.right:

            queue.append(node.right)

            count2 += 1

        count1 -= 1

        if count1 == 0:

            count1 = count2

            count2 = 0

            hight += 1

    return hight

```