---
title: "LeetCode 354:俄罗斯信封套娃问题"
date: 2021-03-05T09:26:34+08:00
draft: true
toc: false
images:
tags: 
  - LeetCode
---

### 题目

给定一些标记了宽度和高度的信封，宽度和高度以整数对形式 (w, h) 出现。当另一个信封的宽度和高度都比这个信封大的时候，这个信封就可以放进另一个信封里，如同俄罗斯套娃一样。

请计算最多能有多少个信封能组成一组“俄罗斯套娃”信封（即可以把一个信封放到另一个信封里面）。

说明:
不允许旋转信封。

示例:

> 输入: envelopes = [[5,4],[6,4],[6,7],[2,3]]
> 输出: 3 
> 解释: 最多信封的个数为 3, 组合为: [2,3] => [5,4] => [6,7]。

### 思路

> 来源：[labuladong](https://labuladong.gitbook.io/algo/)

这道题目其实是最长递增子序列 (Longes Increasing Subsequence, 简写为 LIS) 的一个变种，因为很显然，每次合法的嵌套都是大的套小的，相当于找一个最长递增的子序列，其长度就是最多能嵌套的信封个数。

但是难点在于，标准的LIS算法只能在数组中寻找最长子序列，而我们的信封是由```[w, h]```这样的二维数对形式表示的，如何把LIS算法运用过来呢？

```w * h```计算面积的形式是行不通的，```1 * 10``` 大于 ```3 * 3```，但是明显这样的两个信封是无法相互嵌套的。

### 解法

这道题的解法是比较巧妙的：

先对宽度 w 进行升序排列，如果遇到 w 相同的情况，则按照高度 h 降序排列。之后把所有的 h 取出，填入一个数组，在这个数组上计算 LIS 的长度就是我们的答案。

示例：

```
|  宽度w   高度h
|  [ 1  ,  8 ] 
|  [ 2  ,  3 ]
|  [ 5  ,  4 ]|降
|  [ 5  ,  2 ]|序
|  [ 6  ,  7 ]|降
|  [ 6  ,  4 ]|序
升
序  
```

很明显，高度 h 组成的数组中 ```3 -> 4 -> 7 ```，就是我们要找的LIS，其最大长度为3

这个解法的关键在于，对于宽度 w 相同的数对，要对其高度 h 进行降序排序。因为两个宽度相同的信封不能相互包含的，逆序排序保证 w 相同的数对中最多只选取一个。

### 代码

```go

func maxEnvelopes(envelopes [][]int) int {
	count := len(envelopes)

	// 快速排序
	sort.Slice(envelopes, func(i, j int) bool {
		if envelopes[i][0] == envelopes[j][0] {
			return envelopes[i][1] > envelopes[j][1]
		}
		return envelopes[i][0] < envelopes[j][0]
	})

	// 取出h
	height := make([]int, count)
	for i := 0; i < count; i++ {
		height[i] = envelopes[i][1]
	}

	// 计算LIS
	return lengthOfLIS(height)
}

// LIS 计算
func lengthOfLIS(nums []int) int {
	piles, count := 0, len(nums)
	top := make([]int, count)

	for i := 0; i < count; i++ {
		num := nums[i]
		left, right := 0, piles

		for left < right {
			mid := (left + right) / 2
			if top[mid] >= num {
				right = mid
			} else {
				left = mid + 1
			}
		}
		if left == piles {
			piles++
		}
		top[left] = num
	}
	return piles
}

```

### wiki

> [最长递增子序列扩展到二维而已](https://leetcode-cn.com/problems/russian-doll-envelopes/solution/zui-chang-di-zeng-zi-xu-lie-kuo-zhan-dao-er-wei-er/)
>
> [动态规划设计方法&&纸牌游戏讲解二分解法](https://leetcode-cn.com/problems/russian-doll-envelopes/solution/zui-chang-di-zeng-zi-xu-lie-kuo-zhan-dao-er-wei-er/)