# 手撕： 带过期时间的LRU

## **题目分析**
本题要求在 **LRU（Least Recently Used）缓存** 的基础上增加 **过期时间**，即：

1. `get(key)`:
    - 如果 `key` 存在且**不过期**，返回 `value`，并更新其为最近使用。
    - 如果 `key` 过期或不存在，返回 `-1`。
2. `put(key, value, expireTime)`:
    - 若 `key` 存在，则更新其 `value` 和 `expireTime`，并移动到头部。
    - 若 `key` 不存在，则插入新数据，并设置过期时间。
    - 若缓存超过容量，则淘汰**最久未使用**的元素。
3. **数据自动过期**：
    - 每次 `get` 时检查数据是否过期，过期则删除。

## **数据结构**
1. **哈希表（HashMap）** 存储 `key → Node`，快速查找 `O(1)`;
2. **双向链表（LinkedList）** 维护访问顺序，最近使用的在头部，最久未使用的在尾部；
3. **每个节点存储 **`expireTime`，判断是否过期。

---

## **代码实现（ACM模式）**
```java
import java.util.*;

// LRU缓存实现（带过期时间）
class LRUCache {
    // 双向链表节点
    private static class Node {
        int key, value;
        long expireTime; // 过期时间（时间戳）
        Node prev, next;
        
        Node(int key, int value, long expireTime) {
            this.key = key;
            this.value = value;
            this.expireTime = expireTime;
        }
    }

    private final int capacity; // 缓存容量
    private final Map<Integer, Node> map; // 存储键值对
    private final Node head, tail; // 维护双向链表的头尾节点
    
    // 初始化LRU缓存
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        
        // 伪头和伪尾节点，便于操作
        head = new Node(-1, -1, -1);
        tail = new Node(-1, -1, -1);
        head.next = tail;
        tail.prev = head;
    }

    // 获取值
    public int get(int key) {
        if (!map.containsKey(key)) return -1; // 不存在返回-1
        
        Node node = map.get(key);
        long currentTime = System.currentTimeMillis();
        
        // 过期处理
        if (node.expireTime < currentTime) {
            removeNode(node);
            map.remove(key);
            return -1;
        }
        
        moveToHead(node); // 访问后更新最近使用
        return node.value;
    }

    // 插入或更新值（带过期时间）
    public void put(int key, int value, int ttl) {
        long expireTime = System.currentTimeMillis() + ttl * 1000L; // 计算过期时间
        
        if (map.containsKey(key)) {
            Node node = map.get(key);
            node.value = value;
            node.expireTime = expireTime;
            moveToHead(node);
        } else {
            Node newNode = new Node(key, value, expireTime);
            map.put(key, newNode);
            addToHead(newNode);
            
            if (map.size() > capacity) {
                removeTail();
            }
        }
    }

    // 移动节点到头部（最近使用）
    private void moveToHead(Node node) {
        removeNode(node);
        addToHead(node);
    }

    // 删除节点
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    // 将节点添加到头部
    private void addToHead(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    // 删除尾部节点（最久未使用）
    private void removeTail() {
        Node last = tail.prev;
        removeNode(last);
        map.remove(last.key);
    }
}

// ACM输入输出格式
public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int capacity = scanner.nextInt(); // 读取LRU缓存容量
        LRUCache lru = new LRUCache(capacity);
        
        int operations = scanner.nextInt(); // 读取操作数
        for (int i = 0; i < operations; i++) {
            String op = scanner.next();
            if (op.equals("get")) {
                int key = scanner.nextInt();
                System.out.println(lru.get(key));
            } else if (op.equals("put")) {
                int key = scanner.nextInt();
                int value = scanner.nextInt();
                int ttl = scanner.nextInt(); // 过期时间（秒）
                lru.put(key, value, ttl);
            }
        }
        
        scanner.close();
    }
}
```

---

## **时间与空间复杂度分析**
### **时间复杂度**
+ `get(key)`：
    - 哈希表查询 `O(1)`
    - 链表操作（移动到头部）`O(1)`
    - **总复杂度：O(1)**
+ `put(key, value, ttl)`：
    - 哈希表插入 `O(1)`
    - 链表插入 `O(1)`
    - **总复杂度：O(1)**

### **空间复杂度**
+ **哈希表存储 **`O(capacity)`
+ **双向链表存储 **`O(capacity)`
+ **总复杂度：O(capacity)**

---

## **输入 & 输出示例**
### **输入**
```plain
2
6
put 1 100 3
put 2 200 5
get 1
put 3 300 4
get 2
get 1
```

### **输出**
```plain
100
200
-1
```

### **解释**
1. `put(1, 100, 3)` → 插入 `{1=100}`，过期时间 `+3` 秒。
2. `put(2, 200, 5)` → 插入 `{1=100, 2=200}`，过期时间 `+5` 秒。
3. `get(1)` → `1` 没有过期，返回 `100`，移动到头部。
4. `put(3, 300, 4)` → 插入 `{1=100, 2=200, 3=300}`。
5. `get(2)` → `2` 没有过期，返回 `200`，移动到头部。
6. `get(1)` → `1` 过期，删除并返回 `-1`。

---

## **总结**
1. **哈希表 + 双向链表** 是最优解，保证 **O(1) 查找 + O(1) 维护顺序**。
2. **新增 **`expireTime`** 字段**，用于判断数据是否过期。
3. `get`** 时检查过期**，删除已过期数据，提高缓存利用率。

这是一种高效的 **LRU缓存+过期时间** 解决方案！🎯



> 更新: 2025-03-12 14:14:47  
> 原文: <https://www.yuque.com/neumx/ko4psh/uwulwvd7pwoxa5o4>