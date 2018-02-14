# SLA optimization strategy

## Introduction

I tried couple examples and concluded some typical scenarios for SLA optimization. I hope to define the principle of SLA first, then it will be easy to implement.

In SLA, we need to consider success rate, time cost, incentive cost (money). Success rate and time cost will be based on the historical data. Particularly, they may change according to different location and device. Incentive cost is very complicated to calculate. It might be reverse acution. In this article, I only give an arbitrary price, and we can discuss later. 

To optimize the execution flow, we need to not only calculate the possibility distribution of the task, but also balance the success rate, time cost and incentive cost by assigning number tasks to participants. Before optimization, we need to define a thredhold for the success rate. Here I assume the threshold of the overall success rate is 80%. 

Then I will use some examples to explain.

## Scenarios

### Scenario 1: Linear Flow

![Scenario 1](https://github.com/JunjieCheng/SLA/blob/master/scenario1.png)

Linear flow is simply consective tasks without any fork. For this task, if we only assign one person for each stage, the overall success rate will be 75% * 50% = 37.5%, with time cost 3s and incentive cost $3. To satisfy the threshold, we must guarentee that the success rate of every stage must be higher than the threshold. The formula used to decided the number needed is N = ln(1 - T)/ln(1 - S), where T is the threshold and S is the success rate. Then pick the one stage with the lowest cost to increase. 

Note that the success rate of a single stage is decrease progresively. It cannot reach 100%. Therefore, I plan to use a list to compare the ratio of success rate and cost on every stage. Then iteratly add person util it reaches the threshold.
