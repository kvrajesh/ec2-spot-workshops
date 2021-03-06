+++
title = "ECS Cluster Autoscaling (CAS)"
weight = 60
+++

ECS Cluster Auto Scaling (CAS) is a new capability for ECS to manage the scaling of EC2 Auto Scaling Groups (ASG). CAS relies on ECS capacity providers.  

When creating a capacity provider, you can optionally enable managed scaling. When managed scaling is enabled, Amazon ECS manages the scale-in and scale-out actions of the Auto Scaling group. On your behalf, Amazon ECS creates an AWS Auto Scaling scaling plan with a target tracking scaling policy based on the target capacity value you specify. Amazon ECS then associates this scaling plan with your Auto Scaling group. For each of the capacity providers with managed scaling enabled, an Amazon ECS managed CloudWatch metric with the prefix AWS/ECS/ManagedScaling is created along with two CloudWatch alarms. The CloudWatch metrics and alarms are used to monitor the container instance capacity in your Auto Scaling groups and will trigger the Auto Scaling group to scale in and scale out as needed.

 The scaling policy uses a new CloudWatch metric called  CapacityProviderReservation that ECS publishes for every ASG capacity provider that has managed scaling enabled.

The new metric CloudWatch metric CapacityProviderReservation is defined as follows.

CapacityProviderReservation  = (M/N ) x 100

N = The current number of instances in the Autoscaling group(ASG) that are already running 
M = The number of instances are running in an ASG to meet the needs of the tasks assigned to that ASG, including tasks already running as well as tasks the customer is trying to run that don’t fit on the existing instances. 

Given this assumption, if N = M, scaling out is not required, and scaling in isn’t possible. On the other hand, if N < M, scale out is required because you don’t have enough instances.  Lastly, if N > M, scale in is possible (but not necessarily required) because you have more instances than you need to run all of your ECS tasks.

The CapacityProviderReservation metric is a relative proportion of Target capacity value and dictates how much scale-out / scale-in should happen.  CAS always tries to ensure CapacityProviderReservation is equal to specified Target capacity value either by increasing or decreasing number of instances in ASG.

In other words, the scale-out activity triggered if CapacityProviderReservation > Target capacity for 1 datapoints with 1 minute duration. That means it takes 1 min to scale out the capacity in teh ASG. And the scalee-in activity triggered if CapacityProviderReservation < Target capacity for 15 datapoints with 1 minute duration.  

All of this is well explained in this workshop.
