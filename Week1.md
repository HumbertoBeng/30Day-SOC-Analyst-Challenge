# Week 1 of 30 Day SOC Analyst Challenge. Created by MyDFIR

## Day 1 - Logical Diagram

We created a diagram to show the workflow of the project and the resources we are going to be using over the course of this project. The diagram was created using <a href="https://app.diagrams.net/">Draw.io</a>.

![Day1-LogicalDiagram-Revised(2)](https://github.com/user-attachments/assets/c9e1e77a-85ee-40dc-81b9-4bf16fe40b9d)

For this project we are going to be using:
  - 6 Servers
    * Elastic & Kibana (Ubuntu Server 24.02)
    * Fleet Server (Ubuntu Server 22.02)
    * Windows Server (Windows Server 2022)
    * Ubuntu Server (Ubuntu Server 24.02)
    * OS Ticket Server
    * C2 Server Mythic
  - 2 Virtual Machines
    * Atacker Machine (Kali Linux)
    * SOC Analyst Machine (Windows 10)

All machines will be hosted using a cloud service provider called Microsoft Azure.
For our Virtual Private Network (VPC) we are going to assign the private IP of 172.31.0.0/24 with a range of 172.31.0.1 - 254 with a subnet of 255.255.255.0.

## Day 2 - ELK Stack Introduction

_Elasticsearch_, _Logstash_ and _Kibana_ also known as ELK Stack is a group of technologies used for managing and analyzing large volumes of data. It allows users to collect, process, and visualize data in rea-time, making it a powerful solution for log management and analytics.

### Elasticsearch, Logstash and Kibana

**Elasticsearch** is a database where different types of logs are stored.

**Logstash** is a pipeline that collects telemetry across various sources. There are different ways to collect telemetry, but the most popular are *Beats* and *Elastic Agents*.
  - Beats
    * File Beat collects Logs.
    * Metric Beat collects metrics.
    * Packet beat collects Network Data.
    * Winlog Beat collects Windows Events.
    * Audit Beat collects Audit Data.
    * Heartbeat Beat cocllects Uptime.
  
  - Elastic Agent
    * Collects all kinds of data with a single agent.

**Kibana** is a data visualization and exploration tool used for log and time-series analytics, application monitoring, and operational intelligence use cases. It offers powerful and easy-to-use features such as histograms, line graphs, pie charts, heat maps, and built-in geospatial support.

### Benefits of using ELK Stack

- Centralized Logging
  * Meet Compliance requirements and search data
- Flexibility
  * Customized Ingestion
- Visualizations
  * Observe information at-a-glance
- Scalability
  * Easy to configure to handle larger environments
- Ecosystem
  * Many integrations and rich community


## Day 3.- Setting up Elasticsearch

### 3.1.-Setting up a Virtual Network in Microsoft Azure

First we are going to look for the Icon for "Virtual Network" as show in the image.

![imagen](https://github.com/user-attachments/assets/abf8f42a-0c6a-4719-bf43-d9fb97ad0583)

Once inside we'll need to click on the button "Create Virtual Network". Now we need to choose a subscription, in my case is the "Azure Subscription 1" since I'm using the free tier from Azure, then select a Resource group, the Resource group is going to be the group where all the machines we are going to use and that we will be using for all the other stuff related to the project. So to create a new Resource group just click on "Create new" and then select a name, in this case I'll be using the name of this Challenge.

![imagen](https://github.com/user-attachments/assets/01403ddc-c563-44d1-90b0-3371d888dd14)

Next we are going to give a name to our virtual Network and then select the Region that is closest to us as to not run into any latency problems.

![imagen](https://github.com/user-attachments/assets/956a9616-8505-40bc-b64a-3de5d8f454c0)

We will not be setting "Security" for now. So in the "IP address" tab we are going to be setting the range of IPs we will be using for our virtual machines. Looking at the Diagram we created early we said we were going to be using the range of IPs of "172.31.0.0/24".

![imagen](https://github.com/user-attachments/assets/5da99aef-a8ed-4fef-a9b7-4ba46ac6cec1)

Finally we click on the "Review + create" button to create our Virtual Network.


### 3.2.-Creating our Virtual Machines

First we are going to look for the Icon for "Virtual Machines" as show in the image.

![imagen](https://github.com/user-attachments/assets/bab2b917-7a30-462a-95a1-759e691892e9)

Once inside we are going to click on the "Create" button and select the first option of "Azure Virtual Machine"

![imagen](https://github.com/user-attachments/assets/49e6b5eb-a8dd-46d0-9b0b-9c92133cc3dc)

The Suscription and Resource group field should be the same as the one from the Virtual Network.

![imagen](https://github.com/user-attachments/assets/a00cfe1a-7568-447a-bb1b-6791caae165f)

So now we are going to give the Virtual Machine a name, since the first VM is going to be the Ubuntu Server 22.04, I'll name it "CSBarista-ELK". NOTE: The region of the machine should match the Region we selected for our Virtual Network. 
The next field we are going to leave it as default. Then in the "Authentication Type" we are going to be choosing "Password" and we need to set a secure password.

Once finished we can go ahead and click "Next" we will take us to the "Disks" tab. Here we are only going to change the disk size for the machine to 64gb. Then click "Next" again which will take us to the "Networking" tab, here if the Virtual Network should be already selected by default.

![imagen](https://github.com/user-attachments/assets/cf2c4c08-5f65-41de-9a80-79532d52ae4b)

As for the configuration for this Virtual Machine that should be all, so to finish creating the VM we can go ahead and click on "Review + create".
The IP for the CSBarista-ELK machine is 172.31.0.4.

To verify if our server is running we can connect to it using SSH. I'll be using my own machine to connecto through SSH using the command `ssh [User]@[public ip of the machine]` then pressing Enter, the User is the one we created earlier in the configuration of the Virtual Machine and to find the public IP of the machine we can go to Overview and it should be in the right side of the screen under the specification of the VM. Once we connect to our VM we can go ahead and use the command of `apt-get update && apt-get upgrade -y`.

Finally we are going to need to put up some security mesures to our VM. To do that we need to go to the "Networking" setting within our VM and select "Network Settings. Once inside we need to scroll down a bit and in the "Rules" section we are going to click the rule already created called "SSH" and click it. 

![imagen](https://github.com/user-attachments/assets/6135cc6f-bad7-4f5c-a382-197916bbf167)

We are going to change the field of "Source" and change it to "My IP address" so that nobody other than ourselves can have access to our VM.

### 3.3.- Installing Elasticsearch

To install Elasticsearch we can search for "Elasticsearch" and it the first result should be the oficial page "Elastic.io" and there should be an option to download which will take us to that section of the oficial site.

![imagen](https://github.com/user-attachments/assets/07e39ef8-e177-42e5-9a5c-3a36d264bf83)

Once inside the Download page we are going to choose the platform as "deb x86_64", then we are going to hover over the Download button and we are going to right click it and select "Copy link address" so that we can paste the link in CMD so we can download it in our VM, and will look something like this `wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.0.0-amd64.deb`

![imagen](https://github.com/user-attachments/assets/3f9d9ee8-9bb9-4455-ac12-556110a95ff8)

To make sure it was installed correctly we can input the command `ls` and verify that the files are there. 

![imagen](https://github.com/user-attachments/assets/1e594a66-85ed-4060-9f39-2c8d6dcbb1d9)

Now to install Elasticsearch in our VM we are going to input the next command `dpkg -i elasticsearch-9.0.0-amd64.deb` and press Enter.

![imagen](https://github.com/user-attachments/assets/b259abfc-8d84-407d-b9b9-31782b233f15)

After the installation is done there is one thing that we need to save within the description of the installation and is about the "Security aotuconfiguration information" which contains the password of the elastic built-in superuser and we will need it in the future so we will want to save it. To be safe we should copy the whole messace within the "Security aotuconfiguration information" section.

![imagen](https://github.com/user-attachments/assets/20f1979f-0d0b-40c0-8765-04c689fbc354)

NOTE: In case you forget to copy the password for the superuser within the "Security aotuconfiguration information" we just copy there is a command that will allow us to reset the password for the user created of "elastic", with that we will be able to get another password for the user.

Now to configure Elasticsearch we need to go to the elasticsearch folder which contains the configuration file. This file is located in /etc/elasticsearch and the file is called "elasticsearch.yml". To edit the configuration we are going to be using the `nano elasticsearch.yml` command, but before that we are going to need the ip of the machine which can be obtained using `ip a`, we are going to need the IP we used to access the machine.

![imagen](https://github.com/user-attachments/assets/b5802193-5b9d-4964-bbf8-ac5cf1e5a663)

![imagen](https://github.com/user-attachments/assets/7fa17732-08e3-495e-966e-1f5bc774bd87)

Once inside the configuration file we are going to look for the section of "Network" and locate the "network.host" which will be the line we are going to edit. First to make it so that the changes take effect we will need to remove the "#" symbol which makes it so that the line is skipped when executing said file. Then we are going to change the default ip address with the one we copyed earlier. We will also remove the comment from the http.port line. In the end should look something like this.

![imagen](https://github.com/user-attachments/assets/63450b2a-301f-418b-a6b8-81bf7afedaf0)

After changing the lines we can save it pressing CTRL + X and then pressing y and finally pressing Enter.

Now what we want to do is start the Elasticsearch service and to do that we are going to input the next commands: `systemctl daemon-reload` `systemctl enable elasticsearch.service` `systemctl start elasticsearch.service` and to verify if the service is running we can use `systemctl status elasticsearch.service` and it should appear as shown in the image.

![imagen](https://github.com/user-attachments/assets/805ca041-da32-459b-a98d-d63a159a2b22)

## Day 4.- Setting up Kibana

To install Kibana in our VM we are going to do the same as before when we installed Elasticsearch. In the official page for Elastic we are going to look for Kibana in the ELK Stack and head to its download page, once here we need to select the platform, once again is going to be "DEB x86_64" and then right click the Download button and copy the link.

First we are going to download it in our VM, to do that we are going to use the command `wget https://artifacts.elastic.co/downloads/kibana/kibana-9.0.0-amd64.deb`.

![imagen](https://github.com/user-attachments/assets/ef79f580-9022-473f-ac3a-b709e8368bd2)

Now to install it we are going to use the command `dpkg -i kibana-9.0.0-amd64.deb` and then press Enter.

![imagen](https://github.com/user-attachments/assets/57747e03-c22d-42db-a0a6-cc67d5d78175)

Next we need to make a couple of changes to the configuration file of kibana, which is located in etc/kibana/kibana.yml. We can edit the file using the command `nano etc/kibana/kibana.yml`
Inside the configuration file we are going to be editing 2 lines. The first contains "server.port: 5601" and the second "server.host: "localhost". To make it so that the lines are read after executing Kibana we are going to remove the # symbol at the start of the line. Then we are going to replace the "localhost" string in the server.host line and change it to the Public IP of our VM.

![imagen](https://github.com/user-attachments/assets/eda90e4b-125f-436e-bebc-3eaa44cbe91b)

After editing the file we are going to start the service using the next commnands, `systemctl daemon-reload`, `systemctl enable kibana.service`, `systemctl start kibana.service` and finally we are going to use the commnad `systemctl status kibana.service` to verify if the service is running.

![imagen](https://github.com/user-attachments/assets/1bea11bf-6f8c-4f72-9950-089986b9a860)

Before we access Kibana we need to generate a Elasticsearch enrollment token. To do that we are going to head to the directory of `cd /usr/share/elasticsearch/bin` inside this directory we are interested in the file "elasticsearch-create-enrollment-token". To generate the token we will use this command ` ./elasticsearch-create-enrollment-token --scope kibana`

![imagen](https://github.com/user-attachments/assets/641521df-8575-42ed-a1b7-6e2ed4f47e6b)

After running the command a string of characters will be displayed, we will copy it and save it.

Now we will be accessing our Kibana instance via web browser using the public IP of our VM and the port 5601.

![imagen](https://github.com/user-attachments/assets/328fb186-e450-410d-a298-37eef7af56ca)

Here is where we are going to be using the token we generated earlier. Justo go ahead and paste it in and then click "COnfigure Elastic". After that is going to ask us for another code, which can be obtained by first heading to the directory of usr/share/kibana/bin, here we are going to be running the command `./kibana-verification-code`.

![imagen](https://github.com/user-attachments/assets/1de9429f-1ba4-4fbe-b60f-46e874310121)

After it finish configuring Elastic, it will prompt us for a username and a password. Earlier in this project we copy a couple of lines for the installation of Elasticsearch with the words "Security autoconfiguration information", these contain the user and password for Elastic. The default user is Elastic and the password is contained within the text.

![imagen](https://github.com/user-attachments/assets/76a794cf-0ff8-4aa4-b12c-0dfa9419bd6d)

In this new screen we will hit "explore on my own". Now finally we will do one more configuration, we can start by clicking on the 3 lines on the top right corner of the screen and then selecting "Alerts" within the Security section.

![imagen](https://github.com/user-attachments/assets/abd6c8f3-56f6-45d9-bc06-f126f2418cf2)

Once inside it will give us an error, the same alert give us the instructions to solve it.

![imagen](https://github.com/user-attachments/assets/f7ec24ff-604e-4e59-962c-c69a66e961f4)

Going back to our VM, we need to run the "kibana-encryption-keys" file. To run it we can use the command `./kibana-encryption-keys generate`. This will generate 3 encryption keys which we will need to copy and save them. This keys will be paste in the "kibana-keystore" file, to do that we will use the command `./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey`. Once we run this command it will ask us for the key. After doing that we will repeat the same process with the other 2 keys.

![imagen](https://github.com/user-attachments/assets/83ddf6e6-12c4-4730-af93-47379d45fdf2)

After doing that we need to restart the service using `systemctl restart kibana.service` and then reload the webpage. Once done that we would have successfully installed Kibana.

![imagen](https://github.com/user-attachments/assets/4192f125-8c4e-4aea-849a-1575cd7f6c1b)

## Day 5.-Creating a Windows Server 2022 Virtual Machine

We are going to be creating a Windows Server Virtual Machine in Microsoft Azure. This VM is going to be the one being attacked and where we are going to get the logs from.

To start we are going to go to Microsoft Azure Portal and select "Virtual Machines" and then click "Create" followed by "Azure Virtual Machine". For the name of this VM I'll be using "CSBarista-WINSER", select the same region as the other VM we've created, and in image we select "Windows Server 2022 Datacenter - x64 Gen2".

![image](https://github.com/user-attachments/assets/757a90a8-19c8-43e9-810b-b1bacad3b5cc)

Once done that we need to create a user and a password so that we can have access to the machine and for the field of "Select inbound ports" we'll need to add HTTP and HTTPS. For the rest of the fields we'll leave it as dafault.

![image](https://github.com/user-attachments/assets/cc510d8a-3fcb-45e4-8527-f33666162ca9)

To continue we are going to leave the "Disks" section as default and in the "Networking" section we are going to create a default Virtual Network, we are not going to be using the Virtual Network we created earlier because we don't want outsiders who compromised our Windows Server machine to have access to the rest of the VMs. So we are going to create a different Virtual Network and leave everything else by default.

![image](https://github.com/user-attachments/assets/a498ffc4-2193-4943-a353-47eabea33c72)

Finally we are not going to be configuring anything else, so we can go ahead and click "Review + Create" and then "Create". Once the machine has been deploy we can try to connect to it using the public IP given to us by Azure. To connect to the machine we can use "Remote Desktop Connection" an application that comes by default with windows that will allow us to connect remotely to another machine using an IP, a User, and a Password. To open the application we press "Windows Key + R" and search by "mstsc".

![image](https://github.com/user-attachments/assets/fbc1e107-c6fe-4164-bada-d009a4437a13)

![image](https://github.com/user-attachments/assets/cae42a9b-d9ef-442f-9588-d872948e8967)

Once we click connect it will ask us to provide a User and a Password, we will need to input the one we used to create the VM. After that it will show a message saying that the identity of the remote computer cannot be verified, in this case we can ignore that and click "Yes".

![image](https://github.com/user-attachments/assets/aeb873d7-ddde-4b00-905b-34d53c4ac470)

## Day 6.- Elastic Agent and Fleet Server Introduction

**Elastic Agent** provies a unified way to add monitoring for logs, metrics, and many different types of data. Elastic Agents are part of the Elastic ecosystem designed to simplify and unify data collection across various sources, facilitating observability, security, and search functionalities. They allow for efficient monitoring of logs, metrics, and other types of data from hosts, thereby enhancing the deployment and management of monitoring infrastructure. Elastic Agents operate under a single, unified approach, aiming to streamline data collection processes, configuration, and scalability across different environments.

**Fleet server** Is a component that connects the Elastic Agent to Fleet that will allow us to manage multiple agents in a centralized locationis a component that connects Elastic Agents to Fleet. It supports many Elastic Agent connections and serves as a control plane for updating agent policies, collecting status information, and coordinating actions across Elastic Agents. It also provides a scalable architecture.


### 6.1.- Difference between a Beat and an Agent
A Beat makes it more easy to collect specific type of logs from machines whereas an Agent collects all kinds of data from a machine.


## Day 7.- Setting up Elastc Agent and Fleet Server

First things we are going to do is create another Virtual Machien for our Fleet Server. Same as the other VMs we've created earlier we are going to create a new VM instance in Microsoft Azure, I'll be giving it the name of "CSBarista-FleetServ", using the image uf Ubuntu Server 22.04, choosing a Username and Password and finally we are going to add it to our Virtual Network so that it can comunicate to our ELK server.

![image](https://github.com/user-attachments/assets/ef390842-fc71-4834-9b16-003d98cdf531)

While the VM finishes deploying, lets head to Elastic using the "public IP:5601" and then head to the top left corner click the 3 lines icon and search for "Fleet" within the "Management" section.

![image](https://github.com/user-attachments/assets/5c29cc4f-cb74-4113-b713-a89efaa78f8e)

Here we will be presented with a "Add Fleet Server" which we are going to click it. To add a Fleet server we are going to use the "Quick Start" option, then we are going to give the server a name, I'll be using "CSBarista-Fleet-Server" and for the URL we are going to be using the public IP of the VM we just created ,which should be done deploying by now, and add https://, after that we can click on "Generate Fleet Server policy".

![image](https://github.com/user-attachments/assets/7359d651-3ead-49bb-ba75-1d55f97e9551)

After the process finishi creating the new policy, we are going to install the Fleet Server Agent on our newly created VM. But before doing that we are going to run the usual command of `apt-get update && apt-get upgrade -y`. 

Continuing with the installation of our Agent we are going to copy the command for "Linux Tar" which is:
`curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.0.0-linux-x86_64.tar.gz
tar xzvf elastic-agent-9.0.0-linux-x86_64.tar.gz
cd elastic-agent-9.0.0-linux-x86_64
sudo ./elastic-agent install \
  --fleet-server-es=https://172.31.0.4:9200 \
  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE3NDY0OTIxMzE2NTA6dV9CRU03Z29SQTJRUDJJakx3TW0xZw \
  --fleet-server-policy=fleet-server-policy \
  --fleet-server-es-ca-trusted-fingerprint=e36035cef7fb4939a59466e09d30b2f837b1953a768492c0fe325556e6348326 \
  --fleet-server-port=8220 \
  --install-servers`

and paste it in the VM we created for Fleet. it will prompt us with a question and we can just answer witn "y". After a couple of minutes when the installation finishes we should see in our Elastic page that the Fleet Server is now connected to Elastic.

![image](https://github.com/user-attachments/assets/ee62427b-d551-40ff-bd2c-cdccf3c4ba00)

We may run into errors in the future due to the network configuration for rules we created early in this project when configuring our ELK server, so let's add a rule to our server firewall to allow traffic from port 9200.

![image](https://github.com/user-attachments/assets/65ee0c73-e408-4138-9702-8f6f2f1e4dbd)

Now continuing with the configuration of our Elastic Agent we are going to click on "Continue enroilling Elastic Agent". It will ask us "What type of host do you want to monitor?" so we are going to give it a name to the host, I'll be using "CSBarista-Windows-Policy" as the name, after that we can click on "Create policy". 

![image](https://github.com/user-attachments/assets/380082f5-8e83-496d-b834-d79d2def80ec)

Once the policy is created we can scroll down and select the host we are going to be using, in this case we are going to be using Windows so we are going to copy the command. Note that we are going to need admin privileges to run this command.

![image](https://github.com/user-attachments/assets/5ffff378-d48d-409c-b9d2-c07af1824bde)

After running the command the installation failed saying that it couldn't connect to our fleet-server, so running through the documentation for installing a Elastic Agent it says that the connection is usually made in port 8220 but in the configuration the port is set to 443. To make that change we are going to go to the Settings tab within our Fleet page and change the port of our IP from 443 to 8220 clicking on the pencil.

![image](https://github.com/user-attachments/assets/6fbf8f40-f2aa-40d3-9266-e03a87582a92)

Also before we run the command we are going to add `--insecure` at the end, otherwise it will give another failed installation because the web page doesn't have a certificate.

![image](https://github.com/user-attachments/assets/347bc521-6abd-4fc7-abc1-32965ba31ead)

In Elastic it will also show our Fleet server and our Agent.

![image](https://github.com/user-attachments/assets/257c4230-d03a-4e5f-940d-58c755eae259)























