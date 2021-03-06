---
title: "Add Fargate Capacity Providers to ECS Cluster"
weight: 5
---

Before we deploy tasks on ECS Fargate, let us first add Fargate capacity providers to the ECS Cluster. 
There are two default ECS Fargate capacity providers i.e. FARGATE and FARGATE_SPOT. You just need to add them to the ECS Cluster.

Run the below command to add Fargate capacity providers to the ECS Cluster.

```
aws ecs put-cluster-capacity-providers   \
        --cluster EcsSpotWorkshop \
        --capacity-providers FARGATE FARGATE_SPOT CP-OD CP-SPOT  \
        --default-capacity-provider-strategy capacityProvider=FARGATE,weight=1   capacityProvider=FARGATE_SPOT,weight=1 \
        --region $AWS_REGION
```
Note that when you update an ECS Cluster with new capacity providers, ensure that all the existing capacity providers are included.

Note that above ECS cluster create command also specifies a default capacity provider strategy.

The strategy sets a weight of 1 both FARGATE and FARGATE_SPOT as the default capacity provider. That means for equal distribution of tasks on FARGATE and FARGATE_SPOT.

The ECS Cluster should now contain 4 capacity providers i.e. CP-OD, CP-SPOT, FARGATE and FARGATE_SPOT

![Fargate Capacity Providers](/images/ecs-spot-capacity-providers/ecs_fargate_cps.png)
