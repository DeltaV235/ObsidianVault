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

在树中加入一条边，一定会导致环路的出现，具体有以下两种情况。

根据下方官方题解，有以下几个设定：

1. 一条边只会被记录为冲突边或者环路边，如果图中的边同时是冲突边和环路边，那么该边将被记录为冲突边。
2. 如果被记录为冲突边，那么这个边不会在并查集中被 union。

**Case 1:**

![[Redundant-Connection-Case-1.excalidraw]]

如果没有冲突边（所有节点的入度约为 1），那么一定存在被记录的环路边，最后一个被记录的环路边即为附加边。

**Case 2:**

![[Redundant-Connection-Case-2.excalidraw]]

如上图，存在冲突边也存在坏路边，那么附加边应为同时是冲突边和环路边的边（4-1）。

**Case 2.1:**

![[Redundant-Connection-Case-2-1.excalidraw]]

图中边的数字对应其被遍历的顺序。

4 号边被记录为冲突边。

如果在遍历所有边之后，只记录了冲突边，没有环路边被记录，那么意味着这条被记录的冲突边同时也是环路边，因为图中必定出现环路边，此时返回这条边。

**Case 2.2:**

![[Redundant-Connection-Case-2-2.excalidraw]]

3 被记录为环路边，4 被记录为冲突边。

如果在遍历所有边之后，同时记录了冲突边和环路边，那么被记录冲突边指向的节点的另一条边即为附加边。环路边和冲突边被同时记录，意味着被记录的冲突边不在环路上，而是指向环路的节点。

因为冲突边不会被 union，只有被记录的冲突边不在环路上的时候，才能找到环路边，一旦冲突边位于环路，那么环路上的所有边都不会被记录为环路边，因为缺少了被标记为冲突边的那条边的 union 操作。此时的状况对应 Case 2.1。

```java
class Solution {
    public int[] findRedundantDirectedConnection(int[][] edges) {
        int n = edges.length;
        UnionFind uf = new UnionFind(n + 1);
        // 记录每个节点的父节点，不作为并查集使用
        int[] parent = new int[n + 1];
        for (int i = 1; i <= n; ++i) {
            parent[i] = i;
        }
        int conflict = -1;
        int cycle = -1;
        for (int i = 0; i < n; ++i) {
            int[] edge = edges[i];
            int node1 = edge[0], node2 = edge[1];
            // parent[node2] != node2 说明 node2 有两个父节点，即存在冲突
            // 一条边只会被记录为冲突边或者环路边，如果图中的边同时是冲突边和环路边，那么该边将被记录为冲突边
            if (parent[node2] != node2) {
                conflict = i;
            } else {
                // 记录 node2 的父节点
                parent[node2] = node1;
                // 如果 node1 和 node2 已经连通，说明存在环路
                if (uf.find(node1) == uf.find(node2)) {
                    cycle = i;
                } else {
                    // 将 node1 和 node2 连通
                    uf.union(node1, node2);
                }
            }
        }
        if (conflict < 0) {
            // 如果不存在冲突边，返回环路边
            int[] redundant = {edges[cycle][0], edges[cycle][1]};
            return redundant;
        } else {
            // 如果存在冲突边，返回冲突边或者冲突边的父节点
            int[] conflictEdge = edges[conflict];
            if (cycle >= 0) {
                // 如果同时存在被记录的环路边，返回冲突节点的另一个边，即指向冲突节点的最先被记录的边
                int[] redundant = {parent[conflictEdge[1]], conflictEdge[1]};
                return redundant;
            } else {
                // 如果不存在环路边，返回冲突边
                int[] redundant = {conflictEdge[0], conflictEdge[1]};
                return redundant;
            }
        }
    }
}



class UnionFind {
    int[] ancestor;

    public UnionFind(int n) {
        ancestor = new int[n];
        for (int i = 0; i < n; ++i) {
            ancestor[i] = i;
        }
    }

    public void union(int index1, int index2) {
        ancestor[find(index1)] = find(index2);
    }

    public int find(int index) {
        if (ancestor[index] != index) {
            ancestor[index] = find(ancestor[index]);
        }
        return ancestor[index];
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
