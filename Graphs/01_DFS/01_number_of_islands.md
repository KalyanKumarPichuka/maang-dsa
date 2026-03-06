# Number of Islands — Complete Explanation

> **Problem:** Given an `m x n` binary grid of `'1'`s (land) and `'0'`s (water), return the number of islands.
> An island is formed by connecting adjacent lands **horizontally or vertically**.

---

## Core Idea Across All Approaches

Every approach shares the same underlying insight:

> **An island = a connected component of `'1'`s.**
> The goal is to **count how many such components exist** in the grid.

The three approaches differ only in *how* they explore each component.

---

---

# Approach 1 — DFS (Depth-First Search)

## How It Works

1. Scan every cell in the grid.
2. When you hit an unvisited `'1'` → you've found a **new island**, increment the counter.
3. Immediately run a **DFS** from that cell to "sink" the entire island (change all connected `'1'`s to `'0'`), so it won't be counted again.
4. DFS explores in all 4 directions recursively until it hits water or the grid boundary.

## The Code

```python
class Solution:
    def num_islands(self, grid: list[list[str]]) -> int:
        if not grid:
            return 0

        rows, cols = len(grid), len(grid[0])
        island_count = 0

        # nested function — uses grid, rows, cols from enclosing scope
        def dfs(row, col):
            # stop if we stepped outside the grid or hit water
            if row < 0 or col < 0 or row >= rows or col >= cols or grid[row][col] == '0':
                return

            grid[row][col] = '0'        # sink this cell so we never revisit it

            # recurse in all 4 directions to sink the entire island
            dfs(row - 1, col)            # up
            dfs(row + 1, col)            # down
            dfs(row, col - 1)            # left
            dfs(row, col + 1)            # right

        # scan every cell in the grid
        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == '1':        # hit unvisited land — new island
                    island_count += 1
                    dfs(row, col)                # sink all connected land

        return island_count
```

### Line-by-Line Explanation

| Line | What it does |
|------|-------------|
| `if not grid` | Guard against empty input |
| `rows, cols = ...` | Cache grid dimensions once |
| `def dfs(row, col)` | Nested helper — captures `grid`, `rows`, `cols` via closure |
| `if grid[row][col] == '1'` | Found a new unvisited island |
| `island_count += 1` | Count it immediately |
| `dfs(row, col)` | Explore and "sink" all connected land cells |
| `if row < 0 or ...` | Stop recursion at boundaries or water |
| `grid[row][col] = '0'` | Mark visited by turning land into water |
| Four recursive calls | Go up, down, left, right |

---

## Example 1 — Single Large Island

**Input:**
```
1 1 1
1 1 0
0 1 0
```

**Execution:**
- Scanner hits `(0,0)` → `'1'` → `island_count = 1`, DFS starts
- DFS from `(0,0)` → marks `(0,0)` as `'0'`, explores neighbors
- Eventually visits and sinks: `(0,0),(0,1),(0,2),(1,0),(1,1),(2,1)` — all connected
- Grid becomes all `'0'`s; no more `'1'`s found in the scan

**Output: `1`**

---

## Example 2 — Three Separate Islands

**Input:**
```
1 1 0 0 0
1 1 0 0 0
0 0 1 0 0
0 0 0 1 1
```

**Execution:**
- `(0,0)` → `'1'` → `island_count = 1`, DFS sinks `(0,0),(0,1),(1,0),(1,1)`
- Scanner continues, hits `(2,2)` → `'1'` → `island_count = 2`, DFS sinks `(2,2)`
- Scanner hits `(3,3)` → `'1'` → `island_count = 3`, DFS sinks `(3,3),(3,4)`

**Output: `3`**

---

## Example 3 — All Water

**Input:**
```
0 0 0
0 0 0
0 0 0
```

**Execution:**
- Scanner finds no `'1'` in the entire grid
- `island_count` stays at `0`

**Output: `0`**

---

## Full Execution Flow — DFS (Detailed)

Using this grid:
```
1 1 0
0 1 0
0 0 1
```

**Step 1:** Scanner at `(0,0)` → `'1'` found → `island_count = 1` → call `dfs(grid, 0, 0)`

```
DFS CALL STACK:
dfs(0,0) → marks (0,0)='0', calls dfs(-1,0), dfs(1,0), dfs(0,-1), dfs(0,1)
  dfs(-1,0) → row < 0 → RETURN (base case)
  dfs(1,0)  → grid[1][0]='0' → RETURN (base case)
  dfs(0,-1) → col < 0 → RETURN (base case)
  dfs(0,1)  → grid[0][1]='1' → marks (0,1)='0', calls:
    dfs(-1,1) → row < 0 → RETURN
    dfs(1,1)  → grid[1][1]='1' → marks (1,1)='0', calls:
      dfs(0,1)  → grid[0][1]='0' → RETURN
      dfs(2,1)  → grid[2][1]='0' → RETURN
      dfs(1,0)  → grid[1][0]='0' → RETURN
      dfs(1,2)  → grid[1][2]='0' → RETURN
      ← all 4 branches exhausted, dfs(1,1) returns
    dfs(0,1)  → grid[0][1]='0' → RETURN
    dfs(0,2)  → grid[0][2]='0' → RETURN
    ← dfs(0,1) returns
← dfs(0,0) returns
```

**Grid after first DFS:**
```
0 0 0
0 0 0
0 0 1   ← this '1' is untouched
```

**Step 2:** Scanner continues, reaches `(2,2)` → `'1'` → `island_count = 2` → call `dfs(grid, 2, 2)`
- Marks `(2,2)='0'`, all neighbors are out-of-bounds or water → returns immediately

**Final Answer: `2`**

---

## Complexity

| | Complexity |
|-|-----------|
| **Time** | `O(M × N)` — every cell visited at most once |
| **Space** | `O(M × N)` — recursion stack in worst case (all land) |

---

---

# Approach 2 — BFS (Breadth-First Search)

## How It Works

Same high-level idea as DFS, but instead of going *deep first*, BFS **fans out layer by layer** using a queue.

1. Scan the grid for `'1'`s.
2. On finding one, add it to a queue and mark it visited immediately.
3. Process the queue: for each cell, enqueue its unvisited `'1'` neighbors.
4. When the queue is empty, the entire island has been explored.

## The Code

```python
from collections import deque

class Solution:
    def num_islands(self, grid: list[list[str]]) -> int:
        if not grid:
            return 0

        rows, cols = len(grid), len(grid[0])
        directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]  # down, up, right, left
        island_count = 0

        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == '1':       # hit unvisited land — new island
                    island_count += 1

                    # seed the queue with this starting cell and sink it
                    queue = deque([(row, col)])
                    grid[row][col] = '0'

                    # explore the entire island level by level
                    while queue:
                        cur_row, cur_col = queue.popleft()
                        for d_row, d_col in directions:
                            new_row, new_col = cur_row + d_row, cur_col + d_col
                            # if neighbour is valid land, enqueue and sink right away
                            # (sinking on enqueue, not dequeue, prevents duplicates)
                            if 0 <= new_row < rows and 0 <= new_col < cols and grid[new_row][new_col] == '1':
                                queue.append((new_row, new_col))
                                grid[new_row][new_col] = '0'

        return island_count
```

### Key Difference from DFS

> DFS marks cells visited when the **recursive call executes**.
> BFS marks cells visited **when they are enqueued** — this is critical to prevent adding the same cell to the queue twice from different neighbors.

---

## Example 1 — Linear Island (Horizontal Strip)

**Input:**
```
0 0 0
1 1 1
0 0 0
```

**BFS Execution:**
- Scanner hits `(1,0)` → `'1'` → `island_count = 1`
- Queue: `[(1,0)]`, mark `(1,0)='0'`
- Process `(1,0)`: neighbor `(1,1)` is `'1'` → enqueue, mark `'0'`
- Queue: `[(1,1)]`
- Process `(1,1)`: neighbor `(1,2)` is `'1'` → enqueue, mark `'0'`
- Queue: `[(1,2)]`
- Process `(1,2)`: all neighbors are `'0'` or out-of-bounds
- Queue empty → island fully explored

**Output: `1`**

---

## Example 2 — Diagonal (Separate) Islands

**Input:**
```
1 0 0
0 1 0
0 0 1
```

**BFS Execution:**
- `(0,0)` → `island_count = 1`, BFS: only cell in component, queue empties quickly
- `(1,1)` → `island_count = 2`, same
- `(2,2)` → `island_count = 3`, same
> Note: diagonal adjacency does NOT count — each `'1'` is isolated

**Output: `3`**

---

## Example 3 — U-Shaped Island

**Input:**
```
1 0 1
1 0 1
1 1 1
```

**BFS Execution:**
- `(0,0)` found → `island_count = 1`, BFS explores the entire U shape
- BFS visits: `(0,0)→(1,0)→(2,0)→(2,1)→(2,2)→(1,2)→(0,2)` (order may vary)
- All cells of the U shape are in one component → fully sunk
- No more `'1'`s remain

**Output: `1`**

---

## Full Execution Flow — BFS (Detailed)

Using this grid:
```
1 1 0
0 1 0
0 0 1
```

**Step 1:** Scanner at `(0,0)` → `'1'` found → `island_count = 1`
Enqueue `(0,0)`, sink it → `grid[0][0] = '0'`

```
BFS QUEUE TRACE:

queue = [(0,0)]           grid[0][0] = '0'

  Dequeue (0,0):
    check (1,0) → grid[1][0]='0' → skip
    check (-1,0) → out of bounds → skip
    check (0,1) → grid[0][1]='1' → enqueue, sink → grid[0][1]='0'
    check (0,-1) → out of bounds → skip

queue = [(0,1)]

  Dequeue (0,1):
    check (1,1) → grid[1][1]='1' → enqueue, sink → grid[1][1]='0'
    check (-1,1) → out of bounds → skip
    check (0,2) → grid[0][2]='0' → skip
    check (0,0) → grid[0][0]='0' → skip

queue = [(1,1)]

  Dequeue (1,1):
    check (2,1) → grid[2][1]='0' → skip
    check (0,1) → grid[0][1]='0' → skip
    check (1,2) → grid[1][2]='0' → skip
    check (1,0) → grid[1][0]='0' → skip

queue = []  ← empty, island 1 fully explored
```

**Grid after first BFS:**
```
0 0 0
0 0 0
0 0 1   ← this '1' is untouched
```

**Step 2:** Scanner continues, reaches `(2,2)` → `'1'` → `island_count = 2`
Enqueue `(2,2)`, sink it → `grid[2][2] = '0'`

```
queue = [(2,2)]

  Dequeue (2,2):
    check (3,2) → out of bounds → skip
    check (1,2) → grid[1][2]='0' → skip
    check (2,3) → out of bounds → skip
    check (2,1) → grid[2][1]='0' → skip

queue = []  ← empty, island 2 fully explored
```

**Final Answer: `2`**

---

## Complexity

| | Complexity |
|-|-----------|
| **Time** | `O(M × N)` |
| **Space** | `O(min(M, N))` — BFS queue holds at most a diagonal's worth of cells |

> BFS has **better space complexity** than DFS because the queue never grows as large as the call stack can in the worst case.

---

---

# Approach 3 — Union-Find (Disjoint Set Union)

## How It Works

Union-Find treats each land cell as a **node** in a graph. Start by assuming every `'1'` is its own island. Then merge adjacent `'1'`s together. Each merge reduces the island count by 1.

1. Initialize: every `'1'` is its own "set" (its own island). Count = number of `'1'`s.
2. For each `'1'`, check its **right and bottom** neighbors (to avoid re-checking pairs).
3. If a neighbor is also `'1'`, call `union()` → merge the two sets, decrement count.
4. Return the final count.

## The Code

```python
class Solution:
    def num_islands(self, grid: list[list[str]]) -> int:
        if not grid:
            return 0

        rows, cols = len(grid), len(grid[0])
        uf = UnionFind(grid)

        # only check right and down — avoids merging the same pair twice
        directions = [(1, 0), (0, 1)]

        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == '1':
                    for d_row, d_col in directions:
                        new_row, new_col = row + d_row, col + d_col
                        if new_row < rows and new_col < cols and grid[new_row][new_col] == '1':
                            # convert 2D coords to flat 1D index: row * cols + col
                            uf.union(row * cols + col, new_row * cols + new_col)

        return uf.get_count()


class UnionFind:
    def __init__(self, grid):
        rows, cols = len(grid), len(grid[0])
        self.parent = [0] * (rows * cols)
        self.rank = [0] * (rows * cols)
        self.count = 0          # tracks number of disjoint sets (islands)

        # every land cell starts as its own island
        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == '1':
                    node_id = row * cols + col
                    self.parent[node_id] = node_id  # point to itself (self is root)
                    self.count += 1

    def find(self, node):
        # path compression — flatten tree so every node points directly to root
        if self.parent[node] != node:
            self.parent[node] = self.find(self.parent[node])
        return self.parent[node]

    def union(self, node_a, node_b):
        root_a, root_b = self.find(node_a), self.find(node_b)

        if root_a == root_b:
            return              # already part of the same island, nothing to do

        # union by rank — attach the shorter tree under the taller one
        if self.rank[root_a] > self.rank[root_b]:
            self.parent[root_b] = root_a
        elif self.rank[root_a] < self.rank[root_b]:
            self.parent[root_a] = root_b
        else:
            self.parent[root_b] = root_a
            self.rank[root_a] += 1      # tree grew one level taller

        self.count -= 1         # two separate islands just became one

    def get_count(self):
        return self.count
```

### Key Concepts

| Concept | Explanation |
|---------|-------------|
| **parent[]** | `parent[i]` stores who `i`'s representative (root) is |
| **rank[]** | Keeps trees shallow by always merging smaller into larger |
| **Path Compression** | `find()` flattens the tree so future lookups are faster |
| **2D → 1D index** | Cell `(r, c)` maps to `r * cols + c` for flat array storage |

---

## Example 1 — Simple 2×2 Island

**Input:**
```
1 1
1 0
```

**Union-Find Execution:**
- Init: `count = 3` (three `'1'`s at indices 0, 1, 2)
- `parent = [0, 1, 2, 3]`
- At `(0,0)`: right neighbor `(0,1)` is `'1'` → `union(0, 1)` → `count = 2`
- At `(0,0)`: bottom neighbor `(1,0)` is `'1'` → `union(0, 2)` → `count = 1`
- At `(0,1)`: right=out-of-bounds, bottom `(1,1)='0'` → skip
- At `(1,0)`: right `(1,1)='0'` → skip

**Output: `1`**

---

## Example 2 — Two Separate Islands

**Input:**
```
1 1 0
0 0 0
0 1 1
```

**Union-Find Execution:**
- Init: `count = 4` (indices 0,1,7,8)
- `(0,0)` right → `union(0,1)` → `count = 3`
- `(2,1)` right → `union(7,8)` → `count = 2`
- No other adjacent `'1'` pairs

**Output: `2`**

---

## Example 3 — All Land (One Giant Island)

**Input:**
```
1 1
1 1
```

**Union-Find Execution:**
- Init: `count = 4`
- `(0,0)` right → `union(0,1)` → `count = 3`
- `(0,0)` down → `union(0,2)` → `count = 2`
- `(0,1)` down → `union(1,3)` → `find(1)=root of 0,1,2`, `find(3)=3` → merge → `count = 1`
- `(1,0)` right → `union(2,3)` → `find(2)` and `find(3)` → same root → skip

**Output: `1`**

---

## Full Execution Flow — Union-Find (Detailed)

Using this grid:
```
1 1 0
0 1 0
0 0 1
```

Grid indices (2D → 1D), `cols = 3`:
```
(0,0)=0  (0,1)=1  (0,2)=2
(1,0)=3  (1,1)=4  (1,2)=5
(2,0)=6  (2,1)=7  (2,2)=8
```

**Init:** Only `'1'` cells become nodes. Land at indices 0, 1, 4, 8.
```
parent = [0, 1, _, _, 4, _, _, _, 8]    (_ = water, unused)
rank   = [0, 0, _, _, 0, _, _, _, 0]
count  = 4
```

**Scan (0,0) — index 0:**
- Check right `(0,1)` → `'1'` → `union(0, 1)`
  ```
  find(0) = 0, find(1) = 1 → different roots
  rank[0] == rank[1] → parent[1] = 0, rank[0] = 1
  count = 3
  parent = [0, 0, _, _, 4, _, _, _, 8]
  ```
- Check down `(1,0)` → `'0'` → skip

**Scan (0,1) — index 1:**
- Check right `(0,2)` → `'0'` → skip
- Check down `(1,1)` → `'1'` → `union(1, 4)`
  ```
  find(1): parent[1]=0, parent[0]=0 → root = 0   (path compression: 1→0)
  find(4) = 4 → different roots
  rank[0]=1 > rank[4]=0 → parent[4] = 0
  count = 2
  parent = [0, 0, _, _, 0, _, _, _, 8]
  ```

**Scan (1,1) — index 4:**
- Check right `(1,2)` → `'0'` → skip
- Check down `(2,1)` → `'0'` → skip

**Scan (2,2) — index 8:**
- Check right `(2,3)` → out of bounds → skip
- Check down `(3,2)` → out of bounds → skip

**Final state:**
```
parent = [0, 0, _, _, 0, _, _, _, 8]
count  = 2

Island 1: nodes {0, 1, 4} → cells (0,0), (0,1), (1,1)
Island 2: node  {8}        → cell  (2,2)
```

**Final Answer: `2`**

---

## Complexity

| | Complexity |
|-|-----------|
| **Time** | `O(M × N × α(M×N))` — α is the inverse Ackermann function, effectively constant |
| **Space** | `O(M × N)` for `parent[]` and `rank[]` arrays |

---

---

# Approach Comparison

| Feature | DFS | BFS | Union-Find |
|---------|-----|-----|-----------|
| **Paradigm** | Recursive exploration | Iterative queue | Mathematical merging |
| **Time Complexity** | `O(M×N)` | `O(M×N)` | `O(M×N×α)` ≈ `O(M×N)` |
| **Space Complexity** | `O(M×N)` stack | `O(min(M,N))` queue | `O(M×N)` arrays |
| **Modifies grid?** | Yes (sinks `'1'`→`'0'`) | Yes | No |
| **Best for** | Simplicity & readability | Memory-constrained | Dynamic connectivity problems |
| **Tricky part** | Stack overflow on huge grids | Marking visited *on enqueue* | 2D→1D index mapping |

---

## Which Should You Use?

- **Interview / readability** → **DFS** (cleanest, most intuitive)
- **Memory efficiency** → **BFS** (smaller space footprint)
- **Dynamic graphs / online merging** → **Union-Find** (best for problems where edges are added over time)

---

## Tricks to Remember

### The Core Pattern (All Three Approaches)

> **"Scan → Find a '1' → Count it → Erase the whole island → Keep scanning."**

The only difference is *how* you erase: DFS dives deep, BFS fans wide, Union-Find merges sets.

### DFS — "Sink the Island"

- Think of pouring water on the land cell — it floods recursively in all 4 directions.
- **Template to memorize:**
  1. Base case: out of bounds or water → return
  2. Sink current cell (`grid[r][c] = '0'`)
  3. Four recursive calls (up, down, left, right)
- **Gotcha:** On very large grids (e.g. 500×500 all land), Python's default recursion limit (1000) will blow up. Use `sys.setrecursionlimit()` or switch to BFS.

### BFS — "Ripple Outward"

- Picture dropping a stone in water — the ripple spreads level by level.
- **Key rule: sink ON ENQUEUE, not on dequeue.** If you forget this, the same cell gets added to the queue multiple times from different neighbors.
- **Template to memorize:**
  1. Find a `'1'` → enqueue it, sink it immediately
  2. While queue not empty → dequeue → check 4 neighbors → enqueue and sink any `'1'`

### Union-Find — "Merge Neighbors"

- Think of each land cell starting as a solo team. When two neighbors are both land, they merge teams. Final answer = number of remaining teams.
- **Two key optimizations to always include:**
  1. **Path compression** in `find()` — makes future lookups nearly O(1)
  2. **Union by rank** in `union()` — keeps trees flat
- **Only check right and down** — since you scan left-to-right, top-to-bottom, checking all 4 directions would double-count every pair.
- **2D → 1D trick:** `node_id = row * cols + col` — use this whenever you need to flatten a grid into a 1D array.

### Quick Mental Checklist Before Coding

1. **Empty grid?** → return 0 (handle edge case first)
2. **What counts as connected?** → 4 directions (horizontal + vertical), NOT diagonal
3. **How to mark visited?** → DFS/BFS: flip `'1'` to `'0'` in-place. Union-Find: no modification needed.
4. **When to increment count?** → DFS/BFS: when you *first discover* a new `'1'`. Union-Find: start with total `'1'`s, *decrement* on each merge.
