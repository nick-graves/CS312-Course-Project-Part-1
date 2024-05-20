# CS312-Course-Project-Part-1

## Table of Contents
* Setting up your EC2 instance
* Automating Minecraft server set up with a custom script

## Setting up your EC2 instance
1. Log into the AQS console UI
2. In the top right select the correct region. This region will correspond to your player's location. In my case that is Oregon (us-west-2)
3. Use the search bar to navigate to the "EC2 Dashboard" and click launch instance
4. Configure your instance with the following settings
  * Name: Minecraft Server West
  * Application OS Images: Amazon Linux 2023 AMI
  * Architecture: 64-bit(Arm)
  * Instance Type: t4g.small
  * Key pair name: create a new key pair (and save this file locally)
5. Scroll down to the network settings and select edit in the top right corner. This will create a security group for your EC2 instance. Configure with the following settings
  * VPC: default VPC
  * Subnet: No preference
  * Auto-assign public IP: Enable
6. Under the firewall settings configure with the following settings
  * Firewall: Create security group
  * Security Group Name: Minecraft Security Group
  * Description: Security Group with ports for Minecraft server and SSH
7. Scroll down to the inbound security group rules. You will need two inbound rules
  * Under the default SSH traffic rule change the source type to Custom and source to 0.0.0.0/0.
  * Add another inbound rule on your security group to allow TCP traffic from anywhere (0.0.0.0/0) for port 25565. This will allow Minecraft players to join your server.
8. Scroll


nano setup-minecraft.sh
chmod +x setup-minecraft.sh
