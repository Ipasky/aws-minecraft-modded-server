# aws_minecraft_forge_server
AWS EC2 Minecraft Forge server

## Introduction
This repository is a guide for everyone who wants to set up their private Minecraft Forge server. We are going to use AWS EC2 to host the server.

![](https://github.com/Ipasky/home-hpc)

## [Slurm](https://www.elastic.co)

> [!WARNING]
> Warning test

> [!NOTE]
> Note test

In the node master you must do the next configuration to have plenty acces to the other nodes
```shell
ssh-keygen -t rsa
ssh-copy-id root@node01
ssh-copy-id root@node02
```

```shell
sudo apt update
sudo apt install -y slurm-wlm munge
sudo systemctl enable munge
sudo systemctl start munge
sudo systemctl status munge
```
We need to do the same for each node, in my case since i have Alma Linux in the other nodes i need to do:
```shell
sudo dnf update
sudo apt install -y slurm-wlm munge
sudo systemctl enable munge
sudo systemctl start munge
sudo systemctl status munge
```
![](https://github.com/Ipasky/home-hpc/img/img_01.jpg)

## AWS EC2 Tutorial
### Game Version and Mod List
The first step is choosing the game version and all the mods you want to use in-game. In our case, we are going to play with **Forge 1.20.1-47.2.20** because it's a stable version and there is a large variety of mods available.

Here you can see all the mods we are using on our server:  
![](https://github.com/Ipasky/aws)

Once you have chosen the version and all the mods, you need to decide your **budget** and how many players will join. This is important because AWS offers various types of virtual machines, and you need to choose one that fits your requirements.  
If you're planning to play with 200 mods and multiple players, you should pick a more powerful instance.  
We’ll discuss and compare the different options in the next section.

AWS offers an incredible number of services. For this project, we are going to use **EC2 instances** to host our Minecraft server.  
EC2 has a whole ecosystem around it, starting with the dashboard:  
![alt text](image.png)

The resources we’re going to use are:  
- **Instances**  
- **Volumes**  
- **Key Pairs**  
- **Security Groups**

> [!NOTE]
> I'm not an expert in AWS ecosystem, if i did wrong one step or i have an incorrect configuration please contact with me (ipasauriz@gmail.com) and I will correct the mistake. Thank you.


### On-Demand vs Spot Instance
First of all, let’s discuss which purchase model you should choose for your instance.

You have to decide whether you want to use an **On-Demand** or a **Spot Instance**.  
The difference between them is the following:

- **On-Demand Instances** are for applications that **cannot be interrupted**. You can run the server 24/7 without any problems. Additionally, On-Demand instances allow you to **start and stop the instance whenever you want**, so you have **full control** over your server and usage.

- **Spot Instances**, on the other hand, let you **take advantage of unused EC2 capacity** at a much lower price. However, AWS can **shut down your instance at any time** if that capacity is needed elsewhere — you'll receive a 2-minute warning before it's terminated.  
  The downside is that **you can’t start or stop Spot Instances manually** unless you set up complex automation. They're more suited for services that run continuously, like a 24h/day server without manual control.

In our case, we decided to go with **On-Demand Instances**.  
Even though Spot Instances are significantly cheaper (for example, for a `t3.2xlarge` instance we would pay **$0.3328/hour On-Demand** vs **$0.0966/hour Spot**:  
![alt text](image-1.png) and ![alt text](image-2.png)), I prefer having **full control over the server's active hours**.  

We don’t need the server running all day, especially if no one is playing — that would just waste money on unused time.

If you prefer to have the server available 24/7 and want to save money, **Spot Instances** might be a better fit.  
It really depends on your group’s **play style, schedule, and budget**.


### Choosing Instance type
Second you need to decide what instance type it fits more with your server. In my experience for a server about a 100 mods with 2-4 people playing simultaneously a t3.xlarge is enough. Also for a small servers t3.large or t3.medium will be perfect for the prupose (and more cheap). One advantage of this services is, if you try one instance type and isnt enought powerfull, you only pay for the hours of testing. It will be easy to set-up another instance with more capabilities.

In our case we are going to use a 8 CPU and 32GiB ram spot instance, that because in previous servers we tested the t3.xlarge with 4 CPU and 16GiB ram, and we see that it wasn't enought power to run all the mods and the players. This time we doubled the specs because we have more than 100 mods and about 5-10 players.

### Spot Instance set-up
In the EC2 panel you should go to your left vertical panel in the section Instances and go Spot Requests ![alt text](image-3.png). Then click on Create Spot Fleet Request and select the Manually configure launch parameters option. ![alt text](image-5.png)
Next we are going to select a Amazon Linux 2 AMI Kernel 5.10 version, the AMI are the template that it will be installed in our server, for our purpose a clean Linux version is enough. Additionaly we need to specify a Key pair name to save localy the key for stablish a remote SSH connection. Here you just click on create new key pair and put the name you want. In my case how i want to connect via MobaXterm i'm going to select the key in (.pem) format and download it localy. ![alt text](image-4.png)

Also in "Additional launch parameters" we are going to put in EBS Volumes 20GiB instead of the 8GiB that comes default. ![alt text](image-7.png) That will increase a bit the cost, aproximate 2$/month but its esencialy for set up a server. In your case if you are not using more than 20 mods and you are not going to explore a lot in your world go with the 8GiB option. But in our case the last server weigh about 10GiB (all the server, including mods, map exploration and other resources). I dont want to change this in the middle of the server.

Morover, here we should create a Security Group, thats acts like a firewall, we need to specify that all the IP's v4 going inbound to port 25565 (minecraft default port) are alowed to TCP and UDP. Additionally for use SSH remote, you need to put your public IP from the computer you are going to connect via SSH and allow the inbound connections.
![alt text](image-6.png)

Then you should put at least 3 instance types that are similar in capabilityes. In our case we putted "m7i.2xlarge, m6a.2xlarge, t3a.2xlarge, t3.2xlarge" that are all 8 CPU and 32GiB ram and have a similar price. ![alt text](image-8.png)

### Connect via SSH
With you request done you need to wait a few minutes to set-up the instance, then if you go to your instances you should see the new service running. ![alt text](image-9.png) Select the instance and click on Connect top bar button, here it will apear in the SSH client the sentence to connect via mobaxterm using the (.pem) file that previosly we download ![alt text](image-10.png) Copy the example sentence and go to MobaXterm or other SSH terminal in your local computer. Paste it and voila! you are into your machine. Make shure, when you paste the sentence, you are in the same directory as your (.pem) file or specify where is the file in your computer. ![alt text](image-11.png)

### Java Installation

FartLansBBK25K_MCServer

### Minecraft forge server installation