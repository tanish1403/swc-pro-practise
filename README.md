# 🚀 Samsung R&D SWC Professional Track - Masterclass (Part 1)
Welcome to Part 1 of your ultimate Samsung SWC Pro preparation guide. As your DSA tutor, this repository is structured to not just give you the code, but to teach you the **core patterns** so you know exactly *why* a specific approach is used.
All solutions use the **C++ Standard Template Library (STL)** to ensure optimal performance and minimal boilerplate. Every snippet includes a main() function with **Easy, Medium, and Hard test cases** so you can confidently test edge cases.
## 📚 Table of Contents (Part 1)
 1. Points on a Path (Line Sweep / Geometry)
 2. Warehouse Inventory (Bitmask DP)
 3. Red and Blue Stones (Prefix Sum + Hash Map)
 4. City Logistics (State-Space BFS)
 5. Tiles Selection (Binary Search + Sliding Window)
 6. Stone Removal Cost (1D State Machine DP)
 7. String Merge (LIS Pattern)
 8. Array Score Maximization (Binary Search)
## 1. Points on a Path
### 📝 Problem Description
You are given a continuous path on an infinite 2D lattice. The path consists of connected line segments parallel to either the x-axis or y-axis. You are also given a set of N isolated points in space.
Write a program to find exactly how many of these N points lie directly on *any* segment of the given path.
### 💡 LeetCode Style Examples
**Example 1 (Easy):**
 * **Input:** Path Turning Points: (1, 1) -> (1, 5) -> (2, 5), Given Points: [(1, 3), (2, 2), (1, 5), (3, 5)]
 * **Output:** 2
 * **Explanation:** Point (1,3) lies on the vertical segment x=1. Point (1,5) lies exactly on the corner intersection. (2,2) and (3,5) are off the path.
### 👨‍🏫 Tutor's Corner: The Pattern
> **Why this approach?** This is a 2D Geometry problem, but because lines are parallel to the axes, we don't need complex math like cross products! We can break the path down into strict 1D boundaries (Horizontal and Vertical segments).
> **The Catch:** We must *normalize* our segments so start is always \le end. Otherwise, our boundary checks (start <= target <= end) will fail if the path goes backwards!
> 
### 🛠️ Step-by-Step Solution
 1. **Extract Segments:** Loop through consecutive path points. If x1 == x2, it forms a vertical line. If y1 == y2, it forms a horizontal line.
 2. **Normalize Boundaries:** Store segments as {constant_axis, min_val, max_val} using std::min and std::max.
 3. **Query Point:** For each point, check if it falls on any vertical segment (matches the constant X, and Y is within bounds) or horizontal segment.
 4. **Avoid Double Counting:** Once a point is found on *any* segment, immediately break to the next point so corners aren't counted twice.
### 💻 C++ Code & Test Cases
```cpp
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

// Structure to cleanly hold normalized line segments
struct Segment {
    int constant_val; 
    int start, end;   
};

int countPointsOnPath(int N, vector<int>& pX, vector<int>& pY, int M, vector<int>& pathX, vector<int>& pathY) {
    vector<Segment> vertical, horizontal; 

    // 1. Build and normalize segments
    for (int i = 0; i < M - 1; i++) {
        if (pathX[i] == pathX[i+1]) {
            // Vertical: X is constant. Ensure Y-start is <= Y-end
            vertical.push_back({pathX[i], min(pathY[i], pathY[i+1]), max(pathY[i], pathY[i+1])});
        } else {
            // Horizontal: Y is constant. Ensure X-start <= X-end
            horizontal.push_back({pathY[i], min(pathX[i], pathX[i+1]), max(pathX[i], pathX[i+1])});
        }
    }

    int points_on_path = 0;
    
    // 2. Query each point
    for (int i = 0; i < N; i++) {
        int px = pX[i], py = pY[i];
        bool found = false;

        // Check vertical lines
        for (const auto& v : vertical) {
            if (px == v.constant_val && py >= v.start && py <= v.end) {
                points_on_path++; found = true; break; 
            }
        }
        if (found) continue;

        // Check horizontal lines
        for (const auto& h : horizontal) {
            if (py == h.constant_val && px >= h.start && px <= h.end) {
                points_on_path++; break;
            }
        }
    }
    return points_on_path;
}

int main() {
    cout << "--- 1. Points on a Path ---\n";
    
    // Easy: L-Shape
    vector<int> pX1 = {1, 2, 1, 3}, pY1 = {3, 2, 5, 5};
    vector<int> pathX1 = {1, 1, 2}, pathY1 = {1, 5, 5};
    cout << "Easy Test Expected: 2 | Got: " << countPointsOnPath(4, pX1, pY1, 3, pathX1, pathY1) << "\n";

    // Hard: Backtracking path overlapping itself
    vector<int> pX2 = {0, 2, 4}, pY2 = {2, 3, 1};
    vector<int> pathX2 = {0, 0, 4, 4}, pathY2 = {0, 3, 3, 0};
    cout << "Hard Test Expected: 3 | Got: " << countPointsOnPath(3, pX2, pY2, 4, pathX2, pathY2) << "\n\n";
    return 0;
}

```
## 2. Warehouse Inventory
### 📝 Problem Description
A warehouse has an initial stock array A. Every day, a shipment arrives, increasing stock by array B. At the end of every day, you must choose exactly *one* item to export, instantly reducing its stock to 0. Find the minimum number of days required to make the total warehouse stock \le K. If impossible, return -1.
### 💡 LeetCode Style Examples
**Example 1 (Medium):**
 * **Input:** N = 2, A = [1, 2], B = [2, 1], K = 4
 * **Output:** 1
 * **Explanation:** * Day 1 Start: A = [1+2, 2+1] = [3, 3]. Total = 6.
   * End of Day 1: Export item 0. A = [0, 3]. Total = 3. Target Total <= 4 is met in 1 day!
### 👨‍🏫 Tutor's Corner: The Pattern
> **Why this approach?** This screams **Bitmask DP**. Why? The problem asks us to choose an optimal subset of items to export over time. Since N is small (implied \le 20), we can represent a chosen subset as an integer bitmask (e.g., 101 means items 0 and 2 are exported).
> **The Trick:** Once an item is exported, it stops accumulating its daily B inflow. We must track both the flat stock removed AND the daily inflow prevented!
> 
### 🛠️ Step-by-Step Solution
 1. **Initialize DP:** dp[count] stores the absolute maximum stock we can prevent from accumulating if we use exactly count days (meaning we export exactly count bits/items).
 2. **Iterate Masks:** For each subset mask from 1 to 2^N - 1:
   * Find the newly added item using __builtin_ctz (index of the lowest set bit).
   * Total removed stock = (stock removed from previous mask) + (accumulated B for previously removed items) + (initial A + B of the newly removed item).
 3. **Verify:** Day by day, calculate the theoretical total stock if nothing was ever exported. If Theoretical Total - dp[day] <= K, return day.
### 💻 C++ Code & Test Cases
```cpp
#include <vector>
#include <algorithm>
#include <numeric>
#include <iostream>
using namespace std;

int getMinimumDays(int n, vector<int>& A, vector<int>& B, long long K) {
    vector<int> total_removed(1 << n, 0);
    vector<int> subset_sum(1 << n, 0); // Tracks prevented daily inflow
    vector<int> dp(n + 1, 0); 

    for (int mask = 1; mask < (1 << n); mask++) {
        // Find the newest exported item in this subset
        int idx = __builtin_ctz(mask);
        int prev_mask = mask ^ (1 << idx); 

        // Stock removed = previous removals + prevented inflow + new item stock
        total_removed[mask] = total_removed[prev_mask] + subset_sum[prev_mask] + A[idx] + B[idx];
        subset_sum[mask] = subset_sum[prev_mask] + B[idx];

        int count = __builtin_popcount(mask);
        dp[count] = max(dp[count], total_removed[mask]); 
    }

    long long A_tot = accumulate(A.begin(), A.end(), 0LL);
    long long B_tot = accumulate(B.begin(), B.end(), 0LL);

    // Validate day by day
    for (int day = 1; day <= n; day++) {
        long long theoretical_total = A_tot + (day * B_tot);
        if (theoretical_total - dp[day] <= K) return day;
    }
    return -1; 
}

int main() {
    cout << "--- 2. Warehouse Inventory ---\n";
    
    // Easy: Target reached in 1 day
    vector<int> A1 = {1, 2}, B1 = {2, 1};
    cout << "Easy Test Expected: 1 | Got: " << getMinimumDays(2, A1, B1, 4) << "\n";

    // Hard: Impossible (inflow is too high)
    vector<int> A2 = {1, 1}, B2 = {100, 100};
    cout << "Hard Test Expected: -1 | Got: " << getMinimumDays(2, A2, B2, 0) << "\n\n";
    return 0;
}

```
## 3. Red and Blue Stones
### 📝 Problem Description
Given a necklace of 'R' and 'B' stones (as a string), make the counts of Red and Blue equal by removing the minimum number of stones strictly from the left or right ends.
### 💡 LeetCode Style Examples
**Example 1:**
 * **Input:** s = "BBRRBRBRBRBBR"
 * **Output:** 1
 * **Explanation:** The central subarray BBRRBRBRBRBB (indices 0 to 11) has 6 B's and 6 R's. Its length is 12. Total length 13 - Max balanced length 12 = 1 removal.
### 👨‍🏫 Tutor's Corner: The Pattern
> **Why this approach?** This maps perfectly to the **Continuous Subarray Sum** pattern. By treating 'B' as +1 and 'R' as -1, a balanced subarray will sum exactly to 0.
> If we keep a running prefix sum, and our current sum is X, and we saw X 5 indices ago, the subarray between them perfectly canceled itself out to 0! std::unordered_map handles this look-up in O(1).
> 
### 🛠️ Step-by-Step Solution
 1. Treat 'B' as +1 and 'R' as -1.
 2. Track current_sum sequentially.
 3. Map Initialization: Add map[0] = -1 (a sum of 0 exists conceptually right before the array starts).
 4. If current_sum exists in the map, current_index - map[current_sum] gives us a balanced length. Maximize this.
 5. If it doesn't exist, store its first occurrence to maximize future lengths.
 6. Return Total Length - Max Length.
### 💻 C++ Code & Test Cases
```cpp
#include <string>
#include <unordered_map>
#include <algorithm>
#include <iostream>
using namespace std;

int getMinRemovals(const string& s) {
    unordered_map<int, int> first_occ;
    first_occ[0] = -1; // Virtual index -1: to handle balanced arrays from index 0

    int current_sum = 0, max_length = 0;

    for(int i = 0; i < (int)s.length(); ++i) {
        current_sum += (s[i] == 'B' ? 1 : -1); 

        if(first_occ.count(current_sum)) {
            // Same prefix sum seen earlier → the subarray between them sums to 0!
            max_length = max(max_length, i - first_occ[current_sum]);
        } else {
            // First time seeing this sum; record position to maximize future lengths
            first_occ[current_sum] = i;
        }
    }

    return s.length() - max_length;
}

int main() {
    cout << "--- 3. Red and Blue Stones ---\n";
    
    // Medium: Remove 1 from the end
    cout << "Medium Test Expected: 1 | Got: " << getMinRemovals("BBRRBRBRBRBBR") << "\n";
    
    // Easy: Already balanced
    cout << "Easy Test Expected: 0 | Got: " << getMinRemovals("BRBR") << "\n";
    
    // Hard: Heavily skewed, massive removals
    cout << "Hard Test Expected: 2 | Got: " << getMinRemovals("BBBBBRRR") << "\n\n";
    return 0;
}

```
## 4. City Logistics
### 📝 Problem Description
A grid represents a city (0=Road, 1=Tree, 2=Garage, 3=Warehouse, 4=Airport). A truck starts at the Garage, loads goods from Warehouses, and unloads at the Airport. Moving one block costs 1 + number of goods carried. Trees are impassable. Find the maximum goods unloaded at the Airport given a budget C.
### 💡 LeetCode Style Examples
**Example 1:**
 * **Input:** C = 10, Grid = [[2, 0, 3, 0, 4]]
 * **Output:** 1
 * **Explanation:** Garage -> Road (cost 1) -> Warehouse (cost 1, load 1 good) -> Road (cost 2) -> Airport (cost 2). Total Cost = 6. Max goods = 1.
### 👨‍🏫 Tutor's Corner: The Pattern
> **Why this approach?** This is **State-Space BFS**. A simple 2D visited[x][y] array fails completely here! Why? Because arriving at cell (x, y) with 0 goods is a completely different state than arriving with 5 goods (the cost to move forward changes). We must expand our visited array into 3D: min_cost[x][y][goods_carried].
> 
### 🛠️ Step-by-Step Solution
 1. **Queue Setup:** Use std::queue storing a struct (x, y, goods, cost).
 2. **Explore:** From a cell, look in 4 directions. Calculate next_cost = current_cost + 1 + goods.
 3. **Branching at Warehouses:** If moving into a warehouse (3), you have *two valid branches*: pass through (goods stay same) OR load a good (goods + 1). Push both valid states to the queue.
 4. **Airport Logic:** If we hit the airport (4) with cost <= C, update max_delivered.
### 💻 C++ Code & Test Cases
```cpp
#include <vector>
#include <queue>
#include <algorithm>
#include <iostream>
using namespace std;

struct State { int x, y, goods, cost; };
int dx[] = {-1, 1, 0, 0}; 
int dy[] = {0, 0, -1, 1}; 

int getMaxGoods(int h, int w, vector<vector<int>>& city, int max_cost) {
    // 3D tracking: min_cost[x][y][goods_carried]
    vector<vector<vector<int>>> min_cost(h, vector<vector<int>>(w, vector<int>(20, 1e9)));
    queue<State> q;

    // Find Garage (2)
    for(int i = 0; i < h; ++i) {
        for(int j = 0; j < w; ++j) {
            if(city[i][j] == 2) {
                q.push({i, j, 0, 0});
                min_cost[i][j][0] = 0;
            }
        }
    }

    int max_delivered = 0;
    while(!q.empty()) {
        auto [x, y, goods, cost] = q.front(); q.pop();

        if(city[x][y] == 4 && cost <= max_cost)
            max_delivered = max(max_delivered, goods);

        for(int i = 0; i < 4; ++i) {
            int nx = x + dx[i], ny = y + dy[i];
            
            // Bounds check & Tree (1) avoidance
            if(nx >= 0 && nx < h && ny >= 0 && ny < w && city[nx][ny] != 1) {
                int next_cost = cost + 1 + goods; 
                
                if(next_cost <= max_cost) {
                    // Branch 1: Pass through / Move normally
                    if(next_cost < min_cost[nx][ny][goods]) {
                        min_cost[nx][ny][goods] = next_cost;
                        q.push({nx, ny, goods, next_cost});
                    }
                    
                    // Branch 2: If warehouse, ALSO explore state where we load 1 good
                    if(city[nx][ny] == 3 && next_cost < min_cost[nx][ny][goods + 1]) {
                        min_cost[nx][ny][goods + 1] = next_cost;
                        q.push({nx, ny, goods + 1, next_cost});
                    }
                }
            }
        }
    }
    return max_delivered;
}

int main() {
    cout << "--- 4. City Logistics ---\n";
    // Easy: Cost multiplier makes picking up goods too expensive
    vector<vector<int>> city1 = {{2, 0, 3}, {1, 1, 0}, {4, 0, 0}};
    cout << "Easy Test Expected: 0 | Got: " << getMaxGoods(3, 3, city1, 5) << "\n";

    // Medium: Straight line
    vector<vector<int>> city2 = {{2, 0, 3, 0, 4}};
    cout << "Medium Test Expected: 1 | Got: " << getMaxGoods(1, 5, city2, 10) << "\n\n";
    return 0;
}

```
## 5. Tiles Selection
### 📝 Problem Description
Given N tiles of varying widths and heights. Select K tiles to minimize the maximum difference between any two selected tiles. Difference between two tiles is defined as max(|h1-h2|, |w1-w2|) globally across the K chosen tiles.
### 💡 LeetCode Style Examples
**Example 1:**
 * **Input:** N=4, K=2, Tiles (w,h): [(1, 5), (2, 4), (6, 1), (8, 2)]
 * **Output:** 1
 * **Explanation:** Selecting tiles (1,5) and (2,4) gives a width difference of 1 and height difference of 1. Overall max diff = 1.
### 👨‍🏫 Tutor's Corner: The Pattern
> **Why this approach?** When a problem asks to "minimize the maximum difference", your brain should instantly scream **Binary Search on the Answer**.
> We guess an allowed difference D. We sort the tiles by width, and use a sliding window to guarantee the width span is \le D. Inside that window, we use std::multiset to automatically keep the heights sorted, allowing us to verify the height condition in O(1) time!
> 
### 🛠️ Step-by-Step Solution
 1. **Sort:** Sort the tiles ascending by width.
 2. **Binary Search:** Set low = 0, high = 1e9. Guess mid as the max difference D.
 3. **Sliding Window:** Expand the right pointer. If width[right] - width[left] > D, shrink the window from the left. Maintain heights in a multiset.
 4. **Validate:** If window size \ge K, iterate through the height multiset. Check if any subsegment of length K has max_height - min_height <= D. If yes, this D is achievable.
### 💻 C++ Code & Test Cases
```cpp
#include <vector>
#include <algorithm>
#include <set>
#include <iostream>
using namespace std;

bool isPossible(vector<pair<int,int>>& tiles, int k, int max_diff) {
    multiset<int> heights; 
    int left = 0;

    for (int right = 0; right < (int)tiles.size(); ++right) {
        heights.insert(tiles[right].second);

        // Shrink window if width condition is violated
        while (tiles[right].first - tiles[left].first > max_diff) {
            heights.erase(heights.find(tiles[left].second));
            left++;
        }

        // If window has at least K items, verify height constraint
        if ((int)heights.size() >= k) {
            auto it = heights.begin();
            auto end_it = std::next(it, k - 1);

            while(end_it != heights.end()) {
                if (*end_it - *it <= max_diff) return true; // Valid subset found
                it++; end_it++;
            }
        }
    }
    return false;
}

int getMinDifference(vector<pair<int,int>>& tiles, int k) {
    sort(tiles.begin(), tiles.end()); 
    int low = 0, high = 1e9, ans = high;

    while(low <= high) {
        int mid = low + (high - low) / 2;
        if(isPossible(tiles, k, mid)) { 
            ans = mid;         // Possible! Try tighter bounds
            high = mid - 1; 
        }
        else { 
            low = mid + 1;     // Too tight, loosen the constraints
        }
    }
    return ans;
}

int main() {
    cout << "--- 5. Tiles Selection ---\n";
    // Easy: K=2
    vector<pair<int,int>> t1 = {{1,5},{2,4},{6,1},{8,2}};
    cout << "Easy Test Expected: 1 | Got: " << getMinDifference(t1, 2) << "\n";

    // Hard: K=3 forces much wider spread
    vector<pair<int,int>> t2 = {{1,10},{2,8},{3,5},{10,4},{11,3}};
    cout << "Hard Test Expected: 5 | Got: " << getMinDifference(t2, 3) << "\n\n";
    return 0;
}

```
## 6. Stone Removal Cost
### 📝 Problem Description
Find the minimum cost to remove a sequence of N stones sequentially. Removing stone i costs A[i] if exactly 1 immediate neighbor is still present, B[i] if 2 exist, and 0 if 0 exist.
### 💡 LeetCode Style Examples
**Example 1:**
 * **Input:** A = [1, 3, 2, 4], B = [5, 6, 7, 8]
 * **Output:** 3
 * **Explanation:** DP handles the optimal sequential deletion from left to right, determining neighbor status dynamically.
### 👨‍🏫 Tutor's Corner: The Pattern
> **Why this approach?** This is a **1D DP State Machine**.
> A stone's cost relies on its left and right neighbors. Processing left to right, we don't know the future of the right neighbor! So, we create two parallel universes: dp[0] (universe where right neighbor is alive) and dp[1] (universe where right neighbor is already dead).
> 
### 🛠️ Step-by-Step Solution
 1. dp[0][i]: Min cost clearing stones up to index i, assuming stone i+1 is still present.
 2. dp[1][i]: Min cost clearing stones up to index i, assuming stone i+1 has already been removed.
 3. For dp[0][i], the right neighbor is alive (+1 neighbor). Look at i-1. Was it removed before i? If yes, i has 1 neighbor total \rightarrow cost A. If after, i has 2 neighbors \rightarrow cost B.
 4. Apply the same logic for dp[1][i] but assume the right neighbor is dead.
### 💻 C++ Code & Test Cases
```cpp
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

long long getMinRemovalCost(int n, vector<long long>& A, vector<long long>& B) {
    vector<vector<long long>> dp(2, vector<long long>(n, 0));

    // Base Case: Stone 0
    dp[0][0] = A[0]; // Right exists, left doesn't -> 1 neighbor
    dp[1][0] = 0;    // Right removed, left doesn't -> 0 neighbors

    for (int i = 1; i < n; i++) {
        // STATE 0: Right neighbor exists
        long long c1 = dp[1][i-1] + A[i]; // Left removed earlier -> 1 neighbor total (A)
        long long c2 = dp[0][i-1] + B[i]; // Left removed later -> 2 neighbors total (B)
        dp[0][i] = min(c1, c2);

        // STATE 1: Right neighbor removed
        long long c3 = dp[1][i-1] + 0;    // Left removed earlier -> 0 neighbors (0)
        long long c4 = dp[0][i-1] + B[i]; // Left removed later -> 1 neighbor (Rule specifies B for this state variant)
        dp[1][i] = min(c3, c4);
    }
    
    // The very last stone eventually has its right neighbor "removed" (since it doesn't exist)
    return dp[0][n-1];
}

int main() {
    cout << "--- 6. Stone Removal Cost ---\n";
    vector<long long> A1 = {3, 2}, B1 = {5, 4};
    cout << "Easy Test Expected: 2 | Got: " << getMinRemovalCost(2, A1, B1) << "\n";

    vector<long long> A2 = {1, 3, 2, 4}, B2 = {5, 6, 7, 8};
    cout << "Medium Test Expected: 3 | Got: " << getMinRemovalCost(4, A2, B2) << "\n\n";
    return 0;
}

```
## 7. String Merge
### 📝 Problem Description
You receive an array of stringified integers. You can merge arr[i] and arr[j] if i < j and last_digit[i] == first_digit[j]. Find the max length of a final sequence where ultimate_first_digit == ultimate_last_digit.
### 💡 LeetCode Style Examples
**Example 1:**
 * **Input:** arr = ["12", "21", "99"]
 * **Output:** 4
 * **Explanation:** "12" ends with 2, "21" starts with 2. Merge -> "1221". Length 4. First == Last == 1.
### 👨‍🏫 Tutor's Corner: The Pattern
> **Why this approach?** This is a direct **Longest Increasing Subsequence (LIS)** variant.
> Instead of looking for arr[i] > arr[j], our connection rule is simply first_digit[i] == last_digit[j]. The DP structure is identical to standard LIS! dp[i] depends on finding the best valid dp[j] that came before it.
> 
### 🛠️ Step-by-Step Solution
 1. Preprocess: Extract first_digit, last_digit, and length for every string.
 2. Initialize dp[i] = length[i] (a string on its own is a valid subchain).
 3. **LIS Inner Loop:** For each i, loop backwards j < i. If first_digit[i] == last_digit[j], string i can attach to the end of chain j. Update dp[i] = max(dp[i], dp[j] + len[i]).
 4. Validate: If first_digit[i] == last_digit[i], verify against global max_answer.
### 💻 C++ Code & Test Cases
```cpp
#include <vector>
#include <algorithm>
#include <string>
#include <iostream>
using namespace std;

int maxMergedStringLength(const vector<string>& arr) {
    int n = arr.size();
    vector<int> first(n), last(n), len(n), dp(n);
    int max_ans = 0;

    for(int i = 0; i < n; i++) {
        first[i] = arr[i].front() - '0'; 
        last[i]  = arr[i].back()  - '0'; 
        len[i]   = arr[i].length();
        dp[i]    = len[i]; 

        // LIS logic: Look back and find best valid attachment
        for(int j = 0; j < i; j++) {
            if(first[i] == last[j]) {
                dp[i] = max(dp[i], dp[j] + len[i]);
            }
        }

        // Validity check: chain must start and end with same digit
        if(first[i] == last[i]) {
            max_ans = max(max_ans, dp[i]);
        }
    }
    return max_ans;
}

int main() {
    cout << "--- 7. String Merge ---\n";
    vector<string> arr1 = {"12", "21", "99"};
    cout << "Easy Test Expected: 4 | Got: " << maxMergedStringLength(arr1) << "\n";

    vector<string> arr2 = {"13", "31", "13", "31", "11"};
    cout << "Medium Test Expected: 8 | Got: " << maxMergedStringLength(arr2) << "\n\n";
    return 0;
}

```
## 8. Array Score Maximization
### 📝 Problem Description
Given two arrays A and B. An element scores 1 point if it is \le D, and 2 points if > D. Find the threshold D to maximize the difference Total_Score_A - Total_Score_B.
### 💡 LeetCode Style Examples
**Example 1:**
 * **Input:** A = [5, 6], B = [1, 2]
 * **Output:** 2
 * **Explanation:** If D = 4: A elements are all > 4 (score 4), B elements are all <= 4 (score 2). Diff = 2.
### 👨‍🏫 Tutor's Corner: The Pattern
> **Why this approach?** The infinite search space for D can be condensed! The optimal D will ALWAYS be exactly equal to one of the numbers currently residing in A or B.
> Collect them, sort them, and use std::upper_bound to instantly find how many elements are \le D in O(\log N) time.
> 
### 🛠️ Step-by-Step Solution
 1. Collect all elements of A and B into a candidates array. Sort and remove duplicates.
 2. Sort A and B separately.
 3. For each D in candidates:
   * Find iterator idx using upper_bound in A. Subtracting .begin() gives the COUNT of elements \le D.
   * Score = (idx * 1) + ((total_size - idx) * 2).
   * Calculate difference vs B and maximize.
### 💻 C++ Code & Test Cases
```cpp
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

long long maximizeScoreDifference(vector<int>& A, vector<int>& B) {
    sort(A.begin(), A.end());
    sort(B.begin(), B.end());

    // Coordinate Compression: Collect all valid thresholds
    vector<int> cands = A;
    cands.insert(cands.end(), B.begin(), B.end());
    sort(cands.begin(), cands.end());
    cands.erase(unique(cands.begin(), cands.end()), cands.end());

    long long max_diff = -1e18;

    for (int D : cands) {
        // upper_bound returns iterator to first element > D.
        // Distance from begin() is the COUNT of elements <= D!
        int a_leq = upper_bound(A.begin(), A.end(), D) - A.begin();
        long long score_A = a_leq * 1LL + (A.size() - a_leq) * 2LL;

        int b_leq = upper_bound(B.begin(), B.end(), D) - B.begin();
        long long score_B = b_leq * 1LL + (B.size() - b_leq) * 2LL;

        max_diff = max(max_diff, score_A - score_B); 
    }
    return max_diff;
}

int main() {
    cout << "--- 8. Array Score Maximization ---\n";
    vector<int> A1 = {5, 6}, B1 = {1, 2};
    cout << "Easy Test Expected: 2 | Got: " << maximizeScoreDifference(A1, B1) << "\n";

    vector<int> A2 = {1,2,3,4,5}, B2 = {6,7,8,9,10};
    cout << "Medium Test Expected: 0 | Got: " << maximizeScoreDifference(A2, B2) << "\n\n";
    return 0;
}

```
