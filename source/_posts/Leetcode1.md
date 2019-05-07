---
title: Leetcode-1 Two Sum
date: 2017-04-30 21:38:18
tags: "Leetcode"
categories: "Leetcode"
---
***
题目要求：给定一个数组nums以及一个数target，求出和为target的两个元素的索引。

示例：nums = [2, 7, 11, 15], target = 9，因为nums[0] + nums[1] = 2 + 7 = 9,所以返回[0, 1]

思路：
1. 传统的暴力方法，两次遍历数组，求出所有二元组的和，复杂度O(n^2)
2. 使用Map，数值(key)-索引(value)，遍历一次数组，根据当前nums[i]在Map中查找target-nums[i]

代码：
```
import java.util.Arrays;
public class Solution {
    public int[] twoSum(int[] nums, int target) {
        int N = nums.length;
        if (N < 2)
            return null;
        int[] result = new int[2];
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        for (int i = 0; i < N; i++) {
            if (map.containsKey(target-nums[i]) {
                result[0] = map.get(target-num[i]);
                result[1] = i;
            }
            map.put(nums[i], i);
        }
        return result;
    }
}
```

