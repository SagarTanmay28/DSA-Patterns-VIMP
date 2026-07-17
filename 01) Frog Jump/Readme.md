
# Frog Jump — All DP Variants (C++)

There isn't just one "Frog Jump" problem — it's a family. Here's every common variant, each with **Recursion → Memoization → Tabulation (Bottom-Up)**, plus space optimization where it applies.

---

## VARIANT 1: Classic Frog Jump (1 or 2 steps)

**Problem:** Frog is at stone 0, wants to reach stone n-1. It can jump to `i+1` or `i+2`. Cost of a jump = `abs(height[i] - height[j])`. Find **minimum total cost**.

**Core idea:** `dp[i]` = min cost to reach stone `i`.
`dp[i] = min( dp[i-1] + |h[i]-h[i-1]| , dp[i-2] + |h[i]-h[i-2]| )`

### 1. Recursion
```cpp
int solve(int i, vector<int>& h) {
    if (i == 0) return 0;               // base case: already at start
    int left = solve(i-1, h) + abs(h[i]-h[i-1]);
    int right = INT_MAX;
    if (i > 1)
        right = solve(i-2, h) + abs(h[i]-h[i-2]);
    return min(left, right);
}
// call: solve(n-1, h)
```
Exponential time — recomputes same states again and again.

### 2. Memoization (Top-Down)
```cpp
int solveMemo(int i, vector<int>& h, vector<int>& dp) {
    if (i == 0) return 0;
    if (dp[i] != -1) return dp[i];       // already computed

    int left = solveMemo(i-1, h, dp) + abs(h[i]-h[i-1]);
    int right = INT_MAX;
    if (i > 1)
        right = solveMemo(i-2, h, dp) + abs(h[i]-h[i-2]);

    return dp[i] = min(left, right);
}
// call:
// vector<int> dp(n, -1);
// solveMemo(n-1, h, dp);
```
Same logic as recursion, but we **store** each answer so it's computed once. O(n) time, O(n) space (dp array + recursion stack).

### 3. Tabulation (Bottom-Up)
```cpp
int frogJumpTab(vector<int>& h) {
    int n = h.size();
    vector<int> dp(n);
    dp[0] = 0;                           // base case

    for (int i = 1; i < n; i++) {
        int left = dp[i-1] + abs(h[i]-h[i-1]);
        int right = INT_MAX;
        if (i > 1) right = dp[i-2] + abs(h[i]-h[i-2]);
        dp[i] = min(left, right);
    }
    return dp[n-1];
}
```
No recursion stack. O(n) time, O(n) space.

### 4. Space Optimized
We only ever need the last two values.
```cpp
int frogJumpOptimal(vector<int>& h) {
    int n = h.size();
    int prev = 0, prev2 = 0;

    for (int i = 1; i < n; i++) {
        int left = prev + abs(h[i]-h[i-1]);
        int right = INT_MAX;
        if (i > 1) right = prev2 + abs(h[i]-h[i-2]);
        int cur = min(left, right);
        prev2 = prev;
        prev = cur;
    }
    return prev;
}
```
O(n) time, **O(1) space**.

---

## VARIANT 2: Frog Jump with K distances

**Problem:** Same as above, but frog can jump `1, 2, ..., K` steps ahead. Minimize cost.

`dp[i] = min over j=1..K of ( dp[i-j] + |h[i]-h[i-j]| )` for valid `i-j >= 0`.

### Recursion
```cpp
int solveK(int i, int k, vector<int>& h) {
    if (i == 0) return 0;
    int best = INT_MAX;
    for (int j = 1; j <= k; j++) {
        if (i - j >= 0) {
            int cost = solveK(i-j, k, h) + abs(h[i]-h[i-j]);
            best = min(best, cost);
        }
    }
    return best;
}
```

### Memoization
```cpp
int solveKMemo(int i, int k, vector<int>& h, vector<int>& dp) {
    if (i == 0) return 0;
    if (dp[i] != -1) return dp[i];

    int best = INT_MAX;
    for (int j = 1; j <= k; j++) {
        if (i - j >= 0) {
            int cost = solveKMemo(i-j, k, h, dp) + abs(h[i]-h[i-j]);
            best = min(best, cost);
        }
    }
    return dp[i] = best;
}
```

### Tabulation
```cpp
int frogJumpKTab(vector<int>& h, int k) {
    int n = h.size();
    vector<int> dp(n, 0);

    for (int i = 1; i < n; i++) {
        int best = INT_MAX;
        for (int j = 1; j <= k; j++) {
            if (i - j >= 0) {
                int cost = dp[i-j] + abs(h[i]-h[i-j]);
                best = min(best, cost);
            }
        }
        dp[i] = best;
    }
    return dp[n-1];
}
```
O(n*k) time, O(n) space. (No O(1)-space version here since we need up to K previous values, not just 2 — you *could* keep a sliding window of size K if needed.)

---

## VARIANT 3: Min Cost Climbing Stairs (a close cousin)

**Problem:** `cost[i]` = cost of stepping on stair `i`. You can climb 1 or 2 steps. Start from index 0 or 1. Find min cost to reach top (beyond last stair).

### Recursion
```cpp
int solveMCS(int i, vector<int>& cost) {
    if (i <= 1) return 0;
    int one = cost[i-1] + solveMCS(i-1, cost);
    int two = cost[i-2] + solveMCS(i-2, cost);
    return min(one, two);
}
// call: solveMCS(n, cost)  where n = cost.size()
```

### Memoization
```cpp
int solveMCSMemo(int i, vector<int>& cost, vector<int>& dp) {
    if (i <= 1) return 0;
    if (dp[i] != -1) return dp[i];
    int one = cost[i-1] + solveMCSMemo(i-1, cost, dp);
    int two = cost[i-2] + solveMCSMemo(i-2, cost, dp);
    return dp[i] = min(one, two);
}
```

### Tabulation
```cpp
int minCostClimbingStairs(vector<int>& cost) {
    int n = cost.size();
    vector<int> dp(n+1, 0);
    for (int i = 2; i <= n; i++)
        dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2]);
    return dp[n];
}
```

### Space Optimized
```cpp
int minCostOptimal(vector<int>& cost) {
    int n = cost.size();
    int prev2 = 0, prev1 = 0;
    for (int i = 2; i <= n; i++) {
        int cur = min(prev1 + cost[i-1], prev2 + cost[i-2]);
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

---

## VARIANT 4: Min Jumps to Reach End (LeetCode "Jump Game II")

**Problem:** `nums[i]` = max jump length from index `i`. Find **minimum number of jumps** to reach the last index (not cost — count of jumps).

### Recursion
```cpp
int solveJumps(int i, vector<int>& nums) {
    int n = nums.size();
    if (i >= n-1) return 0;
    int best = INT_MAX;
    for (int step = 1; step <= nums[i]; step++) {
        if (i + step < n) {
            int res = solveJumps(i+step, nums);
            if (res != INT_MAX) best = min(best, res+1);
        }
    }
    return best;
}
```

### Memoization
```cpp
int solveJumpsMemo(int i, vector<int>& nums, vector<int>& dp) {
    int n = nums.size();
    if (i >= n-1) return 0;
    if (dp[i] != -1) return dp[i];

    int best = INT_MAX;
    for (int step = 1; step <= nums[i]; step++) {
        if (i + step < n) {
            int res = solveJumpsMemo(i+step, nums, dp);
            if (res != INT_MAX) best = min(best, res+1);
        }
    }
    return dp[i] = best;
}
```

### Tabulation
```cpp
int jumpTab(vector<int>& nums) {
    int n = nums.size();
    vector<int> dp(n, INT_MAX);
    dp[n-1] = 0;

    for (int i = n-2; i >= 0; i--) {
        for (int step = 1; step <= nums[i] && i+step < n; step++) {
            if (dp[i+step] != INT_MAX)
                dp[i] = min(dp[i], dp[i+step] + 1);
        }
    }
    return dp[0];
}
```
Note: this DP is O(n²) worst case. The optimal solution for this exact problem is actually a **greedy O(n)** approach (not DP) — ask if you want that too.

---

## VARIANT 5: Frog Jump on Stones (LeetCode 403 — "Can the Frog Cross?")

**Problem:** Frog is on stone 0. Stones are at given positions. If last jump was size `k`, next jump must be `k-1`, `k`, or `k+1` (and > 0). Can frog reach the last stone? (Boolean DP, not cost).

This one needs **state = (position, last jump size)**.

### Recursion
```cpp
bool solveCross(int idx, int jumpSize, vector<int>& stones, unordered_map<int,int>& pos) {
    int n = stones.size();
    if (idx == n-1) return true;

    for (int newJump : {jumpSize-1, jumpSize, jumpSize+1}) {
        if (newJump <= 0) continue;
        int nextStone = stones[idx] + newJump;
        if (pos.count(nextStone)) {
            if (solveCross(pos[nextStone], newJump, stones, pos))
                return true;
        }
    }
    return false;
}
// call: solveCross(0, 0, stones, pos)  // pos maps stone-value -> index
```

### Memoization
```cpp
bool solveCrossMemo(int idx, int jumpSize, vector<int>& stones,
                     unordered_map<int,int>& pos,
                     map<pair<int,int>, bool>& dp) {
    int n = stones.size();
    if (idx == n-1) return true;

    auto key = make_pair(idx, jumpSize);
    if (dp.count(key)) return dp[key];

    for (int newJump : {jumpSize-1, jumpSize, jumpSize+1}) {
        if (newJump <= 0) continue;
        int nextStone = stones[idx] + newJump;
        if (pos.count(nextStone)) {
            if (solveCrossMemo(pos[nextStone], newJump, stones, pos, dp))
                return dp[key] = true;
        }
    }
    return dp[key] = false;
}
```

### Tabulation
```cpp
bool canCross(vector<int>& stones) {
    int n = stones.size();
    unordered_map<int,int> pos;
    for (int i = 0; i < n; i++) pos[stones[i]] = i;

    // dp[i] = set of jump sizes that can land you ON stone i
    vector<unordered_set<int>> dp(n);
    dp[0].insert(0);

    for (int i = 0; i < n; i++) {
        for (int jumpSize : dp[i]) {
            for (int newJump : {jumpSize-1, jumpSize, jumpSize+1}) {
                if (newJump <= 0) continue;
                int nextStone = stones[i] + newJump;
                if (pos.count(nextStone)) {
                    int j = pos[nextStone];
                    if (j > i) dp[j].insert(newJump);
                }
            }
        }
    }
    return !dp[n-1].empty();
}
```
This one **cannot** be space-optimized to O(1) since state depends on jump history, not just previous index.

---

## Quick summary table

| Variant | State | Transition | Complexity |
|---|---|---|---|
| Classic (1/2 step) | `dp[i]` = min cost to i | from i-1, i-2 | O(n) / O(1) space |
| K-distance | `dp[i]` | from i-1..i-k | O(n·k) |
| Min cost climbing stairs | `dp[i]` | from i-1, i-2 | O(n) / O(1) space |
| Min jumps (Jump Game II) | `dp[i]` = min jumps from i | try all reachable steps | O(n²) DP (O(n) greedy) |
| Frog crossing river | `dp[stone][lastJump]` | jumpSize±1 | O(n²) |

**Pattern to remember across all of them:**
1. Write recursion first — define what `i` (and any extra state) means, and the recurrence.
2. Add an `dp[]` cache + a check "if already computed, return it" → memoization.
3. Convert recursion into a loop going from base case upward, filling the same `dp[]` → tabulation.
4. If transition only needs the last 1–2 states, drop the array for variables → O(1) space.

