# Coding Interview Cheat Sheet

A single-page, Ctrl-F-able reference for the most-asked coding interview questions, grouped
by technique. Built for **senior SWE** interviews where the bar is *recognize the pattern
fast, say the right things, write a clean template, then handle the twist*.

## How to use this

1. **Read the problem, find the match.** Skim the [recognition index](#recognition-index)
   below — the keyword/signal you heard maps to a technique.
2. **Lift the template.** Each entry has a skeleton (the *shape*, not the polished full
   solution). Type it from memory; it's small on purpose.
3. **Talk while you code.** The **Talk track** line is what to say out loud: brute force →
   optimal, complexity, the key insight, edge cases.
4. **Adapt the twist.** Real problems are *cheat-sheet entry + a small change*. The
   **Twists** line lists the common mutations so you recognize them as variations, not new
   problems.

Where a full walkthrough exists, the entry links to a **deep dive** page.

## Recognition index

| If you hear / see… | Reach for | Section |
|---|---|---|
| "find a pair/triple summing to X", "seen before?", "group by" | hash map / sorting | [Arrays & Hashing](#arrays-hashing) |
| sorted array, "pair from both ends", "in place", palindrome | two pointers | [Two Pointers](#two-pointers) |
| "contiguous subarray/substring", "window of size k", "longest/shortest where…" | sliding window | [Sliding Window](#sliding-window) |
| "matching/nesting", "most recent", "next greater/warmer" | stack / monotonic stack | [Stack](#stack) |
| sorted + "find index/first/last", "min capacity/speed so that…" | binary search | [Binary Search](#binary-search) |
| `ListNode`, "reverse/reorder/merge", "cycle", "nth from end" | linked list / fast-slow | [Linked List](#linked-list) |
| `TreeNode`, "depth/path/ancestor", "level order", BST queries | tree DFS/BFS | [Trees](#trees) |
| "prefix", "autocomplete", "dictionary of words" | trie | [Tries](#tries) |
| "top/k-th largest/smallest", "median of a stream", "k most" | heap | [Heap / Priority Queue](#heap) |
| "all combinations/permutations/subsets", "generate every…" | backtracking | [Backtracking](#backtracking) |
| grid/matrix flood fill, "connected", "shortest steps in grid" | graph BFS/DFS | [Graphs](#graphs) |
| "prerequisites/order", "groups merging", "cheapest path" | topo sort / union-find / Dijkstra | [Advanced Graphs](#advanced-graphs) |
| "ways to…", "min/max to reach", "can you make…" (1 sequence) | 1-D DP | [1-D DP](#dp-1d) |
| two strings/sequences, grid paths, "with constraint k" | 2-D DP | [2-D DP](#dp-2d) |
| "max/min and a local greedy choice looks right" | greedy | [Greedy](#greedy) |
| `[start, end]` pairs, "merge/overlap/rooms needed" | intervals | [Intervals](#intervals) |
| "without +/-", "single number", "count bits", XOR smell | bit manipulation | [Bit Manipulation](#bit) |
| rotate/spiral matrix, digits of a number, fast power | math & geometry | [Math & Geometry](#math) |

---

## Arrays & Hashing { #arrays-hashing }

Reach for a **hash map/set** when you need O(1) "have I seen this?" or "count of this", and
**sorting** when relative order or grouping unlocks the answer. The trade is almost always
*extra space for time*.

### Two Sum — LC 1 (Easy)
**Problem:** Return indices of the two numbers that add to `target`.
**Recognize:** "pair that sums to X" + want O(n) → hash map of complements.
**Talk track:** Brute force O(n²). Optimal: one pass storing `value→index`; for each `x`,
check if `target-x` was already seen. O(n) time, O(n) space.
**Template:**
```java
Map<Integer,Integer> seen = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    int need = target - nums[i];
    if (seen.containsKey(need)) return new int[]{seen.get(need), i};
    seen.put(nums[i], i);
}
```
**Twists:** sorted input → two pointers (O(1) space); count pairs → frequency map; 3Sum → fix one + two-pointer.

### Contains Duplicate — LC 217 (Easy)
**Problem:** Does any value appear at least twice?
**Recognize:** "any repeats?" → hash set.
**Talk track:** Add to a set as you go; if `add` returns false (already present), return true. O(n)/O(n). Sorting gives O(n log n)/O(1) if space matters.
**Template:**
```java
Set<Integer> seen = new HashSet<>();
for (int x : nums) if (!seen.add(x)) return true;
return false;
```
**Twists:** "duplicate within k indices" → sliding window set; "duplicate within k indices and value diff ≤ t" → bucketing / TreeSet.

### Valid Anagram — LC 242 (Easy)
**Problem:** Is `t` a rearrangement of `s`?
**Recognize:** "same letters, different order" → frequency count.
**Talk track:** Count chars in `s`, decrement with `t`; all zero ⇒ anagram. O(n)/O(1) for fixed alphabet. Sorting both is the lazy O(n log n) alternative.
**Template:**
```java
if (s.length() != t.length()) return false;
int[] count = new int[26];
for (int i = 0; i < s.length(); i++) { count[s.charAt(i)-'a']++; count[t.charAt(i)-'a']--; }
for (int c : count) if (c != 0) return false;
return true;
```
**Twists:** unicode → `HashMap`; group anagrams → use the count signature as a key.

### Group Anagrams — LC 49 (Medium)
**Problem:** Group words that are anagrams of each other.
**Recognize:** "group by some canonical form" → map from signature → list.
**Talk track:** Key each word by its sorted letters (or a 26-length count string); bucket into a map. O(n·k log k) sorting keys, or O(n·k) with count keys.
**Template:**
```java
Map<String,List<String>> groups = new HashMap<>();
for (String w : strs) {
    char[] c = w.toCharArray(); Arrays.sort(c);
    groups.computeIfAbsent(new String(c), k -> new ArrayList<>()).add(w);
}
return new ArrayList<>(groups.values());
```
**Twists:** count-array key (`"#1#0#2..."`) avoids the sort for an O(k) key.

### Top K Frequent Elements — LC 347 (Medium)
**Problem:** The `k` most frequent values.
**Recognize:** "k most/least frequent" → count map + (heap **or** bucket sort).
**Talk track:** Count frequencies, then either a size-k min-heap (O(n log k)) or **bucket sort** by frequency (O(n)) since frequency ≤ n.
**Template:**
```java
Map<Integer,Integer> freq = new HashMap<>();
for (int x : nums) freq.merge(x, 1, Integer::sum);
List<Integer>[] bucket = new List[nums.length + 1];
for (var e : freq.entrySet()) {
    int f = e.getValue();
    if (bucket[f] == null) bucket[f] = new ArrayList<>();
    bucket[f].add(e.getKey());
}
// walk bucket high→low, collect until k
```
**Twists:** Top K frequent *words* (tie-break alphabetically) → heap with custom comparator.

### Product of Array Except Self — LC 238 (Medium)
**Problem:** `out[i]` = product of all elements except `nums[i]`, no division.
**Recognize:** "all except self" + "no division" → prefix·suffix products.
**Talk track:** Pass 1 fills `out[i]` with product of everything to the left; pass 2 multiplies by a running product from the right. O(n) time, O(1) extra (output aside).
**Template:**
```java
int n = nums.length; int[] out = new int[n];
out[0] = 1;
for (int i = 1; i < n; i++) out[i] = out[i-1] * nums[i-1];   // left products
int right = 1;
for (int i = n-1; i >= 0; i--) { out[i] *= right; right *= nums[i]; }
```
**Twists:** with division it's trivial — interviewer bans it to force the prefix/suffix idea; handle zeros if division allowed.

### Longest Consecutive Sequence — LC 128 (Medium)
**Problem:** Length of the longest run of consecutive integers (unsorted), in O(n).
**Recognize:** "consecutive" + "O(n)" (so no sort) → hash set + only start counting at run-starts.
**Talk track:** Put all in a set. A number starts a run iff `x-1` not in set; from each start, walk `x+1, x+2, …`. Each element visited O(1) amortized → O(n).
**Template:**
```java
Set<Integer> set = new HashSet<>(); for (int x : nums) set.add(x);
int best = 0;
for (int x : set) {
    if (set.contains(x - 1)) continue;      // not a run start
    int len = 1; while (set.contains(x + len)) len++;
    best = Math.max(best, len);
}
```
**Twists:** longest consecutive in a binary tree → DFS tracking up/down runs.

---

## Two Pointers { #two-pointers }

Two indices moving over the *same* array, usually from **both ends inward** (sorted data,
pair-finding) or as a **slow/fast write head** (in-place filtering). Turns many O(n²) scans
into O(n). If the array is sorted, two pointers is almost always on the table.

### Valid Palindrome — LC 125 (Easy)
**Problem:** Is the string a palindrome, ignoring non-alphanumerics and case?
**Recognize:** "palindrome" / "symmetric from both ends" → converge two pointers.
**Talk track:** `l` from front, `r` from back; skip non-alphanumeric; compare lowercased. O(n)/O(1).
**Template:**
```java
int l = 0, r = s.length() - 1;
while (l < r) {
    while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;
    while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
    if (Character.toLowerCase(s.charAt(l++)) != Character.toLowerCase(s.charAt(r--))) return false;
}
return true;
```
**Twists:** "valid palindrome II" (delete at most one char) → on mismatch, try skipping `l` or `r`.

### Two Sum II — Sorted Input — LC 167 (Medium)
**Problem:** Sorted array; return the 1-indexed pair summing to `target`.
**Recognize:** "sorted" + "pair sum" → both-ends pointers, no extra space.
**Talk track:** Sum too small → move `l` right; too big → move `r` left. O(n)/O(1).
**Template:**
```java
int l = 0, r = nums.length - 1;
while (l < r) {
    int sum = nums[l] + nums[r];
    if (sum == target) return new int[]{l+1, r+1};
    if (sum < target) l++; else r--;
}
```
**Twists:** unsorted → hash map (Two Sum I); 3Sum / 4Sum build on this inner loop.

### 3Sum — LC 15 (Medium)
**Problem:** All unique triples summing to 0.
**Recognize:** "triple sums to X, unique" → sort + fix one + two-pointer inner.
**Talk track:** Sort. For each `i`, two-pointer the rest for `-nums[i]`. Skip duplicates at all three positions. O(n²)/O(1) extra.
**Template:**
```java
Arrays.sort(nums); List<List<Integer>> res = new ArrayList<>();
for (int i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] == nums[i-1]) continue;        // skip dup anchor
    int l = i + 1, r = nums.length - 1;
    while (l < r) {
        int sum = nums[i] + nums[l] + nums[r];
        if (sum < 0) l++;
        else if (sum > 0) r--;
        else { res.add(List.of(nums[i], nums[l], nums[r]));
               while (l < r && nums[l] == nums[l+1]) l++;
               while (l < r && nums[r] == nums[r-1]) r--;
               l++; r--; }
    }
}
```
**Twists:** 3Sum Closest → track min |sum-target|; 4Sum → one more outer loop.

### Container With Most Water — LC 11 (Medium)
**Problem:** Two lines + x-axis form a container; maximize trapped water.
**Recognize:** "max area between two ends" → both-ends pointers, move the shorter wall.
**Talk track:** Area = `min(h[l],h[r]) * (r-l)`. Moving the taller wall can't help (width shrinks, height capped by shorter), so always move the shorter. O(n)/O(1).
**Template:**
```java
int l = 0, r = height.length - 1, best = 0;
while (l < r) {
    best = Math.max(best, Math.min(height[l], height[r]) * (r - l));
    if (height[l] < height[r]) l++; else r--;
}
```
**Twists:** Trapping Rain Water (below) looks similar but sums *all* trapped water, not one container.

### Trapping Rain Water — LC 42 (Hard)
**Problem:** Total water trapped between bars.
**Recognize:** "water trapped over an elevation map" → two pointers tracking left/right max.
**Talk track:** Water over bar `i` = `min(maxLeft, maxRight) - height[i]`. Two-pointer: the side with the smaller max is the bottleneck, so process it. O(n)/O(1). (Prefix-max arrays = O(n) space alternative; monotonic stack also works.)
**Template:**
```java
int l = 0, r = height.length - 1, lMax = 0, rMax = 0, water = 0;
while (l < r) {
    if (height[l] < height[r]) {
        lMax = Math.max(lMax, height[l]);
        water += lMax - height[l]; l++;
    } else {
        rMax = Math.max(rMax, height[r]);
        water += rMax - height[r]; r--;
    }
}
```
**Twists:** 2-D version (Trapping Rain Water II) → min-heap from the border inward.

---

## Sliding Window { #sliding-window }

A **contiguous** window over an array/string whose aggregate you update incrementally
(O(1) per slide) instead of recomputing. *Fixed-size* when a length `k` is given; *variable-size*
(`while`-shrink) when you want the longest/shortest range satisfying a monotone predicate.
*(Deep dives: [Max Points From Cards](sliding-window/max-points-from-cards.md),
[Max Sum of Distinct Subarrays](sliding-window/max-sum-distinct-subarrays.md),
[Fruits Into Baskets](sliding-window/fruits-into-baskets.md).)*

### Best Time to Buy and Sell Stock — LC 121 (Easy)
**Problem:** One buy then one sell, later day; max profit.
**Recognize:** "best single buy/sell", "max future-minus-past gap" → track running min.
**Talk track:** Sweep once, keep cheapest price so far, update best `price - minSoFar`. O(n)/O(1). (A degenerate window: buy = left edge.)
**Template:**
```java
int min = Integer.MAX_VALUE, best = 0;
for (int p : prices) { min = Math.min(min, p); best = Math.max(best, p - min); }
```
**Twists:** unlimited transactions (LC 122) → sum every positive delta (greedy); with cooldown/fee → DP.

### Longest Substring Without Repeating Characters — LC 3 (Medium)
**Problem:** Longest substring with all-distinct characters.
**Recognize:** "longest substring where [no repeats]" → variable window + last-seen map.
**Talk track:** Expand `right`; if the char was seen inside the window, jump `left` past its last index. Track max length. O(n)/O(charset).
**Template:**
```java
Map<Character,Integer> last = new HashMap<>();
int left = 0, best = 0;
for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);
    if (last.containsKey(c) && last.get(c) >= left) left = last.get(c) + 1;
    last.put(c, right);
    best = Math.max(best, right - left + 1);
}
```
**Twists:** at most `k` distinct (LC 340) → shrink while `map.size() > k`; exactly k → atMost(k) - atMost(k-1).

### Longest Repeating Character Replacement — LC 424 (Medium)
**Problem:** Longest substring of one repeated letter after replacing ≤ `k` characters.
**Recognize:** "longest window where (windowLen − maxFreq) ≤ k".
**Talk track:** Window is valid while `len - maxCount <= k` (chars to replace). Shrink when violated. Track `maxCount` of any char in window. O(n)/O(26).
**Template:**
```java
int[] count = new int[26]; int left = 0, maxCount = 0, best = 0;
for (int right = 0; right < s.length(); right++) {
    maxCount = Math.max(maxCount, ++count[s.charAt(right) - 'A']);
    while ((right - left + 1) - maxCount > k) count[s.charAt(left++) - 'A']--;
    best = Math.max(best, right - left + 1);
}
```
**Twists:** "max consecutive ones III" (flip ≤ k zeros) is the same shape with a 0/1 alphabet.

### Minimum Window Substring — LC 76 (Hard)
**Problem:** Smallest substring of `s` containing all chars of `t` (with multiplicity).
**Recognize:** "smallest window containing all of …" → variable window + need-count + `have/need`.
**Talk track:** Expand right, decrement needs; when all needs met (`have == need`), shrink left to minimize while still valid. O(|s|+|t|).
**Template:**
```java
Map<Character,Integer> need = new HashMap<>();
for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);
int have = 0, required = need.size(), left = 0, bestLen = Integer.MAX_VALUE, bestL = 0;
for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);
    if (need.containsKey(c)) { need.merge(c, -1, Integer::sum); if (need.get(c) == 0) have++; }
    while (have == required) {
        if (right - left + 1 < bestLen) { bestLen = right - left + 1; bestL = left; }
        char d = s.charAt(left++);
        if (need.containsKey(d)) { need.merge(d, 1, Integer::sum); if (need.get(d) > 0) have--; }
    }
}
```
**Twists:** Permutation in String / Find All Anagrams → fixed-size window of length |t| + count match.

### Permutation in String — LC 567 (Medium)
**Problem:** Does `s2` contain a permutation of `s1` as a substring?
**Recognize:** "contains a permutation/anagram of X" → fixed window of len(s1) + frequency match.
**Talk track:** Slide a window of size `len(s1)`, maintain a 26-count; when window counts equal `s1` counts, return true. O(n)/O(26).
**Template:**
```java
int[] need = new int[26], win = new int[26];
for (char c : s1.toCharArray()) need[c-'a']++;
int k = s1.length();
for (int i = 0; i < s2.length(); i++) {
    win[s2.charAt(i)-'a']++;
    if (i >= k) win[s2.charAt(i-k)-'a']--;     // evict left
    if (i >= k-1 && Arrays.equals(need, win)) return true;
}
```
**Twists:** Find All Anagrams (LC 438) → collect every start index instead of returning early.

---

## Stack { #stack }

LIFO: the answer depends on the **most recent** unmatched/relevant item. Plain stack for
matching/nesting; **monotonic stack** (kept increasing or decreasing) for "next greater/smaller/warmer".
In Java use `ArrayDeque`, not the legacy `Stack`. *(Deep dive: [Valid Parentheses](stack/valid-parentheses.md).)*

### Valid Parentheses — LC 20 (Easy)
**Problem:** Are the brackets `()[]{}` correctly matched and nested?
**Recognize:** "matching pairs / nesting" → push opens, pop on close.
**Talk track:** Push opener; on a closer, the top must be its partner else invalid; stack empty at end. O(n)/O(n).
**Template:**
```java
Deque<Character> st = new ArrayDeque<>();
Map<Character,Character> match = Map.of(')','(', ']','[', '}','{');
for (char c : s.toCharArray()) {
    if (match.containsKey(c)) { if (st.isEmpty() || st.pop() != match.get(c)) return false; }
    else st.push(c);
}
return st.isEmpty();
```
**Twists:** min-removals to make valid; longest valid parentheses → stack of indices or DP.

### Min Stack — LC 155 (Medium)
**Problem:** Stack with O(1) `push/pop/top/getMin`.
**Recognize:** "min/max alongside stack ops in O(1)" → store the running min with each element.
**Talk track:** Each frame remembers the min *as of when it was pushed* (pair, or a parallel min-stack). Pop discards both. O(1) all ops.
**Template:**
```java
Deque<int[]> st = new ArrayDeque<>();   // {val, minSoFar}
void push(int x) { int m = st.isEmpty() ? x : Math.min(x, st.peek()[1]); st.push(new int[]{x, m}); }
void pop()  { st.pop(); }
int top()   { return st.peek()[0]; }
int getMin(){ return st.peek()[1]; }
```
**Twists:** max stack; queue with min via two stacks / monotonic deque.

### Evaluate Reverse Polish Notation — LC 150 (Medium)
**Problem:** Evaluate an RPN (postfix) expression.
**Recognize:** "postfix / operands then operator" → stack of operands.
**Talk track:** Push numbers; on an operator pop two, apply (mind operand order for `-` and `/`), push result. O(n).
**Template:**
```java
Deque<Integer> st = new ArrayDeque<>();
for (String t : tokens) {
    switch (t) {
        case "+": st.push(st.pop() + st.pop()); break;
        case "*": st.push(st.pop() * st.pop()); break;
        case "-": { int b = st.pop(), a = st.pop(); st.push(a - b); break; }
        case "/": { int b = st.pop(), a = st.pop(); st.push(a / b); break; }
        default: st.push(Integer.parseInt(t));
    }
}
return st.pop();
```
**Twists:** infix evaluation → two stacks (values + ops) with precedence; Basic Calculator.

### Generate Parentheses — LC 22 (Medium)
**Problem:** All valid combinations of `n` pairs of parentheses.
**Recognize:** "generate all valid …" → backtracking with open/close counts (stack-shaped recursion).
**Talk track:** Add `(` while `open < n`; add `)` while `close < open`. Base case length `2n`. O(Catalan(n)).
**Template:**
```java
void build(int open, int close, int n, StringBuilder sb, List<String> out) {
    if (sb.length() == 2*n) { out.add(sb.toString()); return; }
    if (open < n)    { sb.append('('); build(open+1, close, n, sb, out); sb.deleteCharAt(sb.length()-1); }
    if (close < open){ sb.append(')'); build(open, close+1, n, sb, out); sb.deleteCharAt(sb.length()-1); }
}
```
**Twists:** see [Backtracking](#backtracking) for the general template.

### Daily Temperatures — LC 739 (Medium)
**Problem:** For each day, how many days until a warmer temperature.
**Recognize:** "next greater/warmer element" → **monotonic decreasing stack of indices**.
**Talk track:** Keep indices of unresolved days in a decreasing stack; when today is warmer, pop and record the gap. Each index pushed/popped once → O(n).
**Template:**
```java
int[] res = new int[temps.length];
Deque<Integer> st = new ArrayDeque<>();          // indices, decreasing temps
for (int i = 0; i < temps.length; i++) {
    while (!st.isEmpty() && temps[i] > temps[st.peek()]) {
        int j = st.pop(); res[j] = i - j;
    }
    st.push(i);
}
```
**Twists:** Next Greater Element I/II (circular → iterate `2n`); Largest Rectangle, Stock Span — all monotonic-stack.

### Car Fleet — LC 853 (Medium)
**Problem:** Cars on a 1-lane road to a target; count fleets that arrive together.
**Recognize:** "things merge if a faster one catches a slower one ahead" → sort by position desc, monotonic stack of arrival times.
**Talk track:** Sort by start position descending. Compute each car's time-to-target; if a car's time ≤ the fleet ahead, it merges (no new fleet); else it forms a new fleet. O(n log n).
**Template:**
```java
int n = position.length; Integer[] idx = /* indices sorted by position desc */;
Deque<Double> st = new ArrayDeque<>();
for (int i : idx) {
    double time = (double)(target - position[i]) / speed[i];
    if (st.isEmpty() || time > st.peek()) st.push(time);   // new fleet (strictly slower)
}
return st.size();
```
**Twists:** Largest Rectangle in Histogram (LC 84) — the canonical hard monotonic-stack problem (track index + height when popping).

---

## Binary Search { #binary-search }

Halve a search space each step when a **monotone** decision exists: "if mid is too small,
everything left of it is too." Classic on sorted arrays, but the senior move is **binary
search on the answer** — guess a candidate answer, ask a yes/no feasibility question, halve.
*(Deep dive: [Binary Search on Sorted Array](binary-search/binary-search.md).)*

### Binary Search — LC 704 (Easy)
**Problem:** Index of `target` in a sorted array, or -1.
**Recognize:** "sorted" + "find index/exists" → textbook binary search.
**Talk track:** Closed interval `[lo, hi]`, loop while `lo <= hi`, `mid = lo + (hi-lo)/2` to avoid overflow. O(log n).
**Template:**
```java
int lo = 0, hi = nums.length - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] == target) return mid;
    if (nums[mid] < target) lo = mid + 1; else hi = mid - 1;
}
return -1;
```
**Twists:** first/last occurrence → bias mid and keep searching one side; lower_bound/upper_bound.

### Search in Rotated Sorted Array — LC 33 (Medium)
**Problem:** Find `target` in a sorted array rotated at an unknown pivot.
**Recognize:** "sorted but rotated" → binary search where one half is always sorted.
**Talk track:** At each mid, one half `[lo,mid]` or `[mid,hi]` is sorted; check if target lies in the sorted half and recurse there. O(log n).
**Template:**
```java
int lo = 0, hi = nums.length - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] == target) return mid;
    if (nums[lo] <= nums[mid]) {                      // left half sorted
        if (nums[lo] <= target && target < nums[mid]) hi = mid - 1; else lo = mid + 1;
    } else {                                          // right half sorted
        if (nums[mid] < target && target <= nums[hi]) lo = mid + 1; else hi = mid - 1;
    }
}
return -1;
```
**Twists:** with duplicates (LC 81) → shrink `lo`/`hi` when `nums[lo]==nums[mid]==nums[hi]`.

### Find Minimum in Rotated Sorted Array — LC 153 (Medium)
**Problem:** Minimum of a rotated sorted array.
**Recognize:** "rotated, find the pivot/min" → compare mid to `hi`.
**Talk track:** If `nums[mid] > nums[hi]`, the min is right of mid; else it's at mid or left. Converge to the rotation point. O(log n).
**Template:**
```java
int lo = 0, hi = nums.length - 1;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] > nums[hi]) lo = mid + 1; else hi = mid;
}
return nums[lo];
```
**Twists:** compare to `nums[lo]` instead requires handling the not-rotated case carefully — comparing to `hi` is cleaner.

### Koko Eating Bananas — LC 875 (Medium)
**Problem:** Min eating speed `k` so all piles are finished within `h` hours.
**Recognize:** "minimum X such that a condition holds" + monotone feasibility → **binary search on the answer**.
**Talk track:** Feasibility `hoursNeeded(k) <= h` is monotone in `k`. Binary-search `k` in `[1, max(piles)]`, find the smallest feasible. O(n log maxPile).
**Template:**
```java
int lo = 1, hi = Arrays.stream(piles).max().getAsInt();
while (lo < hi) {
    int k = lo + (hi - lo) / 2;
    long hours = 0; for (int p : piles) hours += (p + k - 1L) / k;   // ceil div
    if (hours <= h) hi = k; else lo = k + 1;
}
return lo;
```
**Twists:** Capacity to Ship Packages in D Days, Split Array Largest Sum, Min Days to Make Bouquets — all "min/max feasible value" binary searches; only `feasible()` changes.

### Search a 2D Matrix — LC 74 (Medium)
**Problem:** Search a matrix where each row is sorted and `row[0] > prevRow[last]`.
**Recognize:** "fully sorted if flattened" → binary search on a virtual 1-D index.
**Talk track:** Treat the `m×n` grid as one sorted array of length `m·n`; map `idx → (idx/n, idx%n)`. O(log mn).
**Template:**
```java
int m = matrix.length, n = matrix[0].length, lo = 0, hi = m*n - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2, val = matrix[mid / n][mid % n];
    if (val == target) return true;
    if (val < target) lo = mid + 1; else hi = mid - 1;
}
return false;
```
**Twists:** Search 2D Matrix II (rows & cols sorted, no global order) → staircase from top-right, O(m+n).

### Time Based Key-Value Store — LC 981 (Medium)
**Problem:** `set(key,val,t)` and `get(key,t)` returning the value with the largest timestamp ≤ `t`.
**Recognize:** "latest version at-or-before a timestamp" → per-key sorted list + binary search (upper_bound − 1).
**Talk track:** Timestamps arrive increasing, so append to a per-key list; `get` binary-searches for the rightmost entry with `ts <= t`. O(log n) per get.
**Template:**
```java
Map<String, List<int[]>> store = new HashMap<>();   // key -> sorted [ts, valIdx]
// get: binary search the list for largest ts <= t, return that value (or "")
int lo = 0, hi = list.size() - 1, ans = -1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (list.get(mid)[0] <= t) { ans = mid; lo = mid + 1; } else hi = mid - 1;
}
```
**Twists:** Median of Two Sorted Arrays (LC 23/Hard) → binary search the partition of the smaller array.

---

## Linked List { #linked-list }

Pointer rewiring on nodes you can only traverse forward. Two core tools: the **three-pointer
reversal** (`prev/curr/next`) and **fast/slow pointers** (different speeds to find middle or
detect a cycle; fixed gap for nth-from-end). A **dummy head** simplifies edits at the front.
*(Deep dives: [Reverse Linked List](linked-list/reverse-linked-list.md) plus the
[fast/slow pointer family](linked-list/fast-slow-pointers.md).)*

### Reverse Linked List — LC 206 (Easy)
**Problem:** Reverse a singly linked list, return new head.
**Recognize:** `ListNode` + "reverse / flip direction" → three-pointer rewiring.
**Talk track:** Save `next`, point `curr.next` back to `prev`, march all three. `prev` is the new head when `curr` hits null. O(n)/O(1).
**Template:**
```java
ListNode prev = null, curr = head;
while (curr != null) {
    ListNode next = curr.next;   // save before overwriting
    curr.next = prev;
    prev = curr; curr = next;
}
return prev;
```
**Twists:** reverse between positions m..n; reverse in k-groups (LC 25) — same loop, bounded. *(Deep dive: [Reverse Linked List](linked-list/reverse-linked-list.md).)*

### Linked List Cycle — LC 141 (Easy)
**Problem:** Does the list contain a cycle?
**Recognize:** `ListNode` + "cycle / loop / runs forever" → fast/slow (Floyd).
**Talk track:** slow ×1, fast ×2; on a loop the gap shrinks by 1 each step so they collide; if fast hits null there's no loop. O(n)/O(1).
**Template:**
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next; fast = fast.next.next;
    if (slow == fast) return true;
}
return false;
```
**Twists:** find cycle start (LC 142, below); cycle length → keep walking after collision. *(Deep dive: [Linked List Cycle](linked-list/linked-list-cycle.md).)*

### Linked List Cycle II — LC 142 (Medium)
**Problem:** Return the node where the cycle begins, or null.
**Recognize:** "where does the loop start" → Floyd + reset-to-head.
**Talk track:** After collision, reset one pointer to head; move both at speed 1; they meet at the entry (distance head→entry equals collision→entry mod loop). O(n)/O(1).
**Template:**
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next; fast = fast.next.next;
    if (slow == fast) {                       // collided
        ListNode p = head;
        while (p != slow) { p = p.next; slow = slow.next; }
        return p;                             // cycle entry
    }
}
return null;
```
**Twists:** Find the Duplicate Number (LC 287) is this in disguise — see [Math](#math)/below. *(Deep dive: [Linked List Cycle II](linked-list/linked-list-cycle-ii.md).)*

### Middle of the Linked List — LC 876 (Easy)
**Problem:** Return the middle node (second middle if even length).
**Recognize:** "middle / split in half, one pass" → fast/slow.
**Talk track:** slow ×1, fast ×2; when fast runs off the end, slow is the middle. O(n)/O(1).
**Template:**
```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) { slow = slow.next; fast = fast.next.next; }
return slow;     // start fast at head.next for the FIRST of two middles
```
**Twists:** building block for Reorder List, Palindrome List, Sort List. *(Deep dive: [Middle of the Linked List](linked-list/middle-of-linked-list.md).)*

### Remove Nth Node From End — LC 19 (Medium)
**Problem:** Remove the nth node from the end in one pass.
**Recognize:** "nth from the end" → fixed-gap two pointers + dummy head.
**Talk track:** Give `fast` an `n+1` head start from a dummy, then move both until fast is null; `slow` lands just before the target, splice it out. O(n)/O(1).
**Template:**
```java
ListNode dummy = new ListNode(0, head), slow = dummy, fast = dummy;
for (int i = 0; i <= n; i++) fast = fast.next;     // n+1 ahead
while (fast != null) { slow = slow.next; fast = fast.next; }
slow.next = slow.next.next;                        // unlink
return dummy.next;
```
**Twists:** nth from the end (return node, not remove) → head start of `n`, no dummy needed. *(Deep dive: [Remove Nth Node From End](linked-list/remove-nth-node-from-end.md).)*

### Merge Two Sorted Lists — LC 21 (Easy)
**Problem:** Merge two sorted lists into one sorted list.
**Recognize:** "merge sorted sequences" → dummy head + splice smaller front.
**Talk track:** Walk both with a tail pointer off a dummy; attach the smaller head each step; append the leftover. O(n+m)/O(1).
**Template:**
```java
ListNode dummy = new ListNode(0), tail = dummy;
while (a != null && b != null) {
    if (a.val <= b.val) { tail.next = a; a = a.next; }
    else                { tail.next = b; b = b.next; }
    tail = tail.next;
}
tail.next = (a != null) ? a : b;
return dummy.next;
```
**Twists:** Merge K sorted lists → min-heap of heads, see [Heap](#heap).

### Reorder List — LC 143 (Medium)
**Problem:** Reorder `L0→L1→…→Ln` into `L0→Ln→L1→Ln-1→…`.
**Recognize:** "interleave front and back" → find middle + reverse second half + merge.
**Talk track:** Three reusable subroutines: fast/slow to find middle, reverse the second half, then zip the two halves together. O(n)/O(1).
**Template:**
```java
// 1) middle (slow); 2) reverse second half from slow.next; 3) merge two halves alternately
ListNode slow = head, fast = head;
while (fast.next != null && fast.next.next != null) { slow = slow.next; fast = fast.next.next; }
ListNode second = reverse(slow.next); slow.next = null;
// weave: first = head, second; alternate next pointers until second exhausts
```
**Twists:** Palindrome Linked List (LC 234) → same middle+reverse, then compare halves.

### LRU Cache — LC 146 (Medium)
**Problem:** O(1) `get`/`put` with least-recently-used eviction.
**Recognize:** "O(1) get/put + eviction order" → hash map + doubly linked list.
**Talk track:** Map key→node for O(1) lookup; doubly linked list orders by recency (move-to-front on access, evict from tail). Java shortcut: `LinkedHashMap` with `removeEldestEntry`. O(1) per op.
**Template:**
```java
class LRUCache extends LinkedHashMap<Integer,Integer> {
    private final int cap;
    LRUCache(int cap){ super(16, 0.75f, true); this.cap = cap; }   // accessOrder=true
    public int get(int k){ return super.getOrDefault(k, -1); }
    public void put(int k,int v){ super.put(k, v); }
    protected boolean removeEldestEntry(Map.Entry<Integer,Integer> e){ return size() > cap; }
}
```
**Twists:** LFU Cache (LC 460) → frequency buckets, each an ordered list; expect to hand-roll the DLL if the interviewer bans `LinkedHashMap`.

---

## Trees { #trees }

Most tree problems are **DFS recursion** ("what do I need from my children to answer for
myself?") or **BFS by level** (queue). For BSTs, the in-order traversal is sorted and the
search property prunes half the tree. Default to recursion; mention the iterative stack
version if asked about deep trees.

### Invert Binary Tree — LC 226 (Easy)
**Problem:** Mirror the tree (swap every left/right).
**Recognize:** "mirror / flip a tree" → DFS swap.
**Talk track:** Swap children, recurse both sides. O(n)/O(h).
**Template:**
```java
TreeNode invert(TreeNode root) {
    if (root == null) return null;
    TreeNode l = invert(root.left);
    root.left = invert(root.right);
    root.right = l;
    return root;
}
```
**Twists:** Symmetric Tree (LC 101) → compare left subtree to mirrored right.

### Maximum Depth of Binary Tree — LC 104 (Easy)
**Problem:** Height of the tree.
**Recognize:** "depth/height" → DFS returning 1 + max(children).
**Talk track:** `depth = 1 + max(left, right)`; null is 0. O(n)/O(h).
**Template:**
```java
int depth(TreeNode r) { return r == null ? 0 : 1 + Math.max(depth(r.left), depth(r.right)); }
```
**Twists:** Min depth (careful: a node with one child isn't a leaf); Balanced check; Diameter (below).

### Diameter of Binary Tree — LC 543 (Easy)
**Problem:** Longest path (in edges) between any two nodes.
**Recognize:** "longest path through any node" → DFS returns height, side-effect updates a global best.
**Talk track:** At each node, path through it = leftHeight + rightHeight; track the max while returning height upward. O(n)/O(h).
**Template:**
```java
int best = 0;
int height(TreeNode r) {
    if (r == null) return 0;
    int l = height(r.left), rt = height(r.right);
    best = Math.max(best, l + rt);          // path through r
    return 1 + Math.max(l, rt);
}
```
**Twists:** Max Path Sum (LC 124) → same shape but clamp negative child sums to 0 and track value, not length.

### Lowest Common Ancestor of a BST — LC 235 (Medium)
**Problem:** LCA of two nodes in a BST.
**Recognize:** "LCA in a **BST**" → walk down using the order property.
**Talk track:** If both values < node, go left; both > node, go right; else this node is the split point = LCA. O(h).
**Template:**
```java
TreeNode lca(TreeNode r, TreeNode p, TreeNode q) {
    while (r != null) {
        if (p.val < r.val && q.val < r.val) r = r.left;
        else if (p.val > r.val && q.val > r.val) r = r.right;
        else return r;
    }
    return null;
}
```
**Twists:** LCA of a general binary tree (LC 236) → DFS returning whichever subtree(s) contain p/q.

### Binary Tree Level Order Traversal — LC 102 (Medium)
**Problem:** Values grouped by level, top to bottom.
**Recognize:** "level by level / breadth" → BFS with a queue, snapshot level size.
**Talk track:** Process the queue in chunks of the current level's size. O(n)/O(width).
**Template:**
```java
List<List<Integer>> res = new ArrayList<>();
Queue<TreeNode> q = new LinkedList<>(); if (root != null) q.add(root);
while (!q.isEmpty()) {
    int sz = q.size(); List<Integer> level = new ArrayList<>();
    for (int i = 0; i < sz; i++) {
        TreeNode n = q.poll(); level.add(n.val);
        if (n.left != null) q.add(n.left);
        if (n.right != null) q.add(n.right);
    }
    res.add(level);
}
```
**Twists:** zigzag (reverse alternate levels); Right Side View → last node of each level.

### Validate Binary Search Tree — LC 98 (Medium)
**Problem:** Is the tree a valid BST?
**Recognize:** "valid BST" → DFS with (min, max) bounds, **not** just parent comparison.
**Talk track:** Each node must lie strictly within an inherited open range; tighten the bound as you descend. O(n). (In-order traversal must be strictly increasing — the alternative.)
**Template:**
```java
boolean valid(TreeNode n, long lo, long hi) {
    if (n == null) return true;
    if (n.val <= lo || n.val >= hi) return false;
    return valid(n.left, lo, n.val) && valid(n.right, n.val, hi);
}   // call valid(root, Long.MIN_VALUE, Long.MAX_VALUE)
```
**Twists:** Kth Smallest in BST (LC 230) → in-order, stop at k.

### Construct Tree from Preorder & Inorder — LC 105 (Medium)
**Problem:** Rebuild a tree from its preorder and inorder arrays.
**Recognize:** "rebuild tree from traversals" → preorder gives roots, inorder splits left/right.
**Talk track:** First preorder element is the root; find it in inorder to size the left subtree; recurse. Use a value→index map for O(1) lookups. O(n).
**Template:**
```java
int pre = 0; Map<Integer,Integer> idx;   // value -> inorder index
TreeNode build(int[] preorder, int lo, int hi) {
    if (lo > hi) return null;
    int rootVal = preorder[pre++];
    TreeNode root = new TreeNode(rootVal);
    int mid = idx.get(rootVal);
    root.left  = build(preorder, lo, mid - 1);
    root.right = build(preorder, mid + 1, hi);
    return root;
}
```
**Twists:** from inorder+postorder → consume postorder from the back, build right before left.

### Serialize and Deserialize Binary Tree — LC 297 (Hard)
**Problem:** Encode a tree to a string and back.
**Recognize:** "serialize / encode a tree" → preorder DFS with explicit null markers.
**Talk track:** Preorder with `#` for nulls fully determines the tree. Serialize by DFS join; deserialize by consuming tokens in the same order. O(n).
**Template:**
```java
void ser(TreeNode n, StringBuilder sb) {
    if (n == null) { sb.append("#,"); return; }
    sb.append(n.val).append(',');
    ser(n.left, sb); ser(n.right, sb);
}
TreeNode deser(Deque<String> q) {
    String t = q.poll();
    if (t.equals("#")) return null;
    TreeNode n = new TreeNode(Integer.parseInt(t));
    n.left = deser(q); n.right = deser(q);
    return n;
}
```
**Twists:** BST-only → can skip null markers (bounds reconstruct structure); level-order (BFS) encoding is the common alternative.

---

## Tries { #tries }

A prefix tree: each node is a letter, paths spell words. Reach for it on **prefix** queries,
autocomplete, or "search many words against a board". Lookup/insert is O(word length),
independent of how many words are stored.

### Implement Trie — LC 208 (Medium)
**Problem:** `insert`, `search`, `startsWith`.
**Recognize:** "prefix tree / dictionary with prefix queries" → array-of-26 children + `isEnd`.
**Talk track:** Walk/create child nodes per char; `search` needs `isEnd`, `startsWith` just needs the path to exist. O(L) per op.
**Template:**
```java
class Trie {
    Trie[] next = new Trie[26]; boolean end;
    void insert(String w){ Trie n=this; for(char c:w.toCharArray()){ int i=c-'a'; if(n.next[i]==null) n.next[i]=new Trie(); n=n.next[i]; } n.end=true; }
    boolean find(String w, boolean exact){ Trie n=this; for(char c:w.toCharArray()){ int i=c-'a'; if(n.next[i]==null) return false; n=n.next[i]; } return exact ? n.end : true; }
}
```
**Twists:** map children (`HashMap<Character,Node>`) for large/unicode alphabets.

### Design Add and Search Words — LC 211 (Medium)
**Problem:** Support `.` wildcard matching any one letter in `search`.
**Recognize:** "wildcard in dictionary search" → trie + DFS that branches on `.`.
**Talk track:** Normal chars descend one path; `.` recurses into *all* children. Worst case O(26^L) but tiny in practice. 
**Template:**
```java
boolean dfs(String w, int i, Trie n) {
    if (n == null) return false;
    if (i == w.length()) return n.end;
    char c = w.charAt(i);
    if (c == '.') { for (Trie ch : n.next) if (dfs(w, i+1, ch)) return true; return false; }
    return dfs(w, i+1, n.next[c-'a']);
}
```
**Twists:** see Word Search II.

### Word Search II — LC 212 (Hard)
**Problem:** Find all dictionary words present in a character grid.
**Recognize:** "many words against a board" → build a trie of words, DFS the grid once.
**Talk track:** Trie lets you abandon a grid path the moment no word shares that prefix — far better than running Word Search I per word. Mark visited, prune exhausted trie branches. O(cells · 4^L) worst case, pruned hard in practice.
**Template:**
```java
// 1) insert all words into a trie (store the full word at the end node)
// 2) for each cell, dfs(grid, r, c, trieNode):
//      if char not in node.next -> return
//      advance node; if node.word != null -> add to results, null it out (dedup)
//      mark visited, recurse 4 dirs, unmark
```
**Twists:** dedup found words by nulling the stored word on first hit.

---

## Heap / Priority Queue { #heap }

A heap gives O(log n) insert and O(1) peek of the min/max. Reach for it on **"top/k-th"**,
**"merge k sorted"**, or **"running median"**. Key trick: for *k-th largest*, keep a
size-`k` **min**-heap (the root is your answer); flip for k-th smallest.

### Kth Largest Element in an Array — LC 215 (Medium)
**Problem:** The k-th largest value.
**Recognize:** "k-th largest/smallest" → size-k min-heap (or Quickselect for O(n) avg).
**Talk track:** Maintain a min-heap of size k; the root is the k-th largest. O(n log k). Mention Quickselect for O(n) average if they push.
**Template:**
```java
PriorityQueue<Integer> heap = new PriorityQueue<>();   // min-heap
for (int x : nums) { heap.offer(x); if (heap.size() > k) heap.poll(); }
return heap.peek();
```
**Twists:** Quickselect = partition around a pivot, recurse one side, O(n) average.

### K Closest Points to Origin — LC 973 (Medium)
**Problem:** The k points nearest the origin.
**Recognize:** "k closest/smallest by a metric" → size-k **max**-heap by distance.
**Talk track:** Max-heap of size k keyed on squared distance (no sqrt needed); evict the farthest when over k. O(n log k).
**Template:**
```java
PriorityQueue<int[]> heap = new PriorityQueue<>((a,b) -> (b[0]*b[0]+b[1]*b[1]) - (a[0]*a[0]+a[1]*a[1]));
for (int[] p : points) { heap.offer(p); if (heap.size() > k) heap.poll(); }
```
**Twists:** any "k nearest/cheapest" → same size-k heap with the metric in the comparator.

### Last Stone Weight — LC 1046 (Easy)
**Problem:** Repeatedly smash the two heaviest stones; return the last remaining weight.
**Recognize:** "repeatedly take the largest two" → max-heap.
**Talk track:** Max-heap; poll two, push the difference if nonzero; repeat. O(n log n).
**Template:**
```java
PriorityQueue<Integer> max = new PriorityQueue<>(Collections.reverseOrder());
for (int s : stones) max.offer(s);
while (max.size() > 1) { int a = max.poll(), b = max.poll(); if (a != b) max.offer(a - b); }
return max.isEmpty() ? 0 : max.peek();
```
**Twists:** any "greedily combine extremes" simulation.

### Task Scheduler — LC 621 (Medium)
**Problem:** Min time to run tasks with a cooldown `n` between identical tasks.
**Recognize:** "schedule with cooldown / most frequent first" → greedy with counts (heap or math).
**Talk track:** The most frequent task dictates the skeleton: `(maxCount-1)*(n+1) + (#tasks with maxCount)`; answer is `max(that, total tasks)`. O(n). Heap simulation also works.
**Template:**
```java
int[] freq = new int[26]; for (char c : tasks) freq[c-'A']++;
int max = Arrays.stream(freq).max().getAsInt();
int maxCount = (int) Arrays.stream(freq).filter(f -> f == max).count();
int slots = (max - 1) * (n + 1) + maxCount;
return Math.max(slots, tasks.length);
```
**Twists:** Reorganize String (LC 767) → heap of counts, never place the same char adjacently.

### Find Median from Data Stream — LC 295 (Hard)
**Problem:** Support `addNum` and `findMedian` on a growing stream.
**Recognize:** "running median / balance two halves" → **two heaps** (max-heap low, min-heap high).
**Talk track:** Max-heap holds the smaller half, min-heap the larger; keep sizes balanced (±1). Median is a root or the average of both roots. O(log n) add, O(1) median.
**Template:**
```java
PriorityQueue<Integer> lo = new PriorityQueue<>(Collections.reverseOrder()); // smaller half
PriorityQueue<Integer> hi = new PriorityQueue<>();                            // larger half
void add(int x) {
    lo.offer(x); hi.offer(lo.poll());                 // push to lo, spill its max to hi
    if (hi.size() > lo.size()) lo.offer(hi.poll());   // rebalance
}
double median() { return lo.size() > hi.size() ? lo.peek() : (lo.peek() + hi.peek()) / 2.0; }
```
**Twists:** sliding-window median → two heaps with lazy deletion or a TreeMap.

### Merge K Sorted Lists — LC 23 (Hard)
**Problem:** Merge `k` sorted linked lists into one.
**Recognize:** "merge k sorted sequences" → min-heap of the current heads.
**Talk track:** Push all heads into a min-heap; pop the smallest, append it, push its `next`. O(N log k). (Divide-and-conquer pairwise merge is the O(N log k) alternative.)
**Template:**
```java
PriorityQueue<ListNode> pq = new PriorityQueue<>((a,b) -> a.val - b.val);
for (ListNode l : lists) if (l != null) pq.offer(l);
ListNode dummy = new ListNode(0), tail = dummy;
while (!pq.isEmpty()) {
    ListNode n = pq.poll(); tail.next = n; tail = n;
    if (n.next != null) pq.offer(n.next);
}
return dummy.next;
```
**Twists:** Smallest Range Covering K Lists; merge k sorted arrays — same heap idea.

---

## Backtracking { #backtracking }

Build a candidate incrementally; when it can't lead to a solution, **undo the last choice**
and try the next. The universal shape: *choose → recurse → un-choose*. Reach for it on
"**generate all** subsets/permutations/combinations" or constraint puzzles. Cost is usually
exponential — that's expected; pruning is the senior signal.

### Subsets — LC 78 (Medium)
**Problem:** All subsets (the power set) of distinct numbers.
**Recognize:** "all subsets / power set" → include-or-exclude recursion.
**Talk track:** At each index, branch: take it or skip it. 2^n subsets. O(n·2^n).
**Template:**
```java
void dfs(int i, int[] nums, List<Integer> cur, List<List<Integer>> res) {
    if (i == nums.length) { res.add(new ArrayList<>(cur)); return; }
    cur.add(nums[i]); dfs(i+1, nums, cur, res); cur.remove(cur.size()-1);   // take
    dfs(i+1, nums, cur, res);                                               // skip
}
```
**Twists:** Subsets II (dups) → sort, skip `nums[i]==nums[i-1]` on the skip branch.

### Combination Sum — LC 39 (Medium)
**Problem:** All combinations of candidates (reusable) summing to target.
**Recognize:** "combinations summing to target, reuse allowed" → DFS with a start index, recurse on `i` (not `i+1`).
**Talk track:** Pass `start` to avoid permutations of the same set; recurse on `i` to allow reuse; prune when remaining < 0. O(exponential).
**Template:**
```java
void dfs(int start, int remain, int[] c, List<Integer> cur, List<List<Integer>> res) {
    if (remain == 0) { res.add(new ArrayList<>(cur)); return; }
    if (remain < 0) return;
    for (int i = start; i < c.length; i++) {
        cur.add(c[i]); dfs(i, remain - c[i], c, cur, res); cur.remove(cur.size()-1);
    }
}
```
**Twists:** Combination Sum II (each used once + dups) → recurse on `i+1`, skip duplicate siblings.

### Permutations — LC 46 (Medium)
**Problem:** All orderings of distinct numbers.
**Recognize:** "all permutations / orderings" → swap-in-place or used[] tracking.
**Talk track:** Track which elements are used; append an unused one, recurse, unmark. n! results. O(n·n!).
**Template:**
```java
void dfs(int[] nums, boolean[] used, List<Integer> cur, List<List<Integer>> res) {
    if (cur.size() == nums.length) { res.add(new ArrayList<>(cur)); return; }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true; cur.add(nums[i]);
        dfs(nums, used, cur, res);
        used[i] = false; cur.remove(cur.size()-1);
    }
}
```
**Twists:** Permutations II (dups) → sort + skip `i>0 && nums[i]==nums[i-1] && !used[i-1]`.

### Word Search — LC 79 (Medium)
**Problem:** Does the word exist as a path of adjacent cells in a grid?
**Recognize:** "path spelling a word in a grid" → DFS with backtracking + visited marking.
**Talk track:** From each cell, DFS matching the next char in 4 directions; mark visited (or mutate in place) and restore on backtrack. O(cells·4^L).
**Template:**
```java
boolean dfs(char[][] b, int r, int c, String w, int i) {
    if (i == w.length()) return true;
    if (r<0||c<0||r>=b.length||c>=b[0].length||b[r][c]!=w.charAt(i)) return false;
    char tmp = b[r][c]; b[r][c] = '#';                         // mark visited
    boolean found = dfs(b,r+1,c,w,i+1)||dfs(b,r-1,c,w,i+1)||dfs(b,r,c+1,w,i+1)||dfs(b,r,c-1,w,i+1);
    b[r][c] = tmp;                                             // restore
    return found;
}
```
**Twists:** Word Search II → trie of many words (see [Tries](#tries)).

### Palindrome Partitioning — LC 131 (Medium)
**Problem:** All ways to split a string into palindromic substrings.
**Recognize:** "all partitions where each piece satisfies P" → DFS over cut positions.
**Talk track:** Try every prefix; if it's a palindrome, recurse on the rest. Optionally precompute an `isPalindrome` DP table. O(n·2^n).
**Template:**
```java
void dfs(String s, int start, List<String> cur, List<List<String>> res) {
    if (start == s.length()) { res.add(new ArrayList<>(cur)); return; }
    for (int end = start; end < s.length(); end++) {
        if (isPalindrome(s, start, end)) {
            cur.add(s.substring(start, end+1));
            dfs(s, end+1, cur, res);
            cur.remove(cur.size()-1);
        }
    }
}
```
**Twists:** min cuts (LC 132) → DP, not enumeration.

### N-Queens — LC 51 (Hard)
**Problem:** Place `n` non-attacking queens; return all boards.
**Recognize:** "place items with mutual constraints" → backtrack row by row + O(1) conflict sets.
**Talk track:** One queen per row; track occupied columns and both diagonals (`r+c`, `r-c`) in sets for O(1) checks; place, recurse, remove. O(n!).
**Template:**
```java
Set<Integer> cols = new HashSet<>(), diag = new HashSet<>(), anti = new HashSet<>();
void dfs(int r, int n /*, board */) {
    if (r == n) { /* record board */ return; }
    for (int c = 0; c < n; c++) {
        if (cols.contains(c) || diag.contains(r-c) || anti.contains(r+c)) continue;
        cols.add(c); diag.add(r-c); anti.add(r+c);   /* place */
        dfs(r+1, n);
        cols.remove(c); diag.remove(r-c); anti.remove(r+c);   /* undo */
    }
}
```
**Twists:** N-Queens II → just count; Sudoku Solver → same constraint-set backtracking on a 9×9.

---

## Graphs { #graphs }

Model the problem as nodes + edges, then **DFS** (recursion/stack, connectivity & exploration)
or **BFS** (queue, shortest path in *unweighted* graphs). Grids are graphs in disguise — each
cell is a node with up-to-4 neighbors. Always track **visited** to avoid infinite loops.

### Number of Islands — LC 200 (Medium)
**Problem:** Count connected groups of `'1'` (land) in a grid.
**Recognize:** "count connected regions / flood fill" → DFS/BFS from each unvisited land cell.
**Talk track:** Scan the grid; on unvisited land, flood-fill its whole island (mark visited), increment count. O(m·n).
**Template:**
```java
void sink(char[][] g, int r, int c) {
    if (r<0||c<0||r>=g.length||c>=g[0].length||g[r][c]!='1') return;
    g[r][c] = '0';                       // mark visited
    sink(g,r+1,c); sink(g,r-1,c); sink(g,r,c+1); sink(g,r,c-1);
}
// main: for each cell, if g[r][c]=='1' { count++; sink(g,r,c); }
```
**Twists:** Max Area of Island (return area); Number of Closed Islands; count via union-find (see [Advanced Graphs](#advanced-graphs)).

### Clone Graph — LC 133 (Medium)
**Problem:** Deep-copy a connected undirected graph.
**Recognize:** "clone / deep copy a graph" → DFS/BFS + map old→new.
**Talk track:** Map each original node to its clone to avoid re-cloning and handle cycles; recurse to clone neighbors. O(V+E).
**Template:**
```java
Map<Node,Node> seen = new HashMap<>();
Node clone(Node n) {
    if (n == null) return null;
    if (seen.containsKey(n)) return seen.get(n);
    Node copy = new Node(n.val); seen.put(n, copy);
    for (Node nb : n.neighbors) copy.neighbors.add(clone(nb));
    return copy;
}
```
**Twists:** Copy List with Random Pointer — same old→new map idea on a linked list.

### Rotting Oranges — LC 994 (Medium)
**Problem:** Minutes until all fresh oranges rot, spreading to 4-neighbors each minute.
**Recognize:** "spread/shortest time from multiple sources" → **multi-source BFS**.
**Talk track:** Seed the queue with all initially-rotten cells, BFS level = minute; count fresh, return minutes (or -1 if any fresh remain). O(m·n).
**Template:**
```java
Queue<int[]> q = new LinkedList<>();   // all rotten cells; count fresh
int minutes = 0;
while (!q.isEmpty() && fresh > 0) {
    int sz = q.size();
    for (int i = 0; i < sz; i++) { /* poll, rot fresh 4-neighbors, enqueue, fresh-- */ }
    minutes++;
}
return fresh == 0 ? minutes : -1;
```
**Twists:** Walls and Gates / 01 Matrix → multi-source BFS from all sources at once.

### Pacific Atlantic Water Flow — LC 417 (Medium)
**Problem:** Cells from which water can reach both oceans.
**Recognize:** "reaches both edges / flows to multiple borders" → DFS **inward from the borders**, intersect.
**Talk track:** Instead of testing every cell, flood inward from each ocean's border to find all cells that can reach it (climbing to ≥ height); the answer is the intersection. O(m·n).
**Template:**
```java
// dfs from all Pacific-border cells -> mark reachPacific
// dfs from all Atlantic-border cells -> mark reachAtlantic
// answer = cells where both marks true
void dfs(int[][] h, int r, int c, boolean[][] seen, int prev) {
    if (r<0||c<0||r>=h.length||c>=h[0].length||seen[r][c]||h[r][c]<prev) return;
    seen[r][c] = true;
    dfs(h,r+1,c,seen,h[r][c]); dfs(h,r-1,c,seen,h[r][c]); dfs(h,r,c+1,seen,h[r][c]); dfs(h,r,c-1,seen,h[r][c]);
}
```
**Twists:** Surrounded Regions (LC 130) → mark border-connected `O`s as safe, flip the rest.

### Course Schedule — LC 207 (Medium)
**Problem:** Can you finish all courses given prerequisites (detect a cycle in a directed graph)?
**Recognize:** "prerequisites / ordering / can it be done" → topological sort (cycle ⇒ impossible).
**Talk track:** Build adjacency + indegree; BFS (Kahn's) popping zero-indegree nodes; if you process all `n`, no cycle. O(V+E).
**Template:**
```java
int[] indeg = new int[n]; List<List<Integer>> adj = /* build, indeg[next]++ */;
Queue<Integer> q = new LinkedList<>();
for (int i = 0; i < n; i++) if (indeg[i] == 0) q.add(i);
int done = 0;
while (!q.isEmpty()) {
    int cur = q.poll(); done++;
    for (int nx : adj.get(cur)) if (--indeg[nx] == 0) q.add(nx);
}
return done == n;
```
**Twists:** Course Schedule II → record pop order = the schedule (see [Advanced Graphs](#advanced-graphs)).

---

## Advanced Graphs (Union-Find, Topo, Shortest Path) { #advanced-graphs }

When the graph question is about **ordering** (topological sort), **grouping/connectivity**
(union-find), or **weighted shortest path** (Dijkstra) / **min spanning tree** (Prim/Kruskal).
Recognize the sub-pattern, then drop in the standard structure.

### Course Schedule II — LC 210 (Medium)
**Problem:** Return a valid course order, or empty if impossible.
**Recognize:** "give the order, not just yes/no" → Kahn's topological sort, record output.
**Talk track:** Same as Course Schedule but append nodes as they hit zero indegree; if fewer than `n`, there's a cycle ⇒ return empty. O(V+E).
**Template:**
```java
// build indeg + adj; queue zero-indeg; pop -> order.add(cur); decrement neighbors
return order.size() == n ? order.toArray() : new int[0];
```
**Twists:** Alien Dictionary (LC 269) → derive edges from adjacent word pairs, then topo sort the letters.

### Number of Connected Components — LC 323 (Medium)
**Problem:** Count connected components in an undirected graph.
**Recognize:** "count groups / are these connected / merging sets" → **union-find**.
**Talk track:** Start with `n` components; each edge that unions two different roots reduces the count by 1. Near-O(1) per op with path compression + union by rank. O(E·α).
**Template:**
```java
int[] parent; int find(int x){ while(parent[x]!=x){ parent[x]=parent[parent[x]]; x=parent[x]; } return x; }
int components = n;
for (int[] e : edges) { int a = find(e[0]), b = find(e[1]); if (a != b) { parent[a] = b; components--; } }
```
**Twists:** Redundant Connection (LC 684) → the first edge whose endpoints already share a root; Accounts Merge; Number of Islands via union-find.

### Network Delay Time — LC 743 (Medium)
**Problem:** Time for a signal to reach all nodes from a source (weighted, directed).
**Recognize:** "shortest/cheapest path with **weights**" → Dijkstra (min-heap).
**Talk track:** Min-heap of `(dist, node)`; pop the closest unsettled node, relax its edges. Answer = max settled distance (or -1 if unreachable). O(E log V).
**Template:**
```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> a[0] - b[0]);   // {dist, node}
pq.offer(new int[]{0, src}); int[] dist = new int[n+1]; Arrays.fill(dist, INF);
while (!pq.isEmpty()) {
    int[] cur = pq.poll(); int d = cur[0], u = cur[1];
    if (d > dist[u]) continue;                       // stale
    for (int[] e : adj.get(u)) {                     // e = {v, w}
        if (d + e[1] < dist[e[0]]) { dist[e[0]] = d + e[1]; pq.offer(new int[]{dist[e[0]], e[0]}); }
    }
}
```
**Twists:** Cheapest Flights Within K Stops → Bellman-Ford / Dijkstra with a stop bound; Path With Minimum Effort → Dijkstra on max-edge.

### Min Cost to Connect All Points — LC 1584 (Medium)
**Problem:** Min total cost to connect all points (complete graph, Manhattan weights).
**Recognize:** "connect everything at min total cost" → **Minimum Spanning Tree** (Prim/Kruskal).
**Talk track:** Prim's: grow a tree from any node, always add the cheapest edge to an unvisited node via a min-heap. O(E log V). Kruskal's: sort edges, union-find to skip cycles.
**Template:**
```java
// Prim: min-heap of {cost, node}; pop cheapest unvisited, add cost, push its edges
boolean[] inMST = new boolean[n]; PriorityQueue<int[]> pq = new PriorityQueue<>((a,b)->a[0]-b[0]);
pq.offer(new int[]{0, 0}); int total = 0, used = 0;
while (used < n) {
    int[] cur = pq.poll(); if (inMST[cur[1]]) continue;
    inMST[cur[1]] = true; total += cur[0]; used++;
    for (int j = 0; j < n; j++) if (!inMST[j]) pq.offer(new int[]{dist(cur[1], j), j});
}
```
**Twists:** Kruskal's variant when edges are given explicitly; Connecting Cities With Minimum Cost.

---

## 1-D Dynamic Programming { #dp-1d }

DP = remember subproblem answers to avoid recomputation. Reach for it on "**ways to…**",
"**min/max to reach…**", "**can you make…**" over *one* sequence. The senior approach:
state the **recurrence** (`dp[i]` in terms of earlier `dp`), then optimize space to a few
rolling variables. Start top-down (memoized recursion) if the recurrence is hard to see.

### Climbing Stairs — LC 70 (Easy)
**Problem:** Ways to climb `n` stairs taking 1 or 2 steps.
**Recognize:** "ways to reach n from previous states" → Fibonacci recurrence.
**Talk track:** `dp[i] = dp[i-1] + dp[i-2]`. Only the last two matter → O(n) time, O(1) space.
**Template:**
```java
int a = 1, b = 1;                  // ways to reach step 0 and 1
for (int i = 2; i <= n; i++) { int c = a + b; a = b; b = c; }
return b;
```
**Twists:** Min Cost Climbing Stairs; "1/2/3 steps"; decode-ways shares the shape.

### House Robber — LC 198 (Medium)
**Problem:** Max sum with no two adjacent elements chosen.
**Recognize:** "max with no two adjacent" → take-or-skip recurrence.
**Talk track:** `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` (skip vs rob this). Roll to two vars. O(n)/O(1).
**Template:**
```java
int prev = 0, cur = 0;             // best up to i-2, i-1
for (int x : nums) { int next = Math.max(cur, prev + x); prev = cur; cur = next; }
return cur;
```
**Twists:** House Robber II (circular) → max of robbing [0..n-2] and [1..n-1]; House Robber III (tree DP).

### Coin Change — LC 322 (Medium)
**Problem:** Fewest coins to make `amount` (unbounded coins), or -1.
**Recognize:** "min count to reach a target from given steps" → unbounded knapsack DP.
**Talk track:** `dp[a] = 1 + min over coins of dp[a-coin]`. Bottom-up over amounts. O(amount·coins).
**Template:**
```java
int[] dp = new int[amount + 1]; Arrays.fill(dp, amount + 1); dp[0] = 0;
for (int a = 1; a <= amount; a++)
    for (int c : coins) if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
return dp[amount] > amount ? -1 : dp[amount];
```
**Twists:** Coin Change II (count ways → loop coins outer, see [2-D DP](#dp-2d)); Perfect Squares; Min/Combination Sum IV.

### Longest Increasing Subsequence — LC 300 (Medium)
**Problem:** Length of the longest strictly increasing subsequence.
**Recognize:** "longest increasing subsequence" → O(n²) DP, or O(n log n) patience sorting.
**Talk track:** O(n²): `dp[i] = 1 + max(dp[j])` for `j<i, nums[j]<nums[i]`. Senior answer: keep a `tails` array, binary-search the insertion point → O(n log n).
**Template:**
```java
List<Integer> tails = new ArrayList<>();          // tails[k] = smallest tail of an LIS of length k+1
for (int x : nums) {
    int i = Collections.binarySearch(tails, x);
    if (i < 0) i = -(i + 1);
    if (i == tails.size()) tails.add(x); else tails.set(i, x);
}
return tails.size();
```
**Twists:** Russian Doll Envelopes (2-D LIS); Number of LIS; Longest Increasing Path (grid → DFS+memo).

### Word Break — LC 139 (Medium)
**Problem:** Can `s` be segmented into dictionary words?
**Recognize:** "can the string be split into valid pieces" → DP over prefixes.
**Talk track:** `dp[i]` = is `s[0..i)` segmentable; true if some `j<i` has `dp[j]` and `s[j..i)` in dict. O(n²) (×word check).
**Template:**
```java
Set<String> dict = new HashSet<>(wordDict);
boolean[] dp = new boolean[s.length() + 1]; dp[0] = true;
for (int i = 1; i <= s.length(); i++)
    for (int j = 0; j < i; j++)
        if (dp[j] && dict.contains(s.substring(j, i))) { dp[i] = true; break; }
return dp[s.length()];
```
**Twists:** Word Break II (return all sentences) → DP + backtracking/memoized DFS.

### Maximum Product Subarray — LC 152 (Medium)
**Problem:** Largest product of a contiguous subarray.
**Recognize:** "max product (signs flip)" → track BOTH running max and min.
**Talk track:** A negative flips max↔min, so carry both; reset on the current element. O(n)/O(1).
**Template:**
```java
int max = nums[0], min = nums[0], res = nums[0];
for (int i = 1; i < nums.length; i++) {
    int x = nums[i];
    if (x < 0) { int t = max; max = min; min = t; }   // swap on negative
    max = Math.max(x, max * x);
    min = Math.min(x, min * x);
    res = Math.max(res, max);
}
```
**Twists:** Maximum Subarray (Kadane) is the additive version, see [Greedy](#greedy).

---

## 2-D Dynamic Programming { #dp-2d }

Two changing dimensions: **two strings/sequences** (LCS, edit distance), **a grid**, or
**one sequence + a budget/capacity** (knapsack). State is `dp[i][j]`. The recipe: define what
`dp[i][j]` means, write the transition, set the base row/column, then (often) compress to one
or two rows.

### Unique Paths — LC 62 (Medium)
**Problem:** Count paths from top-left to bottom-right moving only right/down.
**Recognize:** "count grid paths with restricted moves" → `dp[i][j] = dp[i-1][j] + dp[i][j-1]`.
**Talk track:** Each cell is the sum of the cell above and to the left; first row/col are 1. Compress to one row → O(n) space.
**Template:**
```java
int[] row = new int[n]; Arrays.fill(row, 1);
for (int i = 1; i < m; i++)
    for (int j = 1; j < n; j++) row[j] += row[j-1];
return row[n-1];
```
**Twists:** Unique Paths II (obstacles → 0 those cells); Min Path Sum (take min, add cost).

### Longest Common Subsequence — LC 1143 (Medium)
**Problem:** Length of the LCS of two strings.
**Recognize:** "common subsequence/edit between two strings" → 2-D table on the two indices.
**Talk track:** If chars match, `dp[i][j] = dp[i-1][j-1] + 1`; else `max(dp[i-1][j], dp[i][j-1])`. O(mn).
**Template:**
```java
int[][] dp = new int[m+1][n+1];
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        dp[i][j] = (a.charAt(i-1) == b.charAt(j-1))
                 ? dp[i-1][j-1] + 1
                 : Math.max(dp[i-1][j], dp[i][j-1]);
return dp[m][n];
```
**Twists:** Edit Distance (LC 72) → 3 transitions (insert/delete/replace), `1 + min(...)` on mismatch; Longest Common Substring (reset to 0 on mismatch).

### Coin Change II — LC 518 (Medium)
**Problem:** Number of distinct ways to make `amount`.
**Recognize:** "count combinations (order-insensitive)" → coins loop **outside**, amount inside.
**Talk track:** Putting coins on the outer loop prevents counting permutations as distinct. `dp[a] += dp[a-coin]`. O(amount·coins).
**Template:**
```java
int[] dp = new int[amount + 1]; dp[0] = 1;
for (int c : coins)                       // coins outer = combinations, not permutations
    for (int a = c; a <= amount; a++) dp[a] += dp[a - c];
return dp[amount];
```
**Twists:** Combination Sum IV (order *matters*) → swap loops (amount outer); Target Sum (LC 494) → subset-sum DP.

### 0/1 Knapsack (Partition Equal Subset Sum — LC 416, Medium)
**Problem:** Can the array be split into two equal-sum halves?
**Recognize:** "pick a subset hitting an exact sum, each item once" → boolean knapsack.
**Talk track:** Target = total/2 (odd total ⇒ false). `dp[s]` = is sum `s` reachable; iterate sums **downward** so each item is used once. O(n·sum).
**Template:**
```java
int sum = /* total */; if (sum % 2 != 0) return false; int target = sum / 2;
boolean[] dp = new boolean[target + 1]; dp[0] = true;
for (int x : nums)
    for (int s = target; s >= x; s--) dp[s] |= dp[s - x];   // downward = 0/1
return dp[target];
```
**Twists:** Target Sum; Last Stone Weight II — same subset-sum core.

### Best Time to Buy/Sell with Cooldown — LC 309 (Medium)
**Problem:** Max profit, unlimited transactions, 1-day cooldown after selling.
**Recognize:** "state machine: holding / sold / rest" → DP over states.
**Talk track:** Track three states per day — `hold`, `sold` (just sold), `rest`. Transitions encode the cooldown. O(n)/O(1).
**Template:**
```java
int hold = Integer.MIN_VALUE, sold = 0, rest = 0;
for (int p : prices) {
    int prevSold = sold;
    sold = hold + p;                       // sell today
    hold = Math.max(hold, rest - p);       // keep or buy (from rest)
    rest = Math.max(rest, prevSold);       // cooldown day
}
return Math.max(sold, rest);
```
**Twists:** with transaction fee (LC 714); at most k transactions (LC 188) → DP over `[k][hold?]`.

### Longest Increasing Path in a Matrix — LC 329 (Hard)
**Problem:** Longest strictly increasing path in a grid (4 directions).
**Recognize:** "longest path in a DAG-like grid" → DFS + memoization (each cell's answer is reusable).
**Talk track:** From each cell, the longest path only goes to strictly-larger neighbors (so no cycles); memoize per cell. O(m·n).
**Template:**
```java
int[][] memo;   // 0 = uncomputed
int dfs(int[][] g, int r, int c) {
    if (memo[r][c] != 0) return memo[r][c];
    int best = 1;
    for (int[] d : DIRS) {
        int nr = r+d[0], nc = c+d[1];
        if (in(nr,nc) && g[nr][nc] > g[r][c]) best = Math.max(best, 1 + dfs(g, nr, nc));
    }
    return memo[r][c] = best;
}
```
**Twists:** any grid "longest/most where moves are monotone" → DFS + memo.

---

## Greedy { #greedy }

Make the **locally optimal choice** at each step and never reconsider. Hard part is *proving*
it's safe — be ready to argue an exchange argument ("swapping toward the greedy choice never
hurts"). Frequent on max-subarray, jump/reach problems, and scheduling. If greedy is wrong,
it's usually DP.

### Maximum Subarray (Kadane) — LC 53 (Medium)
**Problem:** Largest sum of a contiguous subarray.
**Recognize:** "max contiguous sum" → Kadane: drop the prefix once it goes negative.
**Talk track:** Running sum; if it dips below 0 it can only hurt what follows, so reset to the current element. Track the best. O(n)/O(1).
**Template:**
```java
int cur = nums[0], best = nums[0];
for (int i = 1; i < nums.length; i++) {
    cur = Math.max(nums[i], cur + nums[i]);   // extend or restart
    best = Math.max(best, cur);
}
```
**Twists:** Maximum Product Subarray (track min too, see [1-D DP](#dp-1d)); circular max subarray → total − min subarray.

### Jump Game — LC 55 (Medium)
**Problem:** Can you reach the last index, where `nums[i]` is the max jump from `i`?
**Recognize:** "can you reach the end / furthest reach" → greedy furthest-reachable.
**Talk track:** Track the furthest index reachable so far; if `i` ever exceeds it, you're stuck. O(n)/O(1).
**Template:**
```java
int reach = 0;
for (int i = 0; i < nums.length; i++) {
    if (i > reach) return false;
    reach = Math.max(reach, i + nums[i]);
}
return true;
```
**Twists:** Jump Game II (min jumps) → greedy BFS-by-levels tracking current jump's farthest.

### Gas Station — LC 134 (Medium)
**Problem:** Starting index to complete the circuit, or -1.
**Recognize:** "circular tour feasibility / find a valid start" → greedy with a running tank.
**Talk track:** If total gas ≥ total cost a solution exists. Sweep once; whenever the tank goes negative, the start must be *after* the current index, so reset start. O(n)/O(1).
**Template:**
```java
int total = 0, tank = 0, start = 0;
for (int i = 0; i < gas.length; i++) {
    int diff = gas[i] - cost[i]; total += diff; tank += diff;
    if (tank < 0) { start = i + 1; tank = 0; }      // restart after failure
}
return total >= 0 ? start : -1;
```
**Twists:** Candy (LC 135) → two sweeps (left-to-right, right-to-left) taking the max.

### Partition Labels — LC 763 (Medium)
**Problem:** Split the string into max parts so each letter appears in only one part.
**Recognize:** "partition so no element spans two parts" → greedy by last-occurrence.
**Talk track:** Record each char's last index; sweep extending the current part's end to the max last-index seen; cut when `i` reaches that end. O(n).
**Template:**
```java
int[] last = new int[26]; for (int i = 0; i < s.length(); i++) last[s.charAt(i)-'a'] = i;
int start = 0, end = 0; List<Integer> res = new ArrayList<>();
for (int i = 0; i < s.length(); i++) {
    end = Math.max(end, last[s.charAt(i)-'a']);
    if (i == end) { res.add(end - start + 1); start = i + 1; }
}
```
**Twists:** Merge Intervals viewed as last-occurrence ranges; Hand of Straights (LC 846) → greedy from smallest using a TreeMap.

---

## Intervals { #intervals }

Pairs `[start, end]`. The unlock is almost always **sort first** — by start to *merge*, by end
to *schedule/maximize count*. Then sweep once, comparing each interval to the last one kept.
*(Deep dives: [Merge Intervals](intervals/merge-intervals.md),
[Insert Interval](intervals/insert-interval.md),
[Non-Overlapping Intervals](intervals/non-overlapping-intervals.md),
[Can Attend Meetings](intervals/can-attend-meetings.md),
[Employee Free Time](intervals/employee-free-time.md).)*

### Merge Intervals — LC 56 (Medium)
**Problem:** Merge all overlapping intervals.
**Recognize:** "combine overlapping ranges" → sort by start, extend the last kept.
**Talk track:** Sort by start; if the current starts ≤ last end, extend the last's end; else push a new interval. O(n log n).
**Template:**
```java
Arrays.sort(intervals, (a,b) -> a[0] - b[0]);
List<int[]> out = new ArrayList<>();
for (int[] iv : intervals) {
    if (out.isEmpty() || iv[0] > out.get(out.size()-1)[1]) out.add(iv);
    else out.get(out.size()-1)[1] = Math.max(out.get(out.size()-1)[1], iv[1]);
}
```
**Twists:** Insert Interval (pre-sorted, 3-phase, no re-sort); Interval List Intersections (two-pointer over two sorted lists). *(Deep dive: [Merge Intervals](intervals/merge-intervals.md).)*

### Non-Overlapping Intervals — LC 435 (Medium)
**Problem:** Min intervals to remove so the rest don't overlap.
**Recognize:** "max non-overlapping / fewest removals" → sort by **end**, greedily keep earliest-ending.
**Talk track:** Sort by end; keep an interval only if it starts at/after the last kept end (classic activity selection); count the rest as removals. O(n log n).
**Template:**
```java
Arrays.sort(intervals, (a,b) -> a[1] - b[1]);
int end = Integer.MIN_VALUE, removed = 0;
for (int[] iv : intervals) {
    if (iv[0] >= end) end = iv[1];        // keep
    else removed++;                       // overlaps -> remove
}
```
**Twists:** Max number of non-overlapping intervals = total − removed; Minimum Arrows to Burst Balloons (same, count groups). *(Deep dive: [Non-Overlapping Intervals](intervals/non-overlapping-intervals.md).)*

### Meeting Rooms II — LC 253 (Medium)
**Problem:** Min meeting rooms needed (max concurrent meetings).
**Recognize:** "how many resources at peak overlap" → min-heap of end times, or sweep-line.
**Talk track:** Sort by start; min-heap of end times; before each meeting, free rooms whose end ≤ its start; heap size at the peak = answer. O(n log n).
**Template:**
```java
Arrays.sort(intervals, (a,b) -> a[0] - b[0]);
PriorityQueue<Integer> ends = new PriorityQueue<>();   // earliest end on top
for (int[] iv : intervals) {
    if (!ends.isEmpty() && ends.peek() <= iv[0]) ends.poll();   // reuse a room
    ends.offer(iv[1]);
}
return ends.size();
```
**Twists:** Can Attend Meetings (LC 252) → just check any overlap after sorting; sweep-line with +1/−1 events also gives the peak. *(Deep dives: [Can Attend Meetings](intervals/can-attend-meetings.md), [Employee Free Time](intervals/employee-free-time.md).)*

---

## Bit Manipulation { #bit }

Reach for bit tricks when you see "**without using +/−**", "**single/duplicate number**",
"**count bits**", or tight space constraints. Core moves: `XOR` cancels pairs (`x^x=0`),
`x & (x-1)` clears the lowest set bit, `x & 1` tests parity, `1 << i` masks bit `i`.

### Single Number — LC 136 (Easy)
**Problem:** Every element appears twice except one; find it.
**Recognize:** "everything paired except one" → XOR everything.
**Talk track:** `a^a=0` and XOR is commutative, so all pairs cancel, leaving the unique value. O(n)/O(1).
**Template:**
```java
int x = 0; for (int n : nums) x ^= n; return x;
```
**Twists:** Single Number II (appears 3×) → bit-count mod 3; Single Number III (two uniques) → XOR then split by a differing bit.

### Number of 1 Bits — LC 191 (Easy)
**Problem:** Count set bits (Hamming weight).
**Recognize:** "count set bits / population count" → `n & (n-1)` drops the lowest set bit.
**Talk track:** Each `n &= n-1` removes one set bit, so the loop runs once per 1-bit. O(#bits).
**Template:**
```java
int count = 0; while (n != 0) { n &= (n - 1); count++; } return count;
```
**Twists:** Counting Bits (below) builds an array of these.

### Counting Bits — LC 338 (Easy)
**Problem:** For every `i` in `[0,n]`, count of set bits.
**Recognize:** "set bits for a range" → DP on bits: `dp[i] = dp[i >> 1] + (i & 1)`.
**Talk track:** `i` has the bits of `i/2` plus its lowest bit. O(n).
**Template:**
```java
int[] dp = new int[n + 1];
for (int i = 1; i <= n; i++) dp[i] = dp[i >> 1] + (i & 1);
```
**Twists:** `dp[i] = dp[i & (i-1)] + 1` is an equivalent recurrence.

### Missing Number — LC 268 (Easy)
**Problem:** One number missing from `[0..n]`.
**Recognize:** "one missing from a full range" → XOR indices with values (or Gauss sum).
**Talk track:** XOR all indices `0..n` and all values; pairs cancel, leaving the missing number. O(n)/O(1), overflow-safe (vs sum formula).
**Template:**
```java
int x = nums.length;                       // accounts for index n
for (int i = 0; i < nums.length; i++) x ^= i ^ nums[i];
return x;
```
**Twists:** sum formula `n(n+1)/2 - actualSum`; Find All Numbers Disappeared → index-marking.

### Sum of Two Integers — LC 371 (Medium)
**Problem:** Add two ints without `+` or `−`.
**Recognize:** "add without arithmetic operators" → XOR (sum w/o carry) + AND<<1 (carry).
**Talk track:** XOR adds bit-wise ignoring carry; `(a & b) << 1` is the carry; loop until carry is 0. O(1) (≤32 iterations).
**Template:**
```java
while (b != 0) { int carry = (a & b) << 1; a = a ^ b; b = carry; }
return a;
```
**Twists:** Reverse Bits (LC 190) → shift result left, OR in `n`'s low bit, shift `n` right.

---

## Math & Geometry { #math }

A grab-bag: in-place matrix transforms (rotate, spiral, zero-out), number theory (digits,
GCD), and fast exponentiation. Usually no fancy data structure — the trick is the *insight*
(rotate = transpose + reflect; fast power = square and halve).

### Rotate Image — LC 48 (Medium)
**Problem:** Rotate an `n×n` matrix 90° clockwise, in place.
**Recognize:** "rotate matrix in place" → transpose then reverse each row.
**Talk track:** Transpose (swap across the diagonal), then reverse each row → 90° CW. O(n²)/O(1).
**Template:**
```java
for (int i = 0; i < n; i++)                       // transpose
    for (int j = i + 1; j < n; j++) { int t = m[i][j]; m[i][j] = m[j][i]; m[j][i] = t; }
for (int[] row : m) {                             // reverse each row
    for (int l = 0, r = n - 1; l < r; l++, r--) { int t = row[l]; row[l] = row[r]; row[r] = t; }
}
```
**Twists:** counter-clockwise → transpose + reverse columns (or reverse rows first).

### Spiral Matrix — LC 54 (Medium)
**Problem:** Return all elements in spiral order.
**Recognize:** "traverse in spiral / layers" → four shrinking boundaries.
**Talk track:** Maintain `top/bottom/left/right`; walk right, down, left, up, shrinking each bound after its pass; stop when bounds cross. O(m·n).
**Template:**
```java
int top=0, bot=m-1, left=0, right=n-1;
while (top <= bot && left <= right) {
    for (int c=left;c<=right;c++) res.add(grid[top][c]); top++;
    for (int r=top;r<=bot;r++)   res.add(grid[r][right]); right--;
    if (top<=bot) { for (int c=right;c>=left;c--) res.add(grid[bot][c]); bot--; }
    if (left<=right) { for (int r=bot;r>=top;r--) res.add(grid[r][left]); left++; }
}
```
**Twists:** Spiral Matrix II (fill 1..n² in spiral); the boundary checks on the last two loops prevent double-printing.

### Set Matrix Zeroes — LC 73 (Medium)
**Problem:** If a cell is 0, zero its entire row and column, in place.
**Recognize:** "mark rows/cols to clear, O(1) space" → use first row/col as markers.
**Talk track:** Use row 0 and col 0 as flags for which rows/cols to zero (plus one var for col 0 itself); apply in a second pass. O(mn)/O(1).
**Template:**
```java
// 1) scan: if m[i][j]==0, set m[i][0]=0 and m[0][j]=0 (track col0 separately)
// 2) for i,j >= 1: if m[i][0]==0 || m[0][j]==0 -> m[i][j]=0
// 3) handle first row / first col last using the saved flags
```
**Twists:** the O(m+n) space version (two boolean arrays) is the easy first answer; in-place is the follow-up.

### Pow(x, n) — LC 50 (Medium)
**Problem:** Compute `x^n` efficiently.
**Recognize:** "fast exponentiation" → square-and-halve (binary exponentiation).
**Talk track:** Halve the exponent each step, squaring the base; multiply in the base on odd bits. Handle negative `n` (use `1/x`, careful with `Integer.MIN_VALUE`). O(log n).
**Template:**
```java
double res = 1; long e = Math.abs((long) n);
while (e > 0) { if ((e & 1) == 1) res *= x; x *= x; e >>= 1; }
return n < 0 ? 1 / res : res;
```
**Twists:** modular exponentiation (`% mod` each multiply); matrix power for Fibonacci in O(log n).

### Happy Number — LC 202 (Easy)
**Problem:** Repeatedly replace `n` by the sum of squares of its digits; does it reach 1?
**Recognize:** "iterate a transform, detect a loop" → cycle detection (set or fast/slow).
**Talk track:** The sequence either hits 1 or cycles; detect repeats with a set (or Floyd's fast/slow, O(1) space). 
**Template:**
```java
Set<Integer> seen = new HashSet<>();
while (n != 1 && seen.add(n)) {
    int sum = 0; while (n > 0) { int d = n % 10; sum += d * d; n /= 10; }
    n = sum;
}
return n == 1;
```
**Twists:** O(1) space → reuse the [fast/slow cycle](#linked-list) idea on the transform.

---

## See also

- [Java Quick Hacks](java-quick-hacks.md) — the Java-API idioms (Map.merge, Deque, PriorityQueue, etc.) these templates lean on.
- [Solve by Negation](solve-by-negation.md) — count/detect the *opposite* when the positive case fans out.
- Pattern deep-dives with full walkthroughs: [Sliding Window](sliding-window/index.md), [Intervals](intervals/index.md), [Stack](stack/index.md), [Linked List](linked-list/index.md), [Binary Search](binary-search/index.md).
