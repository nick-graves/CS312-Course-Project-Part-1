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
8. Scroll down to the storage section in EC2 set up and elect 8GB for root volume EBS.

## Automating Minecraft server set up with a custom script
NOTE: The provided script will automatically sign EULA with Minecraft. 

1. To start this step we are going to need to ssh into our ec2 instance. Double-check the instance is running. Click on the instance and scroll down to find the public IPv4 address. Remember this address.
2. Now open a terminal on your local computer and CD into the directory where you stored your key file locally (see step 4) and run the following command
``` ssh -i your-key.pem ec2-user@public IPv4 address```
3. Now we will create a script to install Minecraft and everything we need for the server. Run this command to create a bash script that we will use to install all of this ```nano setup-minecraft.sh```. This will open up nano editor to edit this file. Now copy and paste this in. Please note that the ```MINECRAFTSERVERURL=``` is left blank. You will have to find the latest version. This is a link to version 1.20.1: https://mcversions.net/download/1.20.1 copy the link to the jar download file and copy and paste it in for the ```MINECRAFTSERVERURL=```. 
```
#!/bin/bash

# *** INSERT SERVER DOWNLOAD URL BELOW ***
# Do not add any spaces between your link and the "=", otherwise it won't work. EG: MINECRAFTSERVERURL=https://urlexample


MINECRAFTSERVERURL=


# Download Java
sudo yum install -y java-17-amazon-corretto-headless
# Install MC Java server in a directory we create
adduser minecraft
mkdir /opt/minecraft/
mkdir /opt/minecraft/server/
cd /opt/minecraft/server

# Download server jar file from Minecraft official website
wget $MINECRAFTSERVERURL

# Generate Minecraft server files and create script
chown -R minecraft:minecraft /opt/minecraft/
java -Xmx1300M -Xms1300M -jar server.jar nogui
sleep 40
sed -i 's/false/true/p' eula.txt
touch start
printf '#!/bin/bash\njava -Xmx1300M -Xms1300M -jar server.jar nogui\n' >> start
chmod +x start
sleep 1
touch stop
printf '#!/bin/bash\nkill -9 $(ps -ef | pgrep -f "java")' >> stop
chmod +x stop
sleep 1

# Create SystemD Script to run Minecraft server jar on reboot
cd /etc/systemd/system/
touch minecraft.service
printf '[Unit]\nDescription=Minecraft Server on start up\nWants=network-online.target\n[Service]\nUser=minecraft\nWorkingDirectory=/opt/minecraft/server\nExecStart=/opt/minecraft/server/start\nStandardInput=null\n[Install]\nWantedBy=multi-user.target' >> minecraft.service
sudo systemctl daemon-reload
sudo systemctl enable minecraft.service
sudo systemctl start minecraft.service

# End script
```
4. AFter you finish editing the script Ctrl+O and Ctrl+X to save and exit. Then grand the file execture permissions with ```chmod +x setup-minecraft.sh```.
5. Finally it is time to run the script with ```bash setup-minecraft.sh```



