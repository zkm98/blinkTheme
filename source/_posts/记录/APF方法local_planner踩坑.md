---
title: APF方法的local planner 
description: 人工势场法（Artificial Potential Field）在全局规划与局部规划的应用
date: 2020-09-14 15:19:00
author: zhangyuanes
categories: 记录
mathjax: ture
tags:
  - SLAM
  - 人工势场法
---

## 人工势场法（Artificial Potential Field）

移动机器人在给定初始点和最终点的情况下，在不与障碍物发生碰撞的情况下执行由自动路径规划模块规划出的路径到达最终点。

在实际情况下，部分或全部环境未知，路径规划通常使用基于行为的方法(Behavior based methods)和使用智能控制技术的方法(methods using Intelligent Control Techniques)。

已知环境中的路径规划包括生成供机器人遵循的路径，前提是该环境的整体（静态或动态）都是已知的。在这种情况下，方法可以被大致分为两类：最小化代价方法，势场法。最小化代价方法中一个很典型的算法就是的启发式A*算法。

人工势场法是势场法中避障规划路径的常见方法。人工势场法在障碍物周围使用斥力场来使得机器人离开（不碰撞），目标点周围使用吸引力场来吸引机器人。因此，机器人受到的总力等于总场势梯度的负值（矢量）。这样的合力驱动机器人朝着目标点的位置下降，直到其到达目标点或者停止（可能陷入局部最小，类似于梯度下降）。

人工势场法的主要问题以下几点：

1. 距离很近的障碍物之间没有通道（不能通过）；
2. 在狭小环境下，机器人可能陷入平衡位置，并且会振荡或以闭环运行；
3. GNRON(Goals nonreachable with obstacle near by)无法到达障碍物附近的目标。


不需要发布点云

atan2方法的参数为y，x

theta为世界坐标系中的弧度值

启发式算法以计算角度值代替角速度值