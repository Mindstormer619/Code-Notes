# Interview Questions
> Created: 2022-07-26 06:01

## (Med) Permutation in String

Given two strings `s1` and `s2`, return `true` _if_ `s2` _contains an anagram of_ `s1`_, or_ `false` _otherwise_.

In other words, return `true` if one of `s1`'s anagram (including `s1` itself) is the substring of `s2`.

`s1` and `s2` consist of lowercase English letters.

**Example 1:**

**Input:** s1 = "ab", s2 = "eidbaooo"
**Output:** true
**Explanation:** s2 contains one permutation of s1 ("ba").

**Example 2:**

**Input:** s1 = "ab", s2 = "eidboaoo"
**Output:** false

### Approach

+ Fill 2 hashmaps / array(26), one for small string and one for large one, mapping letter -> frequency.
+ Start with a window on the large string of length equal to the small string. Fill both hashmaps for it, checking all characters for matches. Update a `matches` count accordingly.
+ Each time the sliding window shifts by 1, update the `matches` to remove or add a new match depending on the character that just got removed from the window, and the character that just got added.

## (Med) 3Sum

Given an integer array nums, return all the triplets `[nums[i], nums[j], nums[k]]` such that `i != j`, `i != k`, and `j != k`, and `nums[i] + nums[j] + nums[k] == 0`.

The solution set must not contain duplicate triplets.

### Approach

1. Sort the list.
2. Iterate over numbers from left to right. When seeing a number, check against previous -- if same, then skip it because we already have all the triplets starting with this number.
3. For each, you have now collapsed it to 2Sum which you can solve with 2 pointers, one starting from left side and one starting from right. Increment left when too small and decrement right when sum is too large.


## (Med) Group Anagrams

Given an array of strings `strs`, group **the anagrams** together. You can return the answer in **any order**.

An **Anagram** is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

**Example 1:**

```
**Input:** strs = ["eat","tea","tan","ate","nat","bat"]
**Output:** [["bat"],["nat","tan"],["ate","eat","tea"]]
```

### Approach

Hashmap of 26 characters, mapping letter -> frequency. `O(m*n)` for `m` strings and `n` length of maximum string.

## (Med) Triangle

Given a `triangle` array, return _the minimum path sum from top to bottom_.

For each step, you may move to an adjacent number of the row below. More formally, if you are on index `i` on the current row, you may move to either index `i` or index `i + 1` on the next row.

```
**Input:** triangle = [[2],[3,4],[6,5,7],[4,1,8,3]]
**Output:** 11
**Explanation:** The triangle looks like:
   2
  3 4
 6 5 7
4 1 8 3
The minimum path sum from top to bottom is 2 + 3 + 5 + 1 = 11 (underlined above).
```

### Approach

DP problem. Storage optimization: Built bottom to top, at any given point you only need to store for the immediate upper layer and can discard other layers.

## (Med) Max Product Subarray

Given an integer array `nums`, find a contiguous non-empty subarray within the array that has the largest product, and return _the product_.

The test cases are generated so that the answer will fit in a **32-bit** integer.

A **subarray** is a contiguous subsequence of the array.

```
**Input:** nums = [2,3,-2,4]
**Output:** 6
**Explanation:** [2,3] has the largest product 6.
```

```
**Input:** nums = [-2,0,-1]
**Output:** 0
**Explanation:** The result cannot be 2, because [-2,-1] is not a subarray.
```

### Approach

Can be done in `O(n)`. Keep track of `currentMax` and `currentMin` products as you loop through the array. At every element `n`, update both using the previous max and min as `max(curMax*n, curMin*n, n)` and `min(curMax*n, curMin*n, n)`. Edge case is for `n = 0` where you basically just reset max and min to 1, since you don't want to kill your product. Initialize `result` with `max(inputArray)` because it's possible your input array is just `[-1]`.

## (Med) Word Break

Given a string `s` and a dictionary of strings `wordDict`, return `true` if `s` can be segmented into a space-separated sequence of one or more dictionary words.

**Note** that the same word in the dictionary may be reused multiple times in the segmentation.

**Example 1:**

	**Input:** s = "leetcode", wordDict = ["leet","code"]
	**Output:** true
	**Explanation:** Return true because "leetcode" can be segmented as "leet code".

**Example 2:**

	**Input:** s = "applepenapple", wordDict = ["apple","pen"]
	**Output:** true
	**Explanation:** Return true because "applepenapple" can be segmented as "apple pen apple".
	Note that you are allowed to reuse a dictionary word.

**Example 3:**

	**Input:** s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
	**Output:** false

### Approach

The thing we're storing is an array `dp[]` where `dp[i] == True` means that we found _some_ combo of characters in the substring starting with index `i` that has all words from the dictionary. If it's `False` it means we don't have a match from that character.

Build bottom up, starting at the end and looping back till the beginning. Search at each iteration for whether or not we have a match starting from this character, then equate `dp[i]` to `dp[i + len(match)` because we need to match the remaining characters also. `dp[len(s)]` is the base case which is always True, since at that point the whole string is matched.

## (Med) Coin Change 2

You are given an integer array `coins` representing coins of different denominations and an integer `amount` representing a total amount of money.

Return _the number of combinations that make up that amount_. If that amount of money cannot be made up by any combination of the coins, return `0`.

You may assume that you have an infinite number of each kind of coin.

The answer is **guaranteed** to fit into a signed **32-bit** integer.

**Example 1:**

	**Input:** amount = 5, coins = [1,2,5]
	**Output:** 4
	**Explanation:** there are four ways to make up the amount:
	5=5
	5=2+2+1
	5=2+1+1+1
	5=1+1+1+1+1

**Example 2:**

	**Input:** amount = 3, coins = [2]
	**Output:** 0
	**Explanation:** the amount of 3 cannot be made up just with coins of 2.

**Example 3:**

	**Input:** amount = 10, coins = [10]
	**Output:** 1

### Approach

Recursive memoization can be done on caching `dfs(coinDenomination, currentSumAmount)`. Stop when current sum goes over the target, and recurse for coin denominations greater than or equal to the current one, to prevent duplicates in different order.

DP optimal storage:

![[Pasted image 20220726071557.png]]

You are only looking one spot down when doing it this way, which allows storage improvement to O(n) rather than having to store the entire 2D array.


## (Easy) LCA

Given a binary search tree (BST), find the lowest common ancestor (LCA) node of two given nodes in the BST.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes p and q as the lowest node in T that has both p and q as descendants (where we allow a node to be a descendant of itself).”

## (Med) Generate Parens

Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.

Example 1:

	Input: n = 3
	Output: ["((()))","(()())","(())()","()(())","()()()"]



----

## References
1. https://neetcode.io/