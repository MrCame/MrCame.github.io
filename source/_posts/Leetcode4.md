---
title: Leetcode-4 Median of Two Sorted Arrays
date: 2017-04-30 22:07:09
tags: "Leetcode"
categories: "Leetcode"
---
***
题目要求：给定两个有序数组nums1、nums2，找到合并后数组的中位数。

示例：nums1 = [1, 3]，nums2 = [2]，中位数2.0

思路：
1. 传统思路，将两个数组合并起来再排序，找到中位数。
2. 将问题扩展为在两个有序数组中找到合并后第K个数的问题。

代码：
```
// 采用第二种思路
public class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int m = nums1.length;
        int n = nums2.length;
        int total = m + n;
        // 奇数
        if (total % 2 != 0) {
            return findKth(nums1, 0, m-1, nums2, 0, n-1, total/2 + 1);  // 第total/2+1个元素 
        } else {
            double x = findKth(nums1, 0, m-1, nums2, 0, n-1, total/2);
            double y = findKth(nums1, 0, m-1, nums2, 0, n-1, total/2 + 1);
            return (x + y) / 2;
        }
    }

    //  指针从1开始计数
    private double findKth(int[] a, int aLeft, int aRight, int[] b, int bLeft, int bRight, int k) {
        int m = aRight - aLeft + 1;  // 当前a的扫描长度
        int n = bRight - bLeft + 1;  // 当前b的扫描长度
        if (m > n)
            return findKth(b, bLeft, bRight, a, aLeft, aRight, k);
        if (m == 0)
            return b[k-1];
        if (k == 1)
            return Math.min(a[aLeft], b[bLeft]);

        int partA = Math.min(m, k/2);  // a的指针偏移量
        int partB = k - partA;  // a指针偏移了partA，在合并后的数组中，b指针偏移k-partA

        // a和b哪个偏移后的数值小，说明Kth大于哪个，证明见下文。则可忽略小于偏移量的那些元素。
        if (a[aLeft + partA - 1] > b[bLeft + partB - 1])
            return findKth(a, aLeft, aRight, b, bLeft+partB, bRight, k-partB);
        else if (a[aLeft + partA - 1] < b[bLeft + partB -1])
            return findKth(a, aLeft+partA, aRight, b, bLeft, bRight, k-partA);
        else
            // a[aLeft+partA-1] == b[bLeft+partB-1]，找到Kth
            return a[aLeft + partA - 1];
    }
}
```

分析：考虑一种情况，假设a和b的个数都大于k/2，设p=k/2-1，如果a[p] < b[p]，则合并后Kth > a[p]
证明：用反证法，假设当a[p] < b[p]时，kth < a[p]，不妨令a[p]是合并后第k+1个数。于是，a中有p个数小于a[p]，由假设知b中最多只有p个数小于a[p]，所以总共有2p=k-2个数小于a[p]。a[p]是第k+1个数并且a[p]之前有k-2个数，这是矛盾的，故得证Kth > a[p]。
同理，当a[p] > b[p]时，Kth > b[p]，当a[p] == b[p]时，Kth==a[p]==b[p]

在求Kth时，从相对更短的数组入手来计算偏移量p，主要是比较偏移后元素的大小。如果是上文中假设的情况，则直接取k/2，否则取数组长度，即Math.min(k/2, m)。