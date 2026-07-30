# The Complete DSA Study Guide
### From First Principles to Placement-Ready Mastery

> **How to use this guide:** Read each topic in order — later topics build on earlier ones. For every structure/algorithm you'll find: an intuitive analogy, a step-by-step mechanical walkthrough, complexity analysis (best/average/worst), clean pseudocode, and the interview "pattern recognition" cues that tell you *when* to reach for this tool. Don't just read — trace through the pseudocode with a pencil and a small example before moving on.

---

## Table of Contents

**Part 0 — Foundations**
0. How to Think About Complexity (Big-O)

**Part 1 — Linear Data Structures**
1. Arrays
2. Strings
3. Linked Lists
4. Stacks
5. Queues (incl. Deque & Priority Queue)

**Part 2 — Algorithmic Thinking**
6. Recursion & Backtracking
7. Searching Algorithms
8. Sorting Algorithms
9. Two Pointers & Sliding Window

**Part 3 — Non-Linear Data Structures**
10. Hashing
11. Trees (Binary Tree, BST, Balanced Trees, Heaps)
12. Tries
13. Graphs (Representation, Traversal, Shortest Path, MST, Union-Find)

**Part 4 — Advanced Problem-Solving Paradigms**
14. Dynamic Programming
15. Greedy Algorithms
16. Bit Manipulation

**Part 5 — Wrap-up**
17. Pattern Cheat-Sheet & Roadmap

---

# Part 0 — Foundations

## 0. How to Think About Complexity (Big-O)

### Intuitive Explanation
Imagine you're looking for a friend's phone number.
- If you have an **unsorted pile** of business cards, you must check them one by one — the more cards, the longer it takes proportionally. This is **O(n)**.
- If the cards are **sorted alphabetically**, you can flip to the middle, decide "left half or right half," and repeat — this is **O(log n)**, because you halve the problem every step.
- If you have them memorized in your **phone's contacts app** (a hash map), you find it almost instantly regardless of how many contacts exist — **O(1)**.

Big-O describes **how the work grows as input size (n) grows** — not the exact time, but the *shape* of the growth curve.

### Step-by-Step: How to Derive Big-O
1. Identify the input size variable(s) — usually `n` (array length), sometimes `m, n` for two inputs, or `V, E` for graphs (vertices, edges).
2. Count how many times the core operation (comparison, addition, recursive call) executes as a function of `n`.
3. Drop constants and lower-order terms: `3n² + 5n + 2` → `O(n²)`.
4. If there are nested loops each depending on `n`, multiply their complexities. If loops are sequential, add them (then keep the dominant term).
5. For recursion, use the **recurrence relation**: e.g., `T(n) = 2T(n/2) + O(n)` → solved via the Master Theorem → `O(n log n)`.

### Complexity Ladder (fastest → slowest)
```
O(1) < O(log n) < O(√n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
```

### Best, Average, Worst — Why All Three Matter
- **Best case**: the friendliest possible input (rarely relied upon in interviews).
- **Average case**: expected performance over random/typical inputs (used to justify algorithm choice, e.g. Quicksort).
- **Worst case**: the guarantee — what interviewers care about most, since production systems must survive worst-case inputs (e.g., an already-sorted array fed to Quicksort).

### Space Complexity
Same idea, but counting **extra memory used** (not counting the input itself) — auxiliary arrays, recursion call stack, hash maps, etc. A recursive function with depth `n` uses **O(n) space** on the call stack even if it does no extra array allocation.

### Interview Pattern
Whenever you propose a solution, **immediately state its time and space complexity** and mention whether it's optimal for the constraints given (e.g., `n ≤ 10⁵` hints you need better than O(n²)).

---

# Part 1 — Linear Data Structures

## 1. Arrays

### Intuitive Explanation
An array is like a **row of numbered mailboxes** on a street — every mailbox (element) has a fixed address (index), and because they're all the same size and laid out contiguously, you can jump directly to mailbox #47 without walking past 1–46. That direct jump is what makes indexing O(1).

### Step-by-Step: How It Works
1. Memory is allocated as one **contiguous block** big enough for all elements.
2. Accessing index `i` computes the memory address as `base_address + i * element_size` — pure arithmetic, no traversal needed.
3. Insertion/deletion in the **middle** requires shifting every subsequent element by one slot to keep the array contiguous.
4. Insertion/deletion at the **end** needs no shifting (unless the array is full and must be resized/copied — this is how dynamic arrays like Python's `list` or Java's `ArrayList` work).

### Complexity

| Operation | Best | Average | Worst | Why |
|---|---|---|---|---|
| Access by index | O(1) | O(1) | O(1) | direct address computation |
| Search (unsorted) | O(1) | O(n) | O(n) | may need to scan everything |
| Search (sorted, binary search) | O(1) | O(log n) | O(log n) | halving search space |
| Insert/Delete at end | O(1) | O(1) | O(n)* | *O(n) only when resize/copy triggers |
| Insert/Delete at middle/start | O(n) | O(n) | O(n) | shifting elements |

**Space:** O(n) to store n elements.

### Pseudocode
```
function insertAt(arr, index, value):
    for i from arr.length - 1 down to index:
        arr[i + 1] = arr[i]        // shift right
    arr[index] = value
    arr.length += 1

function deleteAt(arr, index):
    for i from index to arr.length - 2:
        arr[i] = arr[i + 1]        // shift left
    arr.length -= 1
```

### Interview Pattern Recognition
Reach for arrays when:
- You need **O(1) random access** by position.
- The problem mentions "contiguous subarray," "in-place," or gives a fixed-size input.
- You're about to build a **prefix sum**, **two-pointer**, or **sliding window** solution (all assume O(1) indexed access).

---

## 2. Strings

### Intuitive Explanation
A string is essentially an **array of characters** — but treated as immutable in many languages (Java, Python), meaning every "modification" secretly creates a new string. Think of it like a printed sentence: you can't edit a letter in place, you have to reprint the whole line.

### Step-by-Step Concepts
1. **Immutability**: `s = s + "a"` in Python/Java creates a brand-new string object — doing this in a loop is **O(n²)** total. Use a mutable buffer (`StringBuilder` in Java, list + `''.join()` in Python) for O(n) concatenation.
2. **Character sets**: ASCII (256 values) vs Unicode — affects hashing/frequency-count array sizes.
3. Common sub-problems: palindrome check, anagram check, substring search, pattern matching.

### Complexity

| Operation | Complexity | Note |
|---|---|---|
| Access char at index | O(1) | same as array |
| Concatenation (immutable lang) | O(n) per op | use builder/buffer instead |
| Substring search (naive) | O(n·m) | n = text length, m = pattern length |
| Substring search (KMP/Z-algorithm) | O(n + m) | preprocesses pattern to avoid re-scanning |

### Pseudocode: Naive Pattern Search
```
function naiveSearch(text, pattern):
    n = length(text), m = length(pattern)
    for i from 0 to n - m:
        j = 0
        while j < m and text[i+j] == pattern[j]:
            j += 1
        if j == m:
            return i    // match found at index i
    return -1
```

### Pseudocode: Sliding Window Anagram Check (frequency map)
```
function isAnagram(s1, s2):
    if length(s1) != length(s2): return false
    freq = array of 26 zeros
    for c in s1: freq[c - 'a'] += 1
    for c in s2: freq[c - 'a'] -= 1
    return all values in freq == 0
```

### Interview Pattern Recognition
- "Check if two strings are anagrams" → frequency array/hash map.
- "Longest substring without repeating characters" → sliding window + hash set.
- "Pattern occurs in text" → KMP/Rabin-Karp for large inputs, naive search for small ones.
- Palindrome checks → two pointers from both ends.

---

## 3. Linked Lists

### Intuitive Explanation
A linked list is like a **treasure hunt** — each clue (node) tells you the location of the next clue, but you can't jump straight to clue #10 without following clues 1 through 9 first. Unlike arrays (fixed mailboxes), nodes can live anywhere in memory; they're connected purely by pointers/references.

### Step-by-Step: How It Works
**Singly Linked List**: each node stores `data` + a pointer to the `next` node. The list itself just remembers the `head` (first node).

1. **Traversal**: start at `head`, follow `.next` until you hit `null`.
2. **Insertion at head**: create new node, point its `next` to the old head, update `head` to the new node — O(1), no shifting needed (this is the big advantage over arrays!).
3. **Insertion at tail**: traverse to the end (O(n)) unless you maintain a `tail` pointer (O(1)).
4. **Deletion**: find the node *before* the target, re-point its `next` to skip over the target node.

**Doubly Linked List**: each node also has a `prev` pointer — allows O(1) backward traversal and O(1) deletion if you already hold a reference to the node (no need to find "the node before it").

**Circular Linked List**: the last node points back to the head instead of `null` — useful for round-robin scheduling, circular buffers.

### Complexity

| Operation | Singly LL | Doubly LL |
|---|---|---|
| Access by index | O(n) | O(n) |
| Insert/Delete at head | O(1) | O(1) |
| Insert/Delete at tail (no tail ptr) | O(n) | O(n) |
| Insert/Delete at tail (with tail ptr) | O(1) insert / O(n) delete** | O(1) |
| Search | O(n) | O(n) |

**Space:** O(n), plus O(1) extra pointer per node (2 pointers for doubly).
**Deletion at tail in singly LL is O(n) even with a tail pointer, because you must find the second-to-last node to update its `next` to null.*

### Pseudocode
```
class Node:
    data
    next

function insertAtHead(head, value):
    newNode = Node(value)
    newNode.next = head
    return newNode              // new head

function reverseList(head):
    prev = null
    curr = head
    while curr != null:
        nextTemp = curr.next
        curr.next = prev
        prev = curr
        curr = nextTemp
    return prev                 // new head

function detectCycle(head):     // Floyd's Tortoise and Hare
    slow = head
    fast = head
    while fast != null and fast.next != null:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return true         // cycle detected
    return false
```

### Interview Pattern Recognition
- "Reverse a linked list" → iterative 3-pointer technique (prev/curr/next) shown above.
- "Detect a cycle" / "find the middle of a list" → **slow & fast pointers** (Floyd's algorithm).
- "Merge two sorted lists" → two-pointer merge, same idea as merge step in Merge Sort.
- Whenever a problem says **"in O(1) extra space"** for a linked list — that's your cue that pointer manipulation (not auxiliary arrays) is expected.

---

## 4. Stacks

### Intuitive Explanation
A stack is a pile of plates: you can only add a plate to the **top** or remove the **top** plate — you can't grab one from the middle without first removing everything above it. This is **LIFO**: Last In, First Out.

### Step-by-Step: How It Works
- **push(x)**: place `x` on top.
- **pop()**: remove and return the top element.
- **peek()/top()**: look at the top element without removing it.
- Implemented using either a **dynamic array** (append/remove from the end — O(1) amortized) or a **linked list** (insert/delete at head — O(1) always).

### Complexity

| Operation | Time | Space |
|---|---|---|
| push | O(1) | O(1) per element |
| pop | O(1) | — |
| peek | O(1) | — |
| search | O(n) | — |

### Pseudocode
```
class Stack:
    array = []
    function push(x): array.append(x)
    function pop(): return array.removeLast()
    function peek(): return array[last]
    function isEmpty(): return array.length == 0
```

### Classic Stack Problem: Balanced Parentheses
```
function isValid(s):
    stack = []
    pairs = { ')':'(', ']':'[', '}':'{' }
    for char in s:
        if char in "([{":
            stack.push(char)
        else:
            if stack.isEmpty() or stack.pop() != pairs[char]:
                return false
    return stack.isEmpty()
```

### Interview Pattern Recognition
Reach for a stack when the problem involves:
- **Matching/nesting** (parentheses, HTML tags, nested expressions).
- **"Undo" operations** or reversing order of processing.
- **Next Greater/Smaller Element** problems (monotonic stack — see below).
- **Expression evaluation** (infix→postfix conversion, calculator problems).
- Recursion simulation (any recursive algorithm can be rewritten iteratively using an explicit stack).

**Monotonic Stack pattern** (for "next greater element" style problems): keep the stack's elements in increasing or decreasing order by popping elements that violate the order before pushing the new one — this gives O(n) instead of the naive O(n²).
```
function nextGreaterElement(arr):
    result = array of -1, size n
    stack = []                      // stores indices
    for i from 0 to n-1:
        while stack not empty and arr[stack.top()] < arr[i]:
            idx = stack.pop()
            result[idx] = arr[i]
        stack.push(i)
    return result
```

---

## 5. Queues (incl. Deque & Priority Queue)

### Intuitive Explanation
A queue is a **line at a ticket counter**: the first person to join is the first person served. This is **FIFO**: First In, First Out — the opposite discipline from a stack.

### Step-by-Step: How It Works
- **enqueue(x)**: add `x` to the **back** of the line.
- **dequeue()**: remove and return the element at the **front**.
- A naive array implementation wastes space (front keeps advancing) — solved with a **circular queue** (wrap the front/rear pointers using modulo arithmetic) or by using a linked list with head & tail pointers.

### Variants
1. **Circular Queue**: fixed-size array where `rear = (rear + 1) % capacity` — reuses freed slots at the front.
2. **Deque (Double-Ended Queue)**: insertion/removal allowed at **both** ends in O(1) — the Swiss-army knife used in sliding window maximum problems.
3. **Priority Queue**: not FIFO at all — each element has a priority, and `dequeue` always returns the highest (or lowest) priority element. Typically implemented with a **Heap** (see Section 11).

### Complexity

| Operation | Simple Queue | Circular Queue | Deque | Priority Queue (heap-based) |
|---|---|---|---|---|
| enqueue/insert | O(1) | O(1) | O(1) both ends | O(log n) |
| dequeue/remove | O(1)* | O(1) | O(1) both ends | O(log n) |
| peek | O(1) | O(1) | O(1) | O(1) |

**Naive array-based dequeue is O(n) due to shifting; use a linked-list or circular-array queue to get true O(1).*

### Pseudocode: Circular Queue
```
class CircularQueue:
    array = size k
    front = 0, rear = -1, count = 0

    function enqueue(x):
        if count == k: return "full"
        rear = (rear + 1) % k
        array[rear] = x
        count += 1

    function dequeue():
        if count == 0: return "empty"
        val = array[front]
        front = (front + 1) % k
        count -= 1
        return val
```

### Pseudocode: Sliding Window Maximum (Monotonic Deque)
```
function maxSlidingWindow(arr, k):
    deque = []          // stores indices, values decreasing
    result = []
    for i from 0 to n-1:
        while deque not empty and arr[deque.back()] < arr[i]:
            deque.popBack()
        deque.pushBack(i)
        if deque.front() <= i - k:
            deque.popFront()          // out of window
        if i >= k - 1:
            result.append(arr[deque.front()])
    return result
```

### Interview Pattern Recognition
- **BFS traversal** (trees/graphs) → simple queue (level-by-level processing).
- **"Sliding window maximum/minimum"** → monotonic deque.
- **"Kth largest element," "Top K frequent," "Merge K sorted lists"** → priority queue / heap.
- **Task scheduling with priorities** → priority queue.

---

# Part 2 — Algorithmic Thinking

## 6. Recursion & Backtracking

### Intuitive Explanation
Recursion is like **Russian nesting dolls (Matryoshka)**: to open the biggest doll, you open a slightly smaller identical doll inside it, and so on, until you reach the smallest doll that opens directly (the **base case**). Each doll "trusts" that the smaller doll inside knows how to open itself — this trust is the essence of recursive thinking.

Backtracking is recursion with **exploration and undo**: imagine solving a maze by walking a path, and every time you hit a dead end, you walk back to the last junction and try a different direction — you "backtrack."

### Step-by-Step: How Recursion Works
1. Identify the **base case** — the smallest input the function can answer directly without recursing.
2. Identify the **recursive case** — how to break the problem into a smaller version of itself.
3. Trust the recursive call to correctly solve the smaller subproblem (don't try to mentally unfold the entire call chain).
4. Each call adds a **stack frame**; when the base case is hit, frames "pop" back up, combining results.

### Step-by-Step: How Backtracking Works
1. Make a choice (place a queen, pick a number, choose a path).
2. Recurse into the consequences of that choice.
3. If it leads to a dead end (constraint violated) or you've explored it, **undo the choice** (this is the "backtrack" step) and try the next option.
4. Prune early whenever possible — check constraints *before* recursing further to avoid wasted work.

### Complexity
General recursion complexity is problem-specific — use the **recurrence relation**:
- Linear recursion (e.g., factorial): `T(n) = T(n-1) + O(1)` → **O(n)** time, **O(n)** space (call stack depth).
- Binary recursion (e.g., naive Fibonacci): `T(n) = T(n-1) + T(n-2) + O(1)` → **O(2ⁿ)** time — exponential blow-up from recomputing the same subproblems (fixed by memoization → Dynamic Programming, Part 4).
- Backtracking (e.g., N-Queens, permutations): typically **O(k^n)** or **O(n!)** worst case, since it explores a branching tree of choices — pruning reduces the practical runtime drastically.

### Pseudocode: Factorial (simple recursion)
```
function factorial(n):
    if n == 0: return 1              // base case
    return n * factorial(n - 1)      // recursive case
```

### Pseudocode: Generate All Subsets (Backtracking template)
```
function subsets(nums):
    result = []
    function backtrack(start, current):
        result.append(copy of current)
        for i from start to nums.length - 1:
            current.append(nums[i])           // choose
            backtrack(i + 1, current)          // explore
            current.removeLast()               // un-choose (backtrack)
    backtrack(0, [])
    return result
```

### Pseudocode: N-Queens (classic backtracking with pruning)
```
function solveNQueens(n):
    board = n x n grid of '.'
    result = []
    function isSafe(row, col):
        // check column, and both diagonals above current row
        ...
    function place(row):
        if row == n:
            result.append(copy of board)
            return
        for col from 0 to n-1:
            if isSafe(row, col):
                board[row][col] = 'Q'          // choose
                place(row + 1)                  // explore
                board[row][col] = '.'          // un-choose
    place(0)
    return result
```

### Interview Pattern Recognition
- **"Generate all possible ___"** (subsets, permutations, combinations, valid parentheses) → backtracking.
- **"Solve puzzle with constraints"** (N-Queens, Sudoku) → backtracking with pruning.
- A recursive solution that recomputes overlapping subproblems → red flag to **add memoization** (bridge into Dynamic Programming).
- Tree/graph problems phrased as "explore all paths" → recursive DFS is the natural fit.

---

## 7. Searching Algorithms

### 7.1 Linear Search

**Intuitive Explanation**: Checking every house on a street one by one until you find the address you want.

**Step-by-Step**: Start at index 0, compare each element to the target, return the index on a match, else continue to the end.

**Complexity**: Best O(1) (found immediately), Average O(n), Worst O(n). Space O(1).

```
function linearSearch(arr, target):
    for i from 0 to arr.length - 1:
        if arr[i] == target: return i
    return -1
```

**Pattern**: Use when data is unsorted or small; otherwise a smarter search is expected.

### 7.2 Binary Search

**Intuitive Explanation**: Looking up a word in a physical dictionary — you don't start from page 1; you open to the middle, decide if your word is alphabetically before or after, and repeat on the correct half. Requires **sorted** data.

**Step-by-Step**
1. Set `low = 0`, `high = n - 1`.
2. Compute `mid = low + (high - low) / 2` (avoids overflow vs. `(low+high)/2`).
3. If `arr[mid] == target`, done.
4. If `target < arr[mid]`, search left half (`high = mid - 1`).
5. Else search right half (`low = mid + 1`).
6. Repeat until `low > high` (not found).

**Complexity**: Best O(1), Average/Worst O(log n). Space O(1) iterative / O(log n) recursive (call stack).

```
function binarySearch(arr, target):
    low = 0, high = arr.length - 1
    while low <= high:
        mid = low + (high - low) / 2
        if arr[mid] == target: return mid
        else if arr[mid] < target: low = mid + 1
        else: high = mid - 1
    return -1
```

**Interview Pattern Recognition**: 
- Sorted array/rotated sorted array problems.
- **"Find minimum/first/last occurrence satisfying a condition"** → binary search on the **answer space** (not just the array) — a huge category: "minimum capacity to ship packages," "Koko eating bananas," "square root of a number." The tell-tale sign is a monotonic yes/no condition as a function increases (`if x works, does x+1 also work?`).
- Any mention of "O(log n)" in problem constraints/expected complexity.

---

## 8. Sorting Algorithms

### Overview Table (memorize this!)

| Algorithm | Best | Average | Worst | Space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | No |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No | Yes |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Yes |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes | No |
| Radix Sort | O(d·(n+k)) | O(d·(n+k)) | O(d·(n+k)) | O(n+k) | Yes | No |

*(Stable = equal elements keep their relative order; In-place = O(1) extra space, ignoring recursion stack.)*

### 8.1 Bubble Sort
**Analogy**: Bubbles rising in soda water — the largest "bubble" (element) rises to the top with each pass.
**Step-by-Step**: Repeatedly compare adjacent elements, swap if out of order; after each full pass, the largest unsorted element "bubbles" to its correct position at the end.
```
function bubbleSort(arr):
    for i from 0 to n-1:
        for j from 0 to n-i-2:
            if arr[j] > arr[j+1]:
                swap(arr[j], arr[j+1])
```
**Pattern**: Rarely used in practice/interviews except as a baseline or "explain a simple sort" warm-up question.

### 8.2 Insertion Sort
**Analogy**: Sorting a hand of playing cards — you pick up cards one at a time and insert each into its correct position among the cards already sorted in your hand.
```
function insertionSort(arr):
    for i from 1 to n-1:
        key = arr[i]
        j = i - 1
        while j >= 0 and arr[j] > key:
            arr[j+1] = arr[j]
            j -= 1
        arr[j+1] = key
```
**Pattern**: Efficient for small or nearly-sorted arrays; used as the base case inside hybrid sorts (e.g., Timsort switches to insertion sort for small subarrays).

### 8.3 Merge Sort (Divide and Conquer)
**Analogy**: Splitting a deck of cards in half repeatedly until you have single cards (trivially sorted), then merging pairs back together in sorted order — like two people combining two already-sorted piles of papers into one sorted pile.
**Step-by-Step**
1. Divide the array into two halves recursively until each subarray has 1 element.
2. Conquer: merge two sorted halves into one sorted array by comparing front elements of each half.
3. Combine merges bubble back up until the whole array is merged.
```
function mergeSort(arr):
    if arr.length <= 1: return arr
    mid = arr.length / 2
    left = mergeSort(arr[0..mid])
    right = mergeSort(arr[mid..end])
    return merge(left, right)

function merge(left, right):
    result = []
    i = 0, j = 0
    while i < left.length and j < right.length:
        if left[i] <= right[j]: result.append(left[i]); i += 1
        else: result.append(right[j]); j += 1
    append remaining elements of left and right
    return result
```
**Recurrence**: `T(n) = 2T(n/2) + O(n)` → **O(n log n)** by the Master Theorem.
**Pattern**: Guaranteed O(n log n) even in worst case, stable — go-to when stability matters or worst-case guarantees are required (e.g., counting inversions problem reuses the merge step).

### 8.4 Quick Sort (Divide and Conquer)
**Analogy**: Choosing one person (the **pivot**) in a room and asking everyone shorter to stand on the left, taller on the right — then repeating the process independently within each group.
**Step-by-Step**
1. Choose a pivot (first, last, random, or median-of-three element).
2. **Partition**: rearrange the array so elements `< pivot` are left of it and `> pivot` are right of it; pivot lands in its final sorted position.
3. Recursively quicksort the left and right partitions.
```
function quickSort(arr, low, high):
    if low < high:
        pivotIndex = partition(arr, low, high)
        quickSort(arr, low, pivotIndex - 1)
        quickSort(arr, pivotIndex + 1, high)

function partition(arr, low, high):
    pivot = arr[high]
    i = low - 1
    for j from low to high - 1:
        if arr[j] < pivot:
            i += 1
            swap(arr[i], arr[j])
    swap(arr[i+1], arr[high])
    return i + 1
```
**Why worst case is O(n²)**: if the pivot is always the smallest/largest element (e.g., already-sorted array with last-element pivot), each partition only removes one element, giving `n + (n-1) + ... + 1 = O(n²)`. **Randomized pivot selection** avoids this in practice.
**Pattern**: Fastest in-practice general sort (good cache locality, in-place); used when average-case performance matters more than worst-case guarantees. Also the basis for the **Quickselect** algorithm (finding the Kth smallest element in average O(n)).

### 8.5 Heap Sort
See Section 11.4 (Heaps) for the underlying structure. 
**Step-by-Step**: Build a max-heap from the array (O(n)), then repeatedly swap the root (max element) with the last element, shrink the heap by one, and re-heapify — this places elements in sorted order from the back.
```
function heapSort(arr):
    buildMaxHeap(arr)
    for i from n-1 down to 1:
        swap(arr[0], arr[i])
        heapify(arr, 0, i)      // restore heap property on reduced heap
```
**Pattern**: Guaranteed O(n log n) with O(1) space (unlike merge sort) — useful when memory is constrained, though not stable and has worse cache performance than quicksort.

### 8.6 Counting Sort & Radix Sort (non-comparison sorts)
**Analogy for Counting Sort**: Sorting exam scores (0-100) by making 101 buckets, counting how many students got each score, then reading buckets in order — no comparisons needed at all.
```
function countingSort(arr, k):     // k = max value + 1
    count = array of k zeros
    for x in arr: count[x] += 1
    output = []
    for i from 0 to k-1:
        repeat count[i] times: output.append(i)
    return output
```
**Radix Sort** extends this by sorting numbers digit-by-digit (least significant digit first), using counting sort as the stable subroutine for each digit.
**Pattern**: Use when the value range `k` is small/bounded relative to `n` (comparison sorts' O(n log n) lower bound doesn't apply here since we're not comparing elements pairwise).

### Interview Pattern Recognition (Sorting overall)
- If asked to **implement** a sort from scratch: Merge Sort or Quick Sort are the most commonly expected.
- If the problem says **"find the Kth largest/smallest"**: think heap or Quickselect, not a full sort.
- If the problem involves a **small, bounded range of values**: counting sort beats comparison sorts.
- "Sort almost-sorted data" → insertion sort shines (close to O(n)).

---

## 9. Two Pointers & Sliding Window

### Intuitive Explanation
**Two Pointers**: Imagine two people walking towards each other from opposite ends of a sorted line of people, trying to find a pair whose heights sum to a target — instead of checking every pair (O(n²)), they can intelligently move inward, checking O(n) pairs total.

**Sliding Window**: Imagine looking through a physical window frame that slides along a wall of paintings — instead of re-observing every painting from scratch as the frame moves, you just note what enters and what leaves the frame.

### Step-by-Step: Two Pointers (opposite ends)
1. Sort the array (if not already sorted).
2. Place `left = 0`, `right = n - 1`.
3. Evaluate the pair; if the sum is too small, move `left` right (increase sum); if too big, move `right` left (decrease sum).
4. Stop when `left >= right`.

```
function twoSum_sorted(arr, target):
    left = 0, right = arr.length - 1
    while left < right:
        sum = arr[left] + arr[right]
        if sum == target: return [left, right]
        else if sum < target: left += 1
        else: right -= 1
    return [-1, -1]
```

### Step-by-Step: Sliding Window (variable size)
1. Expand the window by moving `right` forward, adding the new element to your running state (sum/count/frequency map).
2. While the window violates a constraint (e.g., sum too big, duplicate found), shrink from the left by moving `left` forward and removing that element from your state.
3. Track the best answer (max/min window length, etc.) at each valid state.

```
function longestSubstringWithoutRepeats(s):
    seen = empty set
    left = 0
    maxLen = 0
    for right from 0 to s.length - 1:
        while s[right] in seen:
            seen.remove(s[left])
            left += 1
        seen.add(s[right])
        maxLen = max(maxLen, right - left + 1)
    return maxLen
```

### Complexity
Both patterns typically turn an O(n²) brute-force into **O(n)**, since each pointer traverses the array at most once (amortized). Space is usually O(1) to O(k) (for a frequency map with bounded alphabet).

### Interview Pattern Recognition
- **Sorted array + "find pair/triplet with sum X"** → two pointers.
- **"Longest/shortest substring/subarray satisfying condition"** → sliding window.
- **"Container with most water," "trapping rain water"** → two pointers from both ends.
- Keywords like **contiguous subarray**, **substring**, **window of size k** are the biggest tells for sliding window.

---

# Part 3 — Non-Linear Data Structures

## 10. Hashing

### Intuitive Explanation
A hash table is like a **coat-check counter with a numbering formula**: instead of searching every hook for your coat, the attendant runs your name through a formula (hash function) that instantly tells them which hook number to check — turning a search into direct lookup.

### Step-by-Step: How It Works
1. A **hash function** converts a key (string, number, object) into an integer (the hash code).
2. That integer is reduced (via modulo) to fit within the table's array size — this becomes the **bucket index**.
3. The key-value pair is stored at that bucket.
4. **Collisions** (two keys mapping to the same bucket) are handled via:
   - **Chaining**: each bucket holds a linked list (or small tree) of all entries that hash there.
   - **Open addressing**: on collision, probe for the next open slot (linear probing, quadratic probing, double hashing).
5. **Resizing**: when the table gets too full (load factor exceeds a threshold, typically 0.7), it's resized (usually doubled) and every entry is rehashed — an O(n) one-time cost, amortized across many O(1) insertions.

### Complexity

| Operation | Average | Worst |
|---|---|---|
| Insert | O(1) | O(n) — all keys collide |
| Search | O(1) | O(n) |
| Delete | O(1) | O(n) |

Worst case happens with a poor hash function or adversarial input causing all keys to collide into one bucket — a well-designed hash function makes this vanishingly rare in practice.
**Space:** O(n).

### Pseudocode: Simple Hash Map with Chaining
```
class HashMap:
    buckets = array of k empty lists

    function hash(key): return key % k

    function put(key, value):
        idx = hash(key)
        for pair in buckets[idx]:
            if pair.key == key: pair.value = value; return
        buckets[idx].append((key, value))

    function get(key):
        idx = hash(key)
        for pair in buckets[idx]:
            if pair.key == key: return pair.value
        return null
```

### Interview Pattern Recognition
Hashing is arguably the **single most common tool** in interviews. Reach for it when you see:
- **"Have you seen this before?"** → use a hash set to track visited/seen elements (e.g., "contains duplicate," "first non-repeating character").
- **"Count frequency of ___"** → hash map from element → count.
- **"Two Sum"**-style problems ("find pair that sums to target," unsorted array) → hash map storing `value → index`, achieving O(n) instead of O(n²).
- Anytime you'd otherwise need a **nested loop just to check membership** — a hash set turns that inner loop into O(1).
- **Grouping** problems ("group anagrams") → hash map from a canonical key (e.g., sorted string) → list of items.

---

## 11. Trees

### 11.1 Binary Trees — Core Concepts

**Intuitive Explanation**: A family tree, but capped at two children per parent. Each node has at most a **left child** and a **right child**. The topmost node is the **root**; nodes with no children are **leaves**.

**Key Terminology**
- **Height** of a node: longest path down to a leaf.
- **Depth** of a node: distance from the root.
- **Balanced tree**: height is O(log n) — left and right subtrees differ in height by a small bounded amount at every node.
- **Complete tree**: every level is fully filled except possibly the last, which fills left-to-right (this is what heaps require).

### Step-by-Step: Tree Traversals
Traversal = the order in which you "visit" nodes. Given node `root` with `left`/`right` children:

- **Preorder** (Root → Left → Right): used to *copy*/*serialize* a tree (you see the root before its children, so you can rebuild top-down).
- **Inorder** (Left → Root → Right): for a **BST**, this visits nodes in **sorted order** — extremely important fact.
- **Postorder** (Left → Right → Root): used when children must be processed before the parent (e.g., deleting a tree, computing subtree sizes/heights).
- **Level-order** (BFS, level by level): uses a queue, not recursion.

```
function preorder(node):
    if node == null: return
    visit(node)
    preorder(node.left)
    preorder(node.right)

function inorder(node):
    if node == null: return
    inorder(node.left)
    visit(node)
    inorder(node.right)

function postorder(node):
    if node == null: return
    postorder(node.left)
    postorder(node.right)
    visit(node)

function levelOrder(root):
    queue = [root]
    while queue not empty:
        node = queue.dequeue()
        visit(node)
        if node.left: queue.enqueue(node.left)
        if node.right: queue.enqueue(node.right)
```

### Complexity (general binary tree)

| Operation | Balanced Tree | Skewed (worst) Tree |
|---|---|---|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Traversal (any order) | O(n) always | O(n) always |

**Space:** O(n) for the structure; O(h) for recursion call stack, where h = height (O(log n) balanced, O(n) skewed).

### 11.2 Binary Search Tree (BST)

**Intuitive Explanation**: A BST is a binary tree with a **rule**: for every node, everything in its left subtree is smaller, and everything in its right subtree is larger. This rule is what makes searching feel like binary search "unrolled" into tree form.

**Step-by-Step: Search**
1. Compare target with `root.value`.
2. If equal, found. If smaller, recurse left. If larger, recurse right.
3. Reaching `null` means not found.

**Step-by-Step: Insertion** — same logic as search, but insert the new node at the `null` spot you land on.

**Step-by-Step: Deletion** (3 cases)
1. **Leaf node**: simply remove it.
2. **One child**: replace the node with its single child.
3. **Two children**: find the **in-order successor** (smallest value in the right subtree — i.e., leftmost node of the right subtree), copy its value into the node being deleted, then recursively delete that successor (which now has at most one child, reducing to case 1 or 2).

```
function insert(node, value):
    if node == null: return new Node(value)
    if value < node.value: node.left = insert(node.left, value)
    else: node.right = insert(node.right, value)
    return node

function search(node, target):
    if node == null: return false
    if node.value == target: return true
    if target < node.value: return search(node.left, target)
    return search(node.right, target)
```

**Complexity**: O(log n) average (balanced), **O(n) worst case** — happens when you insert already-sorted data, degenerating the BST into a linked list. This is exactly why **self-balancing trees** exist.

### 11.3 Self-Balancing Trees (AVL / Red-Black) — Conceptual Overview

**Intuitive Explanation**: An AVL tree is a BST with a strict rule — after every insertion/deletion, if any node's left and right subtree heights differ by more than 1, the tree performs a **rotation** to rebalance itself, like a gymnast automatically adjusting their arms to avoid toppling over.

**Step-by-Step**
1. Insert/delete as in a normal BST.
2. Walk back up from the inserted/deleted node, checking the **balance factor** (`height(left) - height(right)`) at each ancestor.
3. If any node's balance factor becomes ±2, perform a rotation (Left-Left → single right rotation, Right-Right → single left rotation, Left-Right / Right-Left → double rotation) to restore balance.

**You won't be asked to code full AVL rotations from scratch in most interviews**, but you must know:
- **Why** they exist (to guarantee O(log n) worst case, avoiding BST's degenerate case).
- Red-Black trees make a similar guarantee with looser balancing (faster insert/delete, slightly slower lookup) — this is what powers `TreeMap`/`std::map` internally in many languages.

**Complexity**: O(log n) guaranteed for search/insert/delete — **always**, not just on average.

### 11.4 Heaps (Binary Heap) & Priority Queues

**Intuitive Explanation**: A heap is like a **hospital emergency room queue** — not first-come-first-served, but always-treat-the-most-critical-patient-first. A **Max-Heap** always keeps the largest element accessible at the root; a **Min-Heap** keeps the smallest.

**Step-by-Step: Structure**
- A heap is a **complete binary tree** (fills left to right) usually stored as a plain **array** (no pointers needed!): for a node at index `i`, its children are at `2i+1` and `2i+2`, and its parent is at `(i-1)/2`.
- **Heap property**: every parent is ≥ (max-heap) or ≤ (min-heap) both its children — note this does *not* imply sorted order left-to-right, only the parent-child relationship.

**Step-by-Step: Insert**
1. Add the new element at the next open leaf position (end of the array).
2. **Bubble up ("sift up")**: compare with its parent, swap if it violates the heap property, repeat until it doesn't (or reaches the root).

**Step-by-Step: Extract Max/Min (the root)**
1. Save the root value to return.
2. Move the *last* element in the array to the root position.
3. **Bubble down ("sift down"/"heapify")**: compare with children, swap with the larger (max-heap) child if it violates heap property, repeat until settled.

```
function siftUp(heap, i):
    while i > 0 and heap[parent(i)] < heap[i]:
        swap(heap[parent(i)], heap[i])
        i = parent(i)

function siftDown(heap, i, size):
    while true:
        largest = i
        l = 2*i+1, r = 2*i+2
        if l < size and heap[l] > heap[largest]: largest = l
        if r < size and heap[r] > heap[largest]: largest = r
        if largest == i: break
        swap(heap[i], heap[largest])
        i = largest

function extractMax(heap):
    max = heap[0]
    heap[0] = heap.removeLast()
    siftDown(heap, 0, heap.length)
    return max
```

**Complexity**

| Operation | Time |
|---|---|
| Insert | O(log n) |
| Extract max/min | O(log n) |
| Peek max/min | O(1) |
| Build heap from n elements | O(n) *(not O(n log n) — a classic interview trap!)* |

**Space:** O(n).

### Interview Pattern Recognition (Trees & Heaps)
- **"Sorted order"** requirement from a tree → inorder traversal of a BST.
- **"Kth largest/smallest element," "Top K frequent," "median of a stream"** → heap (often *two* heaps — a max-heap for the lower half and min-heap for the upper half — for running-median problems).
- **"Validate BST," "lowest common ancestor," "diameter of tree"** → recursive DFS, often carrying extra info back up (height, boolean validity) via the return value.
- **"Merge K sorted lists/arrays"** → min-heap of size K.
- Any "scheduling by priority" system → priority queue.

---

## 12. Tries (Prefix Trees)

### Intuitive Explanation
A trie is like the **autocomplete on your phone keyboard**: it stores words letter by letter along shared paths, so words with the same prefix (e.g., "CAT," "CAR," "CARD") share the same initial branches. Walking down the tree one character at a time either confirms a word exists or tells you it doesn't, in time proportional to the *word's length*, not the *number of words stored*.

### Step-by-Step: How It Works
1. Each node represents one character position and holds a map/array of children (one per possible next letter) plus a flag `isEndOfWord`.
2. **Insertion**: walk from the root, one character at a time; create a child node if the path doesn't already exist; mark `isEndOfWord = true` at the last character.
3. **Search**: walk the same way; if you fall off the path (a needed child doesn't exist), the word isn't present. If you reach the end, check `isEndOfWord`.
4. **Prefix search** (used for autocomplete): same walk, but you don't need `isEndOfWord` to be true at the end — just that the path exists.

### Complexity

| Operation | Time | Space |
|---|---|---|
| Insert word of length L | O(L) | O(L) per new word (shared prefixes cost nothing extra) |
| Search word of length L | O(L) | — |
| Prefix search of length L | O(L) | — |

Much faster than a hash-set-of-words approach for **prefix-based** queries, since a hash set would need to check every stored word (O(n·L)) to find all words with a given prefix.

### Pseudocode
```
class TrieNode:
    children = map<char, TrieNode>
    isEndOfWord = false

class Trie:
    root = new TrieNode()

    function insert(word):
        node = root
        for char in word:
            if char not in node.children:
                node.children[char] = new TrieNode()
            node = node.children[char]
        node.isEndOfWord = true

    function search(word):
        node = root
        for char in word:
            if char not in node.children: return false
            node = node.children[char]
        return node.isEndOfWord

    function startsWith(prefix):
        node = root
        for char in prefix:
            if char not in node.children: return false
            node = node.children[char]
        return true
```

### Interview Pattern Recognition
- **Autocomplete, spell-checker, "word search with prefixes"** → Trie.
- **"Longest common prefix among strings"** → Trie (or simpler string comparison for small inputs).
- Problems combining **Trie + DFS/Backtracking** (e.g., "Word Search II" on a grid) — build a Trie of the dictionary first, then DFS the grid while walking the Trie in lockstep, pruning paths that don't exist in the Trie.

---

## 13. Graphs

### Intuitive Explanation
A graph is a **social network**: people are **vertices/nodes**, friendships are **edges**. Unlike trees, graphs have no fixed "root" or hierarchy, can have cycles (mutual friend loops), and edges can be one-directional (Twitter "follows" — a **directed** graph) or bidirectional (Facebook "friends" — an **undirected** graph). Edges can also carry a **weight** (e.g., distance between cities, cost of a flight).

### Representations

**Adjacency List** (most common in interviews): a map/array where each vertex stores a list of its neighbors.
```
graph = {
    A: [B, C],
    B: [A, D],
    C: [A],
    D: [B]
}
```
- Space: O(V + E) — efficient for sparse graphs.

**Adjacency Matrix**: a V×V grid where `matrix[i][j] = 1` (or weight) if an edge exists between `i` and `j`.
- Space: O(V²) — wasteful for sparse graphs, but O(1) edge-existence lookup, useful for dense graphs.

### 13.1 Breadth-First Search (BFS)

**Intuitive Explanation**: Ripples spreading outward from a stone dropped in water — you visit all neighbors at distance 1, then all at distance 2, and so on. This guarantees BFS finds the **shortest path in terms of number of edges** (unweighted graphs).

**Step-by-Step**
1. Start at the source node; mark it visited; push to a queue.
2. While the queue isn't empty: dequeue a node, visit it, and enqueue all its unvisited neighbors (marking them visited *at enqueue time*, not dequeue time — a common bug source if done wrong).

```
function bfs(graph, start):
    visited = set([start])
    queue = [start]
    order = []
    while queue not empty:
        node = queue.dequeue()
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.enqueue(neighbor)
    return order
```
**Complexity**: O(V + E) time (every vertex and edge is processed once), O(V) space (visited set + queue).

### 13.2 Depth-First Search (DFS)

**Intuitive Explanation**: Exploring a maze by always taking a new path as far as possible before backtracking — the opposite exploration order of BFS's "ripple."

**Step-by-Step**
1. Visit the start node, mark visited.
2. Recurse (or use an explicit stack) into an unvisited neighbor.
3. When no unvisited neighbors remain, backtrack to the previous node and try its other neighbors.

```
function dfs(graph, node, visited = set()):
    visited.add(node)
    visit(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
```
**Complexity**: O(V + E) time, O(V) space (recursion stack + visited set).

**BFS vs DFS — when to use which**

| Use BFS when... | Use DFS when... |
|---|---|
| You need the shortest path in an unweighted graph | You need to explore all paths/possibilities (backtracking) |
| Level-order processing matters | Detecting cycles, topological sort |
| "Minimum number of steps/moves" | Connected components, path existence |

### 13.3 Topological Sort (Directed Acyclic Graphs only)

**Intuitive Explanation**: Deciding the order to put on clothes — you must put on socks before shoes, underwear before pants. Topological sort produces a valid linear ordering of tasks respecting all "must come before" dependencies.

**Step-by-Step (Kahn's Algorithm — BFS-based)**
1. Compute the **in-degree** (number of incoming edges) of every node.
2. Add all nodes with in-degree 0 to a queue (no prerequisites — can be done first).
3. Dequeue a node, add it to the result, and decrement the in-degree of all its neighbors.
4. If a neighbor's in-degree drops to 0, enqueue it.
5. If the result doesn't include all nodes at the end, the graph has a **cycle** (no valid ordering exists).

```
function topologicalSort(graph, V):
    inDegree = array of V zeros
    for node in graph: for neighbor in graph[node]: inDegree[neighbor] += 1
    queue = [nodes with inDegree == 0]
    result = []
    while queue not empty:
        node = queue.dequeue()
        result.append(node)
        for neighbor in graph[node]:
            inDegree[neighbor] -= 1
            if inDegree[neighbor] == 0: queue.enqueue(neighbor)
    if result.length != V: return "cycle detected — no valid ordering"
    return result
```
**Complexity**: O(V + E).
**Pattern**: "Course schedule / prerequisites," "build order," any "task dependency ordering" problem.

### 13.4 Shortest Path Algorithms

**Dijkstra's Algorithm** (single-source shortest path, non-negative weights)

**Intuitive Explanation**: Like water flooding outward from a source, but flowing faster through "wider pipes" (cheaper edges) — you always expand from the currently-cheapest-reachable node next, using a priority queue instead of a plain queue.

**Step-by-Step**
1. Set distance to source = 0, all others = infinity.
2. Use a min-heap keyed by current known distance; start by pushing the source.
3. Pop the node with the smallest distance; for each neighbor, check if going through this node offers a shorter path ("**relaxation**": `if dist[node] + weight < dist[neighbor]: update and push`).
4. Repeat until the heap is empty.

```
function dijkstra(graph, source):
    dist = map of all nodes to infinity
    dist[source] = 0
    minHeap = [(0, source)]
    while minHeap not empty:
        (d, node) = minHeap.popMin()
        if d > dist[node]: continue      // stale entry, skip
        for (neighbor, weight) in graph[node]:
            newDist = dist[node] + weight
            if newDist < dist[neighbor]:
                dist[neighbor] = newDist
                minHeap.push((newDist, neighbor))
    return dist
```
**Complexity**: O((V + E) log V) with a binary heap. **Fails on negative weights** — a relaxed node might later need to be relaxed again, but Dijkstra never revisits a finalized node.

**Bellman-Ford Algorithm** (handles negative weights, detects negative cycles)
- Relax **all** edges, **V - 1 times** (guaranteed to find shortest paths after that many rounds, since a shortest path has at most V-1 edges).
- A **V-th round** that still finds an improvement means a **negative-weight cycle** exists.
- **Complexity**: O(V · E) — slower than Dijkstra, but more general.

**Floyd-Warshall Algorithm** (all-pairs shortest path)
- Dynamic-programming style: `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])` for every intermediate node `k`.
- **Complexity**: O(V³) — used when you need shortest paths **between every pair** of nodes, and V is small.

### 13.5 Minimum Spanning Tree (MST)

**Intuitive Explanation**: You're wiring electricity to every house in a neighborhood as cheaply as possible — you want every house connected (spanning), using the fewest total cable length (minimum), with no wasteful loops (tree = no cycles).

**Prim's Algorithm**: Grow the MST one vertex at a time — always add the cheapest edge that connects the growing tree to a new, not-yet-included vertex. (Conceptually similar to Dijkstra, but relax by edge weight alone, not cumulative distance.)

**Kruskal's Algorithm**: Sort **all edges** by weight; greedily add each edge to the MST if it doesn't form a cycle with edges already chosen (checked efficiently using **Union-Find**, below).

```
function kruskal(edges, V):
    sort edges by weight ascending
    uf = new UnionFind(V)
    mst = []
    for (u, v, weight) in edges:
        if uf.find(u) != uf.find(v):     // adding this edge won't form a cycle
            uf.union(u, v)
            mst.append((u, v, weight))
    return mst
```
**Complexity**: Kruskal's O(E log E) (dominated by the sort); Prim's O(E log V) with a binary heap.

### 13.6 Union-Find (Disjoint Set Union)

**Intuitive Explanation**: Think of it as tracking "which friend group is each person in" — you can quickly check if two people are in the same group, or merge two groups together, without ever re-scanning the whole group's membership list.

**Step-by-Step**
- Each element points to a "parent"; the **root** of a group is the element that points to itself.
- **find(x)**: follow parent pointers until reaching the root — with **path compression** (point every visited node directly to the root during the walk), this becomes nearly O(1) amortized.
- **union(x, y)**: find both roots; attach one root under the other — with **union by rank/size** (always attach the smaller tree under the bigger one's root) to keep trees shallow.

```
class UnionFind:
    parent = array where parent[i] = i initially
    rank = array of zeros

    function find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])   // path compression
        return parent[x]

    function union(x, y):
        rootX = find(x), rootY = find(y)
        if rootX == rootY: return
        if rank[rootX] < rank[rootY]: swap(rootX, rootY)
        parent[rootY] = rootX
        if rank[rootX] == rank[rootY]: rank[rootX] += 1
```
**Complexity**: O(α(n)) per operation amortized — α is the inverse Ackermann function, which is ≤ 5 for any practically-sized input, so this is **effectively O(1)**.

### Interview Pattern Recognition (Graphs)
- **"Number of islands," "connected components," "friend circles"** → DFS/BFS or Union-Find.
- **"Course schedule / can finish all tasks"** → topological sort / cycle detection in a directed graph.
- **"Shortest path" (unweighted)** → BFS. **(weighted, non-negative)** → Dijkstra. **(weighted, negative allowed)** → Bellman-Ford. **(all pairs)** → Floyd-Warshall.
- **"Cheapest way to connect all nodes/cities"** → MST (Prim's/Kruskal's).
- **"Detect a cycle"** → DFS with recursion-stack tracking (directed) or Union-Find (undirected).
- **"Redundant connection," "accounts merge," "most stones removed"** → Union-Find is the fast, elegant solution.

---

# Part 4 — Advanced Problem-Solving Paradigms

## 14. Dynamic Programming (DP)

### Intuitive Explanation
Imagine climbing a staircase and, at every step, writing down on a sticky note "the number of ways to reach this step" so that if you (or a friend) ever need that number again, you just read the sticky note instead of recalculating from scratch. DP is **recursion + memory** — it only pays off when a problem has **overlapping subproblems** (the same smaller problem is solved repeatedly) and **optimal substructure** (the optimal solution to the big problem is built from optimal solutions to its subproblems).

### Step-by-Step: How to Approach Any DP Problem
1. **Define the state**: what does `dp[i]` (or `dp[i][j]`) represent in plain English? This is the most important step — get this wrong and nothing else works.
2. **Find the recurrence relation**: how does `dp[i]` relate to smaller states (`dp[i-1]`, `dp[i-2]`, etc.)?
3. **Identify the base case(s)**: the smallest state(s) you can answer directly.
4. **Decide the order of computation**: bottom-up (iterative, filling a table from base cases upward) or top-down (recursive with memoization, computing lazily and caching results).
5. **Optimize space if possible**: often you only need the last 1-2 rows/values, not the full table.

### Two Implementation Styles

**Top-Down (Memoization)** — recursion + a cache:
```
memo = {}
function fib(n):
    if n <= 1: return n
    if n in memo: return memo[n]
    memo[n] = fib(n-1) + fib(n-2)
    return memo[n]
```

**Bottom-Up (Tabulation)** — iterative table filling:
```
function fib(n):
    if n <= 1: return n
    dp = array of size n+1
    dp[0] = 0, dp[1] = 1
    for i from 2 to n:
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```
Both turn naive Fibonacci's **O(2ⁿ)** into **O(n)** time — this is the entire point of DP: eliminating redundant recomputation.

### Classic DP Patterns (memorize the shape of each)

**1. 0/1 Knapsack** — "choose items with weight/value constraint, each item used at most once."
```
// dp[i][w] = max value using first i items with capacity w
function knapsack(weights, values, capacity):
    n = weights.length
    dp = 2D array (n+1) x (capacity+1), all zeros
    for i from 1 to n:
        for w from 0 to capacity:
            dp[i][w] = dp[i-1][w]                       // don't take item i
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1])  // take item i
    return dp[n][capacity]
```
**Complexity**: O(n × capacity) time and space (space reducible to O(capacity) using a 1D rolling array).

**2. Longest Common Subsequence (LCS)** — "compare two sequences, find longest matching (not necessarily contiguous) subsequence."
```
// dp[i][j] = LCS length of text1[0..i) and text2[0..j)
function lcs(text1, text2):
    m, n = text1.length, text2.length
    dp = 2D array (m+1) x (n+1), all zeros
    for i from 1 to m:
        for j from 1 to n:
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```
**Complexity**: O(m·n) time and space. This pattern generalizes to **edit distance**, **longest common substring**, and **diff tools**.

**3. Longest Increasing Subsequence (LIS)**
```
// dp[i] = length of the LIS ending exactly at index i
function lis(nums):
    n = nums.length
    dp = array of 1s, size n
    for i from 1 to n-1:
        for j from 0 to i-1:
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```
**Complexity**: O(n²) naive; **O(n log n)** with a patience-sorting + binary search optimization (maintain a list of "smallest tail of an increasing subsequence of each length").

**4. Coin Change (Unbounded Knapsack variant)** — "minimum coins to make an amount, unlimited supply of each coin."
```
// dp[amount] = fewest coins needed for that amount
function coinChange(coins, amount):
    dp = array of size amount+1, filled with infinity
    dp[0] = 0
    for a from 1 to amount:
        for coin in coins:
            if coin <= a:
                dp[a] = min(dp[a], dp[a - coin] + 1)
    return dp[amount] if dp[amount] != infinity else -1
```
**Complexity**: O(amount × number of coin types).

**5. Matrix / Grid Path DP** — "count paths or find min/max cost path through a grid."
```
// dp[i][j] = min cost to reach cell (i, j)
function minPathSum(grid):
    m, n = grid.rows, grid.cols
    dp = 2D array same size as grid
    dp[0][0] = grid[0][0]
    for i from 1 to m-1: dp[i][0] = dp[i-1][0] + grid[i][0]
    for j from 1 to n-1: dp[0][j] = dp[0][j-1] + grid[0][j]
    for i from 1 to m-1:
        for j from 1 to n-1:
            dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])
    return dp[m-1][n-1]
```

### Complexity Summary Table

| DP Pattern | Time | Space (optimized) |
|---|---|---|
| Fibonacci-style (1D) | O(n) | O(1) |
| 0/1 Knapsack | O(n·W) | O(W) |
| LCS / Edit Distance | O(m·n) | O(min(m,n)) |
| LIS | O(n log n) optimized | O(n) |
| Coin Change | O(amount·coins) | O(amount) |
| Grid Path DP | O(m·n) | O(n) |

### Interview Pattern Recognition
DP is signaled by phrases like:
- **"Minimum/maximum number of ways to..."**
- **"Count the number of ways to..."**
- A **greedy or brute-force recursive** approach works but is exponential, *and* you notice the same subproblem being solved repeatedly (draw the recursion tree — repeated branches = DP opportunity).
- Problems on **sequences/strings** asking about subsequences, partitions, or matching (LCS, edit distance, palindrome partitioning).
- Problems that look like **knapsack in disguise**: "can you partition this array into two equal-sum subsets?" is 0/1 knapsack where the target is `sum/2`.

**How to tell DP apart from Greedy**: if making the locally-best choice at each step is *provably* always part of some globally optimal solution, it's Greedy (simpler, faster). If the best local choice can depend on future context you haven't seen yet (you might need to reconsider earlier choices), it's DP.

---

## 15. Greedy Algorithms

### Intuitive Explanation
A greedy algorithm is like giving change with the fewest coins by always grabbing the **largest coin that fits** at each step, trusting that this local best choice leads to the overall best result — without ever reconsidering past choices (no backtracking, no exploring alternatives).

### Step-by-Step: General Greedy Approach
1. Identify a way to rank choices at each step (sort by some criterion — end time, ratio, weight, etc.).
2. At each step, make the locally optimal choice according to that ranking.
3. Never revisit or undo a previous choice.
4. (For rigor, though rarely required in an interview) prove via an **exchange argument** that this local choice never leads to a worse global outcome than any alternative.

### Classic Example: Activity Selection (Interval Scheduling)
**Problem**: Given a set of intervals (start, end), select the maximum number of non-overlapping intervals.
**Greedy Insight**: Always pick the activity that **finishes earliest** among remaining valid options — this leaves the most room for future activities.
```
function activitySelection(intervals):
    sort intervals by end time ascending
    result = [intervals[0]]
    lastEnd = intervals[0].end
    for interval in intervals[1:]:
        if interval.start >= lastEnd:
            result.append(interval)
            lastEnd = interval.end
    return result
```
**Complexity**: O(n log n) — dominated by the sort.

### Classic Example: Fractional Knapsack
**Problem**: Like 0/1 Knapsack, but items can be split (take a fraction of an item).
**Greedy Insight**: Sort items by **value/weight ratio** descending, fill the knapsack greedily, taking a fraction of the last item if it doesn't fully fit.
```
function fractionalKnapsack(items, capacity):
    sort items by (value/weight) descending
    totalValue = 0
    for item in items:
        if capacity == 0: break
        take = min(item.weight, capacity)
        totalValue += take * (item.value / item.weight)
        capacity -= take
    return totalValue
```
*(Note: this greedy approach does NOT work for 0/1 Knapsack, since items can't be split — that's why 0/1 Knapsack needs DP instead. This distinction is a very common interview trap.)*

### Other Well-Known Greedy Algorithms
- **Huffman Coding** (data compression): repeatedly merge the two lowest-frequency nodes using a min-heap, building an optimal prefix-free binary code.
- **Prim's / Kruskal's** (MST, see Section 13.5) — both are greedy: always pick the cheapest valid edge next.
- **Dijkstra's** (see 13.4) is also fundamentally greedy: always finalize the currently-closest unvisited node.

### Complexity
Greedy algorithms are typically fast — dominated by an initial **sort** (O(n log n)) followed by a single linear pass (O(n)), giving **O(n log n)** overall in most classic problems.

### Interview Pattern Recognition
- **"Maximum number of non-overlapping intervals," "minimum number of meeting rooms," "merge intervals"** → sort + greedy scan.
- **"Assign X to Y to minimize/maximize Z"** with a clean, provable local-choice rule → greedy.
- **Red flag that greedy will FAIL**: if you can construct a counter-example where the locally-best choice leads to a worse global outcome — this usually means you actually need DP. Always sanity-check greedy with a small counter-example before committing to it in an interview.

---

## 16. Bit Manipulation

### Intuitive Explanation
Every integer is secretly a row of light switches (bits), each either ON (1) or OFF (0). Bitwise operations let you flip, check, or combine these switches directly — often turning what looks like a loop-heavy problem into a single O(1) arithmetic trick.

### Core Operators

| Operator | Symbol | What it does |
|---|---|---|
| AND | `&` | 1 only if both bits are 1 |
| OR | `\|` | 1 if either bit is 1 |
| XOR | `^` | 1 if bits differ |
| NOT | `~` | flips every bit |
| Left shift | `<<` | shifts bits left, equivalent to ×2 per shift |
| Right shift | `>>` | shifts bits right, equivalent to ÷2 per shift |

### Key Tricks (memorize these — they show up constantly)

```
// Check if the i-th bit is set
isSet = (n >> i) & 1

// Set the i-th bit
n = n | (1 << i)

// Clear the i-th bit
n = n & ~(1 << i)

// Toggle the i-th bit
n = n ^ (1 << i)

// Check if n is a power of 2
isPowerOfTwo = (n > 0) and (n & (n - 1)) == 0

// Count set bits (Brian Kernighan's algorithm)
function countSetBits(n):
    count = 0
    while n > 0:
        n = n & (n - 1)     // clears the lowest set bit
        count += 1
    return count

// Find the single non-duplicate number where every other number appears twice
function singleNumber(nums):
    result = 0
    for num in nums:
        result = result ^ num    // XOR cancels out pairs: a^a=0, a^0=a
    return result
```

### Why `n & (n-1)` clears the lowest set bit
Subtracting 1 flips all bits from the lowest set bit downward (e.g., `...1000 - 1 = ...0111`). ANDing with the original number cancels out everything up to and including that lowest set bit, leaving all higher bits untouched.

### Complexity
Almost everything in bit manipulation is **O(1)** per operation (hardware-native), or **O(log(max value))** / **O(32)** or **O(64)** for operations that must inspect every bit of a fixed-width integer — for practical purposes, this is treated as O(1).

### Interview Pattern Recognition
- **"Find the single number that doesn't repeat"** (others appear twice) → XOR trick.
- **"Count set bits," "check power of two"** → Brian Kernighan's trick.
- **"Generate all subsets"** → each subset corresponds to a binary number from `0` to `2ⁿ - 1`, where bit `i` decides whether element `i` is included — an elegant bitmask alternative to recursive backtracking.
- Problems explicitly mentioning **"O(1) space," "without extra memory," or "without using arithmetic operators"** are strong hints that a bit trick is the intended solution.

---

# Part 5 — Wrap-up

## 17. Pattern Cheat-Sheet & Roadmap

### Master Pattern-Recognition Table

| If the problem says... | Think... |
|---|---|
| Sorted array, find pair/triplet | Two Pointers |
| Contiguous subarray/substring with a condition | Sliding Window |
| Unsorted array, need O(n) lookup/frequency | Hash Map / Hash Set |
| "Kth largest/smallest," "Top K," "median of stream" | Heap / Priority Queue |
| Linked list cycle / middle element | Slow & Fast Pointers |
| Generate all subsets/permutations/combinations | Backtracking |
| Matching brackets, next greater element, undo | Stack (possibly Monotonic Stack) |
| BFS-style shortest path, level order | Queue + BFS |
| Explore all paths, connected components | DFS |
| Prerequisite/dependency ordering | Topological Sort |
| Shortest path with weights | Dijkstra / Bellman-Ford / Floyd-Warshall |
| Cheapest way to connect everything | MST (Prim's/Kruskal's) |
| Grouping/merging disjoint sets, cycle detection (undirected) | Union-Find |
| Overlapping subproblems + optimal substructure | Dynamic Programming |
| Local-best choice provably leads to global-best | Greedy |
| Prefix-based word lookup | Trie |
| "O(1) space," parity/XOR tricks, power of 2 | Bit Manipulation |
| Binary search on a monotonic condition (not just sorted array) | Binary Search on Answer |

### Suggested Study Order (if starting from scratch)
```
Complexity Analysis
   └── Arrays & Strings
          └── Two Pointers / Sliding Window
   └── Linked Lists
          └── Stacks & Queues
                 └── Recursion & Backtracking
                        └── Trees (Binary Tree → BST → Heap)
                               └── Hashing
                                      └── Graphs (BFS/DFS → Topo Sort → Shortest Path → MST → Union-Find)
                                             └── Dynamic Programming
                                                    └── Greedy
                                                           └── Bit Manipulation & Tries (as time permits)
```

### Final Interview Reminders
1. **Always clarify constraints first** (input size, value ranges, duplicates allowed, sorted or not) — they dictate which complexity class is expected.
2. **State your approach in plain English before coding** — interviewers weight communication heavily.
3. **Start with brute force, then optimize out loud** — showing you *recognize* the O(n²) → O(n log n) → O(n) progression is often more valuable than jumping straight to the optimal answer.
4. **Test your code on an edge case** (empty input, single element, all duplicates) before declaring you're done.
5. **Always state final time and space complexity** — this is the expected closing move of nearly every interview answer.

---

*End of guide. Revisit the Pattern Cheat-Sheet whenever you're stuck on a new problem — 90% of interview problems are a variant of one of the patterns above.*
