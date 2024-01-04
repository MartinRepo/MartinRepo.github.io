---
title: "Kattis Rank Climbing Notes"
date: 2023-11-05T22:31:02Z
lastmod: 2023-11-05T22:31:02Z
draft: false
author: ["Martin"]
tags: 
- Algorithm
description: "Take note for some learned algorithms"
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: false # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
---

# Martina DNA
The core thinking is Sliding Window Algorithm.
```c++
int findMinSubstringLength(const vector<int>& s, const unordered_map<int, int>& requiredCounts) {
    unordered_map<int, int> windowCounts;
    int minLength = INT_MAX;
    int left = 0, right = 0;
    int satisfied = 0;

    while (right < s.size()) {
        int rightInt = s[right];
        if (requiredCounts.find(rightInt) != requiredCounts.end()) {
            windowCounts[rightInt]++;
            if (windowCounts[rightInt] == requiredCounts.at(rightInt)) {
                satisfied++;
            }
        }

        while (satisfied == requiredCounts.size()) {
            minLength = min(minLength, right - left + 1);
            int leftInt = s[left];
            if (requiredCounts.find(leftInt) != requiredCounts.end()) {
                windowCounts[leftInt]--;
                if (windowCounts[leftInt] < requiredCounts.at(leftInt)) {
                    satisfied--;
                }
            }
            left++;
        }

        right++;
    }

    return minLength == INT_MAX ? -1 : minLength;
}
```
