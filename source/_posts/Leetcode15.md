---
title: Leetcode-15 Three Sum
date: 2017-05-04 21:45:56
tags: "Leetcode"
categories: "Leetcode"
---
***
题目要求：给定一个数组nums，找出和为0的所有三元组集合。

示例：nums = [-1, 0, 1, 2, -1, -4]，和为0的三元组集合 = [[-1, 0, 1], [-1, -1, 2]]。

思路：将数组排序后，指针指定一个元素后，对该元素之后的元素进行2sum运算，考虑到不能有重复结果，所以需要跳过重复元素。

代码：
```
import java.util.Arrays;

public class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        int N = nums.length;
        List<List<Integer>> result = new ArraysList<>();
        if (nums == null || N == 0 || N < 3)
            return result;
        Arrays.sort(nums);

        for (int i = 0; i < N-2; i++) {
            // 跳过重复元素
            if (i > 0 && num[i-1] == nums[i])
                continue;
            int left = i + 1;
            int right = N - 1;
            int target = -nums[i];
            // 对之后的元素进行2sum
            twoSum(nums, left, right, target, result);
        }
        return result;
    }

    private void twoSum(int[] nums, int left, int right, int target, List<List<Integer>> result) {
        while (left < right) {
            if (nums[left] + nums[right] == target) {
                List<Integer> list = new ArrayList<>();
                list.add(-target);
                list.add(nums[left]);
                list.add(nums[right]);
                result.add(list);

                left++;
                right--;

                // 从左边跳过重复元素
                while (left < right && nums[left-1] == nums[left])
                    left++;
                // 从右边跳过重复元素
                while (left < right && nums[right] == nums[right+1])
                    right--;
            }
            // 和小于target，增大left的值
            else if (nums[left] + nums[right] < target)
                left++;
            // 和大于target，减小right的值
            else
                right--;
        }
    }
}
```
