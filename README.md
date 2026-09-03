# SOC-Analyst-Homelab
## Overview/Objective
In this lab, I plan to achieve a SOC analyst homelab for gaining hands-on cybersecurity knowledge and experience. This homelab will be used for gaining an understanding of how to configure a SOC Analyst homelab and to serve as a foundation for future projects and labs. I will achieve this by configuring a Kali Linux attack machine, a vulnerable Windows 10 target machine, and a Ubuntu server with SIEM and IDS implementation. Specifically, the Ubuntu server will be hosting Splunk as the SIEM and Suricata as the IDS. All virtual machines will be configured to communicate with each other on an isolated network, away from the host machine and host network. This effectively creates a secure SOC Analyst homelab for performing cybersecurity practices.

## Tools & Technologies Utilized
Oracle Virtualbox, Kali Linux, Ubuntu Server 24.04.03 LTS, Microsoft Windows 10, Sysmon, olafhartong’s sysmon-modular config file, Splunk, Splunk Forwarder, and Suricata.

## Steps Taken to Achieve Objective
### Prerequisites 
-	Download and install Oracle Virtualbox.
-	Download the Kali Linux pre-built Virtualbox image
-	Download the Windows 10 ISO
-	Download the Ubuntu Server ISO

### Kali Linux Installation/Configuration
-	In Virtualbox, I selected the “Add” button at the top and browsed for the Kali Linux.vbox file that was downloaded. Selecting the vbox image automatically added it as a machine in Virtualbox.
-	I started the Kali Linux machine and used the credentials “kali” in both of the username and password fields to log in.
-	Once I was at the desktop screen, I opened up the terminal and executed the command “sudo apt update && sudo apt upgrade” to get the latest packages and update the system. I typed in “kali” for the password and “y” to confirm the upgrade.
-	I changed the virtual network adapter to “Internal Network” instead of “NAT” to put the machine on an isolated network that I named “LabNet”

<img width="2560" height="1392" alt="Kali-Isolated-Network" src="https://github.com/user-attachments/assets/0fc8e75e-c028-401b-8b80-2d354a4faa54" />

-	Finally, I changed the network settings to use a manual configuration instead of DHCP. I addressed this machine as 192.168.0.10 with a netmask of 24 and a gateway of 192.168.0.5.
  
<img width="2560" height="1394" alt="Kali-Addressing" src="https://github.com/user-attachments/assets/1226a6fa-7811-4cf2-8235-c7992fe21c91" />

### Ubuntu Server 24.04.03 LTS Installation/Configuration
-	To create the Ubuntu Server virtual machine, I clicked the “New” button at the top of the Virtualbox window. I named it “Ubuntu Server 24.04.03 LTS” and selected the Ubuntu Server ISO image. I unchecked the box for “Proceed with unattended installation” and gave the virtual machine 2 processors, 4GB of ram, and 40GB of fixed hard disk space.
-	I started the virtual machine and selected the “Try or install Ubuntu Server” option at the bootloader.
-	Once in the installer, I selected “English” -> Done -> Ubuntu Server -> Done -> Created a netplan that contains one NIC connecting to the internet and the other for the isolated network -> Done -> Done -> “Use Entire Disk” -> Done -> Done -> Continue.

<img width="1282" height="849" alt="Net-Plan config" src="https://github.com/user-attachments/assets/dd364fd4-6ba8-4ba8-9060-8e6591711d5d" />

-	Next, it asks for names and passwords for the server. For names I chose “Bob” as my name, “ubuntu-server” as the servers name, “bob” as a username, and “bob123” as the password.
-	I skipped the Ubuntu Pro option and selected “Done” to finalize the installation.

### Splunk Installation
-	After booting into Ubuntu Server for the first time, I logged into the user I created and then enabled the firewall by executing the command “sudo ufw enable.”
-	I executed the command wget -O splunk-10.0.2-e2d18b4767e9-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.0.2/linux/splunk-10.0.2-e2d18b4767e9-linux-amd64.deb" to download the Splunk deb package to the server. After that, I ran “sudo dpkg –i splunk-10.0.2-e2d18b4767e9-linux-amd64.deb” to install the Splunk package.
-	Next, I configured Splunk to start at boot with the “sudo /opt/splunk/bin/splunk enable boot-start” command.
-	I agreed to the licensing and went on to create the administrator account. For the username, I chose “admin” and for the password, I chose “splunkadmin”.
-	Finally I ran the command “sudo ufw allow 8000” so that the Splunk webpage can be accessed from other computers on the network without the firewall interfering. 

### Suricata Installation/Configuration
-	To install Suricata, I first had to install the required dependencies by executing “sudo apt -y install software-properties-common”. I then added its repository by running the command “sudo add-apt-repository ppa:oisf/suricata-stable”. I then ran “sudo apt install suricata” to install it.
-	I enabled Suricata as a service so that it starts on boot, I did this by executing the command “sudo systemctl enable suricata.service”.
-	I configured Suricata’s configuration file by running “sudo nano /etc/suricata/suricata.yaml”. In the configuration file, I changed the “HOME_NET” to be “[192.168.0.0/24]” to match the lab network. I also changed the interface under “af-packet” option to be the Ubuntu Server adapter that is connected to the isolated network, which is “enp0s8”. To find out what interface name your system uses, you can simply check by running “ip a” and replacing the default interface name with that.

<img width="1286" height="844" alt="Suricata-Home-net" src="https://github.com/user-attachments/assets/6a969c7e-8613-45bf-9995-49890fdc547c" />

-	Rules need to be added next. This can be done by first searching rules with the command “sudo suricata-update list-sources”. A list of rules will be presented to choose from. In this lab, I chose the “et/open” rule. To install this rule, you simply run the commands “sudo suricata-update enable-source et/open” followed by “sudo suricata-update to update Suricata's rules.

<img width="1279" height="852" alt="Suricata-Rules" src="https://github.com/user-attachments/assets/3e33071e-5c04-43ea-ba50-a361ef517539" />

-	To validate that Suricata's config file is valid and working, I ran the “sudo suricata –T –c /etc/suricata/suricata.yaml –v” command. My config was successfully validated and working.

<img width="1281" height="851" alt="Suricata-Rule-Config-test" src="https://github.com/user-attachments/assets/25ff9879-00f4-459e-b7fe-288669d3094a" />

-	I started the Suricata service by running “sudo systemctl start suricata.service” followed by “sudo systemctl status suricata.service” to make sure the Suricata service is active.
-	To test if Suricata is successfully deployed, I first changed to the root user by using the command “sudo su” followed by “cat /var/log/suricata/fast.log”. I needed root privileges to view the log file and ensure it is currently empty. With Suricata running, it should be actively looking for network-based threats. To confirm this, I changed the af-packet interface to “enp0s3” so it can reach the internet and monitor the NIC and ran "curl http://testmynids.org/uid/index.html". This should generate a Suricata alert and I confirmed this by rechecking the fast.log file. Afterwards, I cleared the log file and reverted back to the interface assigned to af-packet as I intend this to be completely isolated from the internet.

<img width="1280" height="856" alt="Suricata-testmynids-test" src="https://github.com/user-attachments/assets/874ab013-de03-465c-99b2-6f7cd07f8625" />

### Windows 10 Installation/Configuration
-	To create the Windows 10 virtual machine, I clicked the “New” button at the top of the window of Virtualbox. I named the virtual machine “Windows 10”. Next, under the “ISO Image” option, I clicked the down arrow and selected “Other...” to browse and select the Windows 10 ISO image I downloaded. I then checked the box for “Skip Unattended Installation.” In addition to this, I gave it 2 virtual processors, 4GB of ram, and 40GB of fixed hard disk space. I then clicked the “Finish” button and the virtual machine was made.
-	At the Windows Setup screen after booting in, I clicked “Next” -> “Install Now” -> “I don’t have a product key” -> Windows 10 Pro -> “Next” -> checked the “I accept the license terms” box -> “Next.” I was then presented with the installation method screen and selected the “Custom: Install Windows only (advanced)” option, followed by selecting “Drive 0 Unallocated Space” and clicking “Next.” Windows 10 Pro should now be installing to the 40GB virtual hard disk.
-	You will be rebooted into the new installation of Windows 10 Pro where you have to personally set up the system. I selected United States as my region and keyboard layout, Skipped the secondary keyboard layout, and chose “Set up for personal use.” Next, instead of logging into a Microsoft account, I chose the “Offline account” option. If this option doesn’t show, you can alternatively disconnect the virtual network adapter for the virtual machine. Doing so will force you to create a local/offline user account. I then used the fake name “John” and made the password “john123.” For the security questions, I just chose 3 and put random information for each as i couldn’t skip this. I clicked next and accepted everything else after this, which was nothing significant to this project.
-	Next, I downloaded Sysmon, a Windows service that will monitor and log in-depth security events to Event Viewer. First, download the latest Sysmon version from the official Microsoft website. It should come in the form of a ZIP file. I then created a folder on the desktop called “Sysmon” and extracted the contents of the ZIP file into the folder by right-clicking on the ZIP file, selecting Extract all, then browsing for the Sysmon folder on the desktop, and clicking the Extract button. I also downloaded a pre-built config file for Sysmon from Github by Olaf Hartong. I did this by going to the Github link, selecting the default config file, right-clicked on the code and clicked “Save as” to save it to the Sysmon folder.

<img width="2560" height="1397" alt="Sysmon Config" src="https://github.com/user-attachments/assets/ba645fa1-9fed-49c5-ab0d-45e36f754673" />

-	To install Sysmon, I opened Powershell as an admin and typed in the command “cd C:\Users\John\Desktop\Sysmon” and executed it. This changes the directory to the root of the Sysmon folder. Next I ran the command “.\sysmon64.exe –i sysmonconfig.xml” and it opens a EULA that you need to agree to. Once agreed, Sysmon should be successfully installed and running. You can check this by finding if “sysmon64” is running in Services and by checking for Sysmon logs in Event Viewer.

<img width="2560" height="1396" alt="Deploying Sysmon" src="https://github.com/user-attachments/assets/e0f62f73-15eb-4f21-a970-46bb3553343f" />
<img width="2560" height="1398" alt="Confirming Sysmon is Running" src="https://github.com/user-attachments/assets/729c2cde-98c8-4c43-925d-59ccc3cebf06" />

-	The last tool that needs to be downloaded is the Splunk Universal Forwarder so the Sysmon logs can be ingested by the Splunk server. Once downloaded, run the installer and ensure that the Splunk server is currently running on the Ubuntu Server machine. In the setup wizard, I used the same credentials as the Splunk server. “admin” as the username and “splunkadmin” as the password. Next, I skipped the Deployment server option and instead typed the 192.168.0.5:9997 for the Receiver Indexer option. I then finished the installation. Next, I configured the Splunk Forwarders inputs.conf file. This must be configured in order to point event logs into a Splunk index. The input.conf file I used can be found under the Configs folder in Github.
-	The final steps in configuring this virtual machine is to place it in the isolated “LabNet” network with the other machines and give it a valid IP address. To do this, you must change the virtual network adapter from “NAT” to “Internal Network” and name it “LabNet” like before. Finally, we need to configure a manual IP address instead of using DHCP. In the search bar, search for “View Network Connections” and press Enter. This should bring up a window consisting of only one network adapter. Right-click the adapter and select “Properties.” Scroll down the list in the new window to find “Internet Protocol Version 4 (TCP/IPv4).” Select it and click on the “Properties” button. Choose the “Use the following IP address” option and type in the desired IP address and correct subnet mask. I chose the IP address to be “192.168.0.11”, “255.255.255.0” as the subnet mask, and “192.168.0.5” as the gateway. To confirm that it is using this configuration, run Command Prompt and execute “ipconfig”. I then turned off Windows Defender security and firewall security to make the system vulnerable.

<img width="2560" height="1367" alt="Windows-Network-Config" src="https://github.com/user-attachments/assets/f7d8f899-e7bd-454e-992a-ac2ed9da51dd" />

### Splunk Configuration
-	I accessed the Splunk webpage by typing in “http://192.168.0.5:8000" in a web browser on the windows machine and created a receiving port of “9997”.
-	After logging in, I first wanted to setup Splunk to ingest Sysmon logs from the Windows machine. I created an index called “win_log” and configured a receiving port of 9997. For Splunk to be able to receive the Sysmon logs from port 9997, you must open the firewall on that port by executing the command “sudo ufw allow 9997” on the Ubuntu server. 
-	I then installed the app “Splunk Add-on for Sysmon” from the Apps -> Find More Apps section. This provides useful search features for finding information in Sysmon logs.  
-	After Suricata has been installed and configured, I needed to set up Splunk to ingest Suricata’s eve.json log. I did this by creating a new index called “suricata” and a source type called “suricata:eve”. I then went into the Splunk web interface and into Settings -> Local Inputs -> Data Inputs -> Files & Directories -> + Add New -> Browsed for Suricata's eve.json file -> selected the Suricata index and source type. Now, by searching for “index=suricata” in Splunk, it pulls up detailed alerts generated by Suricata.

<img width="2558" height="1364" alt="Suricata-Splunk-Logs" src="https://github.com/user-attachments/assets/3835ec13-60e9-493e-86ca-d3f8c3ce6008" />

- After Sysmon and the win_log Splunk index has been configured, I can now confirm if Splunk is properly ingesting the Sysmon event logs. To test this, I opened Command Prompt on the Windows machine and executed the "ipconfig /all" command. I then searched for the command using the win_log index via Splunk, which revealed one event containing the command. This proves that addition to Suricata alerts, Sysmon event logs are also being ingested by Splunk as well.

<img width="2558" height="1358" alt="Sysmon-Splunk-Logs" src="https://github.com/user-attachments/assets/cfdcd86f-7985-43f3-8c46-259af9728b1e" />

## Cleaning Up
-	Now that the environment has been set up properly, I cleaned up the environment by clearing log files. This is so that when the environment is used as a foundation for future labs, there will be no previous events in the log files to cause any sort of confusion.
-	The main log files that need to be cleared are the Sysmon, Application, Security, and System logs in Event Viewer. The eve.json and fast.log log files from Suricata and the Splunk indexes for both Suricata and Sysmon also need to be cleared as well. 
-	I first started with the Windows machine. I went into Event Viewer and then Windows Logs to clear the Application, Security, and System logs by right-clicking and selecting “Clear Log...” and “Clear” for each. To clear Sysmon events from Event Viewer, I had to go to Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational. I then right-clicked on Operational and selected “Clear Log..” to clear previous events.
-	Next, I cleared the Suricata logs by first stopping the Suricata service. I ran the command “sudo systemctl stop suricata.service” to stop Suricata and prevent further events from being logged for now. I then ran the commands “sudo truncate –s 0 /var/log/suricata/fast.log” and “sudo truncate –s 0 /var/log/suricata/eve.json”. To confirm that the logs were cleared, I used the Cat command to check the log files.
-	Finally, to clear Splunk events I first stopped Splunk by executing “sudo /opt/splunk/bin/splunk stop” and then ran the command “sudo /opt/splunk/bin/splunk clean eventdata”. I then typed “y” to continue. This effectively clear events from both the win_log and suricata indexes in Splunk.
-	The last thing I did was disable the NAT network adapter for the Ubuntu server in Virtualbox. I did this by right-clicking on the Ubuntu Server VM -> Settings -> Network and then unticking the the checkbox for the NAT network adapter. This ensures that the whole environment is isolated from the internet as the setup and configuration process is done. 

## What I Learned & Challenges Overcome
I learned a significant amount of valuable information from this project. A few of the biggest things I learned was setting up and configuring SIEM and IDS technologies such as Splunk, Splunk Forwarder, and Suricata to work properly together. I ran into a number of challenges while setting up and configuring these services as well. One of the biggest challenges I overcame was configuring Splunk to ingest Sysmon events. Setting it up was easy, but ran into the issue of getting all event logs from the Windows machine except for Sysmon. After analyzing the Splunk Forwarder event log, I found that Splunk Fowarder didn’t have the permissions to access the Sysmon operational event log. I corrected the permissions in Windows Services and was able to successfully ingest Sysmon event logs after the correction.

## Results/Conclusion
In this project, I successfully achieved a SOC Analyst homelab that is safe for cybersecurity practices. This project was important to conduct and learn from as it gave me valuable experience implementing and deploying SIEM and IDS technologies like Splunk and Suricata. In addition to this, having a secure homelab environment is crucial to gaining more cybersecurity knowledge and experience safely as well. I now have an isolated environment consisting of an Ubuntu server hosting Splunk and Suricata, a vulnerable Windows machine that can forward Sysmon events to Spunk for ingestion, and a Kali Linux machine for attacking. This serves as a very useful foundation for future projects and labs.
