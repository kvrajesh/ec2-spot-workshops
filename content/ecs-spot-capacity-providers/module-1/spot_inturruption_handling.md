---
title: "EC2 Spot Interruption Handling in ECS"
weight: 80
---

Amazon EC2 Service interrupts your Spot Instance when it needs the capacity back. It provides a Spot Instance interruption notice, 2 minutes before the instance gets terminated.

The EC2 spot interruption notification will be available in two ways:

1. ***Amazon EventBridge Events:*** EC2 service emits an event two minutes prior to the actual interruption. This event can be detected by Amazon CloudWatch Events.

1. ***EC2 Instance Metadata service (IMDS):*** If your Spot Instance is marked for termination by EC2, the instance-action item is present in your instance metadata.

In the Launch Template configuration, we added:
```plaintext
echo "ECS_ENABLE_SPOT_INSTANCE_DRAINING=true" >> /etc/ecs/ecs.config
```
When Amazon ECS Spot Instance draining is enabled on the instance, ECS container agent receives the Spot Instance interruption notice and places the instance in DRAINING status. When a container instance set to DRAINING, Amazon ECS prevents new tasks from being scheduled for placement on the container instance [Click here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/container-instance-spot.html) to learn more.

The web application (app.py) we used to buld docker image shows two ways to handle the EC2 Spot interruption within a docker container. This allows you to perform actions such as preventing the processing of new work, checkpointing the progress of a batch job, or gracefully exiting the application to complete tasks such as ensuring database connections are properly closed

In the first method, it polls the instance metadata service for spot interruption and display a message to web page notifying the users (this is, of course, just for demonstration and not for real world scenarios).

{{% notice warning %}}
In a production environment, you should not provide access from the ECS tasks to the IMDS. This is done in this workshop for simplification purposes.
{{% /notice %}}


```plaintext
URL = "http://169.254.169.254/latest/meta-data/spot/termination-time"
SpotInt = requests.get(URL)
if SpotInt.status_code == 200:
    response += "<h1>This Spot Instance will be terminated at: {} </h1> <hr/>".format(SpotInt.text)
```

In the second method, the application listens to the **SIGTERM** signal. The ECS container agent calls the StopTask API to stop all the tasks running on the Spot Instance.

When StopTask is called on a task, the equivalent of docker stop issued to the containers running in the task. This results in a **SIGTERM** value, and a default 30-second timeout, after which the SIGKILL value is sent, and the containers forcibly stopped.  If the container handles the **SIGTERM** value gracefully and exits within 30 seconds from receiving it, no SIGKILL value sent. Note that we have configured this time timeout value to 120 sec in the EC2 Launch template to allow the applications to take advantage of full 2 min if required, before the EC2 spot instance termination.

```python
class Ec2SpotInterruptionHandler:
  signals = {
    signal.SIGINT: 'SIGINT',
    signal.SIGTERM: 'SIGTERM'
  }

def __init__(self):
   signal.signal(signal.SIGINT, self.exit_gracefully)
   signal.signal(signal.SIGTERM, self.exit_gracefully)

def exit_gracefully(self, signum, frame):
   print("\nReceived {} signal".format(self.signals[signum]))
   if self.signals[signum] == 'SIGTERM':
     print("SIGTERM Signal Received due to EC2 Spot Interruption. Let's wrap up the work within 2 mins..")
```

***Congratulations !!!*** you have successfully completed this workshop. You may continue to **optional** section on how to save costs using ***Fargate Spot*** Capacity Providers.

