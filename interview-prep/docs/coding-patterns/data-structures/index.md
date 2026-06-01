# Custom Data Structure Design (LRU Cache & Trie)

## What it is

At the **Senior Software Engineer (SSE)** level, interviewers want to see how you build and structure low-level utilities. You are no longer just manipulating arrays; you are asked to design optimized, cohesive classes that support multiple operations in guaranteed constant time $O(1)$ or log-time $O(\log n)$.

For **Salesforce** and **ServiceNow**, custom caches and index trees are standard:
*   **LRU Cache (HashMap + Doubly Linked List):** Demonstrates complete control over node pointers and reference mapping. It's the ultimate test of "clean pointer rewiring" under pressure.
*   **Trie (Prefix Tree):** The standard structure for fast text parsing, prefix autocomplete (like search bars, filters, and dynamic queries in relational platforms).

---

## When to reach for it

*   The problem asks you to design a class with multiple operations (e.g. `get(key)`, `put(key, val)`) that must all execute in **$O(1)$ time**.
*   The requirements call for an ordered eviction policy based on access frequency or recency.
*   You are asked to check for words containing a certain prefix, autocomplete options in a dict, or filter elements dynamically by characters.

---

## Core Techniques & Skeletons (Java & Python)

### 1. Hand-Rolled LRU Cache (HashMap + DLL)
Many interviewers *explicitly ban* standard language utilities like Java's `LinkedHashMap` to force you to manage the pointers yourself. The clean recipe for a hand-rolled LRU cache uses:
1.  A **Doubly Linked List** to maintain chronological access order.
2.  A **HashMap** mapping keys to Doubly Linked List nodes for $O(1)$ lookup.
3.  **Dummy Head and Tail** sentinel nodes in the list to eliminate boundary null-checks.

```carousel
```java
// Java Hand-Rolled LRU Cache
import java.util.*;

public class LRUCache {
    private static class Node {
        int key, value;
        Node prev, next;
        Node(int k, int v) { this.key = k; this.value = v; }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0); // Dummy Head
    private final Node tail = new Node(0, 0); // Dummy Tail

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        moveToHead(node); // Accessed, make it most recent
        return node.value;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) {
            Node node = map.get(key);
            node.value = value;
            moveToHead(node);
        } else {
            if (map.size() >= capacity) {
                // Evict the least recently used from tail
                Node lru = tail.prev;
                removeNode(lru);
                map.remove(lru.key);
            }
            Node newNode = new Node(key, value);
            addNode(newNode);
            map.put(key, newNode);
        }
    }

    // Helper: Add a node right after the dummy head
    private void addNode(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    // Helper: Remove an existing node from the list
    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    // Helper: Relocate an active node to the most recent position
    private void moveToHead(Node node) {
        removeNode(node);
        addNode(node);
    }
}
```
<!-- slide -->
```python
# Python Hand-Rolled LRU Cache
class Node:
    def __init__(self, key=0, val=0):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.map = {}
        self.head = Node() # Dummy Head
        self.tail = Node() # Dummy Tail
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key: int) -> int:
        if key not in self.map:
            return -1
        node = self.map[key]
        self._move_to_head(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.map:
            node = self.map[key]
            node.val = value
            self._move_to_head(node)
        else:
            if len(self.map) >= self.capacity:
                lru = self.tail.prev
                self._remove(lru)
                del self.map[lru.key]
            new_node = Node(key, value)
            self._add(new_node)
            self.map[key] = new_node

    def _add(self, node):
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _move_to_head(self, node):
        self._remove(node)
        self._add(node)
```
```

### 2. Trie / Prefix Tree
A **Trie** is a specialized tree used to store associative arrays where keys are strings. It allows $O(L)$ search, insertions, and prefix matchings, where $L$ is the length of the string.

```carousel
```java
// Java Trie Implementation
import java.util.*;

public class Trie {
    private static class TrieNode {
        private final TrieNode[] children = new TrieNode[26];
        private boolean isWord = false;
    }

    private final TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode curr = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) {
                curr.children[idx] = new TrieNode();
            }
            curr = curr.children[idx];
        }
        curr.isWord = true;
    }

    public boolean search(String word) {
        TrieNode node = findNode(word);
        return node != null && node.isWord;
    }

    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }

    private TrieNode findNode(String str) {
        TrieNode curr = root;
        for (char c : str.toCharArray()) {
            int idx = c - 'a';
            if (curr.children[idx] == null) return null;
            curr = curr.children[idx];
        }
        return curr;
    }
}
```
<!-- slide -->
```python
# Python Trie Implementation
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        curr = self.root
        for char in word:
            if char not in curr.children:
                curr.children[char] = TrieNode()
            curr = curr.children[char]
        curr.is_word = True

    def search(self, word: str) -> bool:
        node = self._find(word)
        return node is not None and node.is_word

    def startsWith(self, prefix: str) -> bool:
        return self._find(prefix) is not None

    def _find(self, prefix: str) -> TrieNode:
        curr = self.root
        for char in prefix:
            if char not in curr.children:
                return None
            curr = curr.children[char]
        return curr
```
```

---

## High-Yield Problems

*   **LRU Cache (LC 146):** *The classic custom data structure question.* Standard for senior engineers at Salesforce/ServiceNow to verify solid understanding of thread-safety, reference maps, and double-linked nodes.
*   **Implement Trie (LC 208):** *Building block for dynamic auto-completes.*
*   **Design Add and Search Words Data Structure (LC 211):** *Trie + DFS character wildcards (`.`).* Tests recursion traversal over trie sub-nodes.
*   **LFU Cache (LC 460):** *Hard variation of LRU.* Evicts items with lower access frequencies first. Requires frequency mapping to Doubly Linked Lists.
