# 0x4447 VPN

This folder contains a CloudFormation file that when deployed it will create a L2TP IPSec and a CISCO IPSec VPN server in your selected AWS account.

# How to deploy

Click on this button.

# Manual work

When you deploy this stack not everything is going be done for you. One missing aspect is the Elastic IP, it can't be create by the CF file itself. You'll have to request a fixed IP and take note of the ID, which you'll use while deploying the CloudFormation file. In the following section there is a more details explanation why an Elastic IP is important and how it is used.

# What will be deployed

![0x4447 VPN Stack](https://raw.githubusercontent.com/0x4447/0x4447-products-vpn-contained/assets/stack.png)

The picture above shows the Stack that will be deployed and bellow you can see a detailed list of all the resources that will be created for you.

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

# How dose it work

The VPN servers is created using the [hwdsl2/ipsec-vpn-server](https://github.com/hwdsl2/docker-ipsec-vpn-server) project, a Docker container that contains all the services to create a VPN solution. The Docker container will be deployed using ECS on to a EC2 instance created thanks to the Launch Configuration and the Auto Scaling Group. The benefit of this solution are multi folds:

- If the docker Container crashes at some point, it will be automatically restarted by ECS.
- If the EC2 server for whatever reason gets terminated, it will be recreated automatically.

Another benefit of this is that you can on purpose terminate the EC2 Instance if you realize that the network performance is deteriorating. This can happen since you are on a shared environment and if another instance is running on the same machine as yours and takes more resources the average, you might get a worse performance. If this happens, terminate the instance, wait around 2 min, and hopefully the new EC2 Instance will be created on a different machine with less load.

This way you get a very resilient solution that is hard to brake, and since all the unique settings are stored in the ENVIRONMENT variables, your credentials always stay the same. Same goes for your public IP...

# Elastic IP magic

Another important aspect of this Stack is that it requires an Elastic IP to work. This is important to preserve continuity. The EC2 instance will automatically attach the provided Elastic IP at boot time, meaning even if the EC2 has to be recreated because of an issue, it will always have the same IP, meaning whoever is going to use your VPN will see only a few minutes of down time, and then everything will be back to where it was. Non need to update the DNS settings with a new public IP.

As you can see, once this stack is deployed it will do everything possible to have all the moving parts always available.

# Best practice

The best way to take advantage of this Stack is to have a unique AWS Account just for it. This should be done using the AWS Organization feature. The benefit to do so are:

- One free Elastic IP per AWS account.
- Tree CloudWatch Dashboard for free per account.
- Easier to limit access to this AWS VPN account.
- More precise access to resources thanks to VPC Peering.

# Pricing

This stack will generate expenses only by two resources (if deployed in a separated AWS Account):

- EC2
- Network traffic

Additional charges my apply if not deployed in a unique AWS Account:

- Elastic IP
- CloudWatch Dashboard

## CPU Load

Since we are using a t3.nano type server, the monthly price should be around $0.0052/h * 24/h * 30/d = $3.744 a month. Check the [AWS pricing page](https://aws.amazon.com/ec2/pricing/on-demand/) to make sure this calculation is still valid. Also, since we are using a T3 Instance type you might be charged for Burst mode if the CPU time will go above the average. In general the VPN generates very little CPU load, but since we are running a complex operating system, if any process gets stuck in an endless loop or, the server will be some how compromised – the CPU load could go way up.

## Network

The network traffic price it depends on how much you use it. But this is an example to give an idea of what to expect. If you were to transfer 100GB of data from the EC2 instance to the Internet, then you'd pay 100GB * $0.09/GB = $9.

# Monitoring

To help you get an clear insight in your resources we build out a detailed dashboard that will show exactly what is happening in your EC2 Instance and Container.

## Dashboard

![0x4447 VPN Stack Dashboard](https://raw.githubusercontent.com/0x4447/0x4447-products-vpn-contained/assets/dashboard.png)

The deployed stack will have a detailed CloudWatch Dashboard where you'll have a detail insight in what is happening with your VPN. you'll have a widget displaying Network traffic, CPU time from EC2 instance and ECS, plus a bunch more.

## Alerts

To make it easier for you to sleep well, we have two CloudWatch Alert which makes sure that if the CPU or RAM of the EC2 instance goes above a certain threshold you'll receive an email from AWS telling you about this fact, so you can then take appropriate actions.

## Logs

We also send logs to CloudWatch to make it easy to see exactly what is going on inside the EC2 Instance without need to log in to the machine itself. We exposed the following logs:

- dmesg
- docker
- ecs-agent.log
- messages

# How to work with this project

When you want to deploy the stack you are interested only in the `CloudFormation.json` file. If you'd like to modify the stack we recommend you use the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes). This tool was designed to make it easier to work with Cloud Formation file. You should never edit the main CF file if you want to keep your sanity 🤪.

# The End

If you enjoyed this project, please consider giving it a 🌟. And check out our [0x4447 GitHub account](https://github.com/0x4447), where we have additional resources that you might find useful or interesting.

## Sponsor

This project is brought to you by 0x4447 LLC, a software company specializing in build custom solutions on top of AWS. Find out more by following this link: https://0x4447.com or, send an email at [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).



