---
title: 数据结构(JAVA)
tags:
  - cs
categories:
  - 学习
abbrlink: 13926
date: 2026-03-18 21:05:43
---
# 线性表: 顺序表

## 1. 顺序表的含义

顺序表（Sequential List）是一种用**连续存储空间**保存线性数据的数据结构。  
你可以把它理解成“可按下标随机访问的动态数组”：

- 逻辑上：元素是线性关系（前后相邻）。
- 物理上：元素在内存中连续存放。

这带来两个典型特点：

- 访问快：可通过下标直接访问，时间复杂度 `O(1)`。
- 插入/删除中间位置较慢：需要搬移后续元素，时间复杂度 `O(n)`。

---

## 2. Java 演示：手写 `ArraysList`

下面是一个简化版顺序表实现，包含你要求的：

- `size`：当前有效元素个数
- `capacity`：当前容量
- `add()`：尾插，满了自动扩容
- `insert()`：按下标插入，满了自动扩容
- `remove()`：按下标删除并返回被删除值
- `toString()`：只输出有效元素（`0 ~ size-1`）

```java
public class ArraysList {
    private int[] data;
    private int size;
    private int capacity;

    public ArraysList() {
        this(10);
    }

    public ArraysList(int initCapacity) {
        if (initCapacity <= 0) {
            throw new IllegalArgumentException("initCapacity must be > 0");
        }
        this.capacity = initCapacity;
        this.data = new int[this.capacity];
        this.size = 0;
    }

    public int getSize() {
        return size;
    }

    public int getCapacity() {
        return capacity;
    }

    public void add(int value) {
        if (size == capacity) {
            grow();
        }
        data[size] = value;
        size++;
    }

    public void insert(int index, int value) {
        rangeCheckForInsert(index);

        if (size == capacity) {
            grow();
        }

        // 从后往前搬移，给 index 位置腾出空位
        for (int i = size; i > index; i--) {
            data[i] = data[i - 1];
        }

        data[index] = value;
        size++;
    }

    public int remove(int index) {
        rangeCheck(index);
        int removedValue = data[index];

        // 将 index 之后的元素整体左移一位
        for (int i = index; i < size - 1; i++) {
            data[i] = data[i + 1];
        }

        size--;
        return removedValue;
    }

    private void grow() {
        int newCapacity = capacity * 2;
        int[] newData = new int[newCapacity];

        for (int i = 0; i < size; i++) {
            newData[i] = data[i];
        }

        data = newData;
        capacity = newCapacity;
    }

    private void rangeCheck(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException(
                "index: " + index + ", size: " + size
            );
        }
    }

    private void rangeCheckForInsert(int index) {
        if (index < 0 || index > size) {
            throw new IndexOutOfBoundsException(
                "index: " + index + ", size: " + size
            );
        }
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("[");
        for (int i = 0; i < size; i++) {
            sb.append(data[i]);
            if (i < size - 1) {
                sb.append(", ");
            }
        }
        sb.append("]");
        return sb.toString();
    }
}
```

### 使用示例

```java
public class ArraysListDemo {
    public static void main(String[] args) {
        ArraysList list = new ArraysList(3);
        list.add(10);
        list.add(20);
        list.add(30);
        list.add(40); // 触发扩容：3 -> 6
        list.insert(2, 25); // 在下标 2 插入 25

        System.out.println(list); // [10, 20, 25, 30, 40]
        System.out.println("size=" + list.getSize() + ", capacity=" + list.getCapacity());

        int removed = list.remove(1); // 删除值 20
        System.out.println("removed=" + removed);
        System.out.println(list); // [10, 25, 30, 40]
    }
}
```

# 线性表: 链表

## 1. 链表的含义

链表（Linked List）是一种**不要求连续内存**的线性结构。  
每个元素都放在一个节点（Node）里，节点包含两部分：

- 数据域：存储当前元素值
- 指针域：保存下一个节点的引用

和顺序表相比，链表的典型特点是：

- 插入/删除更灵活：已知位置时不需要整体搬移元素
- 随机访问较慢：访问第 `k` 个元素需要从头逐个走过去

---

## 2. Java 演示：手写 `LinkedList`（单向链表）

下面是一个单向链表实现，包含 `add()`、`remove()`、`insert()`、`get()`、`toString()` 方法，并带必要注释。

```java
public class LinkedList {
    // 链表节点：保存值 + 指向下一个节点
    private static class Node {
        int value;
        Node next;

        Node(int value) {
            this.value = value;
        }
    }

    private Node head;
    private Node tail;
    private int size;

    public int size() {
        return size;
    }

    // 尾插：追加到链表末尾
    public void add(int value) {
        Node newNode = new Node(value);
        if (head == null) {
            head = newNode;
            tail = newNode;
        } else {
            tail.next = newNode;
            tail = newNode;
        }
        size++;
    }

    // 在指定下标插入，允许 index == size（相当于尾插）
    public void insert(int index, int value) {
        rangeCheckForInsert(index);
        Node newNode = new Node(value);

        if (index == 0) {
            newNode.next = head;
            head = newNode;
            if (size == 0) {
                tail = newNode;
            }
        } else if (index == size) {
            tail.next = newNode;
            tail = newNode;
        } else {
            Node prev = nodeAt(index - 1);
            newNode.next = prev.next;
            prev.next = newNode;
        }
        size++;
    }

    // 删除指定下标并返回删除的值
    public int remove(int index) {
        rangeCheck(index);
        int removedValue;

        if (index == 0) {
            removedValue = head.value;
            head = head.next;
            if (size == 1) {
                tail = null;
            }
        } else {
            Node prev = nodeAt(index - 1);
            Node removed = prev.next;
            removedValue = removed.value;
            prev.next = removed.next;

            // 如果删掉的是尾节点，需要更新 tail
            if (removed == tail) {
                tail = prev;
            }
        }

        size--;
        return removedValue;
    }

    // 获取指定下标元素
    public int get(int index) {
        rangeCheck(index);
        return nodeAt(index).value;
    }

    private Node nodeAt(int index) {
        Node current = head;
        for (int i = 0; i < index; i++) {
            current = current.next;
        }
        return current;
    }

    private void rangeCheck(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException(
                "index: " + index + ", size: " + size
            );
        }
    }

    private void rangeCheckForInsert(int index) {
        if (index < 0 || index > size) {
            throw new IndexOutOfBoundsException(
                "index: " + index + ", size: " + size
            );
        }
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("[");
        Node current = head;
        while (current != null) {
            sb.append(current.value);
            if (current.next != null) {
                sb.append(", ");
            }
            current = current.next;
        }
        sb.append("]");
        return sb.toString();
    }
}
```

### 使用示例（单向链表）

```java
public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList list = new LinkedList();
        list.add(10);
        list.add(20);
        list.add(30);
        list.insert(1, 15); // [10, 15, 20, 30]

        System.out.println(list);
        System.out.println("get(2) = " + list.get(2)); // 20

        int removed = list.remove(0); // 删除头结点 10
        System.out.println("removed = " + removed);
        System.out.println(list); // [15, 20, 30]
    }
}
```

---

## 3. Java 演示：手写 `DoublyLinkedList`（双向链表）

双向链表节点同时保存 `next` 和 `prev`，因此可以从前往后、也可以从后往前遍历。

```java
public class DoublyLinkedList {
    // 双向链表节点：保存值 + 前驱/后继引用
    private static class Node {
        int value;
        Node prev;
        Node next;

        Node(int value) {
            this.value = value;
        }
    }

    private Node head;
    private Node tail;
    private int size;

    public int size() {
        return size;
    }

    // 尾插：追加到链表末尾
    public void add(int value) {
        Node newNode = new Node(value);
        if (head == null) {
            head = newNode;
            tail = newNode;
        } else {
            tail.next = newNode;
            newNode.prev = tail;
            tail = newNode;
        }
        size++;
    }

    // 在指定下标插入，允许 index == size（尾插）
    public void insert(int index, int value) {
        rangeCheckForInsert(index);

        if (index == size) {
            add(value);
            return;
        }

        Node nextNode = nodeAt(index);
        Node newNode = new Node(value);
        Node prevNode = nextNode.prev;

        newNode.next = nextNode;
        nextNode.prev = newNode;

        if (prevNode == null) {
            // 插到头部
            head = newNode;
        } else {
            prevNode.next = newNode;
            newNode.prev = prevNode;
        }
        size++;
    }

    // 删除指定下标并返回删除值
    public int remove(int index) {
        rangeCheck(index);
        Node removed = nodeAt(index);
        int removedValue = removed.value;

        Node prevNode = removed.prev;
        Node nextNode = removed.next;

        if (prevNode == null) {
            head = nextNode;
        } else {
            prevNode.next = nextNode;
        }

        if (nextNode == null) {
            tail = prevNode;
        } else {
            nextNode.prev = prevNode;
        }

        size--;
        return removedValue;
    }

    // 获取指定下标元素
    public int get(int index) {
        rangeCheck(index);
        return nodeAt(index).value;
    }

    private Node nodeAt(int index) {
        // 双向链表可根据位置选择从头或尾开始，减少遍历步数
        if (index < size / 2) {
            Node current = head;
            for (int i = 0; i < index; i++) {
                current = current.next;
            }
            return current;
        } else {
            Node current = tail;
            for (int i = size - 1; i > index; i--) {
                current = current.prev;
            }
            return current;
        }
    }

    private void rangeCheck(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException(
                "index: " + index + ", size: " + size
            );
        }
    }

    private void rangeCheckForInsert(int index) {
        if (index < 0 || index > size) {
            throw new IndexOutOfBoundsException(
                "index: " + index + ", size: " + size
            );
        }
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("[");
        Node current = head;
        while (current != null) {
            sb.append(current.value);
            if (current.next != null) {
                sb.append(", ");
            }
            current = current.next;
        }
        sb.append("]");
        return sb.toString();
    }
}
```

### 使用示例（双向链表）

```java
public class DoublyLinkedListDemo {
    public static void main(String[] args) {
        DoublyLinkedList list = new DoublyLinkedList();
        list.add(100);
        list.add(200);
        list.add(300);
        list.insert(2, 250); // [100, 200, 250, 300]

        System.out.println(list);
        System.out.println("get(1) = " + list.get(1)); // 200

        int removed = list.remove(3); // 删除尾结点 300
        System.out.println("removed = " + removed);
        System.out.println(list); // [100, 200, 250]
    }
}
```

# 线性表: 栈

## 1. 栈的含义

栈（Stack）是一种遵循 **LIFO（Last In First Out，后进先出）** 规则的线性结构。  
只能在“栈顶”进行操作：

- `push`：入栈（把元素放到栈顶）
- `pop`：出栈（移除并返回栈顶元素）

典型场景有：函数调用栈、括号匹配、撤销操作（undo）等。

---

## 2. 链表实现：`LinkedStack`

```java
public class LinkedStack {
    // 节点：保存值和下一个节点引用
    private static class Node {
        int value;
        Node next;

        Node(int value) {
            this.value = value;
        }
    }

    private Node top; // 栈顶指针
    private int size;

    public int size() {
        return size;
    }

    public void push(int value) {
        Node newNode = new Node(value);
        // 头插法：新节点指向原栈顶，再更新 top
        newNode.next = top;
        top = newNode;
        size++;
    }

    public int pop() {
        if (size == 0) {
            throw new IllegalStateException("stack is empty");
        }
        int value = top.value;
        top = top.next;
        size--;
        return value;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("[top -> ");
        Node current = top;
        while (current != null) {
            sb.append(current.value);
            if (current.next != null) {
                sb.append(", ");
            }
            current = current.next;
        }
        sb.append("]");
        return sb.toString();
    }
}
```

### 使用示例（链表栈）

```java
public class LinkedStackDemo {
    public static void main(String[] args) {
        LinkedStack stack = new LinkedStack();
        stack.push(10);
        stack.push(20);
        stack.push(30);

        System.out.println(stack); // [top -> 30, 20, 10]
        int popped = stack.pop();  // 弹出 30
        System.out.println("popped = " + popped);
        System.out.println(stack); // [top -> 20, 10]
    }
}
```

---

## 3. 数组实现：`ArrayStack`

```java
public class ArrayStack {
    private int[] data;
    private int size;
    private int capacity;

    public ArrayStack() {
        this(10);
    }

    public ArrayStack(int initCapacity) {
        if (initCapacity <= 0) {
            throw new IllegalArgumentException("initCapacity must be > 0");
        }
        this.capacity = initCapacity;
        this.data = new int[this.capacity];
        this.size = 0;
    }

    public int size() {
        return size;
    }

    public void push(int value) {
        if (size == capacity) {
            grow();
        }
        data[size] = value;
        size++;
    }

    public int pop() {
        if (size == 0) {
            throw new IllegalStateException("stack is empty");
        }
        int value = data[size - 1];
        size--;
        return value;
    }

    private void grow() {
        int newCapacity = capacity * 2;
        int[] newData = new int[newCapacity];

        // 扩容时只复制有效元素
        for (int i = 0; i < size; i++) {
            newData[i] = data[i];
        }

        data = newData;
        capacity = newCapacity;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("[bottom -> ");
        for (int i = 0; i < size; i++) {
            sb.append(data[i]);
            if (i < size - 1) {
                sb.append(", ");
            }
        }
        sb.append(" <- top]");
        return sb.toString();
    }
}
```

### 使用示例（数组栈）

```java
public class ArrayStackDemo {
    public static void main(String[] args) {
        ArrayStack stack = new ArrayStack(2);
        stack.push(100);
        stack.push(200);
        stack.push(300); // 触发扩容

        System.out.println(stack); // [bottom -> 100, 200, 300 <- top]
        int popped = stack.pop();  // 弹出 300
        System.out.println("popped = " + popped);
        System.out.println(stack); // [bottom -> 100, 200 <- top]
    }
}
```

# 线性表: 队列

## 1. 队列的含义

队列（Queue）是一种**先进先出（FIFO, First In First Out）**的线性结构。  
可以把它理解为“排队”：

- 新元素从队尾进入（入队）
- 老元素从队头离开（出队）

典型场景有任务调度、消息缓冲、打印任务等。

---

## 2. 链表实现：`LinkedQueue`

下面是一个单向链表队列实现，包含你要求的 `offer()`、`poll()`、`peek()`、`toString()`，并带必要注释。

```java
import java.util.StringJoiner;

public class LinkedQueue<E> {

    // 节点：保存元素值 + 指向下一个节点
    private static class Node<E> {
        E data;
        Node<E> next;

        Node(E data) {
            this.data = data;
        }
    }

    private Node<E> head; // 队头：出队位置
    private Node<E> tail; // 队尾：入队位置
    private int size;

    public int size() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    // 入队：把元素追加到队尾
    public boolean offer(E e) {
        Node<E> newNode = new Node<>(e);
        if (tail == null) { // 空队列：head 和 tail 同时指向新节点
            head = newNode;
            tail = newNode;
        } else {
            tail.next = newNode;
            tail = newNode;
        }
        size++;
        return true;
    }

    // 出队：移除并返回队头元素；空队列返回 null
    public E poll() {
        if (head == null) {
            return null;
        }

        E value = head.data;
        head = head.next;

        // 如果移除后队列为空，tail 也要置空
        if (head == null) {
            tail = null;
        }

        size--;
        return value;
    }

    // 查看队头：只读取，不删除；空队列返回 null
    public E peek() {
        return head == null ? null : head.data;
    }

    @Override
    public String toString() {
        StringJoiner joiner = new StringJoiner(", ", "[", "]");
        Node<E> current = head;
        while (current != null) {
            joiner.add(String.valueOf(current.data));
            current = current.next;
        }
        return joiner.toString();
    }
}
```

### 使用示例（链表队列）

```java
public class LinkedQueueDemo {
    public static void main(String[] args) {
        LinkedQueue<String> queue = new LinkedQueue<>();

        queue.offer("任务1");
        queue.offer("任务2");
        queue.offer("任务3");

        System.out.println(queue);        // [任务1, 任务2, 任务3]
        System.out.println(queue.peek()); // 任务1（只看不删）
        System.out.println(queue.poll()); // 任务1（出队）
        System.out.println(queue);        // [任务2, 任务3]
        System.out.println(queue.size()); // 2
    }
}
```

# 树: 二叉树

## 1. 二叉树的基本概念

二叉树（Binary Tree）是一种树形结构，每个节点最多有两个子节点：

- 左子节点（left）
- 右子节点（right）

常见术语：

- 根节点（root）：整棵树的起点
- 叶子节点（leaf）：没有子节点的节点
- 子树：某个节点及其后代构成的一棵树
- 深度/高度：用于描述节点层级和树的层数

二叉树常用于表达层级关系，也常作为二叉搜索树、堆、表达式树等结构的基础。

### 补充概念：度、满二叉树、完全二叉树

- 度（Degree）
  - 节点的度：一个节点拥有的子节点个数。
  - 树的度：整棵树中所有节点度的最大值。
  - 在二叉树中，节点度只能是 `0`、`1`、`2`，所以二叉树的度最大为 `2`。

- 满二叉树（Full Binary Tree）
  - 一棵二叉树中，除叶子节点外，每个节点都有两个子节点，并且所有叶子节点都在同一层。
  - 直观上看，每一层节点数都“满”了。

- 完全二叉树（Complete Binary Tree）
  - 除最后一层外，其余各层都达到最大节点数。
  - 最后一层的节点都连续集中在最左侧，中间不能有空位。

简单区分：

- 满二叉树一定是完全二叉树。
- 完全二叉树不一定是满二叉树。

---

## 2. 节点类：`TreeNode`

下面是一个通用的二叉树节点类，包含值、左孩子、右孩子。

```java
public class TreeNode<E> {
    E val;
    TreeNode<E> left;
    TreeNode<E> right;

    public TreeNode(E val) {
        this.val = val;
    }

    public TreeNode(E val, TreeNode<E> left, TreeNode<E> right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
```

### 使用示例（手动构建一棵二叉树）

```java
public class BinaryTreeDemo {
    public static void main(String[] args) {
        //      A
        //     / \
        //    B   C
        //   / \
        //  D   E
        TreeNode<String> root = new TreeNode<>("A");
        root.left = new TreeNode<>("B");
        root.right = new TreeNode<>("C");
        root.left.left = new TreeNode<>("D");
        root.left.right = new TreeNode<>("E");

        System.out.println(root.val);         // A
        System.out.println(root.left.val);    // B
        System.out.println(root.right.val);   // C
    }
}
```

# 树: 二叉树的遍历

## 1. 四种遍历方式的含义

- 前序遍历（Preorder）：`根 -> 左 -> 右`
- 中序遍历（Inorder）：`左 -> 根 -> 右`
- 后序遍历（Postorder）：`左 -> 右 -> 根`
- 层序遍历（Level Order）：按层从上到下、从左到右访问（借助队列）

---

## 2. 代码实现：前序 / 中序 / 后序 / 层序

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.List;
import java.util.Queue;

public class BinaryTreeTraversal {

    // 前序：根 -> 左 -> 右
    public static <E> List<E> preorder(TreeNode<E> root) {
        List<E> result = new ArrayList<>();
        preorderDfs(root, result);
        return result;
    }

    private static <E> void preorderDfs(TreeNode<E> node, List<E> result) {
        if (node == null) {
            return;
        }
        result.add(node.val);               // 先访问根
        preorderDfs(node.left, result);     // 再遍历左子树
        preorderDfs(node.right, result);    // 最后遍历右子树
    }

    // 中序：左 -> 根 -> 右
    public static <E> List<E> inorder(TreeNode<E> root) {
        List<E> result = new ArrayList<>();
        inorderDfs(root, result);
        return result;
    }

    private static <E> void inorderDfs(TreeNode<E> node, List<E> result) {
        if (node == null) {
            return;
        }
        inorderDfs(node.left, result);
        result.add(node.val);               // 左子树处理完后访问根
        inorderDfs(node.right, result);
    }

    // 后序：左 -> 右 -> 根
    public static <E> List<E> postorder(TreeNode<E> root) {
        List<E> result = new ArrayList<>();
        postorderDfs(root, result);
        return result;
    }

    private static <E> void postorderDfs(TreeNode<E> node, List<E> result) {
        if (node == null) {
            return;
        }
        postorderDfs(node.left, result);
        postorderDfs(node.right, result);
        result.add(node.val);               // 最后访问根
    }

    // 层序：按层访问，依赖队列实现 BFS
    public static <E> List<E> levelOrder(TreeNode<E> root) {
        List<E> result = new ArrayList<>();
        if (root == null) {
            return result;
        }

        Queue<TreeNode<E>> queue = new ArrayDeque<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            TreeNode<E> cur = queue.poll();
            result.add(cur.val);

            if (cur.left != null) {
                queue.offer(cur.left);
            }
            if (cur.right != null) {
                queue.offer(cur.right);
            }
        }

        return result;
    }
}
```

### 使用示例（四种遍历）

```java
public class BinaryTreeTraversalDemo {
    public static void main(String[] args) {
        //        A
        //      /   \
        //     B     C
        //    / \     \
        //   D   E     F
        TreeNode<String> root = new TreeNode<>("A");
        root.left = new TreeNode<>("B");
        root.right = new TreeNode<>("C");
        root.left.left = new TreeNode<>("D");
        root.left.right = new TreeNode<>("E");
        root.right.right = new TreeNode<>("F");

        System.out.println("前序: " + BinaryTreeTraversal.preorder(root));   // [A, B, D, E, C, F]
        System.out.println("中序: " + BinaryTreeTraversal.inorder(root));    // [D, B, E, A, C, F]
        System.out.println("后序: " + BinaryTreeTraversal.postorder(root));  // [D, E, B, F, C, A]
        System.out.println("层序: " + BinaryTreeTraversal.levelOrder(root)); // [A, B, C, D, E, F]
    }
}
```
