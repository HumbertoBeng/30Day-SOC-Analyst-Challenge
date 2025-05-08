# Week 2 of 30 Day SOC Analyst Challenge. Created by MyDFIR

## Day 8.- Introduction to Sysmon

Sysmon is a free microsoft tool that is part of the Sysinternals suite and it provides with a lot of telemetry. It has the capability of monitoring events such as Process Creation, Network Connections, File Creations, and many more.

### 8.1 Important Event IDs

Event ID: 1 - Process Creation
The process creation event provides extended information about a newly created process. The full command line provides context on the process execution.

Event ID: 3 - Network Connections
The network connection event logs TCP/UDP connections on the machine. It is disabled by default. Each connection is linked to a process through the ProcessId and ProcessGuid fields.

Event ID: 6/7/8 - Driver/Image Load & Create Remote Thread
This Event IDs could help identify potential defense evasion techniques.

Event ID: 10 - Process Access
The process accessed event reports when a process opens another process, an operation that’s often followed by information queries or reading and writing the address space of the target process. This enables detection of hacking tools that read the memory contents of processes like Local Security Authority (Lsass.exe) in order to steal credentials for use in Pass-the-Hash attacks.

Event ID: 22 - DNS Event (DNS Query)
This event is generated when a process executes a DNS query, whether the result is successful or fails, cached or not. 


## Day 9.- Setting up Sysmon

All the installation and configurations are going to be made in our Windows Server Virtual Machine.

First we are going to go to the official page of "<a href="https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon">Sysmon - Sysinternals</a>". Once we are on the site we are going to download sysmon clicking on the "Download" button.

![image](https://github.com/user-attachments/assets/e2487683-ff39-4141-8e4c-4af5842157e2)

Next step we are going to Extract the contents of the file we just downloaded. After that we are going to be downloading the configuration file made by a user called <a href="https://github.com/olafhartong/sysmon-modular">Olaf</a>. 

![image](https://github.com/user-attachments/assets/2d404602-ca7b-4fb9-a4d5-c5614c517859)

Here we are going to scroll down a bit and search for a file called "sysmonconfig.xml", we are going to click that file and then select "Raw".

![image](https://github.com/user-attachments/assets/26f8a7c5-dd33-4a0c-bea5-11f95345b870)

To download the file we are going to right click and then select "Save as" and save it into Sysmon directory.

![image](https://github.com/user-attachments/assets/05962ba9-4632-4aa3-a876-f9197ad8f05f)

Now what we wanna do is open a powershell window and run it as an administrator. Then we wanna change into the directory of Sysmon, next we are going to run the command ` .\Sysmon64.exe -i .\sysmonconfig.xml`. When a new window shows up we can click on "agree".

![image](https://github.com/user-attachments/assets/951e8e41-2cf0-4320-bb4b-3046eba86f48)


## Day 10.- Feeding Logs to Elasticsearch

### 10.1.- Sysmon

The first thing we are going to do is Login into our Elastic page and in the initial page there should be a blue button called "Add integrations", we are going to click it. This will take us to a page with a lot of different types of way to feed Logs into Elasticsearch, since we are going to be sending Sysmon and Windows Defender logs, in the field "Search for integrations" we are going to be searching for both. Since Sysmon does not show when searching it up, we can search for "Windows" and select the "Custom Windows Event Logs".

![image](https://github.com/user-attachments/assets/07406eb5-fa2b-49f6-9bc6-42f5d1f54b25)

![image](https://github.com/user-attachments/assets/592a34a2-6dc3-4220-b8ca-d8025ea6a49f)

Now inside this integration we are going to click on "Add Custom Windows Event Logs", here we are going to give it a name, I'll give it the name of "CSBarista-WIN-Sysmon". Then we are going to be giving it a description.

![image](https://github.com/user-attachments/assets/d7300536-9ef1-4249-9805-a5b8fceee897)

Scrolling down for the Channel Name, we can obtain that going to our Windows Server VM, searching for "Event Viewer" in the search tab. Then inside the Event Viewer window we are going to click on "Applications and Services Logs" -> "Microsoft" -> "Windows" -> "Sysmon", then we are going to right click "Operational" and select "Properties". In the new window the field with "Full Name:" is going to be what we are going to put in the "Channel Name" field inside Elastic.

![image](https://github.com/user-attachments/assets/3ac4a964-f2a5-451b-85c4-aeefdaf58830)

![image](https://github.com/user-attachments/assets/a9bcc5b0-7660-4a7d-90ef-2022cd1661ab)

Continuing we are leaving the rest as default. At the bottom of the page we are going to add this integration to an Existing Host and we are going to select the one we create for Windows.

![image](https://github.com/user-attachments/assets/5d99190b-b7ba-4406-9465-9bdd8130d2e2)

Finally we can click on "Save and continue" and then "Save and deploy changes".

![image](https://github.com/user-attachments/assets/e2acf2ef-764c-4166-b94d-b54c9e30e563)

### 10.2.- Windows Defender

To add the integration for Logs from Windows defender we are going to repeat the same steps from before but changing the "Integration name" and the "Channel name".

![image](https://github.com/user-attachments/assets/2c9f3851-7361-480d-a609-ee214e0d9282)

To obtain the "Channel Name" we are going to go to the same path in "Event Viewer" within our Windows Server VM. The logs that Windows Defender generate are in a folder called "Windows Defender".

![image](https://github.com/user-attachments/assets/7ec205ef-8213-4421-a321-5355124c5c5a)

Now since Windows Defender also Logs information that is not relevant to us, so for the sake of this project we are only going to be retreiving Logs with event ID of 1116, 1117, and 5001. The Event ID of 1116 is generated when the antivirus detects malware or a potentially unwanted software in our system, the Event ID 1117 is generated when the antivirus performed an action to protect our system from malware or other potentially unwanted software and the last event ID of 5001 is generated when the option for Real-time protection is disabled.

So for the "Channel Name" we are going to right click the "Operational" file and select properties, the field "Full name" is what we are going to be using for this.

![image](https://github.com/user-attachments/assets/1aac9048-aef3-4fcd-8b88-66878a35a45e)

To set the integration so that it only collect the Event IDs of 1116, 1117, and 5001 we are going to click on "Advanced options" and under the field "Event ID" we are going to write the IDs we are going to retrieve. Under this field we can add or exclude Event IDs.

![image](https://github.com/user-attachments/assets/2a1d22fa-b688-4267-9996-7c90e4435da2)

Finally we are going to add this integration to the same Windows-Policy from before and then click on "Save and continue" and "Save and Deploy changes"

![image](https://github.com/user-attachments/assets/4c7259cc-9e52-45e9-a96a-cc223c92a865)

To test if the logs are being received by Elastic we can go to the top left and click on the 3 lines and then select "Discover"







































