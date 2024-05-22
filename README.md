# CS312 Course Project Part 1

## Table of Contents
- [Setting up your EC2 instance](#setting-up-your-ec2-instance)
- [Automating Minecraft server setup with a custom script](#automating-minecraft-server-setup-with-a-custom-script)
- [Connecting to the server](#connecting-to-the-server)

## Setting up your EC2 instance
1. **Log into the AWS console UI**.
2. **Select the correct region**:
   - This region should correspond to your player's location. (e.g., Oregon `us-west-2`).
3. **Navigate to the "EC2 Dashboard"**:
   - Use the search bar to find "EC2 Dashboard" and click `Launch instance`.
4. **Configure your instance**:
   - **Name:** Minecraft Server West
   - **Application OS Images:** Amazon Linux 2023 AMI
   - **Architecture:** 64-bit(Arm)
   - **Instance Type:** t4g.small
   - **Key pair name:** Create a new key pair and save this file locally.
5. **Edit the network settings**:
   - **VPC:** Default VPC
   - **Subnet:** No preference
   - **Auto-assign public IP:** Enable
6. **Configure the firewall settings**:
   - **Firewall:** Create security group
   - **Security Group Name:** Minecraft Security Group
   - **Description:** Security Group with ports for Minecraft server and SSH
7. **Configure inbound security group rules**:
   - Modify the default SSH traffic rule: Change the source type to `Custom` and source to `0.0.0.0/0`.
   - Add another inbound rule: Allow `TCP traffic` from anywhere (`0.0.0.0/0`) for `port 25565`. This will allow Minecraft players to join your server.
8. **Storage settings**:
   - Set the root volume EBS to `8GB`.

## Automating Minecraft server setup with a custom script
**Note:** The provided script will automatically sign the EULA for Minecraft.

1. **SSH into your EC2 instance**:
   - Ensure the instance is running.
   - Click on the instance and note the public IPv4 address.
   - Open a terminal on your local computer, navigate to the directory where you stored your key file (from step 4), and run:
     ```sh
     ssh -i your-key.pem ec2-user@<public_ipv4_address>
     ```
2. **Create the setup script**:
   - Run the following command to create a bash script:
     ```sh
     nano setup-minecraft.sh
     ```
   - Copy and paste the following script into the `nano` editor. **Ensure** to set the `MINECRAFTSERVERURL` to the correct download link (e.g., [Version 1.20.1](https://mcversions.net/download/1.20.1)):
     ```bash
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
3. **Save and exit**:
   - After finishing editing the script, save and exit with `Ctrl+O` and `Ctrl+X`.
   - Grant execute permissions to the script:
     ```sh
     chmod +x setup-minecraft.sh
     ```
4. **Run the script**:
   ```sh
   bash setup-minecraft.sh
   ```
## Connecting to the server

1. **Identify the Minecraft version**:
   - If you used the provided link in [Automating Minecraft server setup with a custom script](#automating-minecraft-server-setup-with-a-custom-script), your version will be `1.20.1`. Start with this version of Minecraft.

2. **Connect to the server**:
   - Click the `Multiplayer` button, then `Add Server`.
   - Input your server IP as `<your_public_ipv4>:25565`.

3. **Join the server**:
   - Click save and join the server.
