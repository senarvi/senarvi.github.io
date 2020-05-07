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
* The state-action value function <span>$q_π(s, a)$</span> is the expected utility at state <span>$s$</span> after taking action <span>$a$</span> and then following policy <span>$π$</span>.

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
