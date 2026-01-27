**LRU 缓存（Least Recently Used，最近最少使用）**
是一种常见的缓存淘汰策略：当缓存满了，需要删除**最久未被访问**的数据，把空间留给新数据。

---

## 核心思想

* **访问过的数据是“新鲜”的**，短期内更可能再次被访问
* **很久没被访问的数据优先淘汰**

---

## 常见实现方式（O(1)）

**哈希表 + 双向链表**

* **哈希表**：`key → 节点`，O(1) 查找
* **双向链表**：

  * 头部：最近使用
  * 尾部：最久未使用（淘汰对象）

### 操作规则

* `Get(key)`

  * 存在：返回值，并把节点移动到链表头
  * 不存在：返回 -1
* `Put(key, value)`

  * 已存在：更新值，移动到头部
  * 不存在：

    * 若已满：删除尾部节点（LRU）
    * 插入新节点到头部

---

## Go 实现示例（经典写法）

```go
type Node struct {
	key, value int
	prev, next  *Node
}

type LRUCache struct {
	capacity int
	cache    map[int]*Node
	head     *Node // dummy head
	tail     *Node // dummy tail
}

func Constructor(capacity int) LRUCache {
	head := &Node{}
	tail := &Node{}
	head.next = tail
	tail.prev = head

	return LRUCache{
		capacity: capacity,
		cache:    make(map[int]*Node),
		head:     head,
		tail:     tail,
	}
}

func (l *LRUCache) Get(key int) int {
	if node, ok := l.cache[key]; ok {
		l.moveToHead(node)
		return node.value
	}
	return -1
}

func (l *LRUCache) Put(key int, value int) {
	if node, ok := l.cache[key]; ok {
		node.value = value
		l.moveToHead(node)
		return
	}

	if len(l.cache) >= l.capacity {
		lru := l.tail.prev
		l.removeNode(lru)
		delete(l.cache, lru.key)
	}

	newNode := &Node{key: key, value: value}
	l.cache[key] = newNode
	l.addToHead(newNode)
}

func (l *LRUCache) addToHead(node *Node) {
	node.prev = l.head
	node.next = l.head.next
	l.head.next.prev = node
	l.head.next = node
}

func (l *LRUCache) removeNode(node *Node) {
	node.prev.next = node.next
	node.next.prev = node.prev
}

func (l *LRUCache) moveToHead(node *Node) {
	l.removeNode(node)
	l.addToHead(node)
}
```

---

## 时间 & 空间复杂度

* `Get / Put`：**O(1)**
* 空间复杂度：**O(capacity)**

---

## 常见应用场景

* 浏览器缓存
* 数据库 / Redis 本地缓存
* 操作系统页面置换
* API / 图像 / 文件缓存

如果你愿意，我也可以：

* 给你一个 **面试讲解版**
* 改成 **泛型 LRU**
* 或写一个 **并发安全（加锁）版本**
