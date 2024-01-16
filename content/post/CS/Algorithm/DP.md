+++
author = "Soeun"
title = "Dynamic Programming 접근법"
date = "2024-01-15"
summary = "leetcode 문제로 보는 DP 접근법"
categories = [
    "CS"
]
tags = [
    "알고리즘"
]
image = ""
+++


Leetcode에서 DP 문제를 풀다가, DP 에 대한 접근법을 아주 잘 설명한 글이 있어 번역했다. 

우선, Leetcode 의 [198. House Robber](https://leetcode.com/problems/house-robber/description/?envType=study-plan-v2&envId=leetcode-75) 문제를 기반으로 한다.

## 문제 설명
```text
You are a professional robber planning to rob houses along a street. 
Each house has a certain amount of money stashed, 
the only constraint stopping you from robbing each of them is that adjacent houses have security systems connected 
and it will automatically contact the police if two adjacent houses were broken into on the same night.

Given an integer array `nums` representing the amount of money of each house, 
return the maximum amount of money you can rob tonight without alerting the police.

**Example 1:**

**Input:** nums = [1,2,3,1]
**Output:** 4
**Explanation:** Rob house 1 (money = 1) and then rob house 3 (money = 3).
Total amount you can rob = 1 + 3 = 4.

**Example 2:**

**Input:** nums = [2,7,9,3,1]
**Output:** 12
**Explanation:** Rob house 1 (money = 2), rob house 3 (money = 9) and rob house 5 (money = 1).
Total amount you can rob = 2 + 9 + 1 = 12.
```

DP 문제는 아래와 같은 단계들을 거쳐서 접근할 수 있다. 
1. Find recursive relation
2. Recursive (top-down)
3. Recursive + memo (top-down)
4. Iterative + memo (bottom-up)
5. Iterative + N variables (bottom-up)

### Step1. Find recursive relation

이 문제에서, 도둑은 2가지 옵션이 있다 : a) rob current house `i` , b) don't rob current house. 

만약 옵션 a 를 선택하면, 도둑은 `i-1` 번째 집은 훔칠 수 없지만, `i-2` 번째 집까지 훔친 것을 모두 더한 것은 가져갈 수 있다. 

옵션 b 를 선택하면, 도둑은 `i-1` 번째 집까지 훔친 것들을 가져갈 수 있다. 

그러면, 이 문제는 두가지 옵션 중 더 큰 값을 가져가는 문제로 바뀐다. 

- 현재 집  `i`  에 있는 돈 훔치기 + `i-2` 까지 훔친 돈들 가져가기 
- `i-1` 까지 훔친 돈들 가져가기 

간단하게 나타내면, `rob(i) = max(rob(i-2) + currentValue, rob(i-1))` 이다. 

### Step2. Recursive (top-down)

Step1 의 식을 코드로 옮기는 것은 어렵지 않다. 

```python
def rob(nums):
    def rob_recursive(nums, i):
        if i < 0:
            return 0
        return max(rob_recursive(nums, i - 2) + nums[i], rob_recursive(nums, i - 1))

    return rob_recursive(nums, len(nums) - 1)

```

하지만 이 알고리즘은 같은 i 를 계속 계산해야 한다. 

### Step3. Recursive + memo (top-down)

```python
def rob(nums):
    memo = [-1] * (len(nums) + 1)

    def rob_recursive(nums, i):
        if i < 0:
            return 0
        if memo[i] >= 0:
            return memo[i]
        result = max(rob_recursive(nums, i - 2) + nums[i], rob_recursive(nums, i - 1))
        memo[i] = result
        return result

    return rob_recursive(nums, len(nums) - 1)
```

이것은 전의 계산 결과를 memo 에 저장해 놓는다. 

### Step4. Iterative + memo (bottom-up)

```python
def rob(nums):
	if (len(nums) == 0) : return 0
	memo = [0] * (len(nums)+1)
	memo[0] = 0
	memo[1] = nums[0]
	for i in range(1,len(nums)):
		val = nums[i]
		memo[i+1] = max(memo[i], memo[i-1] + val)
	return memo[len(nums)]
```


```python
def rob(self, nums: List[int]) -> int:
	n = len(nums)
	if n == 2:
		return max(nums[0], nums[1])
	if n == 1:
		return nums[0]
	dp = [0] * n
	dp[0] = nums[0]
	dp[1] = max(nums[0], nums[1])	
	for i in range(2,n):	
		dp[i] = max(dp[i-1], nums[i] + dp[i-2])
	return dp[n-1]
```

## Referece
- https://leetcode.com/problems/house-robber/solutions/156523/from-good-to-great-how-to-approach-most-of-dp-problems/?envType=study-plan-v2&envId=leetcode-75
