# Day 2 — Tuesday, June 9, 2026

> 6-Month Backend Deep Dive | Month 1: Java + DSA + AWS Fundamentals

---

## What I worked on today

### DSA — Arrays & Prefix Sum Patterns

#### 1. Maximum Subarray — LeetCode #53
- **Pattern:** Kadane's Algorithm
- **Approach:** Started with brute force O(n²), identified the TLE, derived the optimal solution independently
- **Key insight:** Reset `curSum` to 0 when it goes negative — a negative running sum only drags the next element down
- **Brute force bug caught:** Initialising `maxSum = 0` fails for all-negative arrays — fixed to `maxSum = nums[0]`
- **Result:** ✅ Accepted 210/210 | Runtime 1ms | Beats 99.95%
- **Complexity:** Time O(n) · Space O(1)

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int maxSum = nums[0];
        int curSum = 0;
        for (int i = 0; i < nums.length; i++) {
            curSum += nums[i];
            maxSum = curSum > maxSum ? curSum : maxSum;
            curSum = curSum < 0 ? 0 : curSum;
        }
        return maxSum;
    }
}
```

---

#### 2. Contains Duplicate II — LeetCode #219
- **Pattern:** Sliding window + HashSet / HashMap
- **Key insight:** Maintain a fixed-size window of k elements
- **Approach 1 — HashSet:** Trim window by removing `nums[i-k]` when size exceeds k
- **Approach 2 — HashMap:** Store last seen index, check distance mathematically
- **Tradeoff learned:** HashSet = less memory (76MB), HashMap = faster runtime (23ms · 80%)
- **Interview answer:** "Both are O(k) space, but constant factors differ — HashSet requires a remove() call each iteration, HashMap avoids it by storing indices directly"

**HashSet approach:**
```java
Set<Integer> window = new HashSet<>();
for (int i = 0; i < nums.length; i++) {
    if (window.contains(nums[i])) return true;
    window.add(nums[i]);
    if (window.size() > k) window.remove(nums[i - k]);
}
return false;
```

**HashMap approach (faster):**
```java
Map<Integer, Integer> map = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    if (map.containsKey(nums[i]) && i - map.get(nums[i]) <= k) return true;
    map.put(nums[i], i);
}
return false;
```

---

#### 3. Subarray Sum Equals K — LeetCode #560
- **Pattern:** Prefix sum + HashMap (frequency count)
- **Key insight:** `curSum - k` appeared n times before → n valid subarrays end here
- **Why `map.put(0, 1)`:** The empty prefix sum of 0 has been seen once before the array starts — correctly counts subarrays starting from index 0
- **Bug caught:** `map.get(curSum)` throws NullPointerException if key absent → fixed with `map.getOrDefault(curSum, 0) + 1`
- **Result:** ✅ Accepted | Runtime 24ms | Beats 76.60%
- **Complexity:** Time O(n) · Space O(n)

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> map = new HashMap<>();
    map.put(0, 1);
    int curSum = 0, count = 0;
    for (int i = 0; i < nums.length; i++) {
        curSum += nums[i];
        if (map.containsKey(curSum - k)) count += map.get(curSum - k);
        map.put(curSum, map.getOrDefault(curSum, 0) + 1);
    }
    return count;
}
```

---

### AWS — IAM Module (Udemy SAA)

**Concepts covered:**
- IAM Users vs Roles — users have permanent credentials, roles provide temporary credentials via STS AssumeRole
- IAM Groups — attach policies to groups, users inherit all group permissions (additive)
- IAM Policies — JSON documents defining Allow/Deny on actions and resources
- Principle of least privilege — every user/service gets minimum permissions needed for their specific job
- Deny always overrides Allow in any policy evaluation
- Roles are used for: AWS services (EC2, Lambda), cross-account access, federated identity (SSO/AD)

**Hands-on:**
- Created an IAM user with limited permissions in AWS console ✅
- Reviewed policy JSON structure (Effect, Action, Resource)

---

### Java 21 — Pattern Matching for Switch (JEP 441)

**Concepts covered:**
- Pattern matching for `switch` — cleaner than chained `instanceof` checks
- Guarded patterns with `when` clause
- `null` handling in switch (no more NullPointerException)
- Sealed classes + pattern matching = exhaustive switch (compiler enforces all cases)

**Before (old style):**
```java
if (obj instanceof String s) {
    return s.toUpperCase();
} else if (obj instanceof Integer i) {
    return i * 2;
} else {
    return obj.toString();
}
```

**After (Java 21):**
```java
return switch (obj) {
    case String s  -> s.toUpperCase();
    case Integer i -> i * 2;
    default        -> obj.toString();
};
```

---

## Key patterns learned today

| Pattern | When to use |
|---|---|
| Kadane's Algorithm | Maximum subarray sum in O(n) |
| Sliding window + HashSet | Duplicate within distance k |
| Sliding window + HashMap | Duplicate within distance k (faster) |
| Prefix sum + HashMap (frequency) | Count subarrays with exact sum |
| Prefix sum + HashMap (index) | Longest subarray with exact sum |

---

## Bugs caught today (important)

1. `maxSum = 0` fails for all-negative arrays → always initialise to `nums[0]`
2. `map.get(key)` throws NPE if key absent → use `map.getOrDefault(key, 0)`
3. Checking only `j = i + k` misses duplicates closer than k → need full window check

---

## AWS cost lesson learned

- Stopped EC2 ≠ free (EBS storage still charges)
- Terminated EC2 = free (everything destroyed)
- Always **terminate** lab resources, never just stop
- Set billing alert at $5 — AWS will not warn you otherwise

---

## Tomorrow — Day 3 (June 11)

- [ ] Longest Subarray with Sum K (GFG — prefix sum + HashMap, store earliest index)
- [ ] 3Sum + Container With Most Water (two-pointer)
- [ ] AWS EC2 + S3 deep dive + presigned URL lab
- [ ] AWS VPC fundamentals (draw public/private subnet diagram)
- [ ] Java 21 — Virtual threads deep dive (Project Loom)
- [ ] Effective Java Ch 1–4 (catch-up reading)

---

## Stats — Day 2

### LeetCode Results

| # | Problem | Approach | Runtime | Beats | Memory | Beats | Status |
|---|---|---|---|---|---|---|---|
| 53 | Maximum Subarray | Kadane's O(n) | 1ms | 99.95% | 77.42MB | 19.54% | ✅ |
| 219 | Contains Duplicate II | HashSet sliding window | 27ms | 19.50% | 76.55MB | 62.71% | ✅ |
| 219 | Contains Duplicate II | HashMap index | 23ms | 80.33% | 91.00MB | 5.29% | ✅ |
| 560 | Subarray Sum Equals K | Brute force O(n²) | 1539ms | 14.15% | 48.82MB | 47.26% | ✅ |
| 560 | Subarray Sum Equals K | Prefix sum + HashMap | 24ms | 76.60% | 49.00MB | 22.69% | ✅ |

### Summary

| Metric | Value |
|---|---|
| LeetCode problems solved | 3 (5 submissions including optimisations) |
| Best runtime | 1ms · 99.95% — Maximum Subarray |
| Biggest optimisation | 1539ms → 24ms on Subarray Sum Equals K (64x speedup) |
| AWS modules completed | 1 (IAM) |
| Java 21 topics | 1 (Pattern matching JEP 441) |
| Brute force → optimal | 2 problems independently optimised |
| Energy level | TBD |

---

*Part of my 6-month backend upskilling plan — June to November 2026*
*Target: Staff/Principal Java Backend Engineer*
