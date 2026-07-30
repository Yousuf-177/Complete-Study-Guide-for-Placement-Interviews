```
╔══════════════════════════════════════════════════════╗
║   DATA STRUCTURES & ALGORITHMS — Placement Cheat Sheet║
║   Subject: DSA  |  Level: Campus Placements/SDE       ║
╚══════════════════════════════════════════════════════╝
```

---

## ⚡ 1. CORE CONCEPTS (30-second recall)

- **Data Structure** → a way of organizing data for efficient access & modification.
- **Algorithm** → step-by-step procedure to solve a problem.
- **Time Complexity** → how runtime grows with input size (n).
- **Space Complexity** → how memory usage grows with input size (n).
- **In-place algorithm** → uses O(1) extra space (modifies input directly).
- **Stable sort** → preserves relative order of equal elements.
- **Recursion** → function calling itself; needs a base case to terminate.
- **Divide and Conquer** → break problem into subproblems, solve independently, combine results.
- **Dynamic Programming (DP)** → break problem into overlapping subproblems, store results to avoid recomputation.
- **Greedy Algorithm** → makes locally optimal choice at each step, hoping for global optimum (doesn't always work).
- **Backtracking** → try a solution, undo ("backtrack") if it fails, try next option.
- **Amortized Analysis** → average time per operation over a sequence of operations (e.g., dynamic array resizing).
- **Hashing** → mapping data to fixed-size values (hash) for O(1) average lookup.

---

## 📐 2. TIME COMPLEXITY CHEAT TABLE (Big-O)

```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
Best ─────────────────────────────────────────────────────► Worst
```

| Complexity | Name | Example |
|---|---|---|
| O(1) | Constant | Array access, hash lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Single loop, linear search |
| O(n log n) | Linearithmic | Merge sort, Quick sort (avg), Heap sort |
| O(n²) | Quadratic | Bubble/Insertion/Selection sort, nested loops |
| O(2ⁿ) | Exponential | Recursive Fibonacci (no memoization), subsets |
| O(n!) | Factorial | Permutations, brute-force TSP |

💡 **Mnemonic** → "**1**-Constant, **Log**-Halving, **N**-Loop, **N Log N**-Divide & Sort, **N²**-Nested, **2ⁿ**-Choices, **N!**-Arrangements"

---

## 🧩 3. ARRAY vs LINKED LIST

| Basis | Array | Linked List |
|---|---|---|
| Memory | Contiguous | Non-contiguous (nodes linked via pointers) |
| Access | O(1) random access | O(n) sequential access |
| Insertion/Deletion (middle) | O(n) — shifting needed | O(1) if node reference known |
| Insertion/Deletion (start) | O(n) | O(1) |
| Memory overhead | None extra | Extra pointer per node |
| Cache friendliness | High (contiguous) | Low (scattered in memory) |

---

## 🔗 4. LINKED LIST TYPES & CORE OPERATIONS

- **Singly Linked List** → each node points to next; traversal one direction only.
- **Doubly Linked List** → each node has prev + next pointers; traversal both directions.
- **Circular Linked List** → last node points back to first (singly or doubly).

```python
# Reverse a singly linked list (iterative) — VERY common interview question
def reverse(head):
    prev = None
    curr = head
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt
    return prev  # new head
```

**Floyd's Cycle Detection (Tortoise & Hare)** → use slow (1 step) & fast (2 steps) pointers; if they meet, there's a cycle.

---

## 📚 5. STACK vs QUEUE

| Basis | Stack | Queue |
|---|---|---|
| Order | LIFO (Last In First Out) | FIFO (First In First Out) |
| Insertion | push() → top | enqueue() → rear |
| Removal | pop() → top | dequeue() → front |
| Use cases | Recursion, undo, DFS, expression eval, backtracking | Scheduling, BFS, buffering, task queues |

**Variants:** Circular Queue, Deque (double-ended, insert/remove both ends), Priority Queue (highest priority served first, usually via Heap).

💡 **Mnemonic** → "Stack = Stack of plates (top only) | Queue = Queue at ticket counter (front & back)"

---

## 🌳 6. TREES

### Binary Tree Types
| Type | Rule |
|---|---|
| Binary Tree | Each node has at most 2 children |
| Binary Search Tree (BST) | Left subtree < node < right subtree, for every node |
| Balanced BST (AVL, Red-Black) | Height difference between subtrees bounded (self-balancing) |
| Complete Binary Tree | All levels filled except possibly last, filled left to right |
| Full Binary Tree | Every node has 0 or 2 children (never 1) |
| Perfect Binary Tree | All internal nodes have 2 children, all leaves at same level |
| Heap | Complete Binary Tree; Min-Heap (parent ≤ children) or Max-Heap (parent ≥ children) |

### Tree Traversals
```
Inorder (Left, Root, Right)   → gives sorted order for BST
Preorder (Root, Left, Right)  → used to copy/serialize tree
Postorder (Left, Right, Root) → used to delete tree, evaluate expression trees
Level Order (BFS, queue-based) → level by level
```

```python
# Inorder traversal (recursive)
def inorder(root):
    if root:
        inorder(root.left)
        print(root.val)
        inorder(root.right)
```

**BST operations:** Search/Insert/Delete → O(log n) average, O(n) worst case (skewed tree).
**AVL Tree** → self-balancing BST; rotations (LL, RR, LR, RL) keep height balanced → guarantees O(log n).

💡 **Mnemonic** → "In-order = In sorted order (Left-Root-Right); Pre-order = Root first (Prefix); Post-order = Root last (Postfix)"

---

## 🕸️ 7. GRAPHS

### Representations
| Method | Space | Edge Lookup | Best for |
|---|---|---|---|
| Adjacency Matrix | O(V²) | O(1) | Dense graphs |
| Adjacency List | O(V+E) | O(degree) | Sparse graphs (most common) |

### Traversal Algorithms
```
BFS (Breadth First Search) → uses Queue → level-by-level, shortest path in unweighted graph
DFS (Depth First Search)   → uses Stack/Recursion → explores as deep as possible first
```

### Key Graph Algorithms
| Algorithm | Purpose | Complexity |
|---|---|---|
| **Dijkstra's** | Shortest path (non-negative weights) | O((V+E) log V) with min-heap |
| **Bellman-Ford** | Shortest path (handles negative weights, detects negative cycle) | O(V·E) |
| **Floyd-Warshall** | All-pairs shortest path | O(V³) |
| **Kruskal's** | Minimum Spanning Tree (edge-based, uses Union-Find) | O(E log E) |
| **Prim's** | Minimum Spanning Tree (vertex-based, uses min-heap) | O(E log V) |
| **Topological Sort** | Linear ordering of DAG nodes (dependency resolution) | O(V+E) |
| **Union-Find (DSU)** | Detect cycles, track connected components | ~O(α(n)) per op (near constant) |

💡 **Mnemonic** → "**BFS** = Broad First (level by level, like ripples in water); **DFS** = Deep First (go as far as possible, then backtrack)"

---

## 📊 8. SORTING ALGORITHMS COMPARISON

| Algorithm | Best | Average | Worst | Space | Stable? |
|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ No |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ Yes |

**When asked "which sort to use":**
- Nearly sorted data → **Insertion Sort** (adaptive, fast on nearly sorted).
- Guaranteed O(n log n), stability needed → **Merge Sort**.
- Average-case speed, in-place → **Quick Sort** (but worst case O(n²) on bad pivot).
- Fixed extra memory not allowed, O(n log n) guaranteed → **Heap Sort**.

---

## 🔍 9. SEARCHING

```python
# Binary Search — O(log n), array must be sorted
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

**Variants tested in interviews:** search in rotated sorted array, first/last occurrence, search in 2D matrix, floor/ceil of a value.

---

## 🧠 10. DYNAMIC PROGRAMMING (DP)

**Core idea:** if a problem has (1) **Optimal Substructure** (solution built from optimal solutions of subproblems) and (2) **Overlapping Subproblems** (same subproblems solved repeatedly), DP applies.

**Two approaches:**
- **Memoization (Top-Down)** → recursion + cache (store results in a map/array).
- **Tabulation (Bottom-Up)** → build solution iteratively from base cases upward.

📝 **Example: Fibonacci with memoization**
```python
def fib(n, memo={}):
    if n <= 1: return n
    if n in memo: return memo[n]
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]
```

**Classic DP problems to know cold:** 0/1 Knapsack, Longest Common Subsequence (LCS), Longest Increasing Subsequence (LIS), Coin Change, Edit Distance, Matrix Chain Multiplication, Subset Sum.

💡 **Mnemonic** → "DP = Don't (re)Peat work — remember what you've already computed."

---

## 📊 11. GREEDY vs DP vs DIVIDE & CONQUER

| Basis | Greedy | Dynamic Programming | Divide & Conquer |
|---|---|---|---|
| Choice | Locally optimal, never reconsidered | Explores all options, picks best via subproblems | Splits into independent subproblems |
| Subproblems | Not necessarily overlapping | Overlapping (this is why we cache) | Non-overlapping, independent |
| Guarantees global optimum? | Not always | Yes (if applicable) | Yes |
| Example | Activity selection, Huffman coding | Knapsack, LCS | Merge Sort, Quick Sort, Binary Search |

---

## ⚠️ 12. COMMON EXAM/INTERVIEW TRAPS

- ❌ Wrong: Quick Sort is always O(n log n) → ✅ Right: Worst case is O(n²) with bad pivot choice (e.g., already sorted array + first-element pivot).
- ❌ Wrong: Recursion without base case eventually stops → ✅ Right: Missing/wrong base case causes infinite recursion → stack overflow.
- ❌ Wrong: Greedy always gives optimal solution → ✅ Right: Greedy works only for problems with the "greedy choice property" (e.g., fails on 0/1 Knapsack, works on Fractional Knapsack).
- ❌ Wrong: Hash Map lookup is always O(1) → ✅ Right: O(1) average case; worst case O(n) with many collisions.
- ❌ Wrong: BFS and DFS always give the same traversal order → ✅ Right: BFS explores level-by-level (queue), DFS explores depth-first (stack/recursion) — very different orders.
- ❌ Wrong: A Balanced BST and a Heap are the same thing → ✅ Right: BST maintains full sorted order (left<node<right); Heap only maintains parent-child ordering, not full order.
- ❌ Wrong: Array and ArrayList/Vector have identical time complexity for insertion → ✅ Right: Dynamic arrays have amortized O(1) append, but worst-case O(n) when resizing occurs.

---

## 📝 13. QUICK CODE PATTERNS (High-Frequency Interview Templates)

```python
# Two-pointer technique (e.g., pair sum in sorted array)
def two_sum_sorted(arr, target):
    l, r = 0, len(arr) - 1
    while l < r:
        s = arr[l] + arr[r]
        if s == target: return (l, r)
        elif s < target: l += 1
        else: r -= 1
    return None

# Sliding window (e.g., max sum subarray of size k)
def max_sum_window(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i-k]
        max_sum = max(max_sum, window_sum)
    return max_sum

# Kadane's Algorithm (max subarray sum)
def kadane(arr):
    max_end = max_so_far = arr[0]
    for x in arr[1:]:
        max_end = max(x, max_end + x)
        max_so_far = max(max_so_far, max_end)
    return max_so_far
```

---

## 🎯 14. LAST-MINUTE INTERVIEW TIPS

1. Always **state time & space complexity** of your solution out loud, even if not asked.
2. **Clarify constraints first** (input size, sorted or not, duplicates allowed?) before coding — shows structured thinking.
3. Start with a **brute-force approach**, then optimize — interviewers want to see your thought process, not just the final answer.
4. For array/string problems, always consider **two-pointer** or **sliding window** first — they solve a huge fraction of interview questions optimally.
5. For "find optimal value" problems, ask yourself: "Can this be Greedy? Does it have overlapping subproblems (DP)? Or is it simple traversal?"
6. Dry-run your code on a **small example + an edge case** (empty input, single element, duplicates) before saying "done."
7. Know your **language's built-in complexities** (e.g., Python list `.pop(0)` is O(n), dict lookup is O(1) average) — using them wrong can silently blow your complexity analysis.

---

## 🔑 15. ONE-GLANCE SUMMARY BOX

```
┌───────────────────────────────────────────────────────────┐
│  🔑 MUST-KNOW ESSENTIALS — DSA PLACEMENT                   │
│  1. Big-O ladder: 1 < log n < n < n log n < n² < 2ⁿ < n!    │
│  2. Array=fast access/slow insert | List=slow access/      │
│     fast insert at known node                              │
│  3. Stack=LIFO(DFS,recursion) | Queue=FIFO(BFS,scheduling) │
│  4. BST: search/insert/delete O(log n) avg, O(n) worst     │
│  5. BFS=level order(queue) | DFS=depth order(stack/recur)  │
│  6. Merge Sort=stable,O(n log n) guaranteed, O(n) space    │
│     Quick Sort=avg O(n log n), worst O(n²), in-place       │
│  7. DP = optimal substructure + overlapping subproblems    │
│  8. Greedy ≠ always optimal — verify greedy-choice property│
│  9. Dijkstra=non-negative weights | Bellman-Ford=handles   │
│     negative weights                                        │
│  10. Two-pointer & Sliding Window solve most array/string  │
│      interview problems efficiently                         │
└───────────────────────────────────────────────────────────┘
```

---

*Want me to go deeper on any section (e.g., detailed DP problem walkthroughs, graph algorithm derivations, or a mock DSA interview coding round), or create a practice quiz to test yourself?*
