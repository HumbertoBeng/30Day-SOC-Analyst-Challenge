Day 3.- Setting up Elasticsearch

1.-Setting up a Virtual Network in Microsoft Azure

First we are going to look for the Icon for "Virtual Network" as show in the image.

![imagen](https://github.com/user-attachments/assets/abf8f42a-0c6a-4719-bf43-d9fb97ad0583)

Once inside we'll need to click on the button "Create Virtual Network". Now we need to choose a subscription, in my case is the "Azure Subscription 1" since I'm using the free tier from Azure, then select a Resource group, the Resource group is going to be the group where all the machines we are going to use and that we will be using for all the other stuff related to the project. So to create a new Resource group just click on "Create new" and then select a name, in this case I'll be using the name of this Challenge.

![imagen](https://github.com/user-attachments/assets/01403ddc-c563-44d1-90b0-3371d888dd14)

Next we are going to give a name to our virtual Network and then select the Region that is closest to us as to not run into any latency problems.

![imagen](https://github.com/user-attachments/assets/956a9616-8505-40bc-b64a-3de5d8f454c0)

We will not be setting "Security" for now. So in the "IP address" tab we are going to be setting the range of IPs we will be using for our virtual machines. Looking at the Diagram we created early we said we were going to be using the range of IPs of "172.31.0.0/24".

![imagen](https://github.com/user-attachments/assets/5da99aef-a8ed-4fef-a9b7-4ba46ac6cec1)

Finally we click on the "Review + create" button to create our Virtual Network.


2.-Creating our Virtual Machines

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

3.- Installing Elasticsearch

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

Day 4.- Setting up Kibana

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

Day 5.-
