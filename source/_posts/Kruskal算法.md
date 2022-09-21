---
title: Kruskal算法
categories:
  - 数据结构与算法
tags:
  - 算法
  - 图
  - 最小生成树
  - 并查集
date: 2021-02-28 16:37:50
updated: 2021-02-28 16:37:50

---

一月份力扣出了一个月的图论题，可惜那时在封闭开发（爆粗口），有些没时间做。

Kruskal算法是一种用来查找[最小生成树](https://zh.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E7%94%9F%E6%88%90%E6%A0%91)的算法，适合用于查找**稀疏图**的最小生成树。

## 最小生成树

#### 什么是最小生成树

最小生成树是**一副连通加权无向图**中，**一颗权值最小的生成树**。

- 生成树：在[图论](https://zh.wikipedia.org/wiki/图论)中，[无向图](https://zh.wikipedia.org/wiki/图_(数学)) *G* 的**生成树**（英语：Spanning Tree）是具有 *G* 的全部[顶点](https://zh.wikipedia.org/wiki/顶点_(图论))，但边数最少的连通子图。
- 连通：无向图中，任意两个顶点都有**通路**。
- 加权：图分为有权图和无权图，权值可以理解为边的权重。
- 无向：图分为有向图和无向图，可以理解为边是否有方向。
- 权值最小：所有边的权值之和最小。

最小生成树在分布式系统中有非常重要作用。。(我收藏夹里的 MIT6.824 公开课有时间也要看看了。。)

## 并查集

并查集是一种数据结构，不是一种算法。

并查集是Kruskal算法的关键，用于快速判断两个元素是否在同一个集合之中。（结果不能有环）

这里有个大佬的文章，对并查集讲的十分透彻：[https://zhuanlan.zhihu.com/p/93647900](https://zhuanlan.zhihu.com/p/93647900)

go的代码实现：
<!--more-->
```go
type unionFind struct {
	parent []int
	rank   []int // 用于按秩合并
}

func (uf *unionFind) find(x int) int {
	if uf.parent[x] == x {
		return x
	}
	// 路径压缩
	uf.parent[x] = uf.find(uf.parent[x])
	return uf.parent[x]
}

func (uf *unionFind) merge(x, y int) bool {
	fx, fy := uf.find(x), uf.find(y)
	// 如果他们有相同的根节点，无需合并
	if fx == fy {
		return false
	}
	if uf.rank[fx] <= uf.rank[fy] {
		uf.parent[fx] = fy
	} else {
		uf.parent[fy] = fx
	}
	// 如果秩相同
	if uf.rank[fx] == uf.rank[fy] && fx != fy {
		// 这里为什么要把fy的深度加1呢
		// 因为uf.rank[fx] <= uf.rank[fy]时，uf.parent[fx] = fy
		// 所以他们深度本来相同。但是fx挂载了fy下面，所以反fy的深度增加了
		// 如果这里改为uf.rank[fx] < uf.rank[fy],那么会走到else里面的uf.parent[fy] = fx
		// 这时，fx的深度应该加1
		uf.rank[fy]++
	}
	return true
}

func newUnionFind(n int) *unionFind {
	parent := make([]int, n)
	rank := make([]int, n)
	for i := 0; i < n; i++ {
		parent[i] = i
		rank[i] = 1
	}
	return &unionFind{parent, rank}
}
```



## Kruskal(克鲁斯卡尔)

#### 步骤

1. 新建图G，G中拥有原图相同的节点，但没有边。
2. 将原图中所有的边，按照权值从小到大排序。（因为要权值和最小，所以从权值小的边开始挑）
3. 从权值最小的边开始，如果这条边连接的两个节点于图G中不在同一个连通分量中，则添加这条边到图G中。（避免形成环）
4. 重复3，直至图G中所有节点都在一个连通分量重。（一般重复n-1次，n为节点个数）

#### 举例

力扣中等题： [连接所有点的最小费用](https://leetcode-cn.com/problems/min-cost-to-connect-all-points/)

AC代码：

```go
...
// 前文并查集实现代码省略
func minCostConnectPoints(points [][]int) (ans int) {
	n := len(points)
	type edge struct{ v, w, dis int }
	edges := []edge{}
	// 记录边和边的距离信息
	for i, p := range points {
		for j := i + 1; j < n; j++ {
			edges = append(edges, edge{i, j, dist(p, points[j])})
		}
	}
	// 根据曼哈顿距离从小到大排序,题中的曼哈顿距离就是边的权值
	sort.Slice(edges, func(i, j int) bool { return edges[i].dis < edges[j].dis })

	uf := newUnionFind(n)
	// 只需要重复n-1次
	left := n - 1
	for _, e := range edges {
		if uf.merge(e.v, e.w) {
			ans += e.dis
			left--
			if left == 0 {
				break
			}
		}
	}
	return
}

func dist(p, q []int) int {
	return abs(p[0]-q[0]) + abs(p[1]-q[1])
}

func abs(a int) int {
	if a < 0 {
		return -a
	}
	return a
}
```

题中没有步骤的第一步，构造图，并且返回的值不是生成树，而是权值和。但是思想是一样的。

ps:  本来算法就比较渣，也没打算从事算法相关岗位(~~主要因为菜~~)，只是随便写写吧。