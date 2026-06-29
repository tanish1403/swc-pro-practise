# 📘 Samsung SWC-Pro — Problem Explanations & Solutions (Part 2 of 2)

> Based on the actual questions from [kirang193/samsung-pro-coding-questions](https://github.com/kirang193/samsung-pro-coding-questions)
> Each problem includes: problem statement · example · how to identify · step-by-step solution · C++ & Python code with comments · alternative approaches.
>
> **This is Part 2 — covers problems 9 to 16.** Part 1 covers problems 1 to 8 (Segments, Days, RB, Warehouse, Tiles, Stones, Strings, Scores).

---

## 📋 Table of Contents — This Part

| # | File | Topic | Difficulty |
|---|------|-------|------------|
| 9 | [6\_1\_cars.cpp](#9-cars--simultaneous-parking-parity--triangular-numbers) | Math — Parity + Triangular Numbers | ⭐⭐⭐ |
| 10 | [7\_1\_robot.cpp](#10-robot--grid-bfs-simulation) | BFS / Grid Simulation | ⭐⭐ |
| 11 | [7\_2\_digit\_sum.cpp](#11-digit-sum--math) | Math / Number Theory | ⭐⭐ |
| 12 | [min\_subset\_diff.cpp](#12-minimum-subset-difference--dp) | DP — Subset/Partition | ⭐⭐⭐⭐ |
| 13 | [apples.cpp](#13-apples--0-1-bfs-with-direction-state) | 0-1 BFS / Grid + Direction State | ⭐⭐⭐⭐ |
| 14 | [logging\_trees.cpp](#14-logging-trees--greedy--dp-on-tree-lengths) | Greedy + DP on positions | ⭐⭐⭐⭐ |
| 15 | [soldiers.cpp](#15-soldiers--tree-dfs-with-greedy-subtree-balancing) | Tree DFS / Greedy on subtree sums | ⭐⭐⭐ |
| 16 | [min\_cost.cpp](#16-minimum-cost--dp) | Dynamic Programming — Grid Path | ⭐⭐⭐ |

---

---

## 9. Cars — Simultaneous Parking (Parity + Triangular Numbers)

### 🧩 Problem Statement

There's an `M × M` grid (`M` up to `±10^17`) with `N` cars (`N ≤ 100`), each at a starting position `(xi, yi)`. All cars must be parked at the **same target cell** `(p, q)`, moving **simultaneously** every turn. On turn `k`, every car must move **exactly `k` distance** (Manhattan distance) that turn — turn 1 moves distance 1, turn 2 moves distance 2, and so on. Car paths may overlap. Find the **minimum number of turns** to park all cars at `(p, q)`, or `-1` if impossible.

### 📌 Example

```
Cars at distances [3, 5, 7] from target (p, q)

After turn 1: total distance traveled so far = 1
After turn 2: total = 1+2 = 3
After turn 3: total = 1+2+3 = 6
After turn 4: total = 1+2+3+4 = 10

A car needs its total required distance (after some number of turns) to be
>= its distance to target AND have matching parity (since "extra" distance
must be canceled out by back-and-forth moves, which only works if the
leftover difference is even).

All three cars (3, 5, 7) have the same parity (all odd) -> feasible.
Find minimum turns where the cumulative triangular sum >= max distance (7)
and matches parity for all cars.
```

### 🔍 How to Identify This Type

- "Move exactly `k` distance on turn `k`" → the **cumulative distance traveled after `t` turns is the triangular number `t(t+1)/2`** — recognizing this formula is the entire key to the problem
- "All must arrive simultaneously, paths can overlap" → distance alone isn't enough; **leftover distance must be "absorbed" by extra back-and-forth movement**, which is only possible if the **parity** (odd/even) of the leftover matches across all cars
- Keywords: "simultaneous movement", "exactly k distance on turn k", "minimum number of moves/drives", huge coordinate bounds (`10^17`) → signals a **closed-form/binary-search** solution, not simulation
- If any car has a different parity of `(distance to target)` than the others, the answer is immediately `-1`

### 🪜 Step-by-Step Solution

1. For each car, compute its **Manhattan distance** to the target `(p, q)`.
2. Find the **maximum distance** among all cars — this lower-bounds how many turns are needed (the slowest car decides the minimum).
3. Find the smallest `turns` such that the triangular number `turns(turns+1)/2 ≥ max distance` (solve via the quadratic formula / `ceil` on `sqrt`).
4. Compute `actual = turns(turns+1)/2` — the cumulative distance reachable in that many turns.
5. For **every car**, compute `(actual - distance[i]) & 1` — this is the parity of the "leftover" distance that must be canceled by wasted movement.
6. **If all cars don't share the same parity value**, it's impossible → print `-1`.
7. If they do match:
   - If the shared parity is `0` (even leftover) → `turns` is already sufficient, answer = `turns`.
   - If the shared parity is `1` (odd leftover) → you need **one more turn** to flip parity, **unless** that next turn number is itself odd (since adding an odd-numbered turn doesn't always flip cumulative parity correctly) — in that edge case you need **two more turns** instead of one.

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
#define ll long long
#define MAXN 100
using namespace std;

ll a[MAXN];

// Computes minimum turns so triangular(turns) >= max distance,
// then converts each car's distance into a 0/1 parity of leftover slack.
ll parity(ll n) {
    ll x = *max_element(a, a + n);                       // furthest car decides the floor

    // Smallest 'turns' such that turns*(turns+1)/2 >= x  (solve quadratic, take ceiling)
    ll turns = (ll)ceil((sqrt(1 + 8 * x) - 1) / 2);

    ll actual = (turns * (turns + 1)) / 2;                // cumulative distance reachable

    // Replace each car's distance with the PARITY of its leftover slack
    for (int i = 0; i < n; i++)
        a[i] = (actual - a[i]) & 1;

    return turns;
}

void solve() {
    ll n, m;
    cin >> n >> m;
    ll p, q, x, y;
    cin >> p >> q;

    for (int i = 0; i < n; i++) {
        cin >> x >> y;
        a[i] = abs(p - x) + abs(q - y);                   // Manhattan distance to target
    }

    ll turns = parity(n);                                 // converts a[] into parity values in-place

    // All cars must share the same leftover parity, or it's impossible
    for (int i = 0; i < n; i++) {
        if (a[0] != a[i]) {
            cout << -1 << "\n";
            return;
        }
    }

    // a[0] == 1 means there's odd leftover slack to cancel out
    if (a[0] && turns & 1)
        cout << turns + 2 << "\n";   // next turn is odd -> can't flip parity in 1 extra turn
    else if (a[0])
        cout << turns + 1 << "\n";   // next turn is even -> 1 extra turn fixes parity
    else
        cout << turns << "\n";       // already feasible with no extra turns
}

int main() {
    ios_base::sync_with_stdio(false); cin.tie(NULL); cout.tie(NULL);
    ll t, cnt = 1;
    cin >> t;
    while (t--) {
        cout << "# " << cnt << " ";
        solve();
        cnt++;
    }
}
```

### 🐍 Python Code (with comments)

```python
import math

def solve():
    n, m = map(int, input().split())
    p, q = map(int, input().split())

    distances = []
    for _ in range(n):
        x, y = map(int, input().split())
        distances.append(abs(p - x) + abs(q - y))   # Manhattan distance to target

    max_dist = max(distances)

    # Smallest 'turns' such that turns*(turns+1)/2 >= max_dist
    turns = math.ceil((math.sqrt(1 + 8 * max_dist) - 1) / 2)
    actual = turns * (turns + 1) // 2

    # Convert each distance into the PARITY of its leftover slack
    parities = [(actual - d) & 1 for d in distances]

    if len(set(parities)) > 1:
        return -1   # mismatched parity -> impossible to synchronize

    if parities[0] == 0:
        return turns               # already feasible
    elif turns % 2 == 1:
        return turns + 2           # next turn is odd -> needs 2 extra turns
    else:
        return turns + 1           # next turn is even -> 1 extra turn suffices

def main():
    t = int(input())
    for cnt in range(1, t + 1):
        print(f"# {cnt} {solve()}")

main()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **Closed-form triangular number + parity check (repo)** | Direct formula for minimum turns, then parity correction | O(N) per test case | ✅ Optimal — handles the huge `10^17` bound easily |
| **Binary search on number of turns (alternate repo version)** | Binary search smallest `turns` with `triangular(turns) ≥ max_dist`, then parity-correct | O(N log(max_dist)) | Same result, more defensive against edge cases in the sqrt formula |
| **Brute-force turn simulation** | Simulate turn by turn until feasible | Infeasible | `M` up to `10^17` makes this completely impossible |

---

---
## 10. Robot — Grid BFS Simulation

### 🧩 Problem Statement

A robot starts at a position on a grid. Given a sequence of commands (move forward, turn left, turn right), simulate the robot's movement and find its final position, or count cells visited.

### 📌 Example

```
Grid: 5×5, Robot starts at (2,2) facing North
Commands: F F R F L F

Step by step:
  (2,2)→N: F → (1,2)
  (1,2)→N: F → (0,2)
  (0,2)→N: R → now facing East
  (0,2)→E: F → (0,3)
  (0,3)→E: L → now facing North
  (0,3)→N: F → out of bounds? → stay or wrap depending on rules

Answer: final position / visited count
```

### 🔍 How to Identify This Type

- Robot/agent moving on a grid with direction state
- Commands: Forward, Turn Left, Turn Right
- Keywords: "simulate", "robot", "commands", "direction"
- State = `(row, col, direction)` — always track direction!

### 🪜 Step-by-Step Solution

1. Define direction arrays: `dr = [-1,0,1,0]`, `dc = [0,1,0,-1]` for N,E,S,W
2. Use `dir` index (0=N, 1=E, 2=S, 3=W)
3. Turn right: `dir = (dir+1) % 4`
4. Turn left: `dir = (dir+3) % 4`
5. Move forward: `(r,c) → (r+dr[dir], c+dc[dir])`, check bounds
6. Track visited set if needed

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Directions: 0=North, 1=East, 2=South, 3=West
int dr[] = {-1, 0, 1, 0};
int dc[] = {0, 1, 0, -1};

int main() {
    ios::sync_with_stdio(false); cin.tie(NULL);
    int T; cin >> T;
    for (int t = 1; t <= T; t++) {
        int n, m, r, c, dir;
        cin >> n >> m >> r >> c >> dir;  // grid size, start pos, start direction
        string commands; cin >> commands;

        set<pair<int,int>> visited;
        visited.insert({r, c});

        for (char cmd : commands) {
            if (cmd == 'F') {                      // Move forward
                int nr = r + dr[dir], nc = c + dc[dir];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m) {
                    r = nr; c = nc;                // Only move if in bounds
                    visited.insert({r, c});
                }
            } else if (cmd == 'R') {               // Turn right
                dir = (dir + 1) % 4;
            } else if (cmd == 'L') {               // Turn left
                dir = (dir + 3) % 4;              // +3 mod 4 = -1 mod 4
            }
        }

        cout << "#" << t << " " << visited.size() << "\n";
    }
}
```

### 🐍 Python Code (with comments)

```python
def solve():
    T = int(input())
    # Directions: 0=North, 1=East, 2=South, 3=West
    dr = [-1, 0, 1, 0]
    dc = [0, 1, 0, -1]

    for t in range(1, T+1):
        n, m, r, c, d = map(int, input().split())
        commands = input().strip()

        visited = {(r, c)}

        for cmd in commands:
            if cmd == 'F':              # Move forward in current direction
                nr, nc = r + dr[d], c + dc[d]
                if 0 <= nr < n and 0 <= nc < m:
                    r, c = nr, nc
                    visited.add((r, c))
            elif cmd == 'R':            # Turn right
                d = (d + 1) % 4
            elif cmd == 'L':            # Turn left
                d = (d + 3) % 4

        print(f"#{t} {len(visited)}")

solve()
```

### 🔀 Other Approaches

| Approach | Idea | Notes |
|----------|------|-------|
| **Direct simulation (above)** | Follow commands step by step | ✅ Standard approach |
| **BFS variant** | If obstacles exist, use BFS to navigate | More complex |
| **2D prefix sums** | For counting visits in regions | Overkill here |

---

---

## 11. Digit Sum — Math

### 🧩 Problem Statement

Given a number `N`, repeatedly replace it with the sum of its digits until a single digit remains. Find that single digit (also called the **digital root**).

### 📌 Example

```
N = 9875
Step 1: 9+8+7+5 = 29
Step 2: 2+9 = 11
Step 3: 1+1 = 2

Answer: 2
```

### 🔍 How to Identify This Type

- "Sum of digits", "repeat until single digit", "digital root"
- Often combined with modular arithmetic
- Samsung sometimes asks: find the Nth number whose digit sum equals K

### 🪜 Step-by-Step Solution

**Mathematical shortcut (digital root formula):**
- If `N == 0`: answer is 0
- If `N % 9 == 0`: answer is 9
- Else: answer is `N % 9`

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Brute force: sum digits repeatedly
int digitSumBrute(long long n) {
    while (n >= 10) {           // Keep going until single digit
        long long s = 0;
        while (n > 0) {
            s += n % 10;        // Add last digit
            n /= 10;
        }
        n = s;
    }
    return n;
}

// O(1) mathematical formula
int digitalRoot(long long n) {
    if (n == 0) return 0;
    return (n % 9 == 0) ? 9 : n % 9;
}

int main() {
    ios::sync_with_stdio(false); cin.tie(NULL);
    int T; cin >> T;
    for (int t = 1; t <= T; t++) {
        long long n; cin >> n;
        cout << "#" << t << " " << digitalRoot(n) << "\n";
    }
}
```

### 🐍 Python Code (with comments)

```python
def digital_root(n):
    """O(1) formula: digital root of n."""
    if n == 0:
        return 0
    return 9 if n % 9 == 0 else n % 9

def digit_sum_brute(n):
    """Brute force: keep summing digits."""
    while n >= 10:
        n = sum(int(d) for d in str(n))
    return n

T = int(input())
for t in range(1, T+1):
    n = int(input())
    print(f"#{t} {digital_root(n)}")
```

### 🔀 Other Approaches

| Approach | Time | Notes |
|----------|------|-------|
| **Brute force loop** | O(log N per step) | Simple, works fine |
| **Digital root formula** | O(1) | ✅ Best — always use this |
| **String manipulation** | O(digits) per step | Slightly slower than math |

---

---

## 12. Minimum Subset Difference — DP

### 🧩 Problem Statement

Given an array of `N` integers, split them into **two subsets** such that the **absolute difference of their sums is minimised**. Return that minimum difference.

**Input:** Array `arr` of N positive integers
**Output:** Minimum possible `|sum1 - sum2|`

### 📌 Example

```
arr = [1, 2, 3, 4]
total = 10

Possible splits:
  {1,4} vs {2,3} → |5-5| = 0  ✅ minimum!
  {1,2,3} vs {4} → |6-4| = 2
  {1,2,4} vs {3} → |7-3| = 4

Answer: 0
```

### 🔍 How to Identify This Type

- "Split array into two groups", "minimum difference between two halves"
- "Partition", "two teams", "balance two loads"
- This is a classic **0/1 Knapsack / Subset Sum DP** problem
- Key insight: minimize `|S1 - S2|` = minimize `|total - 2*S1|` → find S1 closest to `total/2`

### 🪜 Step-by-Step Solution

1. Compute `total = sum(arr)`
2. Build a DP table: `dp[i][s]` = can we form sum `s` using the first `i` elements?
3. Base case: `dp[*][0] = true` (empty subset has sum 0)
4. Transition: `dp[i][s] = dp[i-1][s] OR dp[i-1][s - arr[i]]`
5. Look at last row: for all achievable sums `s ≤ total/2`, compute `|total - 2*s|`
6. Return the minimum

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Returns true if subset of arr[0..ind] sums to 'target'
bool subsetSum(int ind, int target, vector<int>& arr, vector<vector<int>>& dp) {
    if (target == 0) return dp[ind][target] = true;         // Empty subset
    if (ind == 0)    return dp[ind][target] = (arr[0] == target);

    if (dp[ind][target] != -1) return dp[ind][target];      // Already computed

    bool skip = subsetSum(ind-1, target, arr, dp);           // Don't take arr[ind]
    bool take = false;
    if (arr[ind] <= target)
        take = subsetSum(ind-1, target-arr[ind], arr, dp);  // Take arr[ind]

    return dp[ind][target] = skip || take;
}

int minSubsetDiff(vector<int>& arr) {
    int n = arr.size();
    int total = 0;
    for (int x : arr) total += x;

    // Compute all reachable sums from 0..total
    vector<vector<int>> dp(n, vector<int>(total+1, -1));
    for (int s = 0; s <= total; s++)
        subsetSum(n-1, s, arr, dp);

    // Find minimum |total - 2*s| for all reachable s <= total/2
    int mini = INT_MAX;
    for (int s = 0; s <= total; s++)
        if (dp[n-1][s] == 1)
            mini = min(mini, abs(total - 2*s));

    return mini;
}

int main() {
    vector<int> arr = {1, 2, 3, 4};
    cout << "Min difference: " << minSubsetDiff(arr) << "\n"; // Output: 0
}
```

### 🐍 Python Code (with comments)

```python
def min_subset_diff(arr):
    n = len(arr)
    total = sum(arr)

    # dp[s] = True if sum s is achievable
    dp = [False] * (total + 1)
    dp[0] = True  # Empty subset has sum 0

    for num in arr:
        # Traverse backwards to avoid using same item twice (0/1 knapsack)
        for s in range(total, num - 1, -1):
            if dp[s - num]:
                dp[s] = True

    # Find the achievable sum closest to total//2
    min_diff = float('inf')
    for s in range(total // 2, -1, -1):
        if dp[s]:
            min_diff = total - 2 * s
            break

    return min_diff

# Test
arr = [1, 2, 3, 4]
print(f"Min difference: {min_subset_diff(arr)}")  # Output: 0
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **Recursive + Memoization (repo)** | Top-down DP | O(N × total) | Good for understanding |
| **Bottom-up DP (above Python)** | Iterative bitset-style | O(N × total) | ✅ Cleanest |
| **Meet in the Middle** | Split array, enumerate halves, binary search | O(2^(N/2) log N) | Best for large N (N≤40) |
| **Bitset DP** | Use C++ bitset to speed up transitions | O(N × total / 64) | Very fast in practice |

---

---

## 13. Apples — 0-1 BFS with Direction State

### 🧩 Problem Statement

On an `N × M` map, apples numbered `1` to `M_apples` must be eaten **in order** (apple 2 only appears after apple 1 is eaten, etc.). Cells can be empty (`0`), traps (`-1`, impassable), or contain an apple (positive number = its order). The player starts at `(0,0)` moving right, and **continues moving in the current direction** until it hits a wall, a trap, or **chooses to turn right** (a broken keyboard means **only right turns are possible** — no left turns, no reversing). Find the **minimum number of right turns** needed to eat all apples in order, or `-1` if impossible.

### 📌 Example

```
A small grid where apples 1, 2, 3 are scattered such that:
- Moving straight from start can reach apple 1
- Reaching apple 2 requires one right turn
- Reaching apple 3 requires turning right again to change axis

Each "turn right" has a cost of 1; moving forward any number of cells in
the same direction is "free" in terms of turns (you only pay when you
actually rotate). The minimum number of such rotations to eat all apples
in order, while never reversing or turning left, is the answer.
```

### 🔍 How to Identify This Type

- "Can only turn right (or only turn in one direction)" + "minimize number of turns" → classic **0-1 BFS** (edge weights are only 0 or 1: moving forward costs 0, turning costs 1)
- State must include **direction**, not just position, because the same cell can be visited facing different directions with different "progress" (which apple number has been eaten so far)
- Keywords: "minimum turns", "movement restricted to one rotation direction", "objects must be collected in order"
- The "in order" constraint means the **state must also track how many apples have been eaten so far** — so the full state is `(row, col, apples_eaten, direction)`, not just `(row, col)`

### 🪜 Step-by-Step Solution

1. Model the state as `(x, y, count, dir)` where `count` = number of apples eaten so far (in the correct order) and `dir` ∈ {0,1,2,3} for the 4 cardinal directions.
2. Use a **deque** (double-ended queue) for **0-1 BFS**:
   - Moving forward one cell in the current direction costs **0** → push to the **front** of the deque.
   - Turning right (changing direction without moving) costs **1** → push to the **back** of the deque.
3. From the front state, try **both** transitions: move forward (cost 0) and turn right (cost 1).
4. When moving onto a cell, check if it's the **next expected apple** (`value == count + 1`); if so, increment the count for that new state.
5. Use a `dp[x][y][count][dir]` table to avoid revisiting worse states (only update if you found a strictly better — lower turn-count — way to reach that state).
6. The answer is the minimum value of `dp[x][y][apples_total][dir]` over **all** cells and directions (you can stop anywhere once the last apple is eaten).

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;
    vector<vector<int>> v(n, vector<int>(m));
    int apples = 0;

    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++) {
            cin >> v[i][j];
            apples = max(v[i][j], apples);   // total number of apples to eat
        }

    deque<array<int,4>> dq;   // state: {x, y, apples_eaten, direction}

    // dp[x][y][apples_eaten][dir] = min turns to reach this state
    vector dp(n, vector(m, vector(apples + 1, vector(4, INT_MAX))));

    // Start at (0,0), facing direction 0, having eaten apple 1 if it's there
    dq.push_back({0, 0, v[0][0] == 1, 0});
    dp[0][0][v[0][0] == 1][0] = 0;

    vector<int> dx = {0, 1, 0, -1}, dy = {1, 0, -1, 0};   // 4 directions

    // f = 1 means "turn" (cost 1, push to back); f = 0 means "move forward" (cost 0, push to front)
    auto move = [&](int x, int y, int cnt, int dir, int d, int f) {
        int nx = x + dx[dir], ny = y + dy[dir];
        if (!(nx >= 0 && nx < n && ny >= 0 && ny < m && v[nx][ny] != -1)) return;  // wall/trap

        int ncnt = cnt + (v[nx][ny] == (cnt + 1));   // eat the next apple if it matches
        if (dp[nx][ny][ncnt][dir] > d) {
            if (f) dq.push_back({nx, ny, ncnt, dir});     // cost-1 move -> back of deque
            else   dq.push_front({nx, ny, ncnt, dir});    // cost-0 move -> front of deque
            dp[nx][ny][ncnt][dir] = d;
        }
    };

    while (!dq.empty()) {
        auto [x, y, cnt, dir] = dq.front();
        dq.pop_front();

        move(x, y, cnt, dir, dp[x][y][cnt][dir], 0);             // move forward (cost 0)
        move(x, y, cnt, (dir + 1) % 4, dp[x][y][cnt][dir] + 1, 1); // turn right (cost 1)
    }

    int ans = INT_MAX;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            for (int k = 0; k < 4; k++)
                ans = min(ans, dp[i][j][apples][k]);   // best way to have eaten ALL apples

    cout << ans << endl;
    return 0;
}
```

### 🐍 Python Code (with comments)

```python
from collections import deque

def solve():
    n, m = map(int, input().split())
    grid = [list(map(int, input().split())) for _ in range(n)]
    apples = max(max(row) for row in grid)

    dx = [0, 1, 0, -1]
    dy = [1, 0, -1, 0]

    INF = float('inf')
    # dp[x][y][count][dir] = min turns to reach this state
    dp = [[[[INF] * 4 for _ in range(apples + 1)] for _ in range(m)] for _ in range(n)]

    start_count = 1 if grid[0][0] == 1 else 0
    dp[0][0][start_count][0] = 0
    dq = deque([(0, 0, start_count, 0)])

    def try_move(x, y, cnt, d, dist, is_turn):
        nx, ny = x + dx[d], y + dy[d]
        if not (0 <= nx < n and 0 <= ny < m) or grid[nx][ny] == -1:
            return
        ncnt = cnt + (1 if grid[nx][ny] == cnt + 1 else 0)
        if dp[nx][ny][ncnt][d] > dist:
            dp[nx][ny][ncnt][d] = dist
            if is_turn:
                dq.append((nx, ny, ncnt, d))      # cost 1 -> back
            else:
                dq.appendleft((nx, ny, ncnt, d))  # cost 0 -> front

    while dq:
        x, y, cnt, d = dq.popleft()
        cur = dp[x][y][cnt][d]
        try_move(x, y, cnt, d, cur, False)              # move forward, cost 0
        try_move(x, y, cnt, (d + 1) % 4, cur + 1, True)  # turn right, cost 1

    ans = min(
        dp[i][j][apples][k]
        for i in range(n) for j in range(m) for k in range(4)
    )
    print(ans)

solve()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **0-1 BFS with deque (repo)** | Treat moves as 0-cost, turns as 1-cost, use deque instead of priority queue | O(N×M×apples×4) | ✅ Optimal — avoids the `log` factor of Dijkstra |
| **Dijkstra with priority queue** | Same state space, but use a min-heap instead of deque | O(N×M×apples×4 × log(...)) | Works identically, just slower due to heap overhead |
| **DFS + memoization** | Recursive exploration with state caching | Same complexity as 0-1 BFS | Conceptually simpler but risks recursion depth issues on large grids |

---

---
## 14. Logging Trees — Greedy + DP on Tree Lengths

### 🧩 Problem Statement

A logging robot starts at point `0` on a straight road of length `N` and must reach point `N`, chopping down **all** roadside trees along the way. At each integer point, there may be a tree on the left side and/or the right side (each with its own length, `0` meaning no tree). The robot can move forward/backward (1 minute per unit distance) and can chop+load a tree on either side while standing still (1 minute each). **Trees must be loaded in decreasing order of length** — once you load a tree of some length, every tree loaded afterward must be the same length or shorter. Find the **minimum total time** to chop down all trees and reach the end point.

### 📌 Example

```
Road N=5, trees of various lengths on left/right at certain points.

Naive path: 0 -> 3 -> 4 -> 2 -> 4 -> 5
  Chopping: 4 minutes (4 trees)
  Moving:   9 minutes
  Total:    13 minutes

Smarter path: 0 -> 3 -> 2 -> (stand still, chop another) -> 4 -> 5
  Chopping: 4 minutes
  Moving:   7 minutes
  Total:    11 minutes  <- optimal

Answer: 11
```

### 🔍 How to Identify This Type

- "Items must be loaded/processed in **strictly decreasing (or increasing) order** of some attribute" → process trees **largest length first**, since the order constraint fixes a global processing sequence regardless of position
- Keywords: "load in order of size", "robot moves along a line chopping trees", "minimum total time"
- Once you fix the *order* trees must be processed in (by length, descending), the only remaining freedom is **which end of each length-group to enter from and exit to** → this becomes a **DP over positions**, where the state is "which length are we currently processing, and which extreme position (leftmost or rightmost occurrence of that length) are we standing at"
- The key insight: **within a single length value, the chopping order among trees of equal length is also flexible**, but the **total walking distance to visit all of them is fixed** once you know you'll enter from one end and exit from the other — this lets you precompute a `count[]` array of "cost to traverse all trees of length L" up front

### 🪜 Step-by-Step Solution

1. **Group tree positions by length**: for each tree length `L`, collect the list of road positions where a tree of length `L` exists (regardless of left/right side).
2. For each length group, **sort positions** and compute the cost to **sweep through all of them once** (visiting consecutive positions in order, plus 1 minute per tree to chop): this becomes `count[L]`.
3. Process lengths in **descending order** (largest trees first, since they must be loaded before any shorter tree).
4. Define `f(L, pos)` = minimum total time to finish processing all trees of length `≤ L`, given the robot is currently standing at position `pos` having just finished length `L+1`'s group.
5. **Transition**: for length `L`, you must visit **both ends** of its position list (`left` = smallest position, `right` = largest position) at some point — you can enter from either end:
   - Enter from `left`, exit at `right`: cost = `|pos - left| + count[L] + f(L-1, right)`
   - Enter from `right`, exit at `left`: cost = `|pos - right| + count[L] + f(L-1, left)`
   - Take the **minimum** of these two options.
6. **Base case**: `f(0, pos) = |pos - N|` (no more trees to chop — just walk straight to the end point).
7. **Answer** = `f(max_length, 0)` (start position is `0`, process the largest length group first).

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

void solve() {
    int n;
    cin >> n;

    map<int, vector<int>> mp;   // length -> list of positions with a tree of that length
    int maxLen = 0;

    for (int i = 0; i < n; i++) {
        int l, r;                       // left-side tree length, right-side tree length at position i
        cin >> l >> r;
        maxLen = max({maxLen, l, r});
        if (l != 0) mp[l].push_back(i);
        if (r != 0) mp[r].push_back(i);
    }

    // count[L] = time to sweep through every tree of length L once (chopping all of them)
    vector<int> count(maxLen + 1, 0);
    for (auto &[length, positions] : mp) {
        sort(positions.begin(), positions.end());
        int sz = positions.size();
        int total = 1;                                   // chop the first tree (1 minute)
        for (int i = 1; i < sz; i++) {
            total += (positions[i] - positions[i - 1]);   // walk to next tree of same length
            total++;                                       // chop it (1 minute)
        }
        count[length] = total;
    }

    // dp[L][pos] = min time to finish lengths 1..L, starting from position 'pos'
    vector<vector<int>> dp(maxLen + 1, vector<int>(n + 2, -1));

    auto f = [&](int len, int pos, auto &&f) -> int {
        if (len == 0) return abs(pos - n);   // no more trees, just walk to the end point

        if (dp[len][pos] != -1) return dp[len][pos];

        int left = mp[len].front();          // leftmost position with this tree length
        int right = mp[len].back();          // rightmost position with this tree length

        // Option A: approach from the left end, finish at the right end
        int res = abs(pos - left) + count[len] + f(len - 1, right, f);
        // Option B: approach from the right end, finish at the left end
        res = min(res, abs(pos - right) + count[len] + f(len - 1, left, f));

        return dp[len][pos] = res;
    };

    cout << f(maxLen, 0, f) << "\n";
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    solve();
}
```

### 🐍 Python Code (with comments)

```python
import sys
sys.setrecursionlimit(10000)

def solve():
    n = int(input())
    tree_positions = {}   # length -> list of positions
    max_len = 0

    for i in range(n):
        l, r = map(int, input().split())
        max_len = max(max_len, l, r)
        if l != 0:
            tree_positions.setdefault(l, []).append(i)
        if r != 0:
            tree_positions.setdefault(r, []).append(i)

    # count[L] = time to sweep through every tree of length L once
    count = [0] * (max_len + 1)
    for length, positions in tree_positions.items():
        positions.sort()
        total = 1  # chop the first one
        for i in range(1, len(positions)):
            total += positions[i] - positions[i - 1]  # move to next position
            total += 1                                  # chop it
        count[length] = total

    memo = {}

    def f(length, pos):
        if length == 0:
            return abs(pos - n)   # no more trees, walk straight to the end

        if (length, pos) in memo:
            return memo[(length, pos)]

        positions = tree_positions.get(length, [])
        left, right = positions[0], positions[-1]

        # Enter from left end, exit at right end
        option_a = abs(pos - left) + count[length] + f(length - 1, right)
        # Enter from right end, exit at left end
        option_b = abs(pos - right) + count[length] + f(length - 1, left)

        result = min(option_a, option_b)
        memo[(length, pos)] = result
        return result

    print(f(max_len, 0))

solve()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **Process by descending length + endpoint DP (repo)** | Fix processing order by the size constraint, only track entry/exit endpoints per length | O(N + maxLen) states, each O(1) | ✅ Optimal — the order constraint collapses an exponential search into a linear DP |
| **Full DP over all visited positions** | Track exact subset of trees chopped so far | Exponential | Infeasible — far too many states without the "must load by length order" simplification |
| **Greedy nearest-tree-first** | Always chop whichever unchopped tree (respecting order) is closest | — | Fails — locally optimal moves can strand you far from the next required length group |

---

---
## 15. Soldiers — Tree DFS with Greedy Subtree Balancing

### 🧩 Problem Statement

Each node of a tree represents a military unit with a given number of soldiers. The tree must be **balanced** so that **all children of the same parent have equal subtree sums**. You can only balance by **removing** soldiers (never adding). Find the **minimum total number of soldiers removed**, and report the **final total** number of soldiers remaining after balancing.

### 📌 Example

```
Tree:
        Root(5)
       /        \
   Node(3)     Node(10)
   subtree=3   subtree=10

To balance, both children of Root must have the SAME subtree sum.
Since we can only remove (not add), the target must be min(3, 10) = 3.
Remove 7 soldiers from Node(10)'s subtree to bring it down to 3.

Total removed = 7
Final total = Root(5) + 3 + 3 = 11
```

### 🔍 How to Identify This Type

- "Balance a tree so siblings match" + "can only remove, never add" → the only valid **target value for a group of siblings is the minimum subtree sum among them** (since you can't increase any sibling to match a larger one)
- Keywords: "subtree sum", "balance siblings", "minimum removed", "tree structure given as parent-child pairs"
- This is a **bottom-up DFS** problem: you can't decide a parent's contribution until all of its children's subtree sums are finalized first (post-order traversal)
- A *greedy* choice (always shrink to the minimum sibling) is provably optimal here because removing is the only operation allowed — there's no trade-off to search over, which is what makes this greedy-on-tree fast instead of requiring DP over subsets

### 🪜 Step-by-Step Solution

1. **Build the tree** from the parent-index input; identify the root (the node with no parent, e.g. `parent[i] == -1`).
2. **DFS from the root** (post-order — process children before the current node):
   - For a **leaf node**, its subtree sum is just its own soldier count.
   - For an **internal node**, first recursively compute the subtree sum of **every child**.
3. Among all children's subtree sums, the **target sum** they must all be reduced to is the **minimum** of those sums (you can only shrink, never grow).
4. For each child, the cost to bring it down to the target = `child_subtree_sum - target` (this must be ≥ 0 by definition of target being the minimum).
5. **Accumulate** all these removal costs into a running `totalRemoved` counter.
6. The current node's own subtree sum becomes: `soldiers[node] + target * (number_of_children)`.
7. Return this value up to the parent; repeat until the root is processed.
8. The **final answer** is the subtree sum returned at the root (and `totalRemoved` tracks how many were taken out, if that's separately required).

### 💻 C++ Code (with comments)

```cpp
#include <iostream>
#include <vector>
using namespace std;

const int MAXN = 505;
vector<int> tree[MAXN];     // adjacency list: children of each node
int soldiers[MAXN];         // soldiers initially stationed at each node
int n;
int totalRemoved = 0;       // running total of soldiers removed

// Returns the (possibly reduced) subtree sum rooted at node u, after balancing
int dfs(int u) {
    vector<int> childSums;
    for (int v : tree[u]) {
        childSums.push_back(dfs(v));   // post-order: finalize children first
    }

    if (childSums.empty()) return soldiers[u];   // leaf node: nothing to balance

    // The only valid target for ALL children is the smallest subtree sum
    // (since soldiers can only be removed, never added)
    int target = *min_element(childSums.begin(), childSums.end());

    int costToBalance = 0;
    for (int s : childSums) {
        costToBalance += (s - target);   // how many soldiers must be removed from this child
    }

    totalRemoved += costToBalance;

    // This node's contribution: itself plus all (now-equal) children
    return soldiers[u] + target * (int)childSums.size();
}

int main() {
    cin >> n;
    vector<int> parent(n);
    int root = -1;

    for (int i = 0; i < n; ++i) {
        cin >> parent[i] >> soldiers[i];
        if (parent[i] == -1) {
            root = i;
        } else {
            tree[parent[i]].push_back(i);
        }
    }

    int finalSum = dfs(root);
    cout << finalSum << endl;

    return 0;
}
```

### 🐍 Python Code (with comments)

```python
import sys
sys.setrecursionlimit(10000)

def solve():
    n = int(input())
    tree = [[] for _ in range(n)]
    soldiers = [0] * n
    root = -1

    for i in range(n):
        p, s = map(int, input().split())
        soldiers[i] = s
        if p == -1:
            root = i
        else:
            tree[p].append(i)

    total_removed = [0]   # use a mutable container to update from inside the closure

    def dfs(u):
        child_sums = [dfs(v) for v in tree[u]]   # post-order: children first

        if not child_sums:
            return soldiers[u]   # leaf: nothing to balance

        # Can only shrink, never grow -> target is the smallest child subtree sum
        target = min(child_sums)

        cost = sum(s - target for s in child_sums)
        total_removed[0] += cost

        return soldiers[u] + target * len(child_sums)

    final_sum = dfs(root)
    print(final_sum)
    # print(total_removed[0])  # uncomment if the removed count is also required

solve()
```

### 🔀 Other Approaches

| Approach | Idea | Time | Notes |
|----------|------|------|-------|
| **Post-order DFS + greedy min-of-children (repo)** | Bottom-up traversal, always shrink siblings to their minimum | O(N) | ✅ Optimal — greedy is provably correct since only removal is allowed |
| **BFS level-order with deferred balancing** | Process level by level instead of recursive DFS | O(N) | Same result, avoids recursion depth limits on very deep/skewed trees |
| **Iterative DFS with explicit stack** | Same logic as recursive DFS but manual stack | O(N) | Useful if `depth ≤ 100` constraint were much larger and risked stack overflow |

---

---
## 16. Minimum Cost — DP

### 🧩 Problem Statement

Given a grid of costs, find the path from top-left `(0,0)` to bottom-right `(n-1,m-1)` with **minimum total cost**. You can move right, down, or diagonally down-right.

### 📌 Example

```
Grid:
1  3  1
1  5  1
4  2  1

Paths (right/down/diagonal):
(0,0)→(0,1)→(0,2)→(1,2)→(2,2): 1+3+1+1+1 = 7
(0,0)→(1,0)→(2,0)→(2,1)→(2,2): 1+1+4+2+1 = 9
(0,0)→(0,1)→(1,1)→(2,1)→(2,2): 1+3+5+2+1 = 12
(0,0)→(1,1)→(2,2):              1+5+1 = 7  (diagonal)
(0,0)→(0,1)→(0,2)→(1,2)→(2,2): 1+3+1+1+1 = 7

Minimum: 7
```

### 🔍 How to Identify This Type

- Grid problem asking for minimum/maximum path cost
- Movement restricted to certain directions
- Keywords: "minimum cost", "path", "grid", "top-left to bottom-right"
- Classic **Grid DP** pattern

### 🪜 Step-by-Step Solution

1. Define `dp[i][j]` = minimum cost to reach cell `(i,j)`
2. Base: `dp[0][0] = grid[0][0]`
3. First row: can only come from left → `dp[0][j] = dp[0][j-1] + grid[0][j]`
4. First col: can only come from above → `dp[i][0] = dp[i-1][0] + grid[i][0]`
5. General: `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])`
6. Answer: `dp[n-1][m-1]`

### 💻 C++ Code (with comments)

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false); cin.tie(NULL);
    int T; cin >> T;
    for (int t = 1; t <= T; t++) {
        int n, m; cin >> n >> m;
        vector<vector<int>> g(n, vector<int>(m));
        for (int i=0;i<n;i++) for (int j=0;j<m;j++) cin >> g[i][j];

        vector<vector<int>> dp(n, vector<int>(m, INT_MAX));
        dp[0][0] = g[0][0];

        // Fill first row (only from left)
        for (int j=1;j<m;j++) dp[0][j] = dp[0][j-1] + g[0][j];
        // Fill first col (only from above)
        for (int i=1;i<n;i++) dp[i][0] = dp[i-1][0] + g[i][0];

        // Fill rest: take minimum of left, above, diagonal
        for (int i=1;i<n;i++) {
            for (int j=1;j<m;j++) {
                int best = min({dp[i-1][j],    // from above
                                dp[i][j-1],    // from left
                                dp[i-1][j-1]}); // from diagonal
                dp[i][j] = g[i][j] + best;
            }
        }

        cout << "#" << t << " " << dp[n-1][m-1] << "\n";
    }
}
```

### 🐍 Python Code (with comments)

```python
def solve():
    T = int(input())
    for t in range(1, T+1):
        n, m = map(int, input().split())
        g = [list(map(int, input().split())) for _ in range(n)]

        dp = [[float('inf')] * m for _ in range(n)]
        dp[0][0] = g[0][0]

        # First row: can only come from left
        for j in range(1, m):
            dp[0][j] = dp[0][j-1] + g[0][j]
        # First column: can only come from above
        for i in range(1, n):
            dp[i][0] = dp[i-1][0] + g[i][0]

        # Fill remaining cells
        for i in range(1, n):
            for j in range(1, m):
                best = min(dp[i-1][j],    # above
                           dp[i][j-1],    # left
                           dp[i-1][j-1])  # diagonal
                dp[i][j] = g[i][j] + best

        print(f"#{t} {dp[n-1][m-1]}")

solve()
```

### 🔀 Other Approaches

| Approach | Idea | Notes |
|----------|------|-------|
| **Grid DP (above)** | Classic O(N×M) DP | ✅ Optimal |
| **Dijkstra** | Treat grid as weighted graph | O(NM log NM) — overkill unless weights are non-uniform |
| **DFS + Memoization** | Top-down recursion with cache | Same as DP, slightly more overhead |
| **Space-optimised DP** | Keep only 1 row at a time | O(M) space instead of O(NM) |

---

---

## 📝 Quick Pattern Recognition Guide

| If the problem says... | Think... |
|------------------------|----------|
| "Simulate", "step by step", "direction" | **Simulation** — track state carefully |
| "Grid", "shortest path", "flood" | **BFS** — use queue, mark visited |
| "All paths", "count ways", "combinations" | **DFS + Backtracking** |
| "Minimum/maximum", "split array", "partition" | **Dynamic Programming** |
| "Minimum days/steps", "choose one per round" | **Greedy + DP hybrid** |
| "Digit", "sum of digits", "divisible" | **Math / Number Theory** |
| "Graph", "connected", "cycle" | **DFS/BFS on graph** |
| "Sort first", "greedy choice" | **Greedy — sort by some criterion** |

---

## ⚡ Samsung SWC-Pro Exam Tips

1. **Read both problems first** — 10 minutes reading before writing any code
2. **Check constraints first** — n≤20 means backtracking; n≤10⁵ means O(n log n)
3. **Samsung = C++ only** — no Python allowed in the actual exam
4. **50 test cases** — your solution must be fast AND correct
5. **Test with edge cases** — n=1, all same values, empty grid, maximum values
6. **Template ready** — have your BFS/DFS/DP templates memorised

---

## ⬅️ Back to Part 1

Problems **1–8** (Segments, Days, RB, Warehouse, Tiles, Stones, Strings, Scores) are in **Part 1**.

*Made from actual Samsung SWC-Pro questions in [kirang193/samsung-pro-coding-questions](https://github.com/kirang193/samsung-pro-coding-questions) 🚀*
