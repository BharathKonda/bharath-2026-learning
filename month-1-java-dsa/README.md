# Month 1 — Modern Java 21 + DSA + AWS

# Week 1 - Day 1 - LeetCode Solutions
**Date:** June 7, 2026
**Pattern focus:** Arrays - Two Pointer, One Pass, Two Pointer

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

**Key learning:** When finding a pair/complement in an array → reach for HashMap first.

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

**Key learning:** One pass, track the best seen so far. No need to look back.

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

**Key learning:** One pointer scans, one tracks where valid elements land. Fill rest after.

---

## Day 1 Summary
- Problems solved: 3/3 ✅
- Patterns covered: HashMap, One Pass, Two Pointer
- Energy level: 2/5
- One thing to improve tomorrow: 1% bettermenbt in clean code
