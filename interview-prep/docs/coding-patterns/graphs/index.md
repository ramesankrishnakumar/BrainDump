# Graphs (DFS, BFS & Topological Sort)

## What it is

A **graph** is a collection of nodes (vertices) connected by edges. In coding interviews, graphs represent relationships: prerequisite relationships between courses, dependency trees between software packages, task queues in workflow systems, or organizational structures in databases.

For **Senior Software Engineer (SSE)** interviews at companies like **Salesforce** (hierarchical database schemas, permission trees, tenant dependencies) and **ServiceNow** (task workflows, change-management pipelines, configuration items in the CMDB), graph questions are extremely high-yield.

Almost all graph problems are solved using three core patterns:
1.  **Depth-First Search (DFS):** Reach for DFS when you need to explore paths, detect cycles, evaluate dependency trees, or solve reachability problems. DFS uses recursion (a system call stack) and is simple to implement.
2.  **Breadth-First Search (BFS):** Reach for BFS when you need to find the **shortest path** in an unweighted graph, traverse level-by-level, or model layer-by-layer expansion (e.g. infection/matrix spreading). BFS uses an explicit queue.
3.  **Topological Sort (Kahn's or DFS-based):** Reach for this when you have a Directed Acyclic Graph (DAG) representing dependencies, and you need to find a valid ordering to execute tasks or courses.

---

## When to reach for it

Strong signals you're in graph territory:
*   The input is described as a set of pairs with a prerequisite relationship: `"to do A, you must first do B"`.
*   The question mentions nodes connected by links, networks, dependencies, flight routes, or matrix grid exploration (e.g. islands, water/land).
*   You need to find a **shortest sequence of moves** or operations (unweighted BFS).
*   You are asked to check if there is a **cyclic dependency** that makes a process hang or fail.

---

## Core Techniques & Skeletons (Java)

### 1. Standard Graph BFS (Shortest Path / Level Traversal)
BFS traverses a graph level-by-level using a Queue. To avoid infinite loops in cyclic graphs, always keep a `visited` set.

```java
import java.util.*;

public int bfsShortestPath(int start, int target, Map<Integer, List<Integer>> adj) {
    Queue<Integer> q = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();
    
    q.add(start);
    visited.add(start);
    int steps = 0;
    
    while (!q.isEmpty()) {
        int size = q.size();
        // Process current level
        for (int i = 0; i < size; i++) {
            int curr = q.poll();
            if (curr == target) return steps;
            
            for (int neighbor : adj.getOrDefault(curr, new ArrayList<>())) {
                if (!visited.contains(neighbor)) {
                    visited.add(neighbor);
                    q.add(neighbor);
                }
            }
        }
        steps++; // increment level depth
    }
    return -1; // path not found
}
```

### 2. Topological Sort (Kahn's Algorithm)
Used for task scheduling (e.g. Course Schedule / Package Build Order). It calculates the "in-degree" (number of incoming dependency arrows) for each node, pushes nodes with in-degree `0` into a Queue, and pops them while decrementing their neighbors' in-degrees.

> [!TIP]
> In-degree `0` represents a node that has **no dependencies** and is safe to execute immediately!

```java
import java.util.*;

public List<Integer> topologicalSort(int numNodes, int[][] prerequisites) {
    Map<Integer, List<Integer>> adj = new HashMap<>();
    int[] inDegree = new int[numNodes];
    
    // Build Graph & In-Degrees
    for (int[] edge : prerequisites) {
        int dest = edge[0];
        int src = edge[1];
        adj.computeIfAbsent(src, k -> new ArrayList<>()).add(dest);
        inDegree[dest]++;
    }
    
    // Queue for nodes with 0 incoming dependencies
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < numNodes; i++) {
        if (inDegree[i] == 0) q.add(i);
    }
    
    List<Integer> order = new ArrayList<>();
    while (!q.isEmpty()) {
        int curr = q.poll();
        order.add(curr);
        
        for (int neighbor : adj.getOrDefault(curr, new ArrayList<>())) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] == 0) {
                q.add(neighbor);
            }
        }
    }
    
    // If order size != total nodes, we detected a cyclic dependency!
    return order.size() == numNodes ? order : new ArrayList<>();
}
```

### 3. Cycle Detection in Directed Graph (DFS-based Coloring)
Used to find if a workflow contains a loop. Relies on tracing three states:
*   `0` (Unvisited)
*   `1` (Visiting - currently in recursion stack)
*   `2` (Fully processed - all neighbors explored)

> [!WARNING]
> If you encounter a neighbor that is in the `Visiting` state (`1`), you have found a **back-edge**, which means a cycle exists!

```java
import java.util.*;

public boolean hasCycle(int numNodes, int[][] prerequisites) {
    Map<Integer, List<Integer>> adj = new HashMap<>();
    for (int[] edge : prerequisites) {
        adj.computeIfAbsent(edge[1], k -> new ArrayList<>()).add(edge[0]);
    }
    
    int[] state = new int[numNodes]; // 0=Unvisited, 1=Visiting, 2=Visited
    for (int i = 0; i < numNodes; i++) {
        if (state[i] == 0) {
            if (dfsCycle(i, adj, state)) return true;
        }
    }
    return false;
}

private boolean dfsCycle(int curr, Map<Integer, List<Integer>> adj, int[] state) {
    state[curr] = 1; // Mark as Visiting
    
    for (int neighbor : adj.getOrDefault(curr, new ArrayList<>())) {
        if (state[neighbor] == 1) return true; // Found active recursion ancestor -> Cycle!
        if (state[neighbor] == 0) {
            if (dfsCycle(neighbor, adj, state)) return true;
        }
    }
    
    state[curr] = 2; // Mark as Visited
    return false;
}
```

---

## High-Yield Problems

*   **Course Schedule (LC 207) / Course Schedule II (LC 210):** *Foundational Kahn's / Topological Sort.* Directly models ServiceNow task chains or Salesforce field-dependency parsing.
*   **Clone Graph (LC 133):** *DFS/BFS with deep copy.* Tests memory address separation, pointers, and cyclic object graphs. Very frequent for SSE object mapping queries.
*   **Number of Islands (LC 200) / Pac-Man Grid DFS:** *Matrix BFS/DFS.* Standard reachability and connected component exploration.
*   **Alien Dictionary (LC 269):** *Topological Sort from comparison tuples.* Hard but highly standard dependency-resolution problem.
