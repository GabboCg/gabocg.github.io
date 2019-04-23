
# An Introduction to Statistical Power and Power Analysis


## Hypothesis Testing and Statistical Power

When a hypothesis test is conducted, the goal is to make a determination about the true state of the world. While a true state of the world does in fact exist, it is __unknown__ to researchers, but we can use hypothesis testing to make an informed decision. Because the entire population is generally unavailable, decisions are made using only a sample from the population, which introduces error into the experiment.

There are two ways in which the conclusion of a hypothesis test can be wrong. The null hypothesis can be rejected, when it is in fact true. This is referred to as a __Type I Error__ or sometimes a false positive and denoted by $$\alpha$$. We get  __Type II Error__ when we fail to reject a null hypothesis that is in fact false. This is also referred to as a false negative and denoted by $$\beta$$

We can use an example to vizualize this. Imagine a company is attempting to improve conversion rate by making a change to its website, however it wants to test the change and make sure it truly has an effect. The population who sees the original website converts (makes a purchase upon visiting the site) at a rate of 10%. We don't yet know what the new site converts at, but we suspect it converts at a higher rate. The population has a standard deviation of 15% and our sample size is 25. Let's explicitly state our hypotheses:

+ $$H_0:$$ The conversion rate of the control variant (original website) is less than or equal to the conversion rate of the treatment variant.
+ $$H_A:$$ The conversion rate of the treatment variant is greater than the conversion rate of the control variant.

In order to perform a hypothesis test, we need to set our value for $$\alpha$$. In this case, we'll use $$\alpha = 0.05$$. By setting $$\alpha$$ to 0.05 we are saying that 5% of the time we expect to have a Type I Error. In other words, if we repeat this test over and over again, using random samples each time, a true null hypothesis will be rejected once every twenty times this test is performed.

We'll also use alpha to calculate our critical value. Our critical value tells us the value beyond which we can reject the null hypothesis. In this case: 

$$z_{crit} = \mu_1 + z_{\alpha}*\frac{\sigma}{\sqrt n} = 14.93$$

Now imagine we take a sample from our treatment population and see that it converts at a rate of 20%. This is beyond our critical value, so we can reject the null hypothesis at an alpha of 0.05. 

The figure below shows the two sampling distributions and critical value. The region shaded red represents Type I Error. We can clearly see that we can reject our null as $$\mu_2 > z_{crit}$$. We look again at $$z_{crit}$$ to determine Type II Error. The area under the green curve and to the left of $$z_{crit}$$ shaded in yellow represents Type II Error. __Statistical power__ is simply 1 - Type II Error. In other words: the probability of detecting an effect __given that there is an effect to detect.__ In the figure below: power is the area under the green curve to the right of the cutoff line.

<img src="/img/powerviz.png" alt="power" width="450"/>


## What's a Power Analysis and When Should I Do One?

So now that we have a handle on what statistical power is, let's take a deeper dive into why a power analysis is useful.

Power analyses come in several flavors and are applied under different circumstances:

* __A Priori__
  * Performed before the experiment. It determines what sample size is required to detect a given effect size.
* __Sensitivity__
  * Performed when there are constraints on the experiment, for example if the sample size is limited due to budget, a sensitivity power analysis can be performed. In this case you can determine what effect size is possible with the sample size that is available.
* __Post Hoc__
  * Not particularly useful, your p-value will give you the same information, and if you want to know why that is true, there is an excellent in-depth post [here](http://daniellakens.blogspot.com/2014/12/observed-power-and-what-to-do-if-your.html)

## The Four Key Values of A Power Analysis

Effect size, significance level, sample size and statistical power are interrelated such that if we know three of the values we can solve to find the fourth. In this tutorial, we'll walk through an A Priori power analysis, in which we know effect size, significance level and power and solve for sample size. First, let's make sure we understand each of our four values. 

* __Effect Size__
  * We've referred to effect size a number of times, however have not explicitly defined it. Effect size is a fairly intuitive concept in that it measures the __magnitude__ of a particular phenomenon. Familiar examples of effect size are regression coefficients or Cohen's d.

* __Significance Level ($$\alpha$$)__
  * This is the significance level used in the hypothesis test. As discussed above, it is also referred to as Type I Error. The significance level is specified prior to the experiment and represents the less extreme boundary for determining whether the test was significant. 
  * In terms of the website conversion example, if we set the significance level to 0.05 and then perform a hypothesis test which results in a p-value of 0.02, we would reject our null hypothesis and be 95% confident that our effects were not due to chance.
* __Sample Size__
  * Simply how large our sample is. In relation to statistical power, as $$n$$ increases, so does statistical power. We can go back to our equation for $$z_{crit}$$ to see why this is true.

  $$z_{crit} = \mu_1 + z_{\alpha}*\frac{\sigma}{\sqrt n}$$

  * Recall that everything to the left of $$z_{crit}$$ and under the pdf of the alternative hypothesis is a Type II Error and everything to the right is statistical power. If all else is held equal, but sample size is increased, we can see from the above equation that our critical value will decrease. This will cause Type II Error to decrease and statistical power to increase. 

* __Statistical Power__
  * See discussion above

## A Priori Power Analysis Calculation





Now that we have some background on the key concepts that go into a power analysis and how they work together, let's walk through an example of an a priori analysis. Again, for this type of power analysis, we will determine a sample size for a given effect size. We will continue to use our website example, but keep in mind that for a true a priori calculation, the effect size would be based on prior research rather than a true sample as the calculation is being done prior to the experiment.  

+ There are __two equations__ we need in order to calculate sample size from power. We've already seen the first:

$$z_{crit} = \mu_1 + z_{1- \alpha}*\frac{\sigma}{\sqrt n}$$
  
+ However there is one other equation which must be true in order for our conditions to be met. Here we use the z-score calculated from $\beta$, which is equal to 1 - statistical power.

$$z_{crit} = \mu_2 + z_{\beta}*\frac{\sigma}{\sqrt n}$$

+ We now have two equations and two unknowns ($$z_{crit}, n$$) so we can easily solve for n!
  + In this case we'll set our desired statistical power to 0.9. This implies that $$\beta$$ is equal to 1 - 0.9 = 0.1. From the standard normal distribution we get $$z_{\beta} = -1.28$$

$$10 + 1.645*(\frac{15}{\sqrt n}) = 20 - 1.28 * (\frac{15}{\sqrt n})$$

$$10 = (1.645 + 1.28)\frac{15}{\sqrt n}$$

$$10 = \frac{43.875}{\sqrt n}$$

$$\sqrt n = 4.3875$$

$$n = 19.25 \approx 20$$
