---
title: "Longest Palindromic Substring"
date: 2020-02-11T17:18:35+08:00
draft: false
hideToc: true
enableToc: true
enableTocContent: true
tags:
- dynamic programming
categories:
- string
- pattern matching
series:
- palindromic
libraries:
- katex
---

<!--more-->

## Description

Given a string `s`, find the longest palindromic substring in `s`. You may assume that the maximum length of `s` is 1000.

Example 1:
```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```
Example 2:
```
Input: "cbbd"
Output: "bb"
```

## Dynamic Programming Solution

Let `dp[i][j]` indicates that `s[i:j+1]` is a palindromic sequence, we have

$$ dp[i][j] = \begin{cases} True,  & \text{$i$ = $j$} \\\ s[i] = s[j], & \text{$j$ - $i$ = 1}  \\\ (s[i] = s[j]) \\ \text{and} \\ dp[i+1][j-1], & \text{$j$ - $i$ > 1} \end{cases} $$

```
              dp[i][j]
            /
dp[i+1][j-1]
```

Filling order is as described below:

| i\j | b | a | b | a | d  |
|-----|---|---|---|---|----|
| b   | 0 | 1 | 3 | 6 | 10 |
| a   |   | 2 | 4 | 7 | 11 |
| b   |   |   | 5 | 8 | 12 |
| a   |   |   |   | 9 | 13 |
| d   |   |   |   |   | 14 |

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        max_substr = ""
        dp = [[False]*len(s) for _ in range(len(s))]
        for j in range(len(s)):
            for i in range(j+1):
                if i == j:
                    dp[i][j] = True
                elif j-i == 1:
                    dp[i][j] = s[i] == s[j]
                else:
                    dp[i][j] = (s[i] == s[j]) and dp[i+1][j-1]
                
                if dp[i][j] and j-i+1 > len(max_substr):
                    max_substr = s[i:j+1]
        return max_substr
```