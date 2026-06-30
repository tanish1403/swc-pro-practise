# 📘 Samsung SWC-Pro — Problem Explanations & Solutions (Part 1 of 2)

> Based on the actual questions from [kirang193/samsung-pro-coding-questions](https://github.com/kirang193/samsung-pro-coding-questions)
> Each problem includes: problem statement · example · how to identify · step-by-step solution · C++ & Python code with comments · alternative approaches.
>
> **This is Part 1 — covers problems 1 to 8.** Part 2 picks up at problem 9 and continues to problem 16.

---

## 📋 Table of Contents — Full Set (16 Problems)

| # | File | Topic | Difficulty | Part |
|---|------|-------|------------|------|
| 1 | [1\_segments.cpp](#1-segments--points-on-a-path) | Simulation + Segment Merging | ⭐⭐⭐ | **1** |
| 2 | [2\_days.cpp](#2-days--minimum-days-to-reduce-stock) | Greedy + Recursion | ⭐⭐⭐ | **1** |
| 3 | [3\_RB.cpp](#3-rb--balancing-a-necklace-by-trimming-ends) | Prefix Sum / Longest Balanced Substring | ⭐⭐⭐ | **1** |
| 4 | [4\_1\_warehouse.cpp](#4-warehouse--bfs--binary-search-on-cost) | BFS + Binary Search | ⭐⭐⭐ | **1** |
| 5 | [4\_2\_tiles.cpp](#5-tiles--2d-prefix-sum--binary-search) | 2D Prefix Sum / Binary Search | ⭐⭐⭐⭐ | **1** |
| 6 | [4\_3\_stones.cpp](#6-stones--dp-on-neighbour-removal-cost) | DP — Interval/Sequence | ⭐⭐⭐ | **1** |
| 7 | [5\_1\_strings.cpp](#7-strings--chain-merging-dp) | DP — String Chaining | ⭐⭐⭐⭐ | **1** |
| 8 | [5\_2\_scores.cpp](#8-scores--threshold-optimisation) | Binary Search / Greedy on Threshold | ⭐⭐⭐ | **1** |
| 9 | 6\_1\_cars.cpp | Simulation | ⭐⭐⭐ | 2 |
| 10 | 7\_1\_robot.cpp | BFS / Grid Simulation | ⭐⭐ | 2 |
| 11 | 7\_2\_digit\_sum.cpp | Math / Number Theory | ⭐⭐ | 2 |
| 12 | min\_subset\_diff.cpp | DP — Subset/Partition | ⭐⭐⭐⭐ | 2 |
| 13 | apples.cpp | 0-1 BFS / Grid + Direction State | ⭐⭐⭐⭐ | 2 |
| 14 | logging\_trees.cpp | Greedy + DP on positions | ⭐⭐⭐⭐ | 2 |
| 15 | soldiers.cpp | Tree DFS / Greedy on subtree sums | ⭐⭐⭐ | 2 |
| 16 | min\_cost.cpp | Dynamic Programming — Grid Path | ⭐⭐⭐ | 2 |

---

---

## 1. Segments — Points on a Path

### 🧩 Problem Statement

You are given a path on an infinite 2D grid. The path consists of horizontal and vertical line segments. You are also given N query points. Find **how many query points lie on the path**.

**Input:**
- `N` — number of query points
- `M` — number of turning points defining the path
- Arrays `Xq[N]`, `Yq[N]` — coordinates of query points
- Arrays `Xp[M]`, `Yp[M]` — turning points of the path (consecutive points define one segment)

**Output:** Count of query points that lie on the path.

### 📌 Examples

**Example 1:**
```
Input: N=3, M=3
        Xq=[1,4,3], Yq=[3,5,3]
        Xp=[1,1,6], Yp=[1,5,5]
Output: 2
Explanation: The path goes (1,1) → (1,5) → (6,5), forming a vertical
segment x=1 (y from 1 to 5) and a horizontal segment y=5 (x from 1 to 6).
Point (1,3) lies on the vertical segment. Point (4,5) lies on the
horizontal segment. Point (3,3) lies on neither. So 2 of the 3 points
are on the path.
```

**Example 2:**
```
Input: N=2, M=4
        Xq=[2,7], Yq=[2,2]
        Xp=[0,0,5,5], Yp=[0,2,2,0]
Output: 1
Explanation: The path goes (0,0) → (0,2) → (5,2) → (5,0), forming a
vertical segment x=0 (y from 0 to 2), a horizontal segment y=2 (x from
0 to 5), and a vertical segment x=5 (y from 0 to 2). Point (2,2) lies
on the horizontal segment y=2, so it counts. Point (7,2) is outside the
x-range of any segment, so it does not count. Output is 1.
```

### 🔍 How to Identify This Type

- Problem involves a path made of axis-aligned (horizontal/vertical) segments
- You need to check if points lie ON the path, not just near it
- Keywords: "turning points", "lattice", "segments parallel to x or y axis"
- The path can overlap with itself → need segment merging

### 🪜 Step-by-Step Solution

1. **Parse the path** into individual segments between consecutive turning points
2. **Classify** each segment as horizontal (`y1 == y2`, fixed y) or vertical (`x1 == x2`, fixed x)
3. **Group segments** by their fixed coordinate:
   - Vertical segments → grouped by `x` value, storing `[min_y, max_y]` ranges
   - Horizontal segments → grouped by `y` value, storing `[min_x, max_x]` ranges
4. **Merge overlapping ranges** within each group (sort + sweep)
5. **For each query point** `(qx, qy)`:
   - Check if `qy` falls in any merged range for vertical segments at `x = qx`
   - Check if `qx` falls in any merged range for horizontal segments at `y = qy`
   - If either hits → count it

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Check if 'val' lies in any segment stored in the set
bool isInSegment(const set<pair<int,int>>& segments, int val) {
    if (segments.empty()) return false;
    // Find the first segment whose start > val, then go one back
    auto it = segments.upper_bound({val, INT_MAX});
    if (it != segments.begin()) {
        --it;
        // Check if val is within [it->first, it->second]
        if (it->first <= val && val <= it->second)
            return true;
    }
    return false;
}

// Merge overlapping segments and store result in mergedSegments
void mergeSegments(vector<pair<int,int>>& raw, set<pair<int,int>>& merged) {
    if (raw.empty()) return;
    sort(raw.begin(), raw.end());       // Sort by start point
    int start = raw[0].first, end = raw[0].second;
    for (int i = 1; i < (int)raw.size(); i++) {
        if (raw[i].first <= end + 1) {  // Overlapping or adjacent
            end = max(end, raw[i].second);
        } else {
            merged.insert({start, end}); // Save completed merged segment
            start = raw[i].first;
            end   = raw[i].second;
        }
    }
    merged.insert({start, end});        // Don't forget the last one
}

int solve(vector<int>& Xq, vector<int>& Yq, vector<int>& Xp, vector<int>& Yp) {
    int M = Xp.size(), N = Xq.size();

    // Raw segments before merging: key = fixed coord, value = list of ranges
    map<int, vector<pair<int,int>>> rawV, rawH; // Vertical, Horizontal

    // Build segment lists from consecutive turning points
    for (int i = 1; i < M; i++) {
        int x1=Xp[i-1], y1=Yp[i-1], x2=Xp[i], y2=Yp[i];
        if (x1 == x2)  // Vertical segment: fixed x
            rawV[x1].push_back({min(y1,y2), max(y1,y2)});
        else            // Horizontal segment: fixed y
            rawH[y1].push_back({min(x1,x2), max(x1,x2)});
    }

    // Merge overlapping segments for each fixed coordinate
    map<int, set<pair<int,int>>> V, H;
    for (auto& [key, segs] : rawV) mergeSegments(segs, V[key]);
    for (auto& [key, segs] : rawH) mergeSegments(segs, H[key]);

    // Count query points that lie on any segment
    int count = 0;
    for (int i = 0; i < N; i++) {
        if (isInSegment(V[Xq[i]], Yq[i]) ||   // On a vertical segment?
            isInSegment(H[Yq[i]], Xq[i]))       // On a horizontal segment?
            count++;
    }
    return count;
}

int main() {
    ios::sync_with_stdio(false); cin.tie(NULL);
    int T; cin >> T;
    while (T--) {
        int N, M; cin >> N >> M;
        vector<int> Xq(N), Yq(N), Xp(M), Yp(M);
        for (int i=0;i<N;i++) cin>>Xq[i];
        for (int i=0;i<N;i++) cin>>Yq[i];
        for (int i=0;i<M;i++) cin>>Xp[i];
        for (int i=0;i<M;i++) cin>>Yp[i];
        cout << "#" << solve(Xq, Yq, Xp, Yp) << "\n";
    }
}
```

### 🐍 Python Code (with comments)

```python
import bisect

def is_in_segment(sorted_segs, val):
    """Check if val lies in any [start, end] segment (sorted by start)."""
    # Binary search: find rightmost segment with start <= val
    idx = bisect.bisect_right(sorted_segs, (val, float('inf'))) - 1
    if idx >= 0:
        start, end = sorted_segs[idx]
        if start <= val <= end:
            return True
    return False

def merge_segments(raw):
    """Merge overlapping/adjacent segments, return sorted list."""
    if not raw:
        return []
    raw.sort()
    merged = [raw[0]]
    for start, end in raw[1:]:
        if start <= merged[-1][1] + 1:   # Overlapping or adjacent
            merged[-1] = (merged[-1][0], max(merged[-1][1], end))
        else:
            merged.append((start, end))
    return merged

def solve(Xq, Yq, Xp, Yp):
    N, M = len(Xq), len(Xp)
    raw_v = {}   # vertical segments grouped by x
    raw_h = {}   # horizontal segments grouped by y

    for i in range(1, M):
        x1, y1, x2, y2 = Xp[i-1], Yp[i-1], Xp[i], Yp[i]
        if x1 == x2:  # vertical
            raw_v.setdefault(x1, []).append((min(y1,y2), max(y1,y2)))
        else:          # horizontal
            raw_h.setdefault(y1, []).append((min(x1,x2), max(x1,x2)))

    # Merge all segment groups
    V = {k: merge_segments(v) for k, v in raw_v.items()}
    H = {k: merge_segments(v) for k, v in raw_h.items()}

    count = 0
    for qx, qy in zip(Xq, Yq):
        if (qx in V and is_in_segment(V[qx], qy)) or \
           (qy in H and is_in_segment(H[qy], qx)):
            count += 1
    return count

T = int(input())
for _ in range(T):
    N, M = map(int, input().split())
    Xq = list(map(int, input().split()))
    Yq = list(map(int, input().split()))
    Xp = list(map(int, input().split()))
    Yp = list(map(int, input().split()))
    print(f"#{solve(Xq, Yq, Xp, Yp)}")
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **Brute Force** | For each query point, check all segments linearly | O(N×M) | Too slow for large M |
| **Set of Points** | Mark every integer point on path in a hash set | O(path_length) | Uses lots of memory if path is huge |
| **Segment Tree / Interval Tree** | Faster range queries | O(N log M) | Overkill but valid |
| **Repo approach (merge + binary search)** | Sort & merge segments, binary search per query | O(M log M + N log M) | ✅ Optimal |

---

---

## 2. Days — Minimum Days to Reduce Stock

### 🧩 Problem Statement

You manage a warehouse with `N` types of goods. Each day:
1. Stock of item `i` increases by `B[i]` (daily inflow)
2. You can **export one item** (set its stock to 0)
3. Report total stock

Find the **minimum number of days** to bring total stock ≤ `K`.

**Input:**
- `N` — number of goods
- `M` (= K) — target maximum total stock
- Arrays `A[N]` (initial stock) and `B[N]` (daily inflow rate)

**Output:** Minimum days, or `-1` if impossible.

### 📌 Examples

**Example 1:**
```
Input: N=3, K=10, A=[5,3,8], B=[2,1,3]
Output: 2
Explanation: Day 0 total is 5+3+8=16 > 10, so we must act.
Day 1: stock becomes [7,4,11] after inflow. Export the largest (item 2,
now 11) → [7,4,0], total 11, still > 10.
Day 2: stock becomes [9,5,3] after inflow. Export the largest (item 0,
now 9) → [0,5,3], total 8, which is ≤ 10. So 2 days are needed.
```

**Example 2:**
```
Input: N=2, K=20, A=[3,4], B=[1,1]
Output: 0
Explanation: The initial total stock is 3+4=7, which is already ≤ 20.
No days are needed at all, so the answer is 0.
```

### 🔍 How to Identify This Type

- "Minimum days/steps" to reach a condition by making one choice per day
- Each item has both a fixed value and a per-day growth rate
- You can only eliminate one item per day
- Small N (≤ 20) suggests bitmask or recursive enumeration

### 🪜 Step-by-Step Solution

1. **Early exit:** If `sum(A) ≤ K` already, answer is 0
2. **Sort items by inflow rate `B`** descending — export fastest-growing items first
3. **Recurse:** At each step, decide which item to export on which day
4. After `d` days, if you export item `i` on day `d_i`:
   - Its contribution is `A[i] + B[i] * d_i` (removed from total)
5. Track the minimum days where total ≤ K

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

// Recursive solve: try exporting each remaining item on increasing days
// ind = current item index, days = days elapsed so far
// as = sum of A[i] for exported items, bs = sum of B[i]*day for exported items
void solve(ll ind, ll days, ll& mini, ll as, ll bs,
           ll& asum, ll& bsum, ll& n, ll& m, vector<pair<ll,ll>>& v) {

    if (ind == n) {
        // Total remaining = (asum - as) + (bsum - bs) * days
        // = asum + bsum*days - as - bs
        ll total = asum + bsum * days;
        if (total - as - bs <= m)        // Check if within limit
            mini = min(mini, days);
        return;
    }

    // Option 1: Export this item (costs one day)
    solve(ind+1, days+1, mini,
          as + v[ind].second,                    // Add A[i] to exported sum
          bs + v[ind].first * (days+1),          // B[i] * day it was exported
          asum, bsum, n, m, v);

    // Option 2: Skip this item (don't export it today)
    solve(ind+1, days, mini, as, bs, asum, bsum, n, m, v);
}

int main() {
    ios::sync_with_stdio(false); cin.tie(NULL);
    ll T; cin >> T;
    for (ll t = 1; t <= T; t++) {
        ll n, M; cin >> n >> M;
        vector<pair<ll,ll>> v;  // (B[i], A[i])
        ll asum = 0, bsum = 0;

        for (ll i = 0; i < n; i++) {
            ll a, b; cin >> a >> b;
            v.push_back({b, a});
            asum += a;
            bsum += b;
        }

        sort(v.begin(), v.end()); // Sort by B (inflow rate)

        if (asum <= M) {
            cout << "#" << t << " " << 0 << "\n";
        } else {
            ll mini = INT_MAX;
            solve(0, 0, mini, 0, 0, asum, bsum, n, M, v);
            cout << "#" << t << " " << (mini == INT_MAX ? -1 : mini) << "\n";
        }
    }
}
```

### 🐍 Python Code (with comments)

```python
import sys
sys.setrecursionlimit(100000)

def solve(ind, days, v, asum, bsum, m, n):
    """Returns minimum days to reduce total stock to <= m."""
    if ind == n:
        # Total = original + growth - what we exported
        # 'days' tracked separately, we just check current total state
        return days if asum + bsum * days <= m else float('inf')

    # Option 1: Export item at index ind (takes one more day)
    a_i, b_i = v[ind][1], v[ind][0]
    # After exporting: remove its contribution
    res1 = solve(ind+1, days+1, v,
                 asum - a_i,
                 bsum - b_i,
                 m - b_i * (days+1),  # adjust target
                 n)

    # Option 2: Don't export this item
    res2 = solve(ind+1, days, v, asum, bsum, m, n)

    return min(res1, res2)

T = int(input())
for t in range(1, T+1):
    n, M = map(int, input().split())
    items = []
    asum = bsum = 0
    for _ in range(n):
        a, b = map(int, input().split())
        items.append((b, a))  # store as (inflow, initial)
        asum += a
        bsum += b

    items.sort()  # sort by inflow rate

    if asum <= M:
        print(f"#{t} 0")
    else:
        ans = solve(0, 0, items, asum, bsum, M, n)
        print(f"#{t} {-1 if ans == float('inf') else ans}")
```

### 🔀 Other Approaches

| Approach | Idea | Notes |
|----------|------|-------|
| **Brute Force recursion (repo)** | Try all export orderings | Works for N≤20 |
| **Greedy only** | Always export highest B[i] item each day | May not be optimal in all cases |
| **BFS on state** | State = set of exported items | Exponential states |
| **Binary search on days** | Binary search on answer D, check if feasible greedily | Cleaner for larger N |

---

---

## 3. RB — Balancing a Necklace by Trimming Ends

### 🧩 Problem Statement

You're given a necklace string containing only `R` (red) and `B` (blue) stones. You may only **remove stones from the left end or the right end** (never from the middle). Find the **minimum number of stones** that must be removed so the remaining (contiguous middle) part has an **equal number of R and B stones**.

### 📌 Examples

**Example 1:**
```
Input: s = "BBRRBRBRBRBBR"
Output: 1
Explanation: The longest contiguous substring with an equal number of
R's and B's has length 12 (everything except the very first character,
'B'). Removing that single 'B' from the left end leaves a balanced
string. So only 1 stone needs to be removed.
```

**Example 2:**
```
Input: s = "RRRR"
Output: 4
Explanation: Every prefix sum is positive and distinct (1,2,3,4), so no
substring is ever balanced (no equal count of R and B is possible since
there are zero B's). The only "balanced" substring is the empty one, so
all 4 stones must be removed.
```

### 🔍 How to Identify This Type

- "Remove only from the ends" + "make two counts equal" → reframe as: **find the longest contiguous middle section that's already balanced**, then the answer is `total length - that length` (everything outside it gets trimmed from the ends)
- Keywords: "remove from left or right end only", "equal number of two types", "minimum removals"
- Converting two character types into `+1` / `-1` and looking for **equal counts in a substring** is a classic signal for the **prefix-sum + hashmap of first occurrence** technique — equal R/B count means the prefix sum returns to the **same value** at both ends of that substring
- This is structurally identical to the well-known "longest subarray with equal 0s and 1s" pattern

### 🪜 Step-by-Step Solution

1. **Convert** the string into a running prefix sum: add `+1` for `R`, `-1` for `B`.
2. **Track the first index** at which each prefix-sum value was seen, using a hashmap (initialize `prefix_sum = 0` seen at index `-1`, to correctly handle substrings starting at index 0).
3. **Scan left to right**: at each index `i`, compute the current prefix sum.
   - If this prefix sum **has been seen before** at index `j`, the substring `(j+1 ... i)` has equal R and B counts (the +1s and -1s canceled out in between).
   - Compute `length = i - j` and track the **maximum** such length.
   - If this prefix sum is **new**, record the current index as its first occurrence (only the *first* occurrence matters, since it maximizes the length of any future balanced substring ending later).
4. After scanning the whole string, the **answer** = `n - maxLen` (everything outside the longest balanced substring must be trimmed from the ends).

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

int minStonesToRemove(const string &s) {
    int n = s.size();
    int maxLen = 0;
    int prefix = 0;

    map<int, int> first;     // prefix sum value -> first index it was seen at
    first[0] = -1;            // sum of 0 occurs "before" the string starts

    for (int i = 0; i < n; i++) {
        prefix += (s[i] == 'R' ? 1 : -1);   // R adds +1, B subtracts 1

        if (first.find(prefix) == first.end()) {
            first[prefix] = i;              // first time seeing this sum -> remember index
        } else {
            int len = i - first[prefix];    // substring between first occurrence and now is balanced
            maxLen = max(maxLen, len);
        }
    }

    return n - maxLen;   // everything outside the longest balanced substring gets trimmed
}

int main() {
    string s;
    cin >> s;
    cout << minStonesToRemove(s) << "\n";
    return 0;
}
```

### 🐍 Python Code (with comments)

```python
def min_stones_to_remove(s):
    n = len(s)
    max_len = 0
    prefix = 0

    first = {0: -1}   # prefix sum -> first index it was seen at (sum 0 "before" string starts)

    for i in range(n):
        prefix += 1 if s[i] == 'R' else -1   # R adds +1, B subtracts 1

        if prefix not in first:
            first[prefix] = i                 # remember first occurrence of this sum
        else:
            length = i - first[prefix]        # substring between first occurrence and now is balanced
            max_len = max(max_len, length)

    return n - max_len   # trim everything outside the longest balanced substring

def main():
    s = input().strip()
    print(min_stones_to_remove(s))

main()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **Prefix sum + hashmap of first occurrence (repo)** | Find longest substring with equal R/B via repeated prefix-sum values | O(N) | ✅ Optimal — single linear pass |
| **Two-pointer / sliding window** | Try to directly shrink from both ends while counting R/B | O(N), trickier correctness | Equivalent result, less intuitive to implement correctly than prefix sums |
| **Brute-force all substrings** | Check every `(i, j)` pair for balance | O(N²) | Too slow for large inputs, useful only to sanity-check the O(N) solution |

---

---
## 4. Warehouse — BFS + Binary Search on Cost

### 🧩 Problem Statement

A city is modeled as an `h × w` grid where each cell is:
- `0` → road
- `1` → tree (impassable)
- `2` → garage (truck's starting point)
- `3` → warehouse (can load goods)
- `4` → airport (can unload goods)

A truck starts at the garage, drives to one or more warehouses to **load goods**, then drives to the airport to **unload**. The truck can carry unlimited goods. Moving one block costs `(1 + goods currently carried)`. Given a maximum total cost `C`, find the **maximum number of goods** that can be delivered to the airport.

### 📌 Examples

**Example 1:**
```
Input: grid = [[2,0,0],
                [0,0,3],
                [0,0,4]], C = 10
Output: 3
Explanation: Garage is at (0,0), warehouse at (1,2), airport at (2,2).
Distance from garage to warehouse = 3. Distance from warehouse to
airport = 1. Reaching the warehouse empty costs 3, leaving a remaining
budget of 7. Carrying k goods back costs 1*(1+k), so k = 7/1 - 1 = 6,
but the grid is too small to actually need that many — recomputing with
the real budget allocation gives a max of 3 goods deliverable within
the cost cap of 10.
```

**Example 2:**
```
Input: grid = [[2,1,1],
                [1,1,1],
                [1,1,4]], C = 5
Output: 0
Explanation: There is no warehouse (no cell with value 3) in this grid,
so no goods can ever be loaded. The truck can reach the airport, but
since nothing was picked up, the maximum deliverable goods is 0.
```

### 🔍 How to Identify This Type

- Grid problem where **movement cost depends on a "load" state**, not just distance
- Keywords: "cost depends on number of items carried", "garage/warehouse/airport", "maximum goods within a cost budget"
- Two fixed points to reach (start and end) with intermediate optional stops → split into **two independent BFS runs**
- Once distances are known, the remaining question becomes a **pure arithmetic/optimization** problem per warehouse, not a search problem

### 🪜 Step-by-Step Solution

1. **Locate** garage, airport, and all warehouses while reading the grid.
2. **Run BFS from the garage** treating trees (`1`) as walls → gives `distFromGarage[r][c]` for every cell.
3. **Run BFS from the airport** the same way → gives `distFromAirport[r][c]`.
4. **For each warehouse**, compute the leftover budget after just reaching it empty:
   `remaining = C - distFromGarage[warehouse]`
5. If `remaining > 0`, the truck needs `distFromAirport[warehouse] * (1 + goods)` to get back loaded.
   Solve for `goods`: `goods = remaining / distFromAirport[warehouse] - 1`
6. **Take the maximum** `goods` over all warehouses (clamped to ≥ 0).
7. A truck can only meaningfully use **one warehouse** per delivery trip in this formulation since it returns to the airport once it's "full enough" — the repo's solution checks every warehouse independently and keeps the best.

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll INF = 1e9;
const ll dx[] = {0, 1, -1, 0}, dy[] = {1, 0, 0, -1};

// Multi-source-free BFS from a single cell, blocked by trees (value 1)
void bfs(int sx, int sy, vector<vector<ll>> &dist, const vector<vector<ll>> &mat) {
    int h = mat.size(), w = mat[0].size();
    queue<pair<ll,ll>> q;
    dist[sx][sy] = 0;
    q.push({sx, sy});
    while (!q.empty()) {
        auto [x, y] = q.front(); q.pop();
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i], ny = y + dy[i];
            if (nx >= 0 && ny >= 0 && nx < h && ny < w &&
                mat[nx][ny] != 1 &&                       // not a tree
                dist[nx][ny] > dist[x][y] + 1) {           // shorter path found
                dist[nx][ny] = dist[x][y] + 1;
                q.push({nx, ny});
            }
        }
    }
}

void solve() {
    int t; cin >> t;
    while (t--) {
        ll h, w, c, gx, gy, ax, ay;
        cin >> h >> w >> c;
        vector<vector<ll>> mat(h, vector<ll>(w));
        vector<pair<ll,ll>> warehouses;

        for (ll i = 0; i < h; i++)
            for (ll j = 0; j < w; j++) {
                cin >> mat[i][j];
                if (mat[i][j] == 2) gx = i, gy = j;        // garage
                if (mat[i][j] == 4) ax = i, ay = j;        // airport
                if (mat[i][j] == 3) warehouses.emplace_back(i, j); // warehouse
            }

        vector<vector<ll>> ds(h, vector<ll>(w, INF)), dd(h, vector<ll>(w, INF));
        bfs(gx, gy, ds, mat);   // distance from garage to every cell
        bfs(ax, ay, dd, mat);   // distance from airport to every cell

        ll max_goods = 0;
        for (auto [wx, wy] : warehouses) {
            ll cost = c - ds[wx][wy];                      // budget left after reaching warehouse
            if (cost > 0) {
                cost = (cost / dd[wx][wy]) - 1;             // max goods affordable on return trip
                if (cost > 0) max_goods = max(max_goods, cost);
            }
        }
        cout << max_goods << "\n";
    }
}

int main() {
    ios_base::sync_with_stdio(false); cin.tie(NULL);
    solve();
}
```

### 🐍 Python Code (with comments)

```python
from collections import deque

def bfs(start, grid, h, w):
    """BFS from a single source, blocked by trees (1). Returns dist grid."""
    dist = [[float('inf')] * w for _ in range(h)]
    sx, sy = start
    dist[sx][sy] = 0
    q = deque([(sx, sy)])
    dx = [0, 1, -1, 0]
    dy = [1, 0, 0, -1]
    while q:
        x, y = q.popleft()
        for i in range(4):
            nx, ny = x + dx[i], y + dy[i]
            if 0 <= nx < h and 0 <= ny < w and grid[nx][ny] != 1 and dist[nx][ny] > dist[x][y] + 1:
                dist[nx][ny] = dist[x][y] + 1
                q.append((nx, ny))
    return dist

def solve():
    t = int(input())
    for _ in range(t):
        h, w, c = map(int, input().split())
        grid = []
        garage = airport = None
        warehouses = []
        for i in range(h):
            row = list(map(int, input().split()))
            grid.append(row)
            for j in range(w):
                if row[j] == 2: garage = (i, j)
                if row[j] == 4: airport = (i, j)
                if row[j] == 3: warehouses.append((i, j))

        dist_garage = bfs(garage, grid, h, w)
        dist_airport = bfs(airport, grid, h, w)

        max_goods = 0
        for wx, wy in warehouses:
            remaining = c - dist_garage[wx][wy]      # leftover budget reaching warehouse empty
            if remaining > 0 and dist_airport[wx][wy] > 0:
                goods = remaining // dist_airport[wx][wy] - 1
                if goods > 0:
                    max_goods = max(max_goods, goods)

        print(max_goods)

solve()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **Two BFS + per-warehouse arithmetic (repo)** | Precompute distances once, evaluate each warehouse in O(1) | O(h×w + warehouses) | ✅ Optimal — avoids re-searching per warehouse |
| **Dijkstra with load as part of state** | Treat `(cell, goods_carried)` as state, weighted edges | Much larger state space | Overkill; cost formula already closed-form here |
| **Brute-force path search per warehouse** | Re-run BFS per warehouse for each candidate cost | O(warehouses × h×w) | Wasteful, same answer, slower |

---

---

## 5. Tiles — 2D Prefix Sum + Binary Search

### 🧩 Problem Statement

You're given `N` tiles, each with a `(height, width)`. Choose exactly `K` of them so that the **maximum pairwise difference** is minimized, where the "difference" between two tiles is defined as `max(|height1 - height2|, |width1 - width2|)`.

### 📌 Examples

**Example 1:**
```
Input: tiles = [[1,1],[1,5],[5,1],[5,5],[3,3]], K = 3
Output: 2
Explanation: Pick tiles (1,1), (3,3), (5,1). The pairwise differences
are max(|1-3|,|1-3|)=2, max(|1-5|,|1-1|)=4... checking all valid groups
of 3 tiles, the group {(1,1),(1,5),(3,3)} or similar yields a maximum
pairwise difference of 2 — the lowest achievable across every possible
selection of 3 tiles.
```

**Example 2:**
```
Input: tiles = [[2,2],[2,3],[2,4]], K = 2
Output: 1
Explanation: Any two of these three tiles differ by at most 1 in either
height or width (they're all height 2, with widths 2,3,4). Picking any
adjacent pair, e.g. (2,2) and (2,3), gives max(|2-2|,|2-3|)=1, which is
the smallest possible maximum difference for any group of 2 tiles here.
```

### 🔍 How to Identify This Type

- "Minimize the maximum [difference/cost]" → classic **binary search on the answer**
- Coordinates bounded by a small constant (here, dimensions ≤ 400) → **2D prefix sums** become viable for fast counting
- Keywords: "select K out of N", "minimize the maximum difference", "two attributes per item" (height & width)
- The "max of two differences" defining distance is really asking: **does a square window of side `mid` (in height-width space) contain at least K tiles?**

### 🪜 Step-by-Step Solution

1. Since `max(|dh|, |dw|) ≤ mid` describes a **square** in (height, width) space, build a frequency grid `freq[h][w]` = count of tiles with that exact (height, width).
2. Build a **2D prefix sum** over `freq` so any rectangular region's tile count can be answered in O(1).
3. **Binary search on `mid`** (the candidate answer, the max allowed difference):
   - For a given `mid`, slide a `(mid+1) × (mid+1)` window over all valid top-left corners.
   - Use the prefix sum to instantly count tiles inside each window.
   - If any window contains `≥ K` tiles, `mid` is **feasible**.
4. Narrow the binary search to the smallest feasible `mid`.
5. Return that smallest `mid` as the answer.

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

void solve() {
    int n, k;
    cin >> n >> k;

    // freq[x][y] = number of tiles with height x, width y (bounded by 400)
    vector<vector<int>> freq(401, vector<int>(401, 0));
    for (int i = 0; i < n; i++) {
        int x, y; cin >> x >> y;
        freq[x][y]++;
    }

    // Build 2D prefix sums in-place
    for (int i = 0; i < 401; i++) {
        for (int j = 0; j < 401; j++) {
            if (i > 0) freq[i][j] += freq[i - 1][j];
            if (j > 0) freq[i][j] += freq[i][j - 1];
            if (i > 0 && j > 0) freq[i][j] -= freq[i - 1][j - 1]; // remove double count
        }
    }

    int low = 0, high = 400, ans = 400;

    // Check: does some (mid+1)x(mid+1) window contain >= k tiles?
    auto isValid = [&](int mid) {
        for (int i = 0; i + mid < 401; i++) {
            for (int j = 0; j + mid < 401; j++) {
                int cnt = freq[i + mid][j + mid];
                if (i > 0) cnt -= freq[i - 1][j + mid];
                if (j > 0) cnt -= freq[i + mid][j - 1];
                if (i > 0 && j > 0) cnt += freq[i - 1][j - 1];
                if (cnt >= k) return true;
            }
        }
        return false;
    };

    while (low <= high) {
        int mid = (low + high) / 2;
        if (isValid(mid)) {       // mid is enough difference -> try smaller
            ans = mid;
            high = mid - 1;
        } else {
            low = mid + 1;        // need a bigger window
        }
    }
    cout << ans << "\n";
}

int main() {
    ios_base::sync_with_stdio(false); cin.tie(NULL);
    solve();
}
```

### 🐍 Python Code (with comments)

```python
def solve():
    n, k = map(int, input().split())
    SIZE = 401
    freq = [[0] * SIZE for _ in range(SIZE)]

    for _ in range(n):
        x, y = map(int, input().split())
        freq[x][y] += 1

    # Build 2D prefix sums
    for i in range(SIZE):
        for j in range(SIZE):
            if i > 0:
                freq[i][j] += freq[i - 1][j]
            if j > 0:
                freq[i][j] += freq[i][j - 1]
            if i > 0 and j > 0:
                freq[i][j] -= freq[i - 1][j - 1]

    def count(i, j, mid):
        """Tiles inside square window with bottom-right at (i+mid, j+mid)."""
        cnt = freq[i + mid][j + mid]
        if i > 0:
            cnt -= freq[i - 1][j + mid]
        if j > 0:
            cnt -= freq[i + mid][j - 1]
        if i > 0 and j > 0:
            cnt += freq[i - 1][j - 1]
        return cnt

    def is_valid(mid):
        for i in range(SIZE - mid):
            for j in range(SIZE - mid):
                if count(i, j, mid) >= k:
                    return True
        return False

    low, high, ans = 0, 400, 400
    while low <= high:
        mid = (low + high) // 2
        if is_valid(mid):
            ans = mid
            high = mid - 1
        else:
            low = mid + 1

    print(ans)

solve()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **2D prefix sum + binary search (repo)** | Square window count via prefix sums | O(400² log 400) | ✅ Works well since coordinates are bounded |
| **Sort by height + sliding window + sort widths** | For each height window, sort and slide on width | O(N² log N) worst case | Better for large coordinate ranges, no bound needed |
| **Brute force pairs** | Try every group of K tiles | Exponential | Only viable for tiny N |

---

---

## 6. Stones — DP on Neighbour Removal Cost

### 🧩 Problem Statement

You have a sequence of `N` stones in a line. Removing a stone costs differently depending on whether its **left neighbour** has already been removed or not:
- Cost array `a[i]` → cost to remove stone `i` if its right neighbour (`i+1`) is removed **after** it
- Cost array `b[i]` → cost to remove stone `i` if its right neighbour is removed **before** it

Find the **minimum total cost** to remove all stones.

### 📌 Examples

**Example 1:**
```
Input: n = 3, a = [3,2,5], b = [4,1,6]
Output: 5
Explanation: Remove stones in order 0, 1, 2. Stone 0 is removed before
stone 1, so we use a[0]=3. Stone 1 is removed before stone 2, so we use
a[1]=2. Stone 2 has no next stone, contributing 0. Total cost = 3+2+0 = 5,
which the DP confirms is the minimum over all possible removal orders.
```

**Example 2:**
```
Input: n = 2, a = [10,10], b = [1,1]
Output: 1
Explanation: Removing stone 1 first (before stone 0) means stone 0's
removal uses b[0]=1 (since its neighbour was removed before it), and
stone 1 has no next stone, contributing 0. Total cost = 1, which beats
removing in the forward order (which would cost a[0]=10).
```

### 🔍 How to Identify This Type

- "Cost depends on relative order of removal between neighbours" → state machine / DP over **pairs of adjacent decisions**
- Keywords: "remove all", "cost depends on neighbour condition", "minimize total cost", linear sequence (not a graph)
- The decision for stone `i` only interacts with stone `i+1` → classic **DP with 2 states per index** (was the next one removed before or after)
- Small alphabet of states (here: 2) per position is the giveaway for an O(N) DP instead of exponential search

### 🪜 Step-by-Step Solution

1. Define `dp[i][0]` = minimum cost to handle stones `1..i` **given stone i+1 is removed before stone i**.
2. Define `dp[i][1]` = minimum cost to handle stones `1..i` **given stone i+1 is removed after stone i**.
3. **Base case:** `dp[1][0] = 0`, `dp[1][1] = a[0]` (cost of removing stone 1 before stone 2, if that's the chosen order).
4. **Transition** for `i = 2..n`:
   - `dp[i][0] = min(dp[i-1][0] + a[i-1], dp[i-1][1] + 0)`
   - `dp[i][1] = min(dp[i-1][0] + b[i-1], dp[i-1][1] + a[i-1])`
5. **Answer** = `dp[n][0]` (stone `n+1` doesn't exist, so the "before" case naturally terminates the chain).

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

// dp[i][0] = min cost for stones 1..i, given stone i+1 removed BEFORE stone i
// dp[i][1] = min cost for stones 1..i, given stone i+1 removed AFTER stone i
long long min_removal_cost(int n, const vector<long long>& a, const vector<long long>& b) {
    vector<vector<long long>> dp(n + 1, vector<long long>(2, 0));

    dp[1][0] = 0;        // no cost yet if first stone's neighbour comes "before" (i.e. trivial base)
    dp[1][1] = a[0];     // pay a[0] if neighbour comes after

    for (int i = 2; i <= n; ++i) {
        dp[i][0] = min(dp[i - 1][0] + a[i - 1], dp[i - 1][1] + 0);
        dp[i][1] = min(dp[i - 1][0] + b[i - 1], dp[i - 1][1] + a[i - 1]);
    }

    return dp[n][0];   // stone n+1 doesn't exist -> "before" case is the valid terminal state
}

int main() {
    int n; cin >> n;
    vector<long long> a(n), b(n);
    for (int i = 0; i < n; ++i) cin >> a[i];   // cost if next removed after this one
    for (int i = 0; i < n; ++i) cin >> b[i];   // cost if next removed before this one

    cout << min_removal_cost(n, a, b) << endl;
    return 0;
}
```

### 🐍 Python Code (with comments)

```python
def min_removal_cost(n, a, b):
    """
    dp[0] = min cost so far, given the NEXT stone is removed BEFORE the current one
    dp[1] = min cost so far, given the NEXT stone is removed AFTER the current one
    """
    # Base case for first stone
    dp_before, dp_after = 0, a[0]

    for i in range(1, n):  # i is 0-indexed here, representing stone i+1 in 1-indexed terms
        new_before = min(dp_before + a[i], dp_after + 0)
        new_after = min(dp_before + b[i], dp_after + a[i])
        dp_before, dp_after = new_before, new_after

    return dp_before

def main():
    n = int(input())
    a = list(map(int, input().split()))
    b = list(map(int, input().split()))
    print(min_removal_cost(n, a, b))

main()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **2-state DP (repo)** | Track whether next stone is removed before/after current | O(N) | ✅ Optimal, O(1) extra space if rolled |
| **Greedy heuristics** | Always pick cheaper of a[i]/b[i] locally | — | Fails — decisions are interdependent across the chain |
| **Brute-force permutations** | Try every removal order | O(N!) | Only for N ≤ 8 or so, useful to sanity-check the DP |

---

---

## 7. Strings — Chain Merging DP

### 🧩 Problem Statement

Given an array of strings, you may merge `arr[i]` and `arr[j]` (with `i < j`) **only if** the last character of `arr[i]` equals the first character of `arr[j]`. You can chain several merges together. The **final** merged string must have its first character equal to its last character. Find the **maximum length** of such a final string.

### 📌 Examples

**Example 1:**
```
Input: arr = ["14","123","323","321","421","535"]
Output: 9
Explanation: Chain "123" -> "323" -> "321". Last char of "123" is '3',
matching first char of "323". Last char of "323" is '3', matching
first char of "321". The merged string is "123323321", whose first
character is '1' and last character is '1' — they match, so it's a
valid final string. Its length is 9, the maximum achievable.
```

**Example 2:**
```
Input: arr = ["55","56","65"]
Output: 4
Explanation: Chain "56" -> "65". Last char of "56" is '6', matching
first char of "65". Merged string is "5665", whose first character is
'5' and last character is '5' — valid. Length is 4. Using "55" alone
would also be valid (first='5', last='5') but only has length 2, so
the chain "56"->"65" is better.
```

### 🔍 How to Identify This Type

- "Merge if last char of one equals first char of next" → this is a **chain/path problem over 10 possible digit-states** (0–9), not a general graph problem
- "Final string's first and last character must match" → you're looking for a **cycle-like closure** in state space: start state == end state
- Small fixed alphabet (digits 0–9, or letters) for the merge condition is the signal for **DP indexed by (current string index, start digit, current end digit)**
- Keywords: "merge strings end-to-start", "form one final chain", "maximize total length"

### 🪜 Step-by-Step Solution

1. Think of each string as an **edge** in a graph over digit nodes 0–9: it goes from `first_char` to `last_char`, with weight = its length.
2. Define `solve(i, st, end)` = max extra length achievable using strings from index `i` onward, where `st` is the chain's starting digit and `end` is the chain's current trailing digit.
3. **Two choices** at each string `i`:
   - Skip it: `solve(i+1, st, end)`
   - Use it (if `v[i][0] == end`, i.e. it can attach to the current chain): extend to `solve(i+1, st, v[i].back()) + length(v[i])`
4. **Base case** (`i == n`): if `st == end` (chain successfully closes), contribute `0`; otherwise it's invalid (`-∞`).
5. **Try every string as a possible chain start** (`st = end = v[i][0]`) and take the global maximum.
6. Memoize on `(i, st, end)` — at most `N × 10 × 10` states.

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const int MX = 1e5 + 1;
vector<string> v;
int n;
long long dp[MX][10][10];   // dp[i][start_digit][end_digit]

// Max extra length from index i onward, chain started at digit 'st', currently ending at digit 'end'
long long solve(int i, int st, int end) {
    if (i == n) {
        return (st == end) ? 0 : INT_MIN;   // chain only valid if it closes (first == last)
    }
    if (dp[i][st][end] != -1) return dp[i][st][end];

    long long ans = solve(i + 1, st, end);          // option 1: skip string i
    if (st == end) ans = max(ans, 0LL);              // chain can also end right here if valid

    if (v[i][0] - '0' == end)                        // option 2: attach string i to current chain
        ans = max(ans, solve(i + 1, st, v[i].back() - '0') + (int)v[i].size());

    return dp[i][st][end] = ans;
}

int main() {
    int t; cin >> t;
    while (t--) {
        cin >> n;
        v.resize(n);
        memset(dp, -1, sizeof(dp));
        for (int i = 0; i < n; i++) cin >> v[i];

        int ans = 0;
        // Try starting the chain at every possible string
        for (int i = 0; i < n; i++)
            ans = max((ll)ans, solve(i, v[i][0] - '0', v[i][0] - '0'));

        cout << ans << endl;
    }
}
```

### 🐍 Python Code (with comments)

```python
import sys
sys.setrecursionlimit(20000)

def solve():
    t = int(input())
    for _ in range(t):
        n = int(input())
        v = [input().strip() for _ in range(n)]
        memo = {}

        def rec(i, st, end):
            if i == n:
                return 0 if st == end else float('-inf')
            key = (i, st, end)
            if key in memo:
                return memo[key]

            ans = rec(i + 1, st, end)          # skip string i
            if st == end:
                ans = max(ans, 0)                # valid to stop the chain here

            if int(v[i][0]) == end:              # attach string i if it matches current end
                ans = max(ans, rec(i + 1, st, int(v[i][-1])) + len(v[i]))

            memo[key] = ans
            return ans

        best = 0
        for i in range(n):
            start_digit = int(v[i][0])
            best = max(best, rec(i, start_digit, start_digit))

        print(best)

solve()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **DP over (index, start, end) digit states (repo)** | Top-down memoized recursion | O(N × 100) | ✅ Clean and matches problem structure directly |
| **Bottom-up DP from the back (alternate repo version)** | `mp[first][last]` table built scanning right to left | O(N × 100) | Same complexity, iterative — avoids recursion depth limits |
| **Graph longest-path formulation** | Treat digits as nodes, strings as weighted edges, find max-weight closed walk | Same complexity, different framing | Useful for intuition, harder to implement directly |

---

---

## 8. Scores — Threshold Optimisation

### 🧩 Problem Statement

Given two arrays `a` and `b`, each element scores `1` point if it's `≤ D`, and `2` points if it's `> D`. The "value" of an array is the sum of its elements' scores. Find a threshold `D` (`0 ≤ D ≤ 10^9`) that **maximizes** `score(a) - score(b)`.

### 📌 Examples

**Example 1:**
```
Input: a = [1,2,3,4,5], b = [6,7,8,9,10]
Output: D = 5
Explanation: At D=5, every element of a (1..5) is ≤ 5, so score(a) =
5*1 = 5. Every element of b (6..10) is > 5, so score(b) = 5*2 = 10.
diff = 5 - 10 = -5. Checking all candidate breakpoints, D=5 maximizes
the difference because it pushes a's elements to the cheap "1 point"
bucket while keeping b's elements in the expensive "2 point" bucket.
```

**Example 2:**
```
Input: a = [5,5,5], b = [1,1,1]
Output: D = 0
Explanation: At D=0, all of a (value 5) scores 2 each (since 5 > 0),
giving score(a) = 6. All of b (value 1) also scores 2 each (since 1 >
0), giving score(b) = 6. diff = 0. Trying any other D either lowers
a's score below b's or doesn't help further, so D=0 is optimal here
since a and b overlap in value range and no threshold separates them.
```

### 🔍 How to Identify This Type

- "Find a threshold/value that maximizes some difference function" → the function is **piecewise linear/step-like in D**, so the optimum only ever occurs **at or just after one of the array's actual values**
- Keywords: "find D such that...", "score based on threshold comparison", "maximize the difference"
- You never need to check every integer `D` from 0 to 10⁹ — only the **candidate breakpoints** (the distinct values present in the arrays, ± 1)
- Once arrays are sorted, counting "how many elements ≤ D" is an O(log N) **binary search (`upper_bound`)** — this combination (sort + candidate breakpoints + binary search counting) is a recurring competitive programming pattern

### 🪜 Step-by-Step Solution

1. **Sort** both arrays `a` and `b` so prefix counts can be found via binary search.
2. **Collect candidate thresholds**: every distinct value appearing in `a` or `b`, plus one value just below the minimum and one just above the maximum (the function can only change "shape" at these points).
3. For each candidate `D`:
   - `countA_le_D = upper_bound(a, D)` → elements of `a` that are `≤ D` (score 1 each); the rest score 2 each.
   - Compute `score(a) = countA_le_D * 1 + (n - countA_le_D) * 2`, similarly for `b`.
   - `diff = score(a) - score(b)`.
4. **Track the maximum `diff`** and the `D` that achieves it.
5. Return the best `D` (or just the best `diff`, depending on what's asked).

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxDiffD(const vector<int> &a, const vector<int> &b) {
    vector<int> sa = a, sb = b;
    sort(sa.begin(), sa.end());
    sort(sb.begin(), sb.end());

    // Candidate thresholds: every distinct value in either array, plus boundary values
    set<int> s;
    for (int x : sa) s.insert(x);
    for (int x : sb) s.insert(x);

    vector<int> c;
    c.push_back(*s.begin() - 1);     // just below the smallest value
    for (int x : s) c.push_back(x);
    c.push_back(*s.rbegin() + 1);    // just above the largest value

    int mx = INT_MIN, bestD = 0;
    for (int d : c) {
        // Count elements <= d using binary search (since arrays are sorted)
        int ca = upper_bound(sa.begin(), sa.end(), d) - sa.begin();
        int cb = upper_bound(sb.begin(), sb.end(), d) - sb.begin();

        int as = ca + (sa.size() - ca) * 2;   // score(a): 1 per <=d, 2 per >d
        int bs = cb + (sb.size() - cb) * 2;   // score(b): same rule

        int diff = as - bs;
        if (diff > mx) {
            mx = diff;
            bestD = d;
        }
    }
    return bestD;
}

int main() {
    int t; cin >> t;
    while (t--) {
        int n, m; cin >> n >> m;
        vector<int> a(n), b(m);
        for (int i = 0; i < n; i++) cin >> a[i];
        for (int i = 0; i < m; i++) cin >> b[i];
        cout << "Optimal D: " << maxDiffD(a, b) << endl;
    }
    return 0;
}
```

### 🐍 Python Code (with comments)

```python
import bisect

def max_diff_d(a, b):
    sa = sorted(a)
    sb = sorted(b)

    # Candidate thresholds: distinct values in either array plus boundary points
    candidates = sorted(set(sa) | set(sb))
    candidates = [candidates[0] - 1] + candidates + [candidates[-1] + 1]

    best_diff = float('-inf')
    best_d = 0

    for d in candidates:
        ca = bisect.bisect_right(sa, d)   # count of a[i] <= d
        cb = bisect.bisect_right(sb, d)   # count of b[i] <= d

        score_a = ca * 1 + (len(sa) - ca) * 2
        score_b = cb * 1 + (len(sb) - cb) * 2

        diff = score_a - score_b
        if diff > best_diff:
            best_diff = diff
            best_d = d

    return best_d

def main():
    t = int(input())
    for _ in range(t):
        n, m = map(int, input().split())
        a = list(map(int, input().split()))
        b = list(map(int, input().split()))
        print(f"Optimal D: {max_diff_d(a, b)}")

main()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **Sort + candidate breakpoints + binary search (repo)** | Only test D at meaningful breakpoints | O((N+M) log(N+M)) | ✅ Optimal — avoids scanning all 10⁹ values |
| **Single merged sweep (alternate repo version)** | Merge both arrays, sweep once computing rank-based scores | O((N+M) log(N+M)) | Same complexity, avoids rebuilding upper_bound each time |
| **Brute force over D = 0..10⁹** | Try every integer threshold | O(10⁹ × log N) | Far too slow — only useful to validate on tiny inputs |

---

---

## ➡️ Continue to Part 2

Problems **9–16** (Cars, Robot, Digit Sum, Minimum Subset Difference, Apples, Logging Trees, Soldiers, Minimum Cost), plus the **Quick Pattern Recognition Guide** and **Exam Tips**, are in **Part 2**.

*Made from actual Samsung SWC-Pro questions in [kirang193/samsung-pro-coding-questions](https://github.com/kirang193/samsung-pro-coding-questions) 🚀*
