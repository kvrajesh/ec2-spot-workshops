---
title: "ECS Managed Scaling(CAS) in action"
weight: 60
---

Click the service name in the [ECS Console](https://console.aws.amazon.com/ecs/home?#/clusters/EcsSpotWorkshop/services/ec2-service-split/details) 

![Capacity Provider](/images/ecs-spot-capacity-providers/CP4.png) 

The pending task count is 10, which will change the Capacity Provider Reservation metric values as ECS calculates a new value for M (from zero initially) to accommodate these pending tasks.
 
 Let us look at the [CloudWatch Dashboard](https://console.aws.amazon.com/cloudwatch/home?#dashboards:name=EcsSpotWorkshop) 
 for the new Capacity Provider Reservations Metric values for both Capacity Providers.


![Capacity Provider Reservation](/images/ecs-spot-capacity-providers/cp5.png) 

The Capacity Provider Reservation metric value is 200. This indicates the new value of M is higher than N by a factor 2X, which indicates the scaling (out) factor. This should trigger the cloud watch alarms which in turn will cause the target tracking policy in the Autoscaling Group to provision more instances in response. 

Go to the CloudWatch console and click on the [Alarms section](https://console.aws.amazon.com/cloudwatch/home?#alarmsV2:!alarmStateFilter=ALARM).

![Cloud Watch Alarms](/images/ecs-spot-capacity-providers/ecs_service_alarms.png)

These alarms will cause the scale out action on both Autoscaling Groups. 

Go to EC2 console, select [OnDemand ASG](https://console.aws.amazon.com/ec2autoscaling/home?#/details/EcsSpotWorkshop-ASG-OD?view=activity) and click the Activity tab. You will see two instances are just getting launched.

![ASG Scale Out](/images/ecs-spot-capacity-providers/ecs_asg_od_scale_out.png)


Go to EC2 console, select [EC2 Spot ASG](https://console.aws.amazon.com/ec2autoscaling/home?#/details/EcsSpotWorkshop-ASG-SPOT?view=activity) and click the Activity tab. You will see two instances are just getting launched.

![ASG Scale Out](/images/ecs-spot-capacity-providers/ecs_asg_spot_scale_out.png)


So you see that Capacity Providers Managed Scaling did its job of responding to the application's intent, and scaled out by launching required capacity in the underlying Autoscaling Group. Move to the next step in the workshop to examine how the tasks distributed across the On-Demand and Spot capacity providers. 