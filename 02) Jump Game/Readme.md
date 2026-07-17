# Jump Game — All Variants (C++)

The "Jump Game" series on LeetCode has 7 problems. They share a name but test very different skills — reachability, min steps, max score, graph BFS, etc. Here's all of them, each explained with **Recursion → Memoization → Tabulation**, and a note when a non-DP approach (greedy/BFS) is actually the optimal one.

---

## JUMP GAME I (LC 55) — Can you reach the last index?

**Problem:** `nums[i]` = max jump length from index `i`. Starting at index 0, return `true` if you can reach the last index.

**State:** `i` = current index. Question: can we reach the end from here?

### Recursion
```cpp
bool solve(int i, vector<int>& nums) {
    int n = nums.size();
    if (i >= n - 1) return true;          // already at/past last index
    for (int step = 1; step <= nums[i]; step++) {
        if (solve(i + step, nums)) return true;
    }
    return false;
}
```

### Memoization
```cpp
bool solveMemo(int i, vector<int>& nums, vector<int>& dp) {
    int n = nums.size();
    if (i >= n - 1) return true;
    if (dp[i] != -1) return dp[i];

    for (int step = 1; step <= nums[i]; step++) {
        if (solveMemo(i + step, nums, dp)) return dp[i] = true;
    }
    return dp[i] = false;
}
```

### Tabulation
```cpp
bool canJump(vector<int>& nums) {
    int n = nums.size();
    vector<bool> dp(n, false);
    dp[n-1] = true;

    for (int i = n-2; i >= 0; i--) {
        for (int step = 1; step <= nums[i] && i+step < n; step++) {
            if (dp[i+step]) { dp[i] = true; break; }
        }
    }
    return dp[0];
}
```
O(n²) worst case.

### Better: Greedy O(n) (the actual optimal solution)
```cpp
bool canJumpGreedy(vector<int>& nums) {
    int n = nums.size();
    int maxReach = 0;
    for (int i = 0; i < n; i++) {
        if (i > maxReach) return false;     // stuck, can't even reach i
        maxReach = max(maxReach, i + nums[i]);
    }
    return true;
}
```
Track the farthest index reachable so far. If you ever reach an index beyond that, you're stuck. This beats the DP — no state really needed here, just one running value.

---

## JUMP GAME II (LC 45) — Minimum jumps to reach the last index

**Problem:** Same array, guaranteed reachable. Find **minimum number of jumps**.

### Recursion
```cpp
int solve(int i, vector<int>& nums) {
    int n = nums.size();
    if (i >= n-1) return 0;
    int best = INT_MAX;
    for (int step = 1; step <= nums[i] && i+step < n; step++) {
        int res = solve(i+step, nums);
        if (res != INT_MAX) best = min(best, res + 1);
    }
    return best;
}
```

### Memoization
```cpp
int solveMemo(int i, vector<int>& nums, vector<int>& dp) {
    int n = nums.size();
    if (i >= n-1) return 0;
    if (dp[i] != -1) return dp[i];

    int best = INT_MAX;
    for (int step = 1; step <= nums[i] && i+step < n; step++) {
        int res = solveMemo(i+step, nums, dp);
        if (res != INT_MAX) best = min(best, res + 1);
    }
    return dp[i] = best;
}
```

### Tabulation
```cpp
int jump(vector<int>& nums) {
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

### Better: Greedy O(n) (BFS-level-by-level idea)
```cpp
int jumpGreedy(vector<int>& nums) {
    int n = nums.size();
    int jumps = 0, curEnd = 0, farthest = 0;

    for (int i = 0; i < n-1; i++) {
        farthest = max(farthest, i + nums[i]);
        if (i == curEnd) {                 // must jump now to progress
            jumps++;
            curEnd = farthest;
        }
    }
    return jumps;
}
```
Think of it as BFS levels: `curEnd` marks the boundary of the current "jump level"; when you hit it, you've used one jump and expand to `farthest`.

---

## JUMP GAME III (LC 1306) — Reach any index with value 0

**Problem:** From index `i`, you may jump to `i + nums[i]` or `i - nums[i]` (if in bounds). Return `true` if you can reach **any** index where `nums[index] == 0`.

This is pure reachability — best modeled as **graph traversal (BFS/DFS)**, not classic DP, but here's recursion+memo(visited) then BFS.

### Recursion (DFS with visited set)
```cpp
bool solve(int i, vector<int>& nums, vector<bool>& visited) {
    int n = nums.size();
    if (i < 0 || i >= n || visited[i]) return false;
    if (nums[i] == 0) return true;

    visited[i] = true;                     // "memo" — mark as explored
    return solve(i + nums[i], nums, visited) ||
           solve(i - nums[i], nums, visited);
}
// call: vector<bool> visited(n,false); solve(start, nums, visited);
```
Here `visited[]` **is** the memo table — same role as `dp[]`, just boolean "already tried."

### BFS (iterative, usually preferred)
```cpp
bool canReach(vector<int>& nums, int start) {
    int n = nums.size();
    vector<bool> visited(n, false);
    queue<int> q;
    q.push(start);
    visited[start] = true;

    while (!q.empty()) {
        int i = q.front(); q.pop();
        if (nums[i] == 0) return true;

        for (int next : {i + nums[i], i - nums[i]}) {
            if (next >= 0 && next < n && !visited[next]) {
                visited[next] = true;
                q.push(next);
            }
        }
    }
    return false;
}
```

---

## JUMP GAME IV (LC 1345) — Minimum steps using equal-value teleports

**Problem:** From index `i` you can move to `i+1`, `i-1`, or **any `j` where `arr[j] == arr[i]`**. Find minimum steps from index 0 to last index.

This is a **shortest path** problem → BFS (unweighted graph, so BFS gives min steps directly; DP doesn't fit naturally since "steps" = graph distance).

```cpp
int minJumps(vector<int>& arr) {
    int n = arr.size();
    if (n == 1) return 0;

    // group indices by value, so we can jump to same-value indices O(1) amortized
    unordered_map<int, vector<int>> sameValue;
    for (int i = 0; i < n; i++) sameValue[arr[i]].push_back(i);

    vector<bool> visited(n, false);
    queue<int> q;
    q.push(0);
    visited[0] = true;
    int steps = 0;

    while (!q.empty()) {
        int size = q.size();
        for (int k = 0; k < size; k++) {
            int i = q.front(); q.pop();
            if (i == n-1) return steps;

            // try same-value jump
            if (sameValue.count(arr[i])) {
                for (int j : sameValue[arr[i]]) {
                    if (!visited[j]) {
                        visited[j] = true;
                        q.push(j);
                    }
                }
                sameValue.erase(arr[i]);   // avoid reprocessing this value (key optimization)
            }
            // try i+1 and i-1
            for (int next : {i+1, i-1}) {
                if (next >= 0 && next < n && !visited[next]) {
                    visited[next] = true;
                    q.push(next);
                }
            }
        }
        steps++;
    }
    return -1;
}
```
Erasing the value group after using it once is what keeps this O(n) instead of O(n²).

---

## JUMP GAME V (LC 1340) — Max indices visitable (strictly decreasing height jumps)

**Problem:** From index `i`, jump to `j` where `|i-j| <= d`, and every index strictly between `i` and `j` has height `< arr[i]`, and `arr[j] < arr[i]`. Find the **max number of indices visitable** starting from any index.

**State:** `dp[i]` = max indices visitable starting at `i`. This one is a genuine DP (on a DAG, since you can only move to strictly smaller heights — no cycles).

### Recursion
```cpp
int solve(int i, vector<int>& arr, int d) {
    int n = arr.size();
    int best = 1;                          // count itself

    // jump right
    for (int j = i+1; j <= min(n-1, i+d) && arr[j] < arr[i]; j++)
        best = max(best, 1 + solve(j, arr, d));

    // jump left
    for (int j = i-1; j >= max(0, i-d) && arr[j] < arr[i]; j--)
        best = max(best, 1 + solve(j, arr, d));

    return best;
}
```

### Memoization
```cpp
int solveMemo(int i, vector<int>& arr, int d, vector<int>& dp) {
    if (dp[i] != -1) return dp[i];
    int n = arr.size();
    int best = 1;

    for (int j = i+1; j <= min(n-1, i+d) && arr[j] < arr[i]; j++)
        best = max(best, 1 + solveMemo(j, arr, d, dp));

    for (int j = i-1; j >= max(0, i-d) && arr[j] < arr[i]; j--)
        best = max(best, 1 + solveMemo(j, arr, d, dp));

    return dp[i] = best;
}

int maxJumps(vector<int>& arr, int d) {
    int n = arr.size();
    vector<int> dp(n, -1);
    int ans = 1;
    for (int i = 0; i < n; i++)
        ans = max(ans, solveMemo(i, arr, d, dp));
    return ans;
}
```

### Tabulation
Process indices in **increasing order of height** (since `dp[i]` only depends on shorter neighbors — this is the topological order of the DAG).
```cpp
int maxJumpsTab(vector<int>& arr, int d) {
    int n = arr.size();
    vector<int> indices(n);
    iota(indices.begin(), indices.end(), 0);
    sort(indices.begin(), indices.end(), [&](int a, int b) {
        return arr[a] < arr[b];              // shortest heights first
    });

    vector<int> dp(n, 1);
    int ans = 1;

    for (int i : indices) {
        for (int j = i+1; j <= min(n-1, i+d) && arr[j] < arr[i]; j++)
            dp[i] = max(dp[i], 1 + dp[j]);
        for (int j = i-1; j >= max(0, i-d) && arr[j] < arr[i]; j--)
            dp[i] = max(dp[i], 1 + dp[j]);
        ans = max(ans, dp[i]);
    }
    return ans;
}
```

---

## JUMP GAME VI (LC 1696) — Max score with jump range k

**Problem:** `nums[i]`, from index `i` you can jump to any `j` in `[i+1, i+k]`. Score = sum of visited `nums` values. Find **max score** to reach the last index.

**State:** `dp[i]` = max score to reach index `i`.
`dp[i] = nums[i] + max(dp[i-k..i-1])`

### Recursion
```cpp
int solve(int i, vector<int>& nums, int k) {
    if (i == 0) return nums[0];
    int best = INT_MIN;
    for (int j = max(0, i-k); j < i; j++)
        best = max(best, solve(j, nums, k));
    return nums[i] + best;
}
```

### Memoization
```cpp
int solveMemo(int i, vector<int>& nums, int k, vector<int>& dp) {
    if (i == 0) return nums[0];
    if (dp[i] != INT_MIN) return dp[i];

    int best = INT_MIN;
    for (int j = max(0, i-k); j < i; j++)
        best = max(best, solveMemo(j, nums, k, dp));

    return dp[i] = nums[i] + best;
}
```

### Tabulation
```cpp
int maxResult(vector<int>& nums, int k) {
    int n = nums.size();
    vector<int> dp(n);
    dp[0] = nums[0];

    for (int i = 1; i < n; i++) {
        int best = INT_MIN;
        for (int j = max(0, i-k); j < i; j++)
            best = max(best, dp[j]);
        dp[i] = nums[i] + best;
    }
    return dp[n-1];
}
```
O(n·k) — fine for small k, too slow for large k.

### Optimized: Monotonic Deque O(n)
```cpp
int maxResultOptimal(vector<int>& nums, int k) {
    int n = nums.size();
    vector<int> dp(n);
    dp[0] = nums[0];
    deque<int> dq;                          // stores indices, dp[] decreasing
    dq.push_back(0);

    for (int i = 1; i < n; i++) {
        while (!dq.empty() && dq.front() < i - k) dq.pop_front();  // out of window
        dp[i] = nums[i] + dp[dq.front()];
        while (!dq.empty() && dp[dq.back()] <= dp[i]) dq.pop_back(); // keep decreasing
        dq.push_back(i);
    }
    return dp[n-1];
}
```
The deque keeps the max of the last `k` dp-values available in O(1) instead of scanning each time.

---

## JUMP GAME VII (LC 1871) — Reachability with binary string + jump range

**Problem:** Binary string `s`, jump range `[minJump, maxJump]`. From index `i` (only if `s[i]=='0'`), can jump to any `j` in `[i+minJump, i+maxJump]` where `s[j]=='0'`. Can you reach the last index?

**State:** `dp[i]` = true if index `i` is reachable.

### Recursion
```cpp
bool solve(int i, string& s, int minJump, int maxJump) {
    int n = s.size();
    if (i == n-1) return true;
    for (int j = i+minJump; j <= min(n-1, i+maxJump); j++) {
        if (s[j] == '0' && solve(j, s, minJump, maxJump)) return true;
    }
    return false;
}
```

### Memoization
```cpp
bool solveMemo(int i, string& s, int minJump, int maxJump, vector<int>& dp) {
    int n = s.size();
    if (i == n-1) return true;
    if (dp[i] != -1) return dp[i];

    for (int j = i+minJump; j <= min(n-1, i+maxJump); j++) {
        if (s[j] == '0' && solveMemo(j, s, minJump, maxJump, dp))
            return dp[i] = true;
    }
    return dp[i] = false;
}
```
Naive memo is still O(n·maxJump) — can TLE on large inputs.

### Tabulation + Sliding Window Count (the efficient O(n) way)
```cpp
bool canReach(string s, int minJump, int maxJump) {
    int n = s.size();
    if (s[n-1] != '0') return false;

    vector<bool> dp(n, false);
    dp[0] = true;
    int reachableCount = 0;                 // count of reachable indices in current window

    for (int i = 1; i < n; i++) {
        // window is [i-maxJump, i-minJump]
        if (i >= minJump) reachableCount += dp[i-minJump];
        if (i > maxJump)  reachableCount -= dp[i-maxJump-1];

        dp[i] = (s[i] == '0') && (reachableCount > 0);
    }
    return dp[n-1];
}
```
Instead of scanning the whole `[i-maxJump, i-minJump]` window each time, maintain a running count of how many reachable indices are inside it — add one as it enters, remove one as it leaves.

---

## Summary table

| Problem | Real nature | Best approach | Naive DP complexity |
|---|---|---|---|
| Jump Game I | Reachability | Greedy O(n) | O(n²) |
| Jump Game II | Min jumps | Greedy O(n) | O(n²) |
| Jump Game III | Reachability (graph) | BFS/DFS | O(n) w/ visited |
| Jump Game IV | Shortest path (graph) | BFS | O(n) w/ grouping |
| Jump Game V | Max path in DAG | DP (topological by height) | O(n·d) |
| Jump Game VI | Max score DP | Monotonic deque | O(n·k) naive |
| Jump Game VII | Reachability DP | Sliding window count | O(n·maxJump) naive |

**Key pattern to notice:** in this series, "Jump Game" is really a placeholder for several different algorithm families — pure DP (V, VI, VII), graph BFS (III, IV), and problems where DP works but greedy is strictly better (I, II). Recognizing *which family* a jump problem belongs to is the actual skill, more than memorizing code.

Want a full dry-run walkthrough on any single one, or the greedy-vs-DP proof for Jump Game I/II?
