---
date: 2025-06-24
title: "{{title}}"
tags:
  - useless
  - insight
---

## Definition

将原本具有半正定约束（如 $\rho \succeq 0$）或线性等式约束（如 $\operatorname{Tr}(\rho)=1$）的量子信息优化问题，通过**参数化或近似策略**，转化为**无约束优化问题**，使其可用于高维训练或神经网络建模中。

## Motivation

* 量子信息中常见 SDP 优化问题 [[SDP]]，但 SDP 的计算复杂度高，求解的维度很难 scaling
* 能否转为无约束优化问题，并用梯度下降求解？

## Methodology

1. 量子态参数化：使用参数矩阵 $T(\theta)$ 表示密度矩阵
  $$
  \rho(\theta) = \frac{T(\theta) T^\dagger(\theta)}{\operatorname{Tr}(T(\theta) T^\dagger(\theta))}
  $$

自动满足：

- $\rho \succeq 0$
- $\operatorname{Tr}(\rho) = 1$

1. 罚函数：将 SDP 约束加入损失函数作为 soft penalty：
  $$
  \mathcal{L}_{\text{total}} = \mathcal{L}_{\text{fit}} + \lambda_1 \|\sum_k E_k - I\|_F^2 + \lambda_2 \sum_k \max(-\lambda_{\min}(E_k), 0)^2
 $$

## Discussion

其实还是没什么用……神经网络的参数量依然虽维度指数上升，idea 暂时作废。
