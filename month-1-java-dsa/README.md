# Month 1 — Modern Java 21 + DSA + AWS

# Week 1 — Day 1 — LeetCode Pattern Notes
**Date:** June 7, 2026
**Problems solved:** 3/3 ✅

---

## 1. Two Sum (#1)
**Pattern:** HashMap complement
**Time:** O(n) | **Space:** O(n)

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> processed = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int needed = target - nums[i];
            if (processed.containsKey(needed)) {
                return new int[]{processed.get(needed), i};
            }
            processed.put(nums[i], i);
        }
        return new int[]{};
    }
}
```

> 💡 **Pattern to remember:**
> *"When you need to find a complement or a pair in an array — reach for a HashMap first."*
>
> **Same pattern appears in:**
> - Group Anagrams
> - Subarray Sum Equals K
> - Two Sum II, 4Sum, 3Sum

---

## 2. Best Time to Buy and Sell Stock (#121)
**Pattern:** One pass, track minimum
**Time:** O(n) | **Space:** O(1)

```java
class Solution {
    public int maxProfit(int[] prices) {
        int maxProfit = 0;
        int leastPrice = Integer.MAX_VALUE;
        for (int price : prices) {
            leastPrice = Math.min(leastPrice, price);
            maxProfit = Math.max(maxProfit, price - leastPrice);
        }
        return maxProfit;
    }
}
```

> 💡 **Pattern to remember:**
> *"One pass, track the best seen so far. No need to look back."*
>
> **Same pattern appears in:**
> - Maximum Subarray
> - Jump Game
> - Trapping Rain Water

---

## 3. Move Zeroes (#283)
**Pattern:** Two pointer
**Time:** O(n) | **Space:** O(1)

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int insertPosition = 0;
        for (int num : nums) {
            if (num != 0) {
                nums[insertPosition++] = num;
            }
        }
        while (insertPosition < nums.length) {
            nums[insertPosition++] = 0;
        }
    }
}
```

> 💡 **Pattern to remember:**
> *"Two pointers — one scans, one tracks where valid elements land. Fill the rest after."*
>
> **Same pattern appears in:**
> - Remove Duplicates
> - Remove Element
> - Sort Colors

---

## Day 1 Summary

| # | Problem | Pattern | Time | Space | Status |
|---|---------|---------|------|-------|--------|
| 1 | Two Sum | HashMap complement | O(n) | O(n) | ✅ |
| 2 | Best Time to Buy/Sell Stock | One pass, track min | O(n) | O(1) | ✅ |
| 3 | Move Zeroes | Two pointer | O(n) | O(1) | ✅ |

**Energy level:** 2 / 5
**One thing to improve tomorrow:** Lets write clean code better by 1%
