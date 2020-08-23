# Bayesian

## Using Bayesian Method on a Bernoulli Multi-Armed Bandit

For an introduction to Multi Armed Bandits, refer to [Tutorial On Bandits](https://genrl.readthedocs.io/en/latest/usage/tutorials/Tutorial_on_bandits.html).

This method is also based on the prinicple - 'Optimism in the face of uncertainty', like [UCB](https://genrl.readthedocs.io/en/latest/api/bandit/genrl.bandit.agents.mab_agents.html#module-genrl.bandit.agents.mab_agents.ucb). We initially _assume_ an initial distribution(prior) over the quality of each of the arms. We can model this prior using a Beta distribution, parametrised by alpha($\alpha$) and beta($\beta$).

$$PDF = \frac{x^{\alpha - 1} (1-x)^{\beta -1}}{B(\alpha, \beta)}$$

Let's just think of the denominator as some normalising constant, and focus on the numerator for now. We initialise $\alpha$ = $\beta$ = 1. This will result in a uniform distribution over the values (0, 1), making all the values of quality from 0 to 1 equally probable, so this is a fair initial assumption. Now think of $\alpha$ as the number of times we get the reward '1' and $\beta$ as the number of times we get '0', for a particular arm. As our agent interacts with the environment and gets a reward for pulling any arm, we will update our prior for that arm using Bayes Theorem. What this does is that it gives a posterior distribution over the quality, according to the rewards we have seen so far.

This is quite similar to [Thompson Sampling](https://genrl.readthedocs.io/en/latest/api/bandit/genrl.bandit.agents.mab_agents.html#module-genrl.bandit.agents.mab_agents.thompson). But what is different here is that we explicity try to calculate the uncertainty of a particular action by calculating the standard deviation($\sigma$) of its posterior. We add this std. dev to the mean of the posterior, giving us an _upper bound_ of the quality of that arm. At each timestep we select a greedy action based on this upper bound we calculated. 

$$a_t = argmax(q_t(a) + \sigma_{q_t})$$

As we try out an action more and more, the standard deviation of the posterior decreases, corresponding to a decrease in the uncertainty of that action, which is exactly what we want. If an action has not been tried that often, it will have a wider posterior, meaning higher chances of it getting selected based on its upper bound.

Code to use Bayesian method on a Bernoulli Multi-Armed Bandit:

```python
import gym
import numpy as np

from genrl.bandit import BayesianUCBMABAgent, BernoulliMAB, MABTrainer

bandits = 10
arms = 5
alpha = 1.0
beta = 1.0

reward_probs = np.random.random(size=(bandits, arms))
bandit = BernoulliMAB(bandits, arms, reward_probs, context_type="int")
agent = BayesianUCBMABAgent(bandit, alpha, beta)

trainer = MABTrainer(agent, bandit)
trainer.train(timesteps=10000)
```

More details can be found in the docs for [BernoulliMAB](https://genrl.readthedocs.io/en/latest/api/bandit/genrl.bandit.bandits.multi_armed_bandits.html#genrl.bandit.bandits.multi_armed_bandits.bernoulli_mab.BernoulliMAB), [BayesianUCBMABAgent](https://genrl.readthedocs.io/en/latest/api/bandit/genrl.bandit.agents.mab_agents.html#module-genrl.bandit.agents.mab_agents.bayesian) and [MABTrainer](https://genrl.readthedocs.io/en/latest/api/common/bandit.html#module-genrl.bandit.trainer).