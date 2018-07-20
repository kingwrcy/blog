---
title: "广度优先算法-golang实现"
date: 2018-07-19T21:33:13+08:00
draft: false
categories: ["golang"]

---

# 广度优先(BFS)

[WIKI介绍](https://zh.wikipedia.org/wiki/%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)

广度优先搜索算法（英语：Breadth-First-Search，缩写为BFS），又译作宽度优先搜索，或横向优先搜索，是一种图形搜索算法。简单的说，BFS是从根节点开始，沿着树的宽度遍历树的节点。如果所有节点均被访问，则算法中止。广度优先搜索的实现一般采用open-closed表。

BFS是一种盲目搜索法，目的是系统地展开并检查图中的所有节点，以找寻结果。换句话说，它并不考虑结果的可能地址，彻底地搜索整张图，直到找到结果为止。BFS并不使用经验法则算法。

从算法的观点，所有因为展开节点而得到的子节点都会被加进一个先进先出的队列中。一般的实现里，其邻居节点尚未被检验过的节点会被放置在一个被称为 open 的容器中（例如队列或是链表），而被检验过的节点则被放置在被称为 closed 的容器中。（open-closed表）

<!--more-->

## 具体算法

1. 首先将根节点放入队列中。
2. 从队列中取出第一个节点，并检验它是否为目标。
3. 如果找到目标，则结束搜索并回传结果。
4. 否则将它所有尚未检验过的直接子节点加入队列中。
5. 若队列为空，表示整张图都检查过了——亦即图中没有欲搜索的目标。结束搜索并回传“找不到目标”。
6. 重复步骤2。

我这里是用`BFS`来实现走迷宫,迷宫中有墙的概念,所以跟上面的算法有点出入就是不用全图搜索,只需要搜索能走的地方即可.

迷宫一般是二维数组实现,所以也没有子节点的概念,而是换成了方向,上左下右四个方向.

具体做法就是把起点入队列,然后出队第一个节点开始,每个节点按照逆时针方向搜索,只要不到迷宫边界,并且搜素到的节点是之前未搜索过的,并且是能走的(非墙),则加入队列.

并把当前所花费的`步数`写入搜索到的节点之中.然后反复循环,直到要么查到了目标节点,要么队列为空(就是死路),如果成功搜索到了终点.

这时可以根据每个节点的`步数`来确定一条从起点到终点的最短路径.


迷宫原始数据:(0代表能走,1代表是墙)
{{< highlight go "linenos=table,linenostart=1" >}}
var data = [5][9]int{
	{0, 1, 0, 1, 0, 0, 0, 1, 0},
	{0, 1, 0, 1, 0, 1, 0, 1, 0},
	{0, 1, 0, 0, 0, 1, 0, 0, 0},
	{0, 0, 0, 1, 1, 1, 1, 1, 0},
	{0, 1, 1, 1, 1, 1, 1, 1, 0},
}

type point struct {
	x, y int //节点坐标
}
// 迷宫定义
type node struct {
	point          //坐标
	vistied   bool //是否已访问过
	available bool //是否可用,0可以,1不可用(墙)
	step      int  //走到这节点所需的步骤数目
}

//初始化迷宫
func initNodes() *maze {
	ll := len(data)    //行
	cl := len(data[0]) //列

	n := maze{}
	for i := 0; i < ll; i++ {
		p := []node{}
		for j := 0; j < cl; j++ {
			p = append(p, node{
				vistied:   false,
				available: data[i][j] == 0,
				point: point{
					x: i,
					y: j,
				},
			})
		}
		n = append(n, p)
	}
	return &n
}
{{< / highlight >}}

循环搜索四个方向的节点,并把到达所需的步数计入节点.

{{< highlight go "linenos=table,linenostart=1" >}}

func find(ns *maze, start, end point) {
	var finded bool
	//开始节点入队列
	queue = append(queue, ns.getPosition(start.x, start.y))

	for len(queue) > 0 {
		//取出队列头部元素，并设置为已访问
		current := queue[0]
		queue = queue[1:]
		current.vistied = true

		//如果当前是目标终点,退出
		if current.equal(end) {
			fmt.Printf("find the target %+v,exit!\n",end)
			finded = true
			printSteps(ns)
			break
		}

		//循环4个方向
		for _, dir := range directions {
			newNode, ok := current.move(ns, dir)
			//如果没出界 并且 新节点是可以走的 并且 新节点是没有访问过的
			if ok && newNode.available && !newNode.vistied {
				//把新节点的 step 置为 当前节点的 step +1
				newNode.step = current.step + 1
				//把新节点入队列
				queue = append(queue, newNode)
			}
		}
	}
	if !finded {
		fmt.Printf("can not fnd a way from start :%+v to end %+v\n", start, end)
	}
}

{{< / highlight >}}

四个方向定义如下:
{{< highlight go "linenos=table" >}}
var directions = []point{
	//四个方向,上->左->下->右
	{-1, 0}, {0, -1}, {1, 0}, {0, 1},
}
{{< / highlight >}}


完整代码见[这里](https://gist.github.com/kingwrcy/d1064c9787496d131e160f7d7c64df30)
