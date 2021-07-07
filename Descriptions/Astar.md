## 一、简介
AStar（又称 A*），它结合了 Dijkstra 算法的节点信息（倾向于距离起点较近的节点）和贪心算法的最好优先搜索算法信息（倾向于距离目标较近的节点）。可以像 Dijkstra 算法一样保证找到最短路径，同时也像贪心最好优先搜索算法一样使用启发值对算法进行引导。

简单点说，AStar的核心在于将游戏背景分为一个又一个格子，每个格子有自己的靠谱值，然后通过遍历起点的格子去找到周围靠谱的格子，接着继续遍历周围…… 最终找到终点。

## 二、截图
![](https://img-blog.csdn.net/2018053118203257?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E3NjQ0MjQ1Njc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](https://img-blog.csdn.net/20180531180754188?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E3NjQ0MjQ1Njc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](https://img-blog.csdn.net/20180531180748935?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E3NjQ0MjQ1Njc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)