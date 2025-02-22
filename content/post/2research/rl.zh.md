---
title: "强化学习"
date: 2025-02-01T10:39:49Z
draft: false
author: ["Martin"]
tags: 
- RL
description: ""
weight:
slug: "rl-note"
comments: true
showToc: true
TocOpen: true
hidemeta: false
disableShare: true
showbreadcrumbs: true
cover:
    image: "/img/rl-note/whatisrl.png"
    caption: ""
    alt: "what-is-rl"
    relative: true
mermaid: true
---
# 多臂老虎机问题(MAB)
## 符号&问题定义
- 大写斜体表示随机变量，例如$A, R, A_t, R_t$
- 小写字母表示这些随机变量的实现，例如$a, r, a_t, r_t, Pr${$A_t=a_t$}
- 花体，区间等表示集合，例如$\mathcal{A}, [0, 1], \mathbb{N}$

Given: a set of k actions, $\mathcal{A}$, number of rounds T.
Repeat for t in T rounds:
1. Algorithm selects arm $A_t \in \mathcal{A}$
2. Algorithm observes reward $R_t \in [0, 1]$
Goal: maximise expected total reward.

预期的奖励被称为Value：$q_*(a) = \mathbb{E}[R_t \mid A_t = a]$
## 问题核心(Explore-exploit dilemma)
由于各个老虎机的奖励分布未知，我们需要在探索和利用之间做出权衡，来最大化长期奖励。

探索（Exploration） 和 利用（Exploitation） 之间的权衡是 MAB 问题的核心：

- Exploration: 选择当前信息不足的选项，以获得关于奖励分布的更多知识。例如，如果某个老虎机的奖励不确定，你可能会尝试拉取它，以收集更多数据。
- Exploitation: 选择当前估计奖励最高的选项，以最大化即时收益。例如，如果你已经知道某个老虎机的平均奖励最高，你会更倾向于持续拉取它。

这种权衡的核心难点在于：
- 过度探索（过多尝试未知选项）可能导致较低的短期收益。
- 过度利用（过早锁定某个选项）可能导致错失更优的长期收益。
## 问题算法
### Action-Value Methods
我们可以估计某个动作$a$所获得奖励的样本均值。，具体函数表示为：
$$
Q_t(a) = \frac{\text{Sum of rewards when taken a so far}}{\text{Number of times taken a so far}}
$$
$$
Q_t(a) = \frac{1}{N_t(a)} \sum^{t-1}_{\tau=1}R _ {\tau=1} \cdot \mathbb{1} _ {A_t=a}
$$
其中
- $N_t(a)$ 是到当前时间 $t$ 为止，动作 $a$ 被选择的次数。
- $R_{\tau}$ 是第 $\tau$ 次执行动作的奖励。
- $\mathbb{1} _ {A_{\tau} = a}$ 是一个**指示函数**，当 $\tau$ 时刻选择了动作 $a$ 时，它的值为 1，否则为 0。

**如果一个动作被选取无穷次**，那么它的**样本均值会收敛**到真实的期望奖励，即：
$$
\lim_{N_t(a) \to \infty} Q_t(a) = q_*(a)
$$
### $\epsilon$- Greedy Action Selection
根据上面的公式，我们可以把exploit表示为
$$
A_t = A_t^* = \mathop{\arg\max}\limits_{a} Q_t(a)
$$
exploration(随机选择action)表示为
$$
A_t\sim\text{Unif}(\mathcal{A})
$$
```python
def epsilon_greedy(Q, epsilon, actions):
    if np.random.rand() < epsilon:
        # 以概率 epsilon 进行探索
        return np.random.choice(actions)
    else:
        # 以概率 (1 - epsilon) 选择当前最优动作
        return np.argmax(Q)
    self.execute(A_t)
    self.observe(R_t)

    self.update(N_t_a, Q_t_a)
```

如何在递增的同时避免重复计算？
$$
Q_n = \frac{R_1 + ... + R_{n-1}}{n-1}
$$
$$
Q_{n+1} = Q_n + \frac{1}{n}[R_n - Q_n]
$$
这也是强化学习中update rules的标准形式
$$
\text{NewEstimate <— OldEstimate + StepSize[Target - OldEstimate]}
$$
### Non-Stationary Problem
假设true action value会随时间而变化，这是强化学习中经常遇到的non-stationary problem，这个时候再用sample average就不合适了，我们引入一个step-size parameter $\alpha\in[0, 1]$来跟踪action value，那么更新函数变为
$$
Q_{n+1} = Q_n + \alpha[R_n - Q_n]
$$
### Stochastic Approximation Convergence Conditions
**随机逼近（Stochastic Approximation, SA）** 是一种用于在噪声环境中迭代逼近最优解的方法。这种方法常用于机器学习、强化学习（如 Q-learning）、优化算法（如随机梯度下降）等。

在强化学习中，**值函数更新** 采用随机逼近的方式，例如：
$$
Q_{t+1}(s, a) = Q_t(s, a) + \alpha_t [R_t + \gamma \max_{a'} Q_t(s', a') - Q_t(s, a)]
$$
其中 $\alpha_t$ 是学习率，目标是让 $Q_t(s,a)$ 收敛到最优值 $Q^*(s,a)$。

为了保证 **随机逼近算法** 能够收敛，必须满足**收敛条件（Convergence Conditions）**:

**学习率满足 Robbins-Monro 条件**
$$
\sum_{t=1}^{\infty} \alpha_t = \infty, \quad \sum_{t=1}^{\infty} \alpha_t^2 < \infty
$$

- **第一项** $ \sum \alpha_t = \infty $ 确保算法在**长期内继续学习**，否则算法可能过早停止更新。
- **第二项** $ \sum \alpha_t^2 < \infty $ 确保学习率最终**足够小**，否则收敛会受噪声影响。

示例：
- **满足条件的学习率：** $ \alpha_t = \frac{1}{t} $、$ \alpha_t = \frac{1}{\sqrt{t}} $
- **不满足条件的学习率：** 固定的 $ \alpha_t = 0.1 $（因为求和不发散）

### Upper Confidence Bound (UCB)

相比于 **$\epsilon$-Greedy** 算法的随机探索策略，UCB 通过构造一个上置信界（Upper Confidence Bound）来进行探索，从而更快地收敛到最优解。

UCB 采用 **置信区间（confidence bound）** 来决定是否探索某个动作：
- **利用（Exploitation）：** 选择历史上平均奖励最高的动作（即当前最优动作）。
- **探索（Exploration）：** 选择那些尝试次数较少、不确定性较大的动作。

**公式：**
$$
A_t = \arg\max_{a} \left[ Q_t(a) + c \sqrt{\frac{\ln t}{N_t(a)}} \right]
$$
其中：
- $ Q_t(a) $ 是动作 $ a $ 在时间 $ t $ 时刻的平均奖励（历史经验）。
- $ N_t(a) $ 是**动作 $ a $ 被选择的次数**。
- $ t $ 是当前时间步（当前实验的总次数）。
- $ c $ 是**探索参数**，控制探索程度（通常 $ c $ 取 $\sqrt{2}$）。
- $ \ln t $ 是对数项，使得探索随时间减少。

**解释：**
- $ Q_t(a) $ 代表 **利用**（Exploitation）：倾向于选择历史上表现最好的动作。
- $ \sqrt{\frac{\ln t}{N_t(a)}} $ 代表 **探索**（Exploration）：当某个动作选择次数 \( N_t(a) \) 很小时，该项较大，鼓励探索。

```python
import numpy as np

def UCB(Q, N, t, c=1.0):
    ucb_values = Q + c * np.sqrt(np.log(t) / (N + 1e-5))  # 避免除零错误
    return np.argmax(ucb_values)  # 选择 UCB 值最高的动作
```
其中：
- `Q`：存储每个动作的**平均奖励**。
- `N`：存储每个动作的**选择次数**。
- `t`：当前时间步数（总实验次数）。
- `c`：探索参数，控制探索强度。

# 马尔可夫决策过程(MDPs)
## 理论框架
马尔可夫决策过程包括状态空间$\mathcal{S}$，动作空间$\mathcal{A}$，奖励空间$\mathcal{R}$。MDP is finite if $\mathcal{S}$, $\mathcal{A}$, $\mathcal{R}$ is finite

Markov property: Future state and reward are independent of past states and actions, given the current state and action:

$Pr${$S_{t+1},R_{t+1} | S_t,A_t,S_{t−1},A_{t−1},...,S_0,A_0$}$ = Pr${$S_{t+1},R_{t+1} | S_t,A_t$}

状态$S_t$是交互历史的充分总结。设计compact Markov states是RL中的一项工程工作。
## 核心数学概念&函数
### Policy
### Returns
### Value Functions
### Bellman Equation

# 动态规划
## Policy Iteration
### Iterative policy evaluation
### Policy Improvement

## Value Iteration 

## Asynchronous and generalised DP

# 蒙特卡洛方法


