# Python应用之基础结构：链表- 删除排序链表中的重复元素_10月月更_向阳逐梦_InfoQ写作社区
存在一个按升序排列的链表，给你这个链表的头节点 head ，请你删除所有重复的元素，使每个元素 只出现一次 。返回同样按升序排列的结果链表。

**示例 1:**

![](_assets/f321e1a00594b0b380bcbe91affbd9b0.png)

> **输入:** head = \[1,1,2\]
> 
> **输出:** \[1,2\]

**示例 2:**

![](_assets/a2d426d9de8b12f12b7d25e9b75f194c.png)

> **输入:** head = \[1,1,2,3,3\]
> 
> **输出:** \[1,2,3\]

**示例 3:**

> **输入:** head = \[\]
> 
> **输出:** \[\]

**初始代码**

```
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None
 
class LinkList:
    def __init__(self):
        self.head=None
 
    def initList(self, data):
        while len(data)==0:return None
        self.head = ListNode(data[0])
        r=self.head
        p = self.head
        for i in data[1:]:
            node = ListNode(i)
            p.next = node
            p = p.next
        return r
    def printlist(self,head):
        a=[]
        if head == None: return []
        node = head
        while node != None:
            a.append(node.val)
            node = node.next
        return a
class Solution:
    def deleteDuplicates(self, head: ListNode) -> ListNode:
        

if __name__ == '__main__':
    print(LinkList().printlist(Solution().deleteDuplicates(LinkList().initList([1,1,2]))))
    print(LinkList().printlist(Solution().deleteDuplicates(LinkList().initList([1,1,2,3,3]))))
    print(LinkList().printlist(Solution().deleteDuplicates(LinkList().initList([]))))
```

*   由于给定的链表是排好序的，因此重复的元素在链表中出现的位置是连续的，因此我们只需要对链表进行一次遍历，就可以删除重复的元素。
    
*   当遍历完整个链表之后，我们返回链表的头节点即可。
    

```
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None
 
class LinkList:
    def __init__(self):
        self.head=None
 
    def initList(self, data):
        while len(data)==0:return None
        self.head = ListNode(data[0])
        r=self.head
        p = self.head
        for i in data[1:]:
            node = ListNode(i)
            p.next = node
            p = p.next
        return r
    def printlist(self,head):
        a=[]
        if head == None: return []
        node = head
        while node != None:
            a.append(node.val)
            node = node.next
        return a
class Solution:
    def deleteDuplicates(self, head: ListNode) -> ListNode:
        b=ListNode(0)
        c=b
        while head:
            c.next=head
            while head.next and head.val==head.next.val:
                head.next=head.next.next
            head=head.next
            c=c.next
        return b.next

if __name__ == '__main__':
    print(LinkList().printlist(Solution().deleteDuplicates(LinkList().initList([1,1,2]))))
    print(LinkList().printlist(Solution().deleteDuplicates(LinkList().initList([1,1,2,3,3]))))
    print(LinkList().printlist(Solution().deleteDuplicates(LinkList().initList([]))))
```

**第 1-30，44-47 行：**  题目中已经给出的信息，运行代码时要根据这些代码进行编辑（具体为创建链表以及列表、链表转换）

**第 31 行：**  创建初始链表 b，头指针数值为 0

**第 32 行：**  保存头指针于变量 c 上，移动的是 b 指针

**第 33 行：**  当 head 不为 None 时，循环直到遍历完整个链表（print(None)的结果为 False）

**第 34 行：**  c 链表的 next 指向 head 链表

**第 35 行：**  判断 head.next 是否非空，若非空，再判断其数值是否与 head 相等

**第 36 行：**  若数值相等，跳过 head.next，并将 head.next 之后的链表 head.next.next 进行循环判断

**第 37 行：**  当两者数值不相等时，将 head 指向 head.next（此时 head.next 已经指向与 head 数值不相等的链表上了）

**第 38 行：**  c 指针移到 c.next，用于为 c.next 赋值后面的链表

**第 39 行：**  返回 b.next（q.next 指向新链表的头指针）

代码运行结果为：

![](_assets/92bc688c89fb4652329f6ca0469faa19.png)

这里用到了基础结构：链表，简单讲解下这个链表：

**链表**

链表是一组数据项的集合，其中每个数据项都是一个节点的一部分，每个节点还包含指向下一个节点的链接链表的结构：data 为自定义的数据，next 为下一个节点的地址。

**基本元素**

节点：每个节点有两个部分，左边称为值域，存放用户数据；右边部分称为指针域，用来存放指向下一个元素的指针。

head:head 节点永远指向第一个节点

tail: tail 永远指向最后一个节点

None:链表中最后一个节点的指针域为 None 值