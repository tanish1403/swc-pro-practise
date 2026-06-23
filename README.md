# Samsung R&D SWC Professional Track - STL Enhanced Problem Repository
This repository contains problem statements, detailed test case examples, step-by-step algorithmic approaches, explicitly defined Dynamic Programming (DP) states, and highly-optimized **STL-enabled C++ implementations** for advanced competency test preparation.
Using the Standard Template Library (std::vector, std::unordered_map, std::queue, std::multiset, etc.) drastically reduces boilerplate code and allows you to focus entirely on core algorithmic logic.
## Table of Contents
 1. Test 1: Points on a Path
 2. Test 2: Warehouse Inventory (Bitmask DP)
 3. Test 3: Red and Blue Stones (Prefix Sum Hash)
 4. Test 4, Q1: City Logistics (Graph / BFS)
 5. Test 4, Q2: Tiles Selection (Sliding Window / Multiset)
 6. Test 4, Q3: Stone Removal Cost (1D DP)
 7. Test 5, Q1: String Merge (LIS Variant)
 8. Test 5, Q2: Array Score Maximization (Binary Search)
 9. Test 6, Q1: Converging Cars (Math / Parity)
 10. Test 6, Q2: Array Reconstruction (Modulo Math)
 11. Test 7, Q1: Robot Garbage Collection (2D DP)
 12. Test 7, Q2: Gift Certificates (Digit DP)
## Test 1: Points on a Path
**Problem Statement:**
You are given a continuous path on an infinite 2D lattice. The path consists of connected line segments parallel to either the x or y-axis. You are also given a set of N isolated points in space. Find how many of these N points lie directly on any segment of the given path.
**Test Case Example:**
 * **Input:**
   * Path Turning Points: (1, 1) -> (1, 5) -> (2, 5)
   * Given Points: (1, 3), (2, 2), (1, 5), (3, 5)
 * **Logic:**
   * Path creates vertical segment from y=1 to y=5 at x=1.
   * Path creates horizontal segment from x=1 to x=2 at y=5.
   * Point (1, 3) lies on the vertical segment. Point (1, 5) lies on the intersection.
 * **Output:** 2
**Step-by-Step Approach:**
 1. Using std::vector, dynamically store vertical and horizontal line segments.
 2. Parse the turning points. If consecutive points share an x-coordinate, form a vertical segment. If they share a y-coordinate, form a horizontal segment.
 3. Ensure the start coordinate is always smaller than the end coordinate using std::min and std::max.
 4. Iterate through each point and check it against the vectors of segments.
**STL C++ Code:**
```cpp
#include <vector>
#include <algorithm>
using namespace std;

struct Segment {
    int constant_val;
    int start, end;
};

int countPointsOnPath(int N, vector<int>& pX, vector<int>& pY, int M, vector<int>& pathX, vector<int>& pathY) {
    vector<Segment> vertical, horizontal;

    for (int i = 0; i < M - 1; i++) {
        if (pathX[i] == pathX[i+1]) { 
            vertical.push_back({pathX[i], min(pathY[i], pathY[i+1]), max(pathY[i], pathY[i+1])});
        } else { 
            horizontal.push_back({pathY[i], min(pathX[i], pathX[i+1]), max(pathX[i], pathX[i+1])});
        }
    }

    int points_on_path = 0;
    for (int i = 0; i < N; i++) {
        int px = pX[i], py = pY[i];
        bool found = false;

        for (const auto& v : vertical) {
            if (px == v.constant_val && py >= v.start && py <= v.end) {
                points_on_path++; found = true; break;
            }
        }
        if (found) continue;

        for (const auto& h : horizontal) {
            if (py == h.constant_val && px >= h.start && px <= h.end) {
                points_on_path++; break;
            }
        }
    }
    return points_on_path;
}

```
## Test 2: Warehouse Inventory (Bitmask DP)
**Problem Statement:**
A warehouse has an initial stock array A (size N). Every day, a shipment arrives, increasing stock by array B (size N), meaning A[i] += B[i]. At the end of every day, you can choose exactly one item to export, instantly reducing its stock to 0. Find the minimum number of days required to make the total warehouse stock <= K. If impossible, return -1.
**Test Case Example:**
 * **Input:** N=2, A=[1, 2], B=[2, 1], K=4
 * **Logic:** * Day 1 Start: A = [1+2, 2+1] = [3, 3]. Total = 6.
   * End of Day 1: Export item 0. A = [0, 3]. Total = 3.
   * Target condition Total <= 4 is met!
 * **Output:** 1
**DP States:**
 * **mask**: Integer representing the subset of items exported (bits set to 1).
 * **dp[count]**: Max stock we can remove globally using exactly count days (removing count items).
**Step-by-Step Approach:**
 1. Iterate bitmasks from 1 to (2^N - 1).
 2. Use __builtin_ctz to quickly find the first active bit representing the newly removed item.
 3. Compute total removed stock by utilizing previously calculated subset states.
 4. Use __builtin_popcount to count the days/items removed for this mask.
 5. Check total stock incrementally day-by-day and subtract the best DP state.
**STL C++ Code:**
```cpp
#include <vector>
#include <algorithm>
#include <numeric>
using namespace std;

int getMinimumDays(int n, vector<int>& A, vector<int>& B, long long K) {
    vector<int> total_removed(1 << n, 0);
    vector<int> subset_sum(1 << n, 0);
    vector<int> dp(n + 1, 0); 
    
    for (int mask = 1; mask < (1 << n); mask++) {
        int idx = __builtin_ctz(mask); // STL-adjacent GCC builtin
        int prev_mask = mask ^ (1 << idx);
        
        total_removed[mask] = total_removed[prev_mask] + subset_sum[prev_mask] + A[idx] + B[idx];
        subset_sum[mask] = subset_sum[prev_mask] + B[idx];
        
        int count = __builtin_popcount(mask);
        dp[count] = max(dp[count], total_removed[mask]);
    }
    
    long long A_tot = accumulate(A.begin(), A.end(), 0LL);
    long long B_tot = accumulate(B.begin(), B.end(), 0LL);
    
    for (int day = 1; day <= n; day++) {
        if ((A_tot + day * B_tot) - dp[day] <= K) return day;
    }
    return -1;
}

```
## Test 3: Red and Blue Stones (Prefix Sum Hash)
**Problem Statement:**
Given a necklace of 'R' and 'B' stones (as a string), make the counts equal by removing the minimum number of stones strictly from the left or right ends.
**Test Case Example:**
 * **Input:** BBRRBRBRBRBBR (Length: 13, 8 B's, 5 R's)
 * **Logic:** The subarray BBRRBRBRBRBB (indices 0 to 11) has 6 B's and 6 R's. Length is 12. Total length (13) - Max balanced length (12) = 1 removal.
 * **Output:** 1
**Step-by-Step Approach:**
 1. Treat 'B' as +1 and 'R' as -1. Target: longest contiguous subarray summing to 0.
 2. Track a current_sum sequentially.
 3. Utilize std::unordered_map to store the *first occurrence* of every prefix sum. This completely removes the need for array offsets and bounds checking.
 4. If sum is seen again, update max balanced length.
**STL C++ Code:**
```cpp
#include <string>
#include <unordered_map>
#include <algorithm>
using namespace std;

int getMinRemovals(const string& s) {
    unordered_map<int, int> first_occ;
    first_occ[0] = -1; // Sum 0 occurs conceptually at index -1
    
    int current_sum = 0, max_length = 0;
    
    for(int i = 0; i < s.length(); ++i) {
        current_sum += (s[i] == 'B' ? 1 : -1);
        
        if(first_occ.count(current_sum)) {
            max_length = max(max_length, i - first_occ[current_sum]);
        } else {
            first_occ[current_sum] = i;
        }
    }
    
    return s.length() - max_length;
}

```
## Test 4, Q1: City Logistics (Graph / BFS)
**Problem Statement:**
A grid represents a city (Road=0, Tree=1, Garage=2, Warehouse=3, Airport=4). A truck starts at Garage, loads goods from Warehouses, and unloads at Airport. Moving one block costs 1 + number of goods carried. Trees are impassable. Find the maximum goods unloaded at the Airport given a budget C.
**Test Case Example:**
 * **Input:** Budget C = 5. Grid:
   ```
   [2, 0, 3]
   [1, 1, 0]
   [4, 0, 0]
   
   ```
 * **Output:** 0 (Can reach airport but cannot afford carrying goods due to high cost multiplier).
**Step-by-Step Approach:**
 1. Use std::queue to handle state-space BFS cleanly.
 2. Use a 3D std::vector to track the minimum cost to reach (x, y) with exactly goods carried.
 3. At a warehouse (3), branch the BFS state: push a state where we load a good, and a state where we just pass through.
 4. At the airport (4), track the max goods delivered strictly within budget.
**STL C++ Code:**
```cpp
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

struct State { int x, y, goods, cost; };
int dx[] = {-1, 1, 0, 0}; int dy[] = {0, 0, -1, 1};

int getMaxGoods(int h, int w, vector<vector<int>>& city, int max_cost) {
    vector<vector<vector<int>>> min_cost(h, vector<vector<int>>(w, vector<int>(20, 1e9)));
    queue<State> q;
    
    for(int i=0; i<h; ++i) {
        for(int j=0; j<w; ++j) {
            if(city[i][j] == 2) { 
                q.push({i, j, 0, 0}); 
                min_cost[i][j][0] = 0; 
            }
        }
    }
    
    int max_delivered = 0;
    while(!q.empty()) {
        auto [x, y, goods, cost] = q.front(); q.pop();
        
        if(city[x][y] == 4 && cost <= max_cost) max_delivered = max(max_delivered, goods);
        
        for(int i=0; i<4; ++i) {
            int nx = x + dx[i], ny = y + dy[i];
            if(nx >= 0 && nx < h && ny >= 0 && ny < w && city[nx][ny] != 1) {
                int next_cost = cost + 1 + goods;
                if(next_cost <= max_cost) {
                    if(next_cost < min_cost[nx][ny][goods]) {
                        min_cost[nx][ny][goods] = next_cost;
                        q.push({nx, ny, goods, next_cost});
                    }
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

```
## Test 4, Q2: Tiles Selection (Sliding Window / Multiset)
**Problem Statement:**
Given N tiles of varying widths and heights. Select K tiles to minimize the maximum difference between any two selected tiles. Difference between two tiles is defined as max(|h1-h2|, |w1-w2|) globally.
**Test Case Example:**
 * **Input:** N=4, K=2. Tiles (w,h): (1, 5), (2, 4), (6, 1), (8, 2)
 * **Output:** 1 (Selecting tiles (1,5) and (2,4) gives max diff of 1).
**Step-by-Step Approach:**
 1. Use std::sort to order tiles ascending by width.
 2. Binary Search on the possible answer D.
 3. Inside the validation function, use a sliding window with std::multiset to automatically keep heights sorted within the current valid width window.
 4. Check *heights.rbegin() - *heights.begin() <= D to instantly verify if the K items satisfy the height condition.
**STL C++ Code:**
```cpp
#include <vector>
#include <algorithm>
#include <set>
using namespace std;

bool isPossible(vector<pair<int,int>>& tiles, int k, int max_diff) {
    multiset<int> heights;
    int left = 0;
    
    for (int right = 0; right < tiles.size(); ++right) {
        heights.insert(tiles[right].second);
        
        while (tiles[right].first - tiles[left].first > max_diff) {
            heights.erase(heights.find(tiles[left].second));
            left++;
        }
        
        if (heights.size() >= k) {
            // Using multiset allows us to quickly check the min/max height in K window
            auto it = heights.begin();
            auto end_it = std::next(it, k - 1);
            
            while(end_it != heights.end()) {
                if (*end_it - *it <= max_diff) return true;
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
        if(isPossible(tiles, k, mid)) { ans = mid; high = mid - 1; } 
        else { low = mid + 1; }
    }
    return ans;
}

```
## Test 4, Q3: Stone Removal Cost (1D DP)
**Problem Statement:**
Find minimum cost to remove a sequence of N stones sequentially. Removing stone i costs A[i] if exactly 1 immediate neighbor is still present, B[i] if 2 exist, and 0 if 0 exist.
**DP States:**
 * **dp[0][i]**: Min cost clearing stones up to index i, assuming stone i+1 has not been removed yet.
 * **dp[1][i]**: Min cost clearing stones up to index i, assuming stone i+1 has already been removed.
**Step-by-Step Approach:**
 1. Base cases: dp[0][0] = A[0], dp[1][0] = 0.
 2. For dp[0][i] (right exists): If left removed before i, cost = A. If left removed after i, cost = B.
 3. For dp[1][i] (right gone): If left removed before i, cost = 0. If left removed after i, cost = B.
**STL C++ Code:**
```cpp
#include <vector>
#include <algorithm>
using namespace std;

long long getMinRemovalCost(int n, vector<long long>& A, vector<long long>& B) {
    vector<vector<long long>> dp(2, vector<long long>(n, 0)); 
    dp[0][0] = A[0]; 
    dp[1][0] = 0;    
    
    for (int i = 1; i < n; i++) {
        long long c1 = dp[1][i-1] + A[i];
        long long c2 = dp[0][i-1] + B[i];
        dp[0][i] = min(c1, c2);
        
        long long c3 = dp[1][i-1] + 0;
        long long c4 = dp[0][i-1] + B[i]; 
        dp[1][i] = min(c3, c4);
    }
    return dp[0][n-1];
}

```
## Test 5, Q1: String Merge (LIS Variant)
**Problem Statement:**
You receive an array of stringified integers. You can merge arr[i] and arr[j] if i < j and last_digit[i] == first_digit[j]. Find the max length of a final sequence where ultimate_first_digit == ultimate_last_digit.
**DP States:**
 * **dp[i]**: Absolute maximum merged string length generated ending exactly at arr[i].
**Step-by-Step Approach:**
 1. Determine first_digit, last_digit, and string length mathematically.
 2. Initialize dp[i] = length[i].
 3. For each i, loop backwards j < i. If first_digit[i] == last_digit[j], update dp[i] = max(dp[i], dp[j] + length[i]).
**STL C++ Code:**
```cpp
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

int maxMergedStringLength(const vector<string>& arr) {
    int n = arr.size();
    vector<int> first(n), last(n), len(n), dp(n);
    int max_ans = 0;

    for(int i = 0; i < n; i++) {
        first[i] = arr[i].front() - '0';
        last[i] = arr[i].back() - '0';
        len[i] = arr[i].length();
        dp[i] = len[i]; 
        
        for(int j = 0; j < i; j++) {
            if(first[i] == last[j]) {
                dp[i] = max(dp[i], dp[j] + len[i]);
            }
        }
        if(first[i] == last[i]) {
            max_ans = max(max_ans, dp[i]);
        }
    }
    return max_ans;
}

```
## Test 5, Q2: Array Score Maximization (Binary Search)
**Problem Statement:**
Given two arrays A and B. An element scores 1 point if it is <= D, and 2 points if > D. Find the threshold D to maximize the difference Total_Score_A - Total_Score_B.
**Test Case Example:**
 * **Input:** A = [1, 2, 3, 4, 5], B = [6, 7, 8, 9, 10]
 * **Output:** 0 (Forces all to score 2, tying the score at 10-10=0, the max diff).
**Step-by-Step Approach:**
 1. Use std::sort on both arrays.
 2. Collect all unique elements into a single candidate vector and use std::unique to deduplicate.
 3. Instead of a manual binary search, use std::upper_bound to instantly find the count of elements <= D.
 4. Compute the score difference and track the maximum.
**STL C++ Code:**
```cpp
#include <vector>
#include <algorithm>
using namespace std;

long long maximizeScoreDifference(vector<int>& A, vector<int>& B) {
    sort(A.begin(), A.end());
    sort(B.begin(), B.end());
    
    vector<int> cands = A;
    cands.insert(cands.end(), B.begin(), B.end());
    sort(cands.begin(), cands.end());
    cands.erase(unique(cands.begin(), cands.end()), cands.end());
    
    long long max_diff = -1e18; 
    
    for (int D : cands) {
        int a_leq = upper_bound(A.begin(), A.end(), D) - A.begin();
        long long score_A = a_leq * 1LL + (A.size() - a_leq) * 2LL;
        
        int b_leq = upper_bound(B.begin(), B.end(), D) - B.begin();
        long long score_B = b_leq * 1LL + (B.size() - b_leq) * 2LL;
        
        max_diff = max(max_diff, score_A - score_B);
    }
    return max_diff;
}

```
## Test 6, Q1: Converging Cars (Math / Parity)
**Problem Statement:**
N cars on a 2D plane need to reach (p,q) simultaneously. Drive T provides exactly T continuous moves. You can overshoot and retrace. Find the minimum number of total drives required for all cars to sync at the destination. Return -1 if impossible.
**Step-by-Step Approach:**
 1. Compute Manhattan distance for a car: dist = abs(px - car_x) + abs(py - car_y).
 2. Find minimum drives n where arithmetic sum (1+2+...+n) >= dist.
 3. Check diff = sum - dist. Because overshooting must be retraced (taking 2 moves), diff must be even.
 4. If diff is odd, increment n until the resulting sum yields an even difference.
 5. All cars must share the same even/odd parity in their required drives to sync.
**STL C++ Code:**
```cpp
#include <vector>
#include <cmath>
#include <algorithm>
using namespace std;

long long getMinimumDrives(int n, long long px, long long py, vector<long long>& cx, vector<long long>& cy) {
    vector<long long> required_drives(n);
    long long max_drives = 0;
    
    for(int i = 0; i < n; i++) {
        long long dist = abs(px - cx[i]) + abs(py - cy[i]);
        long long drives = 0;
        long long sum_moves = 0;
        
        while(sum_moves < dist) {
            drives++;
            sum_moves += drives;
        }
        
        long long diff = sum_moves - dist;
        if(diff % 2 != 0) {
            if((drives + 1) % 2 != 0) drives += 1; 
            else drives += 2; 
        }
        
        required_drives[i] = drives;
        max_drives = max(max_drives, drives);
    }
    
    for(int i = 0; i < n; i++) {
        if((required_drives[i] % 2) != (max_drives % 2)) {
            if (max_drives % 2 == 0) max_drives += 1; 
        }
    }
    return max_drives;
}

```
## Test 6, Q2: Array Reconstruction (Modulo Math)
**Problem Statement:**
Can Array A transform into Array B? Allowed operations on A: Add/subtract e from any element indefinitely, and reorder the array.
**Step-by-Step Approach:**
 1. Transforming by e implies comparing elements modulo e.
 2. Modulo e all elements in A and B in-place using ranged-for loops. Add e and modulo again for negatives.
 3. Use std::sort to order both arrays.
 4. Use the built-in vector equality operator A == B to verify identical frequencies.
**STL C++ Code:**
```cpp
#include <vector>
#include <algorithm>
using namespace std;

bool canTransform(int e, vector<int>& A, vector<int>& B) {
    if (A.size() != B.size()) return false;
    
    for(int& x : A) x = ((x % e) + e) % e;
    for(int& x : B) x = ((x % e) + e) % e;
    
    sort(A.begin(), A.end());
    sort(B.begin(), B.end());
    
    return A == B; // STL allows direct vector comparison
}

```
## Test 7, Q1: Robot Garbage Collection (2D DP)
**Problem Statement:**
An array holds garbage values. Deploying a robot costs fixed m. A robot cleans i, then moves to i+1. The penalty cost at any step is the sum of all uncleaned garbage sitting ahead of the robot. Find min total cost to clear array.
**DP States:**
 * **dp[i][j]**: Min total cost accumulated up to index i, knowing the currently active robot started its run back at index j.
**Step-by-Step Approach:**
 1. Use std::vector for dynamic 2D DP table and prefix sums.
 2. Choice 1: Active robot cleans i. Penalty is uncleaned garbage between j and i.
 3. Choice 2: Pay m to drop new robot at i. Base cost is min over dp[i-1][k].
**STL C++ Code:**
```cpp
#include <vector>
#include <algorithm>
using namespace std;

long long getMinGarbageCost(int m, const vector<long long>& garbage) {
    int n = garbage.size();
    vector<vector<long long>> dp(n, vector<long long>(n, 1e18));
    vector<long long> pref(n, 0);
    
    pref[0] = garbage[0];
    for(int i=1; i<n; ++i) pref[i] = pref[i-1] + garbage[i];
    
    dp[0][0] = m;
    
    for(int i=1; i<n; ++i) {
        for(int j=0; j<=i; ++j) {
            if (j == i) {
                long long min_prev = 1e18;
                for(int p=0; p<i; ++p) min_prev = min(min_prev, dp[i-1][p]);
                dp[i][i] = min_prev + m;
            } else {
                dp[i][j] = dp[i-1][j] + (pref[i] - pref[j]);
            }
        }
    }
    
    long long ans = 1e18;
    for(int j=0; j<n; ++j) ans = min(ans, dp[n-1][j]);
    return ans;
}

```
## Test 7, Q2: Gift Certificates (Digit DP)
**Problem Statement:**
Calculate how many numbers from 1 up to A (where A is massive, up to 10^100 represented as a string) have a digit sum exactly equal to S. Print modulo 10^9 + 7.
**DP States:**
 * **dp[i][j]**: The number of mathematically valid sequences of exactly i digits that sum up to exactly j.
**Step-by-Step Approach:**
 1. Precompute dp table dynamically using std::vector. Base: dp[0][0] = 1.
 2. For numbers with fewer digits than A, sum dp[len-1][S - first_digit] (ignoring leading zeros).
 3. For numbers matching the exact length of A, enforce the limit by reading the string prefix logic.
**STL C++ Code:**
```cpp
#include <string>
#include <vector>
using namespace std;

#define MOD 1000000007

int countCertificates(const string& A, int S) {
    int n = A.length();
    vector<vector<int>> dp(n + 1, vector<int>(S + 1, 0));
    dp[0][0] = 1; 
    
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= S; j++) {
            for (int k = 0; k <= 9; k++) {
                if (j >= k) dp[i][j] = (dp[i][j] + dp[i-1][j-k]) % MOD;
            }
        }
    }
    
    long long ans = 0;
    
    for (int len = 1; len < n; len++) {
        for (int first_digit = 1; first_digit <= 9; first_digit++) {
            if (S >= first_digit) ans = (ans + dp[len-1][S - first_digit]) % MOD;
        }
    }
    
    int current_sum = 0;
    for (int i = 0; i < n; i++) {
        int limit = A[i] - '0';
        int start_digit = (i == 0) ? 1 : 0; 
        
        for (int d = start_digit; d < limit; d++) {
            int remaining_sum = S - current_sum - d;
            if (remaining_sum >= 0) ans = (ans + dp[n - i - 1][remaining_sum]) % MOD;
        }
        current_sum += limit;
    }
    
    if (current_sum == S) ans = (ans + 1) % MOD;
    
    return ans;
}

```
