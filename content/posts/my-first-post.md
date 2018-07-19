---
title: "广度优先算法-golang实现"
date: 2018-07-19T21:33:13+08:00
draft: true
---

``` golang
package main

import (
	"fmt"
)

type point struct {
	x, y int //节点坐标
}

type node struct {
	point
	vistied   bool //是否已访问过
	available bool //是否可用
	step      int  //走到这节点所需的步骤数目
}
// 迷宫
type maze [][]node

// 获取坐标 x,y处的node对象
func (m *maze) getPosition(x, y int) (n *node) {
	return &((*m)[x][y])
}

func (m *maze) getLineCount() int{
	return len(*m)
}

func (m *maze) getColumnCount() int{
	return len((*m)[0])
}

var data = [5][9]int{
	{0, 1, 0, 1, 0, 0, 0, 1, 0},
	{0, 1, 0, 1, 0, 1, 0, 1, 0},
	{0, 1, 0, 0, 0, 1, 0, 0, 0},
	{0, 0, 0, 1, 1, 1, 1, 1, 0},
	{0, 1, 1, 1, 1, 1, 1, 1, 0},
}

var directions = []point{
	//四个方向,上->左->下->右
	{-1, 0}, {0, -1}, {1, 0}, {0, 1},
}

//存放待搜索的队列
var queue = []*node{}

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

// 四个方向移动,出界返回false,否则返回新节点
func (n *node) move(ns *maze, move point) (*node, bool) {
	if n.x+move.x >= ns.getLineCount() || n.x+move.x < 0 || n.y+move.y < 0 || n.y+move.y >= ns.getColumnCount() {
		return nil, false
	}
	x := n.x + move.x
	y := n.y + move.y
	return ns.getPosition(x, y), true
}

func (n *node) equal(p point) bool {
	if &p == nil {
		return false
	}
	return n.x == p.x && n.y == p.y
}

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

func printSteps(ns *maze) {
	for i := 0; i < ns.getLineCount(); i++ {
		p := (*ns)[i]
		for j := 0; j < ns.getColumnCount(); j++ {
			fmt.Printf("%3d", p[j].step)
		}
		fmt.Println()
	}
}

func main() {

	ns := initNodes()
	//开始节点 左上角
	start := point{0, 0}
	//结束节点 右下角
	end := point{ns.getLineCount() - 1, ns.getColumnCount() - 1}
	find(ns, start, end)
}

```