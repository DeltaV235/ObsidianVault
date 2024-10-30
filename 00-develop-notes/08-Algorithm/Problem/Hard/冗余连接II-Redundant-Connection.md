---
title: 冗余连接II-Redundant-Connection
created: 2024-10-29
tags:
    - Algorithm
    - Leetcode
    - UnionFind
    - Tree
---

# 冗余连接II-Redundant-Connection

## Problem

[Leetcode-链接](https://leetcode-cn.com/problems/redundant-connection-ii/)

## Problem Description

在本问题中，有根树指满足以下条件的有向图。该树只有一个根节点，所有其他节点都是该根节点的后继。每一个节点只有一个父节点，除了根节点没有父节点。

输入一个有向图，该图由一个有着N个节点（节点值不重复1, 2, ..., N）的树及一条附加的有向边构成。附加的边的两个顶点包含在1到N中间，这条附加的边不属于树中已存在的边。

结果图是一个以边组成的二维数组。 每一个元素是一个包含两个整数的列表，分别表示图中连接的两个节点。

返回一条可以删去的边，使得结果图是一个有N个节点的树。如果有多个答案，返回最后出现在给定二维数组的答案。

示例 1:

```plaintext
输入: [[1,2], [1,3], [2,3]]
输出: [2,3]
解释: 给定的有向图如下:
  1
 / \
v   v
2-->3
```

示例 2:

```plaintext
输入: [[1,2], [2,3], [3,4], [4,1], [1,5]]
输出: [4,1]
解释: 给定的有向图如下:
5 <- 1 -> 2
     ^    |
     |    v
     4 <- 3
```

注意:

- 二维数组大小的在3到1000范围内。
- 二维数组中的每个整数在1到N之间，其中 N 是二维数组的大小。

## Solution

### UnionFind

```java
class Solution {
    public int[] findRedundantDirectedConnection(int[][] edges) {
        int n = edges.length;
        int[] parent = new int[n + 1];
        // parent[]数组用于记录每个节点的父节点
        for (int i = 1; i <= n; i++) {
            parent[i] = i;
        }
        int[] indegree = new int[n + 1];
        int conflict = -1;
        int cycle = -1;
        for (int i = 0; i < n; i++) {
            int[] edge = edges[i];
            int u = edge[0], v = edge[1];
            if (parent[v] != v) {
                conflict = i;
            } else {
                parent[v] = u;
                if (find(parent, u) == find(parent, v)) {
                    cycle = i;
                }
            }
        }
        if (conflict < 0) {
            return edges[cycle];
        } else {
            int[] conflictEdge = edges[conflict];
            if (cycle >= 0) {
                return new int[]{parent[conflictEdge[1]], conflictEdge[1]};
            } else {
                return conflictEdge;
            }
        }
    }

    public int find(int[] parent, int index) {
        if (parent[index] != index) {
            parent[index] = find(parent, parent[index]);
        }
        return parent[index];
    }
}
```

### DFS

```java
class Solution {
    public int[] findRedundantDirectedConnection(int[][] edges) {
        int n = edges.length;
        int[] parent = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            parent[i] = i;
        }
        int[] indegree = new int[n + 1];
        int conflict = -1;
        int cycle = -1;
        for (int i = 0; i < n; i++) {
            int[] edge = edges[i];
            int u = edge[0], v = edge[1];
            indegree[v]++;
            if (indegree[v] == 2) {
                conflict = i;
            }
        }
        if (conflict < 0) {
            return getRedundantDirectedConnection(edges, n, -1);
        } else {
            int[] conflictEdge = edges[conflict];
            int start = conflictEdge[0], end = conflictEdge[1];
            for (int i = 0; i < n; i++) {
                if (edges[i][1] == end) {
                    cycle = i;
                }
            }
            if (cycle >= 0) {
                return new int[]{edges[cycle][0], edges[cycle][1]};
            } else {
                return conflictEdge;
            }
        }
    }

    public int[] getRedundantDirectedConnection(int[][] edges, int n, int skipEdge) {
        int[] parent = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            parent[i] = i;
        }
        for (int i = 0; i < n; i++) {
            if (i == skipEdge) {
                continue;
            }
            int[] edge = edges[i];
            int u = edge[0], v = edge[1];
            if (find(parent, u) == find(parent, v)) {
                return edge;
            }
            parent[v] = u;
        }
        return new int[0];
    }

    public int find(int[] parent, int index) {
        if (parent[index] != index) {
            parent[index] = find(parent, parent[index]);
        }
        return parent[index];
    }
}
```
