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
3. GNRON（Goals nonreachable with obstacle near by）无法到达障碍物附近的目标。


## 手动实现APF

### 思路

在激光的回调函数中调用，拿到Lansecan的数据，数据结构如下：

```bash
sensor_msgs/LaserScan

angle_min (float32) scan 的开始角度 [弧度]
angle_max (float32) scan 的结束角度 [弧度]
angle_increment (float32) 测量的角度间的距离 [弧度]
time_increment (float32) 测量间的时间 [秒] – 如果扫描仪在移动,这将用于插入 3D 点的位置
scan_time (float32) 扫描间的时间 [秒]
range_min (float32) 最小的测量距离 [米]
range_max (float32) 最大的测量距离 [米]
```

在world中拿到goal（目标点）和机器人所在位置坐标，分别为goal_（x，y），pose_（x，y）。然后计算goal与pose的欧式距离r，再使用atan2（dy，dx）计算出目标点相对当前位置的角度theta_goal，单位 [弧度]，其中dy = goal_（y）-pose_（y），dx同理。这里theta_goal是在世界坐标系中的，而非机器人坐标系。

```c++
pose_x = goal_.pose.position.x - pose_.pose.position.x;
pose_y = goal_.pose.position.y - pose_.pose.position.y;
float theta_goal = atan2(pose_y, pose_x);
float r = sqrt(pow(pose_x, 2) + pow(pose_y, 2));
```

引力场计算公式：

$$
U_{att}(q) = \frac12\xi r^2 
$$

引力场下的引力计算为：

$$
F_{att} = -\Delta U_{att}(q) = \xi r
$$

这里将系数设置为GOAL_FORCE（default=1），需要进行世界坐标系的xy分解：

```c++
pose_x = goal_.pose.position.x - pose_.pose.position.x;
pose_y = goal_.pose.position.y - pose_.pose.position.y;
float theta_goal = atan2(pose_y, pose_x);
float r = sqrt(pow(pose_x, 2) + pow(pose_y, 2));
float goal_force = GOAL_FORCE * r;
float goal_x = goal_force * cos(theta_goal);
float goal_y = goal_force * sin(theta_goal);
```

斥力场计算公式：

$$
U_{rep}(q) = \frac12 \eta \left( \frac{1}{\rho (q,q_{obs})} - \frac1{\rho_0} \right)^2 \rho ^n(q,q_{goal}), \rho (q,q_{obs}) \leq \rho_0
$$

$$
U_{rep}(q) = 0, \rho (q,q_{obs}) > \rho_0
$$

这里需要设置两个系数，参数n和阈值p0,在原有的斥力场中加上目标点和机器人距离的影响，来缓解GNRON问题。参数n一般设置为2，阈值p0是障碍物的影响半径，如果机器人距离障碍物一定距离，即使可以看见障碍物，也对机器人没有影响。

斥力计算：

$$
F_{rep}(q) = -\Delta U_{rep}(q) = F_{rep1}n_{OR} + F_{req2}n_{RG} , \rho (q,q_{obs}) \leq \rho_0
$$

$$
F_{rep1}=\eta \left( \frac1{\rho (q,q_{obs})} - \frac1{\rho_0} \right) \frac{\rho ^n(q,q_{goal})}{\rho (q,q_{obs})},F_{req2}=\frac{n}{2}\eta \left( \frac1{\rho (q,q_{obs})} - \frac1{\rho_0} \right)^2\rho ^{n-1}(q,q_{goal})
$$

### 遇到的问题：

不需要发布点云

atan2方法的参数为y，x

theta为世界坐标系中的弧度值，因为LaserScan中的angle_min与angle_max都是在世界坐标系下的弧度值，且已知（）
启发式算法以计算角度值代替角速度值