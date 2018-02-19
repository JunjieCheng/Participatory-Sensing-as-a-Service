# SLA optimization strategy

## Introduction

I tried couple examples and concluded some typical scenarios for SLA optimization. I hope to define the principle of SLA first, then it will be easy to implement.

In SLA, we need to consider success rate, time cost, incentive cost (money). Success rate and time cost will be based on the historical data. Particularly, they may change according to different location and device. Incentive cost is very complicated to calculate. It might be reverse acution. In this article, I only give an arbitrary price, and we can discuss later. 

To optimize the execution flow, we need to not only calculate the possibility distribution of the task, but also balance the success rate, time cost and incentive cost by assigning number tasks to participants. Before optimization, we need to define a thredhold for the success rate. Here I assume the threshold of the overall success rate is 80%. 

Then I will use some examples to explain.

## Scenarios

### Scenario 1: Linear Flow

![Scenario 1](https://github.com/JunjieCheng/SLA/blob/master/scenario1.png)

Linear flow is simply consective tasks without any fork. 

#### Success rate
Assume the theshold is 80%. For this task, if we only assign one person for each stage, the overall success rate will be only 75% * 50% = 37.5%, with time cost 3s and incentive cost $3. Therefore, we need to assign tasks to more people to increase the success rate. 

To satisfy the threshold, we must guarentee that the success rate of every stage must be higher than the threshold. Think about the example. If the threshold is 80%, then the greatest difference between success rate of two stages are 80% and 100%. Otherwise one will exceed 100%. However, the success rate of a single stage is decrease progresively. It is impossible to reach 100%. 

The formula used to decided the number needed is N = ceilling(ln(1 - T)/ln(1 - S)), where T is the threshold and S is the success rate. In this case, the stage 1 requries 3 people and stage 2 requires 2 people.

#### Cost 
If the overall success rate is still lower than the threshold after using the formula, then pick the one stage with the lowest cost to increase. I plan to use a list to compare the ratio of success rate and cost on every stage. Then iteratly add person util it reaches the threshold.

#### Time
In this scenario we don't need to consider time.

### Scenario 2: Alternative Parallel Flow

![Scenario 2](./scenario2.png)

Alternative parallel flow is a component that retrive data from multiply microservice. If one of these microservices succeed, all other parallel microservices will stop. 

#### Success rate
For this task, if we only assign one person for each microservice, the overall success rate will be 100 - (100 - 25) * (100 - 40)% = 55%.

#### Cost
Depends on the number of people to execute them. I will use a Knapsack algorithm to find the lowest cost. Iterate all parallel microservices and compute (increased success rate/increased cost) after asigning one more people for this microservice. Then select the lowest one to assign. Until the overall success rate is reached. This is not efficient but it won't cost too much time.

#### Time
The total time depends on the longest microservice.

### Scenario 3: Required Parallel Flow

![Scenario 3](./scenario3.jpg)

Required parallel flow is a component that require data from every parallel microservices. If any one of microservices fails, the whole task fails. 

#### Success rate 
It is smiliar to the scenario 1. We need to guarentee the success rate of every microservice is higher than the threshold. The overall success rate of this flow is 50% * 75% = 37.5%. We still use N = ceilling(ln(1 - T)/ln(1 - S)) to assign people first. Then approach the threshold by Knapsack.

#### Cost
Use Knapsack to optimize.

#### Time
The total time depends on the longest microservice.

### Possibility Distribution

![Get Tempreture](./getTemp.png)

At the end, I will give a possibility distribution. 

* 50%, 1s, $1
* 37.5%, 3s, $3
* 6.25%, 4s, $2
* 6.25%, fail

In this case, I assume that the failure of execution will cost time but not cost money.

The first possiblity is the success rate of the microservice nearby sensor. If it fails, we will start the parallel microservices. I will first calculate the possibity of the shortest microservice, because it ends first. If it fails, calculate the rest microservices with the fail rate. If there are some microservices has smiliar execution time, I will calculate them as one overall success rate.
