# ⚡ Samsung SWC-Pro — 10-Day Practice Plan

> **Platforms:** Codeforces & LeetCode only &nbsp;|&nbsp; **Max 7 problems/day** &nbsp;|&nbsp; **Day 9 = Revision** &nbsp;|&nbsp; **Day 10 = Mock Exam**

---

## 📋 At a Glance

| Day | Focus | Contest |
|-----|-------|---------|
| Day 1 | Simulation Basics | — |
| Day 2 | BFS / Grid | — |
| Day 3 | DFS + Backtracking | 🏆 Codeforces Round |
| Day 4 | DP Part 1 — Subset / Partition | 🏆 LeetCode Weekly |
| Day 5 | DP Part 2 — Path / Sequence | — |
| Day 6 | Graph Traversal Advanced | 🏆 Codeforces Div. 3 |
| Day 7 | Greedy + Sorting | 🏆 LeetCode Weekly |
| Day 8 | Strings + Math | — |
| Day 9 | 📝 Revision Day | — |
| Day 10 | 🧪 Mock Exam + Final Contest | 🏆 Codeforces Round |

---

## Day 1 — Simulation Basics

### 📁 From Your Repo
- `segments.cpp`
- `days.cpp`
- `cars.cpp`

### 📖 Topics to Read
- State machines — track position/direction/state across steps
- Use structs or pairs to bundle `(row, col, direction)` as state
- Always handle edge cases: boundary checks before moving
- Modular simulation: write helper functions for each action

### 💡 Tricks & Tips
- Use `int dx[] = {-1,0,1,0}; dy[] = {0,1,0,-1};` for 4-directional movement
- Use a `visited[][]` array to prevent re-processing cells
- Simulate step by step — never shortcut unless proven safe
- Print intermediate grid states while debugging

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [Vasya and a Tree](https://codeforces.com/problemset/problem/1076/E) | Simulation + DFS | Codeforces | 1800 | [ ] |
| 2 | [The Queue](https://codeforces.com/problemset/problem/353/A) | Simulation | Codeforces | 1300 | [ ] |
| 3 | [Robot Return to Origin](https://leetcode.com/problems/robot-return-to-origin/) | Simulation | LeetCode | Easy | [ ] |
| 4 | [Circular Array Loop](https://leetcode.com/problems/circular-array-loop/) | Simulation | LeetCode | Medium | [ ] |
| 5 | [Broken Necklace](https://codeforces.com/problemset/problem/680/B) | String Simulation | Codeforces | 1500 | [ ] |
| 6 | [Flood Fill](https://leetcode.com/problems/flood-fill/) | Grid Simulation | LeetCode | Easy | [ ] |
| 7 | [Bus Stop](https://codeforces.com/problemset/problem/850/A) | Simulation | Codeforces | 1100 | [ ] |

---

## Day 2 — BFS / Grid

### 📁 From Your Repo
- `robot.cpp`
- `warehouse.cpp`

### 📖 Topics to Read
- BFS guarantees shortest path in unweighted grids
- Multi-source BFS: push all sources into queue at start
- 0-1 BFS: use deque, push front for weight-0 edges, back for weight-1
- When to use BFS vs DFS: BFS = shortest path, DFS = connectivity/cycle

### 💡 Tricks & Tips
- Always check bounds AND visited before pushing to queue
- Use `pair<int,int>` in queue; use `dist[n][m]` matrix for distances
- For multi-source BFS (fire, rotting), initialise queue with ALL sources at once
- BFS level = distance; track with a `dist[][]` matrix or a level counter

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [Labyrinth](https://codeforces.com/problemset/problem/1063/B) | Grid BFS | Codeforces | 1900 | [ ] |
| 2 | [Fire](https://codeforces.com/problemset/problem/1033/D) | Multi-source BFS | Codeforces | 1700 | [ ] |
| 3 | [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | BFS on Grid | LeetCode | Medium | [ ] |
| 4 | [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | Grid BFS/DFS | LeetCode | Medium | [ ] |
| 5 | [Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | BFS | LeetCode | Medium | [ ] |
| 6 | [Knight Moves](https://codeforces.com/problemset/problem/62/B) | Grid BFS | Codeforces | 1400 | [ ] |
| 7 | [Walls and Gates](https://leetcode.com/problems/walls-and-gates/) | Multi-source BFS | LeetCode | Medium | [ ] |

---

## Day 3 — DFS + Backtracking
> 🏆 **Contest:** Codeforces Round — check [codeforces.com](https://codeforces.com)

### 📁 From Your Repo
- `RB.cpp`
- `stones.cpp`

### 📖 Topics to Read
- Backtracking = DFS + undo (restore state after each recursive call)
- Pruning: cut branches early when constraints are already violated
- Permutations vs Combinations: use `index` / `used[]` to control order
- Constraint propagation: for Sudoku-style, eliminate options before branching

### 💡 Tricks & Tips
- Always mark `visited` BEFORE recursing; unmark AFTER returning
- Pass current state by reference and undo changes on return
- Use a `used[]` boolean array to skip duplicates in permutations
- Sort input before backtracking to enable early termination
- For grid DFS: check bounds and visited in one `if` condition

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [N-Queens](https://leetcode.com/problems/n-queens/) | Backtracking | LeetCode | Hard | [ ] |
| 2 | [Combination Sum](https://leetcode.com/problems/combination-sum/) | Backtracking + Pruning | LeetCode | Medium | [ ] |
| 3 | [Word Search](https://leetcode.com/problems/word-search/) | Grid DFS Backtracking | LeetCode | Medium | [ ] |
| 4 | [Sudoku Solver](https://leetcode.com/problems/sudoku-solver/) | Backtracking | LeetCode | Hard | [ ] |
| 5 | [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/) | Backtracking | LeetCode | Medium | [ ] |
| 6 | [Target Sum](https://leetcode.com/problems/target-sum/) | Backtracking/DP | LeetCode | Medium | [ ] |
| 7 | [Zuma](https://codeforces.com/problemset/problem/41/D) | Interval Backtracking | Codeforces | 1700 | [ ] |

---

## Day 4 — DP Part 1 — Subset / Partition
> 🏆 **Contest:** LeetCode Weekly — check [leetcode.com](https://leetcode.com)

### 📁 From Your Repo
- `min_subset_diff.cpp`
- `Min_Subset_Diff_2025.cpp`

### 📖 Topics to Read
- Subset DP: `dp[j] = true` if sum `j` is achievable from a subset
- 0/1 Knapsack: iterate items outer, capacity inner (backwards to avoid reuse)
- Unbounded Knapsack: iterate capacity forward (allows item reuse)
- Partition into equal subsets: target = total/2, check if reachable

### 💡 Tricks & Tips
- For subset DP, iterate `j` from high to low to prevent item reuse
- Initialise `dp[0] = true` (empty set achieves sum 0)
- Minimise subset difference = find largest subset `<= total/2`
- Always check if total is odd early — impossible to split exactly

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) | Subset DP | LeetCode | Medium | [ ] |
| 2 | [Last Stone Weight II](https://leetcode.com/problems/last-stone-weight-ii/) | Subset DP Variant | LeetCode | Medium | [ ] |
| 3 | [Knapsack Problem](https://codeforces.com/problemset/problem/19/B) | 0/1 Knapsack | Codeforces | 1500 | [ ] |
| 4 | [Coin Change](https://leetcode.com/problems/coin-change/) | Unbounded Knapsack | LeetCode | Medium | [ ] |
| 5 | [Coin Change II](https://leetcode.com/problems/coin-change-ii/) | Knapsack Count | LeetCode | Medium | [ ] |
| 6 | [Target Sum (DP)](https://leetcode.com/problems/target-sum/) | Subset DP | LeetCode | Medium | [ ] |
| 7 | [Minimum Cost to Fill Bag](https://codeforces.com/problemset/problem/1256/C) | Greedy + DP | Codeforces | 1500 | [ ] |

---

## Day 5 — DP Part 2 — Path / Sequence

### 📁 From Your Repo
- `min_cost.cpp`
- `scores.cpp`

### 📖 Topics to Read
- Grid DP: `dp[i][j]` depends on `dp[i-1][j]` and `dp[i][j-1]`
- LCS: `dp[i][j] = dp[i-1][j-1]+1` if match, else `max(dp[i-1][j], dp[i][j-1])`
- LIS: O(n log n) with patience sort / binary search on active list
- Edit Distance: 3 operations — insert, delete, replace — each costs 1

### 💡 Tricks & Tips
- Initialise DP base cases carefully: `dp[0][j]` and `dp[i][0]`
- For path DP with obstacles, set `dp[blocked] = 0` (or INF for min cost)
- Reconstruct the path by backtracking through the DP table
- Space-optimise 2D DP by keeping only 2 rows when only previous row is needed

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [Unique Paths II](https://leetcode.com/problems/unique-paths-ii/) | Grid DP | LeetCode | Medium | [ ] |
| 2 | [Triangle](https://leetcode.com/problems/triangle/) | DP on Structure | LeetCode | Medium | [ ] |
| 3 | [Edit Distance](https://leetcode.com/problems/edit-distance/) | Classic DP | LeetCode | Medium | [ ] |
| 4 | [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) | LCS DP | LeetCode | Medium | [ ] |
| 5 | [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) | LIS DP | LeetCode | Medium | [ ] |
| 6 | [Maximal Square](https://leetcode.com/problems/maximal-square/) | Grid DP | LeetCode | Medium | [ ] |
| 7 | [Sequence DP](https://codeforces.com/problemset/problem/909/C) | Sequence DP | Codeforces | 1700 | [ ] |

---

## Day 6 — Graph Traversal Advanced
> 🏆 **Contest:** Codeforces Div. 3 — check [codeforces.com](https://codeforces.com)

### 📁 From Your Repo
- `logging_trees.cpp`
- `soldiers.cpp`

### 📖 Topics to Read
- Topological sort: Kahn's BFS (in-degree) or DFS post-order
- Union-Find: path compression + union by rank for near-O(1) ops
- Bipartite check: 2-color with BFS; conflict → not bipartite
- Cycle detection in directed graphs: DFS with 3-color (white/grey/black)

### 💡 Tricks & Tips
- Build adjacency list as `vector<vector<int>> adj(n+1)`
- For Union-Find: `find()` with path compression; `union()` with rank
- Topological sort only works on DAGs — check for cycles first
- Track `in-degree` for Kahn's: decrement neighbors when a node is processed

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [Number of Islands](https://leetcode.com/problems/number-of-islands/) | DFS/BFS Components | LeetCode | Medium | [ ] |
| 2 | [Course Schedule](https://leetcode.com/problems/course-schedule/) | Cycle Detection | LeetCode | Medium | [ ] |
| 3 | [Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/) | BFS Coloring | LeetCode | Medium | [ ] |
| 4 | [Redundant Connection](https://leetcode.com/problems/redundant-connection/) | Union Find | LeetCode | Medium | [ ] |
| 5 | [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | Dijkstra / BFS | LeetCode | Hard | [ ] |
| 6 | [Connected Components](https://codeforces.com/problemset/problem/920/E) | Union Find | Codeforces | 1700 | [ ] |
| 7 | [Round Trip](https://codeforces.com/problemset/problem/500/B) | Cycle Detection DFS | Codeforces | 1600 | [ ] |

---

## Day 7 — Greedy + Sorting
> 🏆 **Contest:** LeetCode Weekly — check [leetcode.com](https://leetcode.com)

### 📁 From Your Repo
- `apples.cpp`
- `digit_sum.cpp`

### 📖 Topics to Read
- Greedy works when local optimal leads to global optimal
- Interval scheduling: sort by end time, pick non-overlapping greedily
- Exchange argument: prove greedy by showing swapping any two choices is no better
- Priority queues for greedy: max-heap or min-heap based on goal

### 💡 Tricks & Tips
- Sort first — most greedy problems need sorted input
- For interval problems: sort by **end time**, not start time
- Max-heap: `priority_queue<int>` — Min-heap: `priority_queue<int,vector<int>,greater<int>>`
- If greedy feels wrong, test with a small counter-example before switching to DP

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Greedy Intervals | LeetCode | Medium | [ ] |
| 2 | [Jump Game II](https://leetcode.com/problems/jump-game-ii/) | Greedy | LeetCode | Medium | [ ] |
| 3 | [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Greedy + Priority Queue | LeetCode | Medium | [ ] |
| 4 | [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Greedy Interval | LeetCode | Medium | [ ] |
| 5 | [Gas Station](https://leetcode.com/problems/gas-station/) | Greedy Circular | LeetCode | Medium | [ ] |
| 6 | [Maximize Toys](https://codeforces.com/problemset/problem/602/B) | Greedy Sort | Codeforces | 1200 | [ ] |
| 7 | [Coins](https://codeforces.com/problemset/problem/158/C) | Greedy | Codeforces | 1500 | [ ] |

---

## Day 8 — Strings + Math

### 📁 From Your Repo
- `strings.cpp`
- `digit_sum.cpp`

### 📖 Topics to Read
- Sliding window: two pointers expanding/contracting on a window
- String hashing: `unordered_map<char,int>` for frequency count
- Palindrome: expand from center O(n²) or Manacher's O(n)
- Digit DP: process a number digit by digit, track tight constraint

### 💡 Tricks & Tips
- For anagram/window problems, use a freq array of size 26 (or `unordered_map`)
- Two pointer: move right to expand, move left to shrink when condition violated
- `s.substr()` is O(n) — avoid calling it inside tight loops
- Digit sum divisibility trick: digit sum divisible by 9 → number divisible by 9
- Stack is ideal for valid parentheses and next-greater-element patterns

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) | String DP | LeetCode | Medium | [ ] |
| 2 | [Group Anagrams](https://leetcode.com/problems/group-anagrams/) | String Hashing | LeetCode | Medium | [ ] |
| 3 | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Sliding Window | LeetCode | Hard | [ ] |
| 4 | [Decode Ways](https://leetcode.com/problems/decode-ways/) | String DP | LeetCode | Medium | [ ] |
| 5 | [Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/) | Stack/DP | LeetCode | Hard | [ ] |
| 6 | [String Game](https://codeforces.com/problemset/problem/779/C) | Binary Search + String | Codeforces | 1700 | [ ] |
| 7 | [Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/) | Hashing | LeetCode | Medium | [ ] |

---

## Day 9 — 📝 Revision Day

### 📁 From Your Repo
- Re-solve problems you got wrong
- Review all notes from Days 1–8

### ✅ Revision Checklist

- [ ] List all problems you could not solve — re-attempt your top 3
- [ ] Review your BFS/DFS template — write it from scratch without help
- [ ] Review your DP subset pattern — trace through `min_subset_diff` logic
- [ ] Practice writing fast I/O: `ios::sync_with_stdio(false); cin.tie(NULL);`
- [ ] Time yourself: solve 1 medium problem in under 45 minutes
- [ ] Write down all edge cases you missed this week — keep as a cheat sheet
- [ ] Pick one Samsung repo problem and write final clean submission-ready code

---

## Day 10 — 🧪 Mock Exam + Final Contest
> 🏆 **Contest:** Codeforces Round (Evening) — check [codeforces.com](https://codeforces.com)
>
> **Morning:** 3-hour mock with 2 unsolved repo problems — no hints, no Google
> **Evening:** Attend the Codeforces live round

### 📖 Topics to Read
- Mixed bag — any topic can appear; stay calm and read carefully
- If stuck after 20 mins: sketch the approach on paper before coding
- Check time complexity before submitting: n=10⁵ needs O(n log n) or better

### 💡 Tricks & Tips
- Read BOTH problems in the first 10 minutes before writing any code
- Pick the problem you are more confident about first
- Leave 5 minutes at the end to test edge cases (n=1, all same values)
- Samsung exam is **C++ only** — make sure your solution compiles clean

### 🧩 Practice Problems

| # | Problem | Topic | Platform | Diff | Done |
|---|---------|-------|----------|------|------|
| 1 | [Virus Spread (Samsung-style)](https://codeforces.com/problemset/problem/600/D) | BFS + Simulation | Codeforces | 1700 | [ ] |
| 2 | [Queue Simulation](https://codeforces.com/problemset/problem/545/C) | Queue Simulation | Codeforces | 1600 | [ ] |
| 3 | [Tetromino-style DFS](https://codeforces.com/problemset/problem/545/D) | DFS + Brute Force | Codeforces | 1700 | [ ] |
| 4 | [Maximum Profit in Job Scheduling](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) | DP + Binary Search | LeetCode | Hard | [ ] |
| 5 | [Critical Connections](https://leetcode.com/problems/critical-connections-in-a-network/) | Tarjan DFS | LeetCode | Hard | [ ] |
| 6 | [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) | Topological Sort | LeetCode | Hard | [ ] |
| 7 | [Bus Routes](https://leetcode.com/problems/bus-routes/) | BFS | LeetCode | Hard | [ ] |

---

## 🛠️ General Tips for SWC-Pro

- Solve repo questions **first** every day — Samsung frequently repeats past questions
- Always use C++ fast I/O template (see below)
- Read constraints before coding — tight limits rule out brute force
- Template your BFS and DFS — write it from memory in under 2 minutes
- In the actual exam: read **both** problems fully before writing any code
- For DP: define the state clearly on paper before coding
- If over 1 hour on a problem — read the editorial and move on
- Samsung exam uses **C++ only** — no Python, no Java
- Test edge cases: `n=1`, all same values, empty grid, max constraints

---

## 📄 C++ Fast I/O Boilerplate

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(NULL);

    int n;
    cin >> n;
    // your code here

    return 0;
}
```

---

## 🧭 BFS Template

```cpp
void bfs(int startR, int startC, vector<vector<int>>& grid) {
    int n = grid.size(), m = grid[0].size();
    int dx[] = {-1, 0, 1, 0};
    int dy[] = {0, 1, 0, -1};
    vector<vector<bool>> visited(n, vector<bool>(m, false));
    queue<pair<int,int>> q;
    q.push({startR, startC});
    visited[startR][startC] = true;
    while (!q.empty()) {
        auto [r, c] = q.front(); q.pop();
        for (int d = 0; d < 4; d++) {
            int nr = r + dx[d], nc = c + dy[d];
            if (nr >= 0 && nr < n && nc >= 0 && nc < m && !visited[nr][nc]) {
                visited[nr][nc] = true;
                q.push({nr, nc});
            }
        }
    }
}
```

---

## 🔁 Backtracking Template

```cpp
void backtrack(int idx, vector<int>& current, vector<int>& nums, vector<vector<int>>& result) {
    // base case
    if (/* goal condition */) {
        result.push_back(current);
        return;
    }
    for (int i = idx; i < nums.size(); i++) {
        current.push_back(nums[i]);       // choose
        backtrack(i + 1, current, nums, result); // explore
        current.pop_back();               // undo
    }
}
```

---

*Good luck on the Samsung SWC-Pro! 🚀*
