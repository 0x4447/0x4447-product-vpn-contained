# 0x4447 Contained VPN

This folder contains a CloudFormation file that, when deployed, will create L2TP, IPSec, and CISCO IPSec VPN servers in your selected AWS account. 

# DISCLAIMER!

This stack is available to anyone at no cost, but on an as-is basis. 0x4447 is not responsible for damage of any kind that occurs when this stack is used. You take full responsibility when you use it.

# Use cases

This stack is ideal for connecting your office to AWS resources and making those resources available to your local office network, as well as securing your internet connection.

# Before you deploy

Don't expect this stack to do everything for you. For example, the CF file itself can't create the Elastic IP part, you'll need to request a fixed IP and take note of the ID, which is used when deploying the CloudFormation file. The next section contains details on why an Elastic IP is important and how it's used.

# How to deploy

WORK IN PROGRESS.

# What deploys

![0x4447 VPN Stack](https://raw.githubusercontent.com/0x4447/0x4447-products-vpn-contained/assets/stack.png)

The image above shows the stack that deploys. You'll find a detailed list of all resources the stack creates for you below.

- 1x ECS
  - 1x Cluster
  - 1x Service
  - 1x Task
- 1x EC2
	- 1x Launch Configuration
	- 1x Auto Scaling Group
	- 1x Security Group
- 1x CloudWatch Dashboard
- 1x CloudWatch Logs
- 2x CloudWatch Alerts
  - EC2 CPU
  - EC2 RAM

# How it works

The VPN servers are created using the [hwdsl2/ipsec-vpn-server](https://github.com/hwdsl2/docker-ipsec-vpn-server) project, which is a Docker container with all the services needed to create a VPN solution. The Docker container is deployed using ECS onto an EC2 instance created thanks to the Launch Configuration and the Auto Scaling Group. This solution provides multifold benefits:

- If the Docker container crashes at some point, ECS automatically restarts it.
- If the EC2 server is terminated for whatever reason, a new instance takes its place.

Additional benefit: You can purposely terminate the EC2 Instance if you notice that network performance is deteriorating, which can happen when working in a shared environment. If your machine is running another instance that uses more resources than average, it may adversely affect performance. If this happens, terminate the instance and then wait around two minutes. Hopefully, the new EC2 Instance will be created on a different machine with less load.

This gives you a very resilient solution that's hard to break, and with all unique settings stored in the ENVIRONMENT variables, your credentials always stay the same. The same goes for your public Elastic IP.

# Elastic IP magic

Another important aspect of this stack is that it requires an Elastic IP to work. This is important for preserving continuity. The EC2 instance automatically attaches the provided Elastic IP at boot time, meaning even if the EC2 has to be recreated because there's an issue, it will always have the same IP. This means that anyone using your VPN will see only a few minutes of downtime, and then everything will be restored. There's no need to update the DNS settings with a new public IP.

As you can see, once this stack is deployed, it does everything possible to ensure that all moving parts are always available.

# Best practice

The best way to take advantage of these benefits is to have a unique AWS Account for this stack alone. This should be done via the AWS Organization feature, which offers the following advantages:

- One free Elastic IP per AWS account
- Free Tree CloudWatch Dashboard for each account
- Easily limited access to this AWS VPN account
- More precise access to resources through VPC Peering

# Pricing

This stack generates expenses via three resources (if deployed in a dedicated AWS Account):

- EC2
- Network traffic
- CloudWatch Alarms

Additional charges may apply if it isn't deployed via a unique AWS Account:

- Elastic IP
- CloudWatch Dashboard

## CPU Load

Since we're using a t3.nano type server, the monthly price should be around $0.0052/h * 24/h * 30/d = $3.744 a month. Check the [AWS pricing page](https://aws.amazon.com/ec2/pricing/on-demand/) to make sure this calculation is still valid. Also, since we are using a T3 type instance you might be charged for Burst mode if the CPU time will go above the average. In general the VPN generates very little CPU load, but since we are running a complex operating system, if any process gets stuck in an endless loop or, the server will be some how compromised – the CPU load could go way up - be aware of this.

## Network

The network traffic price it depends on how much you use it. But this is an example to give an idea of what to expect. If you were to transfer 100GB of data from the EC2 instance to the Internet, then you'd pay 100GB * $0.09/GB = $9. Check the [AWS pricing page](https://aws.amazon.com/ec2/pricing/on-demand/) to make sure this calculation is still valid.

## Alarms

Each alarm in your account will cost you $0.10 per alarm. Check the [AWS pricing page](https://aws.amazon.com/cloudwatch/pricing/) to make sure this calculation is still valid.

# Monitoring

To help you get a clear insight in your resources we built a detailed dashboard that will show graphically exactly what is happening in your EC2 Instance and Container. While also setting alarms to let you know when key aspect of the deployment are acting out, and the EC2 will also push logs to CloudWatch to give you a detailed view of the Instance insides without the need to SSH inside the machine.

## Dashboard

![0x4447 VPN Stack Dashboard](https://raw.githubusercontent.com/0x4447/0x4447-products-vpn-contained/assets/dashboard.png)

As you can see from the image above you get a clear view of what is going on with your resources. We recommend using the full screen feature which will update the widget as new data comes in.

## Alerts

To make it easier for you to sleep well, we have two CloudWatch Alert which makes sure that if the CPU or RAM of the EC2 instance goes above a certain threshold you'll receive an email from AWS telling you about this fact, so you can then take appropriate actions.

## Logs

We also send logs to CloudWatch to make it easy to see exactly what is going on inside the EC2 Instance without the need to log in to the machine itself. We exposed the following logs:

- dmesg
- docker
- ecs-agent.log
- messages

# How to work with this project

When you want to deploy the stack you are interested only in the `CloudFormation.json` file. If you'd like to modify the stack we recommend you use the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes). This tool was designed to make it easier to work with CloudFormation file. You should never edit the main CF file if you want to keep your sanity 🤪.

# The End

If you enjoyed this project, please consider giving it a 🌟. And check out our [0x4447 GitHub account](https://github.com/0x4447), where we have additional resources that you might find useful or interesting.

## Sponsor 🎊

This project is brought to you by 0x4447 LLC, a software company specializing in build custom solutions on top of AWS. Find out more by following this link: https://0x4447.com or, send an email at [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).
