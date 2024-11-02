---
title: 并查集
created: 2024-10-27
tags: 
    - Algorithm
    - UnionFind
---

# 并查集

## 1. 什么是并查集

并查集是一种树形的数据结构，用于处理一些不交集的合并及查询问题。它的主要操作有两个：

- 查找（Find）：确定元素属于哪一个子集。它可以被用来确定两个元素是否属于同一个子集。
- 合并（Union）：将两个子集合并成同一个集合。

## 2. 并查集的实现

并查集的实现一般有两种方式：数组实现和树实现。

### 2.1 数组实现

数组实现是最简单的一种实现方式，每个元素对应一个下标，数组的值表示该元素的父节点。数组的初始化时，每个元素的父节点都是自己。

```java
class UnionFind {
    private int[] parent;

    public UnionFind(int n) {
        parent = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    public void union(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX != rootY) {
            parent[rootX] = rootY;
        }
    }
}
```

### 2.2 树实现

树实现是一种更加高效的实现方式，每个元素对应一个节点，节点之间通过父子关系连接。树的初始化时，每个元素的父节点都是自己。

```java
class UnionFind {
    private int[] parent;
    private int[] rank;

    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            rank[i] = 1;
        }
    }

    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    public void union(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX != rootY) {
            if (rank[rootX] > rank[rootY]) {
                parent[rootY] = rootX;
            } else if (rank[rootX] < rank[rootY]) {
                parent[rootX] = rootY;
            } else {
                parent[rootY] = rootX;
                rank[rootX]++;
            }
        }
    }
}
```

## 3. 并查集的应用

并查集主要用于处理一些不交集的合并及查询问题，例如：

- 判断无向图中是否有环
- 判断有向图中是否有环
- 计算连通分量的个数
- 计算连通分量的大小
- 计算连通分量的最大值

## 4. 并查集的优化

并查集的优化主要有两种方式：路径压缩和按秩合并。

### 4.1 路径压缩

路径压缩是一种优化方式，通过将树的深度降低，使得树的高度更低，从而提高查询的效率。

```java
public int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]);
    }
    return parent[x];
}
```

### 4.2 按秩合并

按秩合并是一种优化方式，通过将深度较小的树合并到深度较大的树上，使得树的高度更低，从而提高查询的效率。

```java
public void union(int x, int y) {
    int rootX = find(x);
    int rootY = find(y);
    if (rootX != rootY) {
        if (rank[rootX] > rank[rootY]) {
            parent[rootY] = rootX;
        } else if (rank[rootX] < rank[rootY]) {
            parent[rootX] = rootY;
        } else {
            parent[rootY] = rootX;
            rank[rootX]++;
        }
    }
}
```

## 5. 并查集的时间复杂度

并查集的时间复杂度主要取决于路径压缩和按秩合并的优化方式，一般情况下，查找和合并的时间复杂度都是 $O(\alpha(n))$，其中 $\alpha(n)$ 是 Ackermann 函数的反函数，它的增长非常缓慢，可以认为是一个很小的常数。

## 6. LeetCode 例题

[冗余连接](https://leetcode.cn/problems/redundant-connection)
