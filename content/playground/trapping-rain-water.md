---
title: "Trapping Rain Water"
date: 2021-08-11T20:09:03+08:00
draft: false
hideToc: true
enableToc: true
enableTocContent: true
libraries:
- katex
---
<!--more-->

https://leetcode.com/problems/trapping-rain-water/

设第$i$个bar的高度为$h_i$, 其蓄水量为$v_i$, 一共有$n$个bar,

对于每一个位置，考虑所有可能的左右bar，计算每一对可以拦截的水量，而每一对拦截水量是$\min(h_l, h_r) - h_i$
所有可能的围挡方式中，最高值就是答案，即
$$\max\limits_{0\leq l<i<r<n} (\min(h_l, h_r) - h_i) = \min (\max\limits_{0\leq l<i} h_l, \max\limits_{i<r<n} h_r) - h_i$$

由此可知，对于任意一个bar，它左边最高的bar和右边最高的bar中比较低的那个决定了蓄水量. 

## "水位"法
对此, 网络上有一个非常直观的的解法：

想象水从两边流向中间, 水位在逐渐上升, 水会逐渐淹没比较低的bar,并且在淹没其之后流向相邻的bar, 从两边逐渐向中间蔓延. 
设水位高度为$level$, 并且水流向了$h_j$.
1. $level > h_j$, 水位比bar高, 水会淹没$h_j$, 此时$j$位置可以拦截的水量即为$v_j = level - h_j$. 证明: 无论水是从左边还是右边流向位置$j$的, 它一定得先漫过左右两边最高的bar中比较低的bar, 否则不能流到此位置.
2. $level \leq h_j$, 那么水平面遇到了更高的bar, 它不会容纳水, 即$v_j = 0$. 因为水从某一侧来就说明$h_j$的高度比那一侧最高的bar更高.

### 优先队列解
维护一个优先队列, 其中包含了水平面接触到但是尚未浸没的bar, 随着水位的上升, 有以下步骤

0. 水位为$0$, 左右两边的bar入队列, 作为初始状态.
1. 最小值出队列, $h_i = q.pop()$. 这代表水位需要浸没该bar
2. 更新水位, 
    a. 如果$level \leq h_i$, 说明水位已经高过该bar, 不用更新水位值.
    b. 如果$level < h_i$, 说明水位需要涨到该bar的高度才能浸没它, 更新水位值.
    c. 综上所述, $level = max(level, h_i)$
3. 浸没$h_i$之后, 水开始探索和$h_i$相邻的bar, 设为$h_{next}$, 
    a. 若$h_{next}$没有入过队列(未被水接触), 将$h_{next}$入队列, 说明水流至此.
    b. 若$h_{next} < level$, 说明水流至此, 并且水位没过$h_{next}$, 计算蓄水量$v_i = level - h_{next}$
4. 重复1~3步骤直至队列为空.

```cpp
int visited[20001];
int trap_heap(vector<int>& height) {
    // 优先队列的数据结构是 {index, height} 分别记录了bar的位置和高度
    auto cmp_height = [](pair<int, int> a, pair<int, int> b) {return a.second > b.second;};
    priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(cmp_height)> q(cmp_height);
    // 设定初始状态, visited记录了该bar是否入过队列.
    int level = 0; int v = 0; memset(visited, 0, sizeof(visited));
    q.push({0, height[0]}); q.push({height.size()-1, height[height.size()-1]});
    visited[0] = 1; visited[height.size()-1] = 1;

    while (!q.empty()) {
        int i = q.top().first; int h = q.top().second; // 最小值出队列
        q.pop();
        level = max(level, h); // 更新水位
        int neighbours[2] = {i-1, i+1};
        for (int next : neighbours) { // 遍历邻居
            if (next >= 0 && next < height.size() && !visited[next]) { // 边界检查, 且已经入过队列的不可以再入队列.
                visited[next] = 1;
                if (level > height[next]) { // 水位高于next
                    v += level - height[next];
                }
                q.push({next, height[next]});
            }
        }
    }
    return v;
}
```

### 双指针解
双指针$l, r$从两边向中间遍历, 双指针其实就是水位法中优先队列里的两个元素. 双指针的移动顺序, 其实就是水浸没bar的顺序. 所以逻辑是完全一致的.

```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int l = 0; int r = height.size() - 1; // l和r对应队列中的index
        int lmax = 0; int rmax = 0; int v = 0; // lmax和rmax代表了左右两边水位的高度, 对应level
        while (l <= r) {
            int hl = height[l]; int hr = height[r]; // hl和hr对应队列中的height
            if (hl <= hr) {  // 左边水位比较低, 先处理左边
                lmax = max(lmax, hl); // 更新水位
                if (lmax > hl) { 
                    v += lmax - hl; // 水位浸没hl
                }
                l++; // 对应hl入队列
            } else {  // 右边水位比较低, 先处理右边
                rmax = max(rmax, hr);
                if (rmax > hr) {
                    v += rmax - hr; // 水位浸没hr
                }
                r--; // 对应hr入队列
            }
        }
        return v;
    }
};
```

## 二维拓展
Extend: https://leetcode.com/problems/trapping-rain-water-ii/
这道题拓展到了2维，其实使用"水位法"依然可解。

想象水从四周流向中间，水位逐渐上升，每当水位升至与某一个$h_i$齐平的时候，它就会"溢出"并且蔓延到和$h_i$接壤的位置，此时需要检查水可不可以"漫"过去。如果接壤的$h_j < h_i$，$h_j$之上可以容纳水的体积就确定为$h_i - h_j$. 因为对$j$来说，在已经把它包围的拓扑里，最低的值是$h_i$

所以和上一题把两边的bar入栈类似的, 先把最外围一圈入栈, 然后和1维情况一样了. [演示动画](https://www.youtube.com/watch?v=cJayBq38VYw)

```cpp
class Solution {
public:
    int visited[201][201];
    int drow[4] = {0, -1, 0, 1}; // clockwise
    int dcol[4] = {-1, 0, 1, 0};
    int trapRainWater(vector<vector<int>>& heightMap) {
        // 堆中元素的结构: {{row, col}, height}
        int nrow = heightMap.size(); int ncol = heightMap[0].size();
        auto cmp_height = [](pair<pair<int, int>, int> a, pair<pair<int, int>, int> b) {return a.second > b.second;};
        priority_queue<pair<pair<int, int>, int>, vector<pair<pair<int, int>, int>>, decltype(cmp_height)> q(cmp_height);
        // 初始化水平面和水量
        int level = 0; int volume = 0;
        memset(visited, 0, sizeof(visited));
        // 第一行和最后一行入队列
        for (int col = 0; col < ncol; col++) {
            q.push({{0, col}, heightMap[0][col]});  // first row
            q.push({{nrow-1, col}, heightMap[nrow-1][col]});  // last row
            visited[0][col] = 1; visited[nrow-1][col] = 1;
        }
        // 第一列和最后一列入队列
        for (int row = 1; row < nrow-1; row++) { // 跳过第一行和最后一行已经入队列的元素
            q.push({{row, 0}, heightMap[row][0]});
            q.push({{row, ncol-1}, heightMap[row][ncol-1]});
            visited[row][0] = 1; visited[row][ncol-1] = 1;
        }

        while (!q.empty()) {
            pair<int, int> coord = q.top().first; int h = q.top().second;
            q.pop();
            level = max(level, h);

            for (int i = 0; i < 4; i++) { // 遍历上下左右四个位置
                pair<int, int> next = {coord.first + drow[i], coord.second + dcol[i]};
                int next_height = heightMap[next.first][next.second];
                if (next.first >= 0 && next.first < nrow && next.second >=0 && next.second < ncol && !visited[next.first][next.second]) {
                    visited[next.first][next.second] = 1;
                    if (level > next_height) {
                        volume += level - next_height;
                    }
                    q.push({next, next_height});
                }
            }
        }
        return volume;
    }
};
```