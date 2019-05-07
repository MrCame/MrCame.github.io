---
title: Leetcode-11 Container With Most Water
date: 2017-05-03 21:12:55
tags: "Leetcode"
categories: "Leetcode"
---
***
题目要求：给定一个非负的整型数组，元素a[i]代表坐标(i, a[i])，得到一组每个点到x轴的垂线，选出两条垂线和x轴构成水箱，求围成的最大的面积。

思路：
左右两个指针扫描数组，计算面积。假设找到最佳的两条线为left，right，易知left左边没有比left更高的垂线，right右边没有比right更高的垂线，即最佳的组合在当前候选的left和right的范围内。收缩底边长度，增大高，当left的高度大于right的，right向左扫描，反之left向右扫描，求出最大面积。

代码：
```
public class Solution {
    public int maxArea(int[] height) {
        int N = height.length;
        if (N < 2)
            return 0;
        int maxArea = 0;
        int left = 0;
        int right = N - 1;
        while (left < right) {
            int currArea = (right - left) * Math.min(height[left], height[right]);
            maxArea = Math.max(maxArea, currArea);
            if (height[left] > height[right])
                right--;
            else
                left++;
        }
        return maxArea;
    }
}
```