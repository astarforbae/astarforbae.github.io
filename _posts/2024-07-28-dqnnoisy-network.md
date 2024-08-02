---
layout: post
title: "DQN改进（一）：Noisy Network"
subtitle: ""
date: 2024-07-28
author: "Yi"
mathjax:    true
header-img: "img/post-bg-2015.jpg"
tags: [RL]
published: false
---

今天介绍一个增强探索能力的方法：Noisy Network。这个方法是由DeepMind在2017年提出的，通过在网络的权重上增加噪声来增加探索能力。

原文链接：[Noisy Networks for Exploration](https://arxiv.org/abs/1706.10295)

## 传统的探索方法

探索与利用是强化学习中的一个重要问题。通常的做法是使用$\epsilon-greedy$的方法，即以$\epsilon$的概率随机选择动作，以$1-\epsilon$的概率选择当前最优的动作。并且让$\epsilon$随着时间逐渐减小。常用的衰减方法比如指数衰减：
$
\epsilon = \epsilon_{min} + (\epsilon_{max} - \epsilon_{min}) \times e^{(-\alpha t)}
$
