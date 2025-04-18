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
马尔可夫决策过程包括状态空间$\mathcal{S}$，动作空间$\mathcal{A}$，奖励空间$\mathcal{R}$。MDP是有限集，如果$\mathcal{S}$, $\mathcal{A}$, $\mathcal{R}$都是有限集

马尔可夫性质：简单说就是未来的状态只与当前一个状态有关，独立于过去的所有状态（Future state and reward are independent of past states and actions, given the current state and action）

$Pr${$S_{t+1},R_{t+1} | S_t,A_t,S_{t−1},A_{t−1},...,S_0,A_0$}$ = Pr${$S_{t+1},R_{t+1} | S_t,A_t$}

状态$S_t$是交互历史的充分总结。设计compact Markov states是RL中的一项工程工作。

以recycling robot为例。
- States
    - high battery level
    - low battery level
- Actions
    - search for can
    - wait for someone to bring can
    - recharge battery at charging station
- Rewards: number of cans collected

| $s$  | $a$    | $s'$ | $p(s'\mid s, a)$ | $r(s, a, s')$       |
| ---- |  ----  | ---- | ----             | ----                |
|high  |search  | high |  $\alpha$        |$r_{\text{research}}$|
|high  |search  | low  | $1 - \alpha$     |$r_{\text{research}}$|
|low   |search  | high | $1 - \beta$      |        -3           |
|low   |search  | low  | $\beta$          |$r_{\text{research}}$|
|high  |wait    | high | 1                |$r_{\text{wait}}$    |
|high  |wait    | low  | 0                |-                    |
|low   |wait    | high | 0                |-                    |
|low   |wait    | low  | 1                |$r_{\text{wait}}$    |
|high  |recharge| high | 1                |0                    | 
|high  |recharge| low  | 0                |-                    |

## 核心数学概念&函数
### Policy
马尔可夫决策过程由policy控制。

$\pi(a\mid s) = $ 在状态$s$的情况下选择动作$a$的概率
| $\pi(a\mid s)$  | search  | wait | recharge |
| ---- |  ----  | ---- | ---- |
|high  |0.9  | 0.1  | 0       |
|low   |0.2  | 0.3  | 0.5     |

> Agent的目标就是学一个能最大化累积奖励的策略

### Returns（回报）
**Total Return**

策略应该最大化预期回报（$G_t$）
$$
G_t = R_{t+1} + R_{t+2} + ... + R_{T} = R_{t+1} + G_{t+1}
$$
where $T$ is final time step

Assumes terminating episodes: 适用于存在明确终止条件的任务。例如在象棋游戏中，一个玩家获胜就终止。通过设置允许的时间步数来强制终止

**Discounted Return**

对于不能终止的（无限的）场景，可以使用折扣率 $\gamma \in [0, 1)$
$$
G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + ... = R_{t+1} + \gamma G_{t+1}
$$
$\gamma$是折扣因子，用于降低远期奖励的价值。

这适用于无限回合任务，例如股票交易策略，智能体持续进行交易，没有固定结束时间。
### Value Functions
这张图给一个overview
![main quantities](/img/rl-note/value_function.png)
**Bellman Equation**
在强化学习和马尔可夫决策过程中，贝尔曼方程用于描述状态值函数和动作值函数的递归关系。这两是强化学习中求解最优策略的重要数学工具。

**State Value Function and the Bellman Equation**

根据Markov property，可以把状态价值函数写成Bellman Equation的递归形式。
$$
v_\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s]
$$
这个公式的意思就是给定当前状态的情况下，策略$\pi$可以获得的回报期望值

将公式展开，得到：

$$
v_\pi(s) = \sum_a \pi(a \mid s)r(s, a) + \gamma\sum_{s' \in S} p(s'\mid s, a)\cdot v_\pi(s')
$$

其中$\sum_a \pi(a \mid s)r(s, a)$表示即时奖励（选择动作$a$的概率$\times$这样做带来的奖励值），$\gamma$是折扣因子，$\sum_{s' \in S} p(s'\mid s, a)\cdot v_\pi(s')$表示expected future value (当前状态$s$, 采取动作$a$之后，有多大概率转到每个可能的$s'$,每个$s'$会带来多大价值，全部加权求和后就是未来的预期价值)

直觉理解：眼前收益+未来收益（带有折扣率来降低影响）

**Action Value Function and the Bellman Equation**
$$
q_\pi(s, a) = r(s, a) + \gamma \sum_{s' \in S} p(s' \mid s, a)\cdot v_\pi(s')
$$
用期望简写
$$
q_\pi(s, a) = r(s, a) + \gamma \mathbb{E} _ {s'}[ v _ \pi(s') ]
$$

## GridWorld Example
GridWorld是强化学习中常见的环境之一。它是一个简单的离散状态空间环境，智能体在其中移动，试图最大化其累积奖励。

## Summary (Main idea)
- Markov Decision Process: the canonical way to model RL problems
- Policy: $\pi$ is a strategy for assigning actions to states.
- Value: $v_\pi(s)$ captures expected cumulative discounted reward (Long term view of the quality of a policy)
- Goal: Find a policy that maximises value.

# 动态规划
动态规划核心思想：use Bellman Equations to organise search for good policies:
## DP Algo1: Policy Iteration 
这个算法包含两个阶段:
- Policy evaluation: compute $v_\pi$ for $\pi$
- Policy improvement: make policy $\pi$ greedy with respect to $v_\pi$

具体表示如下（这个过程会收敛到最优策略）
$$
\pi_0 \xrightarrow{E} v_{\pi_0} \xrightarrow{I} \pi_1 \xrightarrow{E} v_{\pi_1} \xrightarrow{I} ... \xrightarrow{I} \pi_* \xrightarrow{E} v_*
$$
### Iterative policy evaluation
伪代码如下
```plaintext
Input: pi, the policy to be evaluated
Initialize an array V(s) = 0, for all s
Repeat
    delta <- 0
    For each s in S:
        v <- V(s)
        V(s) <- State Value Function
        delta <- max(delta, |v - V(s)|)
until delta < theta (a small positive number)
Output V \approx= v_pi
```

### Policy Improvement
计算在当前state value funtion $V(s)$下，每个状态$s$对不同动作$a$的action value。然后选择最优动作$\pi'(s)$，如果这与之前的$\pi(s)$不同，则策略发生了更新，并继续迭代，否则停止。

## DP Algo2: Value Iteration 
以租车问题为例。有两个车辆租赁点，根据分布随机请求和返回车辆。状态（States）表示为(n1, n2)，其中ni是位置i的汽车数量（每个地方最多20辆）。动作（Actions）表示从一个位置移动到另一个位置的车子的数量。其中如果是从1移到2就是正数，从2移到1就是负数，最多5辆。奖励（Rewards）表示为每个时间步中，租出去一辆车+\$10，移动一辆车-\$2。最后$\gamma$ = 0.9。

Policy iteration使用Bellman equation作为operator:
$$
v_{k+1}(s) = \sum_a \pi(a\mid s)\sum_{s', r}p(s', r\mid s, a) [r+\gamma v_k(s')]
$$
其中$s\in S$

Value iteration使用Bellman optimality equation作为operator:
$$
v_{k+1}(s) = \max_a \sum_{s', r}p(s', r\mid s, a) [r+\gamma v_k(s')]
$$
其中$s\in S$

Value iteration的伪代码如下
![value iteration](/img/rl-note/value_iteration.png)
## Asynchronous and generalised DP
目前为止，动态规划方法执行的是穷举。如果状态空间很大，policy evaluation and improvement就不可计算了。

异步动态规划方法避免了标准动态规划中必须同时更新所有状态的限制，而是选择性地更新某些状态，以提高计算效率和收敛速度。

改进点：
- 状态异步更新：不要求所有状态在同一时间更新，而是按需更新一部分状态。
- 优先级更新：优先更新那些 价值变化较大或与最终策略最相关的状态，使有价值的信息更快传播。
- 加速收敛：减少不必要的计算，特别适用于大规模问题，如机器人控制、路径规划、强化学习等。

# 蒙特卡洛方法

# Temporal DIfference Learning

# Planning and Learning

# Value function approximation

# Policy Gradient Methods


