# Java Quick Hacks

Java-API idioms for interviews — not algorithms, just library moves that keep solutions short. Each section names **when** the snippet wins and which LeetCode families it shows up in.

| Reach for… | Section |
|---|---|
| Pick from both ends | [Both-ends indexing](#index-from-both-ends-with-one-variable) |
| Count / group in a map | [Map.merge](#mapmerge-and-mapcompute-for-count-maps) |
| Dedup or cycle check | [HashSet](#hashset--add-and-contains) |
| Sort then sweep | [Arrays.sort](#arrayssort--1d-and-2d) |
| `[start, end]` rows | [Intervals](#int-intervals--what-length-means) |
| Grid walk / BFS | [Grid + neighbors](#int-vs-flat-int-for-grids) |
| Stack, queue, deque | [Deque](#deque-as-stack-queue-or-deque) |
| Scan a string | [String chars](#access-individual-characters-of-a-string) |
| Fixed lookup table | [Map.of / List.of](#immutable-constants--mapof-listof-setof) |
| Next min / max | [PriorityQueue](#priorityqueue--min--max-heap) |
| Build output string | [StringBuilder](#stringbuilder--build-or-reverse-output) |
| Unvisited = ∞ | [Arrays.fill](#dp--graph-init--arraysfill--sentinels) |
| Visited `(r, c)` | [record](#bfsdfs-state--record-tuples) |

```java
import java.util.*;

Deque<Integer> stack = new ArrayDeque<>();
Queue<Integer> heap = new PriorityQueue<>();
Map<Character, Character> pairs = Map.of('(', ')', '[', ']', '{', '}');
int[][] dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};
```

## Index from both ends with one variable

One index `i` reads the front (`arr[i]`) and back (`arr[n - 1 - i]`) — no negative indices, no wrap-around.

```java
int n = arr.length;
for (int i = 0; i < n; i++) {
    int fromFront = arr[i];
    int fromBack  = arr[n - 1 - i];   // i = 0 -> last element, i = 1 -> second last, ...
}
```

Inward from both sides ("take k from left and right"):

```java
arr[k - 1 - i]   // left inward:  k-1, k-2, ..., 0
arr[n - 1 - i]   // right inward: n-1, n-2, ..., n-k
```

**LeetCode when:** Max Points From Cards, Valid Palindrome, reverse in place *(pick-from-ends, mirrored two-pointer)*.

!!! warning "Don't fake a circular array"
    Resist `arr[(n + start) % n]` — the array isn't circular. `arr[n - 1 - i]` is direct and in bounds.

See [Max Points From Cards](sliding-window/max-points-from-cards.md) (Approach B).

## `Map.merge` and `Map.compute` for count maps

`merge` inserts or combines in one call and **returns the new value** — so decrement-to-zero collapses to one line.

```java
Map<Integer, Integer> counts = new HashMap<>();

counts.merge(key, 1, Integer::sum);    // increment (insert 1 if absent)

if (counts.merge(key, -1, Integer::sum) == 0) {
    counts.remove(key);                // drop zero-count keys when size() = distinct count
}
```

Verbose equivalent: `counts.put(key, counts.getOrDefault(key, 0) + 1)`.

`computeIfAbsent` builds maps-of-lists without a null check:

```java
Map<String, List<String>> groups = new HashMap<>();
groups.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

**LeetCode when:** Fruits Into Baskets, Max Sum of Distinct Subarrays *(sliding-window freq)*; Group Anagrams *(computeIfAbsent bucketing)*.

!!! warning "Returning null deletes the key, and zero-count keys lie"
    A `merge`/`compute` fn that returns `null` **removes** the entry. When `map.size()` tracks distinct elements, remove keys at count zero (as above).

See [Max Sum of Distinct Subarrays](sliding-window/max-sum-distinct-subarrays.md), [Fruits Into Baskets](sliding-window/fruits-into-baskets.md).

## HashSet — `add` and `contains`

**Win when:** you only need membership or "seen before?" — not counts.

```java
Set<Integer> seen = new HashSet<>();

if (!seen.add(x)) { /* x was already present — duplicate */ }
if (seen.contains(x)) { /* ... */ }
```

**LeetCode when:** Contains Duplicate, Two Sum (hash complement), cycle detection.

## `Arrays.sort` — 1D and 2D

Prefer `Integer.compare` in comparators — `(a[0] - b[0])` overflows on large values.

### 1D `int[]`

```java
import java.util.Arrays;

int[] arr = {1, 7, 4, 3, 2};
Arrays.sort(arr);   // ascending
```

Reverse on primitive `int[]`: no `Comparator` overload — box to `Integer[]` + `Collections.reverseOrder()`, or sort ascending then reverse in place ([both-ends indexing](#index-from-both-ends-with-one-variable)).

### 2D `int[][]` — sort rows by a column

```java
int[][] arr = {{1, 0}, {8, 2}, {4, 3}};

Arrays.sort(arr, (a, b) -> Integer.compare(a[0], b[0]));   // by col 0 ↑
Arrays.sort(arr, (a, b) -> Integer.compare(a[1], b[1]));   // by col 1 ↑ (interval end)
Arrays.sort(arr, (a, b) -> Integer.compare(b[0], a[0]));   // by col 0 ↓
```

**LeetCode when:** Merge Intervals *(sort by start)*, Non-Overlapping Intervals / Meeting Rooms II *(sort by end)* — then sweep once.

!!! warning "`Collections.reverseOrder()` is not for `int[]`"
    `Arrays.sort(int[], Comparator)` does not exist. For 2D, flip compare order instead of boxing.

See [Intervals](intervals/index.md).

## `int[][]` intervals — what `length` means

Each row is one span `[start, end]` — a **list of intervals**, not a grid.

| Expression | Meaning |
|---|---|
| `intervals.length` | number of intervals |
| `intervals[i][0]` | start |
| `intervals[i][1]` | end |

```java
int[][] intervals = {{0, 30}, {5, 10}, {15, 20}};
intervals.length;    // 3
intervals[0][1];     // 30
```

After sorting: overlap ⇔ `intervals[i][0] < intervals[i - 1][1]`. Edge case: `length <= 1` → neighbor loop never runs.

**LeetCode when:** Can Attend Meetings, Insert Interval, Employee Free Time.

!!! warning "`length` vs `[i].length`"
    `intervals.length` = how many intervals. `intervals[i].length` = width of one row (usually `2`).

See [Can Attend Meetings](intervals/can-attend-meetings.md).

## `int[][]` vs flat `int[]` for grids

Default: `matrix[r][c]`. Flat `r * cols + c` when the problem gives a 1D buffer.

```java
int rows = 3, cols = 4;
int index = r * cols + c;    // (r,c) → flat
int r = index / cols, c = index % cols;   // flat → (r,c)
```

**4-neighbor BFS/DFS** — pair with `dirs` from the paste block and a `seen` set ([record tuples](#bfsdfs-state--record-tuples) below):

```java
int[][] dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};

for (int[] d : dirs) {
    int nr = r + d[0], nc = c + d[1];
    if (nr < 0 || nc < 0 || nr >= rows || nc >= cols) continue;
    if (matrix[nr][nc] == blocked) continue;
    // visit (nr, nc)
}
```

**LeetCode when:** Number of Islands, Rotting Oranges, Flood Fill *(grid + neighbors)*; Unique Paths *(DP table)*.

## `Deque` as stack, queue, or deque

One type, three roles — flip by which methods you call:

```java
Deque<Integer> dq = new ArrayDeque<>();

// STACK (LIFO) — head only
dq.push(x);  dq.pop();  dq.peek();

// QUEUE (FIFO) — tail in, head out
dq.offer(x); dq.poll(); dq.peek();
```

**LeetCode when:** Valid Parentheses *(stack)*, Binary Tree Level Order *(queue)*, Daily Temperatures *(monotonic stack on deque)*.

!!! warning "Skip `java.util.Stack`"
    Synchronized and iterates bottom-to-top. `pop`/`poll`: former throws on empty, latter returns `null`.

See [Valid Parentheses](stack/valid-parentheses.md).

## Access individual characters of a String

`s.length()` is a **method**; arrays use `.length` field. Strings are immutable.

```java
for (int i = 0; i < s.length(); i++) {
    char c = s.charAt(i);
}

for (char c : s.toCharArray()) { /* no index needed */ }
```

**Fixed-size freq array** — subtract the range base instead of a `HashMap`:

```java
int[] freq = new int[26];   // or new int[128] for all ASCII

freq[c - 'a']++;   // lowercase → 0..25
freq[c - 'A']++;   // uppercase → 0..25
freq[c - '0']++;   // digit     → 0..9
```

**LeetCode when:** Valid Anagram, First Unique Character, permutation-window problems *(char scan + freq)*.

!!! warning "Immutability"
    `s.charAt(i) = x` won't compile. Mutate via [StringBuilder](#stringbuilder--build-or-reverse-output) or `char[]` + `new String(arr)`.

## Immutable constants — `Map.of`, `List.of`, `Set.of`

**Win when:** the table never changes mid-solution — one line, no `put`.

```java
Map<Character, Character> pairs = Map.of('(', ')', '[', ']', '{', '}');
int[][] dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};
Set<Character> vowels = Set.of('a', 'e', 'i', 'o', 'u');
```

More than 10 map entries → `Map.ofEntries(Map.entry(k, v), ...)`.

**LeetCode when:** Valid Parentheses *(pair map + stack)*, Number of Islands / Word Search *(dirs)*.

!!! warning "Immutable — no `put`, no `null`"
    `put`/`add`/`remove` throw. Keys and values cannot be `null`.

See [Valid Parentheses](stack/valid-parentheses.md).

## `PriorityQueue` — min / max heap

Default = **min-heap** (`peek()` = smallest). Reverse comparator for max.

```java
Queue<Integer> minHeap = new PriorityQueue<>();
Queue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

minHeap.offer(x);
int smallest = minHeap.poll();

Queue<int[]> byEnd = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
```

**LeetCode when:** Merge k Sorted Lists, Top K Frequent Elements, Kth Largest *(always expand best next)*.

!!! warning "Default is min-heap; subtraction overflows"
    Forgetting the comparator gives min when you wanted max. Use `Integer.compare`, not `a - b`.

## `StringBuilder` — build or reverse output

**Win when:** output grows character-by-character or you need in-place reverse.

```java
StringBuilder sb = new StringBuilder();
for (char c : s.toCharArray()) sb.append(c);
sb.reverse();
return sb.toString();
```

**LeetCode when:** Reverse String, Decode String, stack-driven reconstruction.

## DP / graph init — `Arrays.fill` + sentinels

**Win when:** "unvisited" or min-distance before relaxing edges.

```java
int[] dist = new int[n];
Arrays.fill(dist, Integer.MAX_VALUE);
dist[src] = 0;
```

**LeetCode when:** Network Delay Time, Cheapest Flights Within K Stops.

!!! warning "`MAX_VALUE + cost` overflows"
    Guard: `if (dist[u] != Integer.MAX_VALUE)` before adding weight.

## BFS/DFS state — `record` tuples

**Win when:** `Set`/`Map` key is `(row, col)` or similar — no hand-written `equals`/`hashCode`.

```java
record Cell(int r, int c) {}

Set<Cell> seen = new HashSet<>();
Deque<Cell> queue = new ArrayDeque<>();
seen.add(new Cell(r, c));
queue.offer(new Cell(nr, nc));   // accessors: cell.r(), cell.c()
```

Pair with [grid neighbor loop](#int-vs-flat-int-for-grids) above.

**LeetCode when:** Rotting Oranges, Walls and Gates, grid shortest-path with visited set.

!!! warning "Field accessors use parentheses"
    `cell.r()` not `cell.r`.
