---
title: "Decision Theory and Reinforcement Learning"
permalink: /notes/reinforcement-learning/
author_profile: true
math: true
---

## Decision Theory
* An agent observes the environment and performs an action that changes the environment
* An example: a player can make moves in a game
* A game tree defines all the possible actions taken by the agents
* Utility defines a score for the leaf nodes of a game tree

### Minimax
* When the game tree is known, minimax decision rule defines the optimal sequence of actions
* Assumes that the other player always chooses the branch with minimum utility
* Always chooses the branch with maximum utility


## Markov Decision Process
* A framework for modeling decision-making problems
* Can be solved using Monte Carlo tree search or reinforcement learning
* An MDP is composed of states, actions, and transition probabilities <span>$p(s’ \mid s, a)$</span>
* Generally, the result of each action is not known in advance (the environment is *stochastic*)
* Markovian transition model: the probability of reaching state <span>$s’$</span> when performing action <span>$a$</span> in state <span>$s$</span> depends only on the current state <span>$s$</span>

### Reward
* In each state the agent receives reward <span>$R(s)$</span>
* Alternatively, we may consider the expected reward <span>$R(s, a, s’)$</span> after taking the action <span>$a$</span>

### Policy
* Policy is a mapping from states to actions <span>$π(s)$</span>, a probability of taking an action <span>$π(a \mid s)$</span>, or e.g. a search process
* Executing the same policy will generally not end up in the same state

### Utility / Return
* Future rewards are discounted by a factor of gamma per time step
* Utility or return is the cumulative discounted reward <span>$U = R(s_0) + γ R(s_1) + γ^2 R(s_2) + ...$</span> of a sequence of states

### Value
* It is useful to assess states with respect to the expected utility
* The value function <span>$v_π(s)$</span> is the expected utility at state <span>$s$</span> by following policy <span>$π$</span>
* The state-action value function <span>$q^π(s, a)$</span> is the expected utility at state <span>$s$</span> after taking action <span>$a$</span> and then following policy <span>$π$</span>.

### Optimal Policy
* Optimal policy is the one that maximizes the expected utility
* Bellman equation: utility of a state is the reward from that state plus the expected utility of the next state, assuming optimal policy

<div>$$
U(s) = R(s) + γ \max_a \sum_{s’} P(s’ \mid s, a) U(s’)
$$</div>


## Monte Carlo Tree Search
* Simulates game play (moving down in the search tree) starting from the current game state according to a fast rollout policy (e.g. randomly)
* After e.g. a given time has elapsed, evaluates the end state
* The node where the simulation started is marked visited
* Propagates the evaluation result up to the root of the game tree and update all the nodes along the way
* Each node stores statistics on how promising it is (reward) and how well it has been explored (number of visits)
* Upper confidence bound (UCD) function decides whether to exploit the promising nodes or explore less visited nodes when choosing how to expand the tree
* May also use a policy network to provide prior probabilities for different moves


## Reinforcement Learning
* A very general framework for phrasing reward-related learning
* The agent cannot necessarily observe the whole environment
* Feedback in form of rewards
* The agent learns the best behaviour by observing the state space
* A state can be defined as a summary of previous actions, observations, and rewards

### Compared to Supervised Learning
* Typically the objective is a discrete metric so a gradient is not available
* Decisions are sequential (the next input depends on the current decision)
* The agent is not told which action to take
* The rewards may be delayed in time

### Problematic Areas
* Continuous control
* Sparse rewards
* Long-term correlations


## Approaches to Reinforcement Learning

### Utility-Based Agents
* Learn a utility function on states
* Model the environment to know the state to which an action will lead

### Action-Value Methods
* Learn the optimal state-action value function <span>$q^*(s, a)$</span> (the maximum <span>$q$</span> under any policy) and use it to design a good policy
* Can be used without a model of the environment (state transition and reward probability functions)
* Cannot look ahead when it's unknown where the action leads
* Off-policy agents learn the optimal policy while performing actions of another policy to ensure adequate exploration
* On-policy agents execute the same policy that they learn

### Policy-Gradient Methods
* Optimize a parameterized policy (for example a neural network) by gradient descent
* Policy gradient theorem reformulates the loss in such a way that only the policy needs to be differentiable
* A value function may be used to learn the policy parameters, but is not required for action selection

### Actor-Critic Methods
* An actor chooses an action to take and a critic evaluates being in a state
* For example [a neural network with actor and critic output layers](https://github.com/pytorch/examples/blob/master/reinforcement_learning/actor_critic.py)
* Policy-gradient is a special case of this more general idea, where the policy is the actor and the reward is used as the critic

### Reflex Agents
* Learn a mapping from states to actions


## Utility Estimation
* The agent performs trials
* Each trial provides a sample of the expected reward-to-go from the states it visited
* Disregard knowledge about the dependency of the utility between states (Bellman equation)


## Q-Learning
* Q-learning is an off-policy action-value method
* The optimal state-action value function <span>$q^*(s, a)$</span> predicts the maximum (under any strategy) expected utility in state <span>$s$</span> after action <span>$a$</span>
* The policy is to select the action with maximal <span>$q$</span>
* <span>$q$</span> obeys the Bellman equation—can be used to define an iterative update so that <span>$q_i$</span> will converge to <span>$q^*$</span>:

<div>$$
q_{i+1}(s, a) = R(s) + γ \max_{a'} \sum_{s’} P(s’ \mid s, a) q_i(s’, a’)
$$</div>

* Can operate in discrete state and action spaces only
* Policy is not differentiable—action probabilities change abruptly when a different action has the maximal value

### Dynamic Programming Approach to Q-Learning
* Think of <span>$q$</span> as the length of a path segment in a graph
* Use dynamic programming to find the shortest path
* Requires a model of the environment

### Sample-Based Approaches to Q-Learning
* Don't require a model of the environment

### Deep Q-Learning
* Calculating <span>$q(s, a)$</span> exactly for every state and action is impractical except in trivial cases.
* <span>$q$</span> can be approximated as a neural network with weights <span>$w$</span>: <span>$q(s, a, w)$</span>
* Deep Q-learning minimizes the squared error of the predicted <span>$q$</span> value and the target, which is the same as the iterative update <span>$q_{i+1}(s, a)$</span>
* The target is nonstationary—it changes at each iteration when the weights are updated


## Policy-Gradient Methods
* Define an explicit policy
* The policy gives smooth action probabilities and is differentiable
* Don't (necessarily) learn the value function
* Effective in high-dimensional / continuous action spaces
* No need for a specific set of rules for the policy
* Popular for robotics

### REINFORCE
* REINFORCE is the first and most well known policy-gradient method
* In the context of sequence generation, the policy is a model that is used to generate the most likely sequence
* The generated sequence is compared to the optimal (ground truth) sequence to compute the reward
* The policy is updated based on the gradient of the loss, which is the negative expected reward
* The expected reward is approximated using one or more samples
