# Active-Directory-Project

## Introduction

This document provides a comprehensive guide for setting up an Active Directory lab environment using virtual machines. The project utilizes Oracle VM VirtualBox to create a network of VMs, including Windows 10, Kali Linux, Windows Server, and Ubuntu Server. This controlled environment is designed for exploring cybersecurity concepts, with a focus on Active Directory administration, security testing, and log analysis.


### Project Objectives

The primary objective of this lab is to provide a hands-on learning experience by:
* Creating and configuring a virtualized environment with a variety of operating systems (Windows 10, Kali Linux, Windows Server, and Ubuntu Server).
* Gaining practical experience in network configuration, security tool deployment (e.g., Splunk and Sysmon), endpoint monitoring, and security testing (e.g., brute force attacks with **Hydra**).
* Integrating Windows machines into an Active Directory domain and enabling Remote Desktop access for remote management.
* Utilizing PowerShell and **Bash** scripting to automate various administrative tasks.
* Overall, this lab aims to equip participants with a foundational understanding of cybersecurity concepts, tools, and techniques in a secure and controlled setting.


### Requirements and Tools

| **Category** | **Tools and Requirements** |
| :--- | :--- |
| **Virtualization** | Oracle VM VirtualBox Manager |
| **Operating Systems** | Windows Server 2022, Microsoft Windows 10, Kali Linux, Ubuntu Server 24.04.1 LTS |
| **Security Tools** | Splunk Server, Splunk Universal Forwarder, Sysmon, Hydra, Atomic Red Team (ART) |
| **Log Analysis** | Microsoft Windows Event Logs |
| **Scripting** | PowerShell, Bash |


## Active Directory Project Diagram:
<img width="629" height="659" alt="active_directory" src="https://github.com/user-attachments/assets/cd553097-9d7a-48c8-b41b-96553e84cb18" />


## 1. Active Directory Project Setup

This section outlines the steps for VM installation, network configuration, and tool deployment.

### 1.1. VM Installation

1. **Windows 10:** Download the Windows 10 ISO file using the Microsoft installation media creation tool. In VirtualBox, create a new VM with the ISO, allocate **6096MB** of RAM, **4 CPU**, and a 50GB virtual disk, then proceed with a custom installation.
2. **Kali Linux:** Download the Kali Linux VM version and use a tool like 7-zip to extract the files. Import the extracted VM into VirtualBox and start it.
3. **Windows Server 2022:** Download the Windows Server 2022 ISO. In VirtualBox, create a new VM, allocate **6096MB** of RAM, **4 CPU**, and a 50GB virtual disk. Follow the on-screen prompts to install the Standard Evaluation (Desktop Experience) edition and set an administrator password.
4. **Ubuntu Server:** Download the Ubuntu Server 24.04.1 LTS ISO. Create a new VM with **8192MB** of RAM, **4 CPUs**, and a 100GB virtual disk. Follow the installation prompts, then log in and execute below command to update the system.
  ```Bash
  sudo apt-get update && sudo apt-get upgrade -y
  ```

### 1.2. Network Configuration

1. **Create a NAT Network:** In VirtualBox, navigate to Tools and select NAT Networks to create a new network. Use the IP range 192.168.10.0/24 for this project.
2. **Assign Network to VMs:** For all the VMs, change their network adapter settings to use the custom NAT network you just created. This enables internet connectivity and direct communication among the machines.
3. **Configure Static IP for Ubuntu Server:** Edit the network configuration file on the Ubuntu Server to assign a static IP address.
    - Open the network configuration file using
       ```Bash
       sudo nano /etc/netplan/00-installer-config.yaml`
       or
       sudo nano /etc/netplan/50-cloud-init.yaml
       ```
    - **Modify the file to include the static IP details:**
    ```yaml
    network:
      version: 2
      ethernets:
        enp0s3:
          dhcp4: no
          addresses: [192.168.10.10/24]
          routes:
            - to: default
              via: 192.168.10.1
          nameservers:
            addresses: [8.8.8.8]
    ```
    - Save the file and apply the changes with
       ```Bash
       sudo netplan apply
       ```
4. **Configure Static IP for Windows 10**
    * Right-click the network icon and go to "Network & Internet settings".
    * Click "Change adapter options", right-click Ethernet, and select Properties.
    * Double-click "Internet Protocol Version 4 (TCP/IPv4)", select "Use the following IP address," and enter the desired IP address and subnet mask.

5. Configure Static IP for Windows Server
    * Configure a static IP address in the same manner as the Windows 10 machine.

6. Configure Static IP for Kali linux
    * Click the network icon in the top-right corner and select "Edit Connections."
    * Choose "Wired connection 1," go to the "IPv4 Settings" tab, change the method to Manual, and add the desired IP address and netmask.
    * Click Save.

### 1.3. Splunk Server Configuration
1. **File Transfer and Guest Additions:** To facilitate file transfers between the host system and the VM, the VirtualBox Guest Additions are required. Install the necessary packages using the following commands:
    ```Bash
    sudo apt-get install virtualbox-guest-additions-iso
    sudo apt-get install virtualbox-guest-utils
    ```
2. After installation, reboot the VM.
3. **Shared Folder Configuration:**
    - Add the current user to the `vboxsf` group to enable shared folder access:
        ```Bash
        sudo adduser [username] vboxsf
        ```
    - Create a directory for the shared folder:
        ```Bash
        mkdir shared
        ```
    - Mount the shared folder:
      ```Bash
      sudo mount -t vboxsf -o uid=1000,gid=1000 [foldername] shared
      ```
      <img width="512" height="356" alt="splunk_config1" src="https://github.com/user-attachments/assets/af599480-1dac-43ab-ae4c-ef2ea9733a5d" />
      <img width="351" height="18" alt="splunk_config2" src="https://github.com/user-attachments/assets/6a1dbcc1-4706-491b-9ca4-4b0e38dbc24b" />
      <img width="512" height="141" alt="splunk_config3" src="https://github.com/user-attachments/assets/33a8a0fb-f14d-4dc3-8090-92ffd1418b64" />

4. **Splunk Installation and Configuration:**
    - Install Splunk using the downloaded installer file:
        ```Bash
        sudo dpkg -i [installer_name]
        ```
    - Navigate to the Splunk installation directory:
       ```Bash
       cd /opt/splunk
       ```
    - To perform administrative tasks, switch to the `splunk` user:
       ```Bash
       sudo -u splunk bash
       ```
    - Navigate to the bin directory:
      ```Bash
      cd /opt/splunk/bin
      ```
    - Start the Splunk service for the first time: `./splunk start`. This will prompt you to read and accept the license agreement and create an administrative username and password.
    - Configure Splunk to start automatically at boot:
      ```Bash
      sudo ./splunk enable boot-start -user splunk
      ```

### 1.4. Install Sysmon on Windows 10 and Windows Server
  1. Download **Sysmon** from [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon).
  2. Download a pre-configured `sysmonconfig.xml` from [Sysmon Modular](https://github.com/olafhartong/sysmon-modular).
  3. Extract the Sysmon files and place the `sysmonconfig.xml` into the extracted directory.
  4. Open PowerShell as an administrator.
  5. Navigate to the Sysmon directory:
      ```PowerShell
      cd C:\Users\Downloads\sysmon
      ```
  6. Install Sysmon.
      ```PowerShell
      .\sysmon64.exe to install Sysmon.
      ```
  7. Verify Sysmon is running:
      ```PowerShell
      Get-Process sysmon64
      ```
  8. Install configuration to Sysmon:
     ```PowerShell
     .\sysmon64.exe -i sysmonconfig.xml
     ```

### 1.5. Install Splunk Universal Forwarder on Windows 10 and Windows Server
1. Download **Splunk Universal Forwarder** from the [Splunk Website](https://www.splunk.com/).
2. Install Splunk Universal Forwarder.
3. Go to the folder where the forwarder is installed and navigate to `splunk > system > local`. Create a file named `inputs.conf` with the following content:
    ```ini
    [WinEventLog://Microsoft-Windows-Sysmon/Operational]
    index = endpoint
    disabled = false
    renderXml = true
    source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

    [WinEventLog://Microsoft-Windows-Windows Defender/Operational]
    index = endpoint
    disabled = false
    source = Microsoft-Windows-Windows Defender/Operational
    blacklist = 1151,1150,2000,1002,1001,1000

    [WinEventLog://Microsoft-Windows-PowerShell/Operational]
    index = endpoint
    disabled = false
    source = Microsoft-Windows-PowerShell/Operational
    blacklist = 4100,4105,4106,40961,40962,53504

    [WinEventLog://Application]
    index = endpoint
    disabled = false

    [WinEventLog://Security]
    index = endpoint
    disabled = false

    [WinEventLog://System]
    index = endpoint
    disabled = false
    ```
4. Launch Splunk from a browser using the server IP and port **8000**, log in, and enable data collection.
5. To parse the Sysmon data, download the **Splunk Add-on for Sysmon** from **Apps > Find More Apps** within the Splunk interface.
6. Create a new index named **"endpoint"** by navigating to **Settings > Indexes**, clicking **New Index**, naming it "endpoint," and saving the configuration.

### 1.6. Active Directory Configuration
1. **Install ADDS:** On the Windows Server, launch **Server Manager**. Navigate to **Manage > Add Roles and Features** to install **Active Directory Domain Services**.
2. **Promote to Domain Controller:** After ADDS installation, **go to** the flag in the top-right corner of Server Manager. Click it and select **Promote this server to a domain controller**. Choose to **add a new forest**, provide a top-level domain name, set a password, and complete the installation. The server will automatically restart. The login screen will display the domain name, confirming the **server** is now a domain controller.
3. **Create Active Directory Users:** In **Server Manager**, go to **Tools > Active Directory Users and Computers**. Right-click your domain name to create a new **Organizational Unit (OU)**. Within this new folder, right-click and create the necessary user accounts.
4 **Join Windows 10 to the Domain:** On the Windows 10 machine, go to **About > Advanced system settings > Computer Name > Change**. Enter the domain name, authenticate with an administrator account, and restart the machine. Log in as one of the newly created domain users.
   
  <img width="512" height="320" alt="ad1" src="https://github.com/user-attachments/assets/8793eb13-641d-41af-8777-c54a87868bc4" />
  <img width="512" height="249" alt="ad2" src="https://github.com/user-attachments/assets/b15588cc-27fb-4e3a-b039-e40120b34bec" />
  <img width="512" height="406" alt="ad3" src="https://github.com/user-attachments/assets/3cc0d0bd-70e8-4970-ac7a-daeb5fa62345" />

### 1.7 Remote Desktop Setup

1. **Enable Remote Desktop:** On the Windows 10 machine, navigate to **System Properties** by going to **Windows > PC > Properties > Advanced system settings**. Log in as an administrator, click the **Remote** tab, and select **Allow remote connections to this computer**.
2. **Add Users:** Click **Select Users** to add the domain users who will have remote access. Confirm the changes by clicking **OK** and **Apply**.

  <img width="512" height="404" alt="rds1" src="https://github.com/user-attachments/assets/3b3d0989-3796-405c-babe-c2d36610a534" />
  <img width="479" height="459" alt="rds2" src="https://github.com/user-attachments/assets/61c78b50-8645-4f9d-b6a0-0de04117b6d9" />

## 2. Security Testing with Hydra and Atomic Red Team

### 2.1. Brute Force Attack
1 **Kali Linux Configuration:** Update the package repositories of Kali Linux. Create a project directory
  ```Bash
sudo apt-get update && sudo apt-get upgrade -y
mkdir ad-project
  ```
2 **Brute Force Attack Preparation:** Copy a password wordlist into your project directory. Create a smaller password list by selecting approximately 20 passwords from the main list and adding the passwords for the users you created on the Windows Server
  ```Bash
    cd /usr/share/wordlists/
    sudo gunzip rockyou.txt.gz
    cp rockyou.txt ~/Desktop/ad-project
    cd ~/Desktop/ad-project
    head -n 20 rockyou.txt > passwords.txt
    nano passwords.txt
  ```
  
  <img width="512" height="347" alt="hydra1" src="https://github.com/user-attachments/assets/1eca479b-0c28-4aee-a080-460aa2487b7c" />
    
3 **Execution:** Use **Hydra** to launch a brute force attack against the Windows 10 machine's Remote Desktop Protocol (RDP) service. The command will attempt to authenticate with the specified user and password list. A successful connection will be established upon finding the correct credentials.
  ```Bash
  hydra -l [username] -P [passwordlist] -V rdp://[target_IP]
  ```

<img width="512" height="218" alt="hydra2" src="https://github.com/user-attachments/assets/fa8f5a73-f913-400b-9445-83856a227ebe" />

4 **Splunk Log Analysis:** In Splunk, use a search query like `index=endpoint username="<username>"` to find events. Expand on **Event ID 4625** to see the failed login attempts originating from the Kali machine's IP address.
  - Multiple event code generated for the target user

  <img width="512" height="467" alt="hydralog1" src="https://github.com/user-attachments/assets/7357ee64-a4f7-456f-b8ac-9def7372a242" />
    
  - Eventcode = 4625 shows failed login attempts

  <img width="512" height="381" alt="hydralog2" src="https://github.com/user-attachments/assets/ca248acd-4f58-4e04-9b86-4b5bacf75782" />

  - Eventcode = 4624 shows successful login

  <img width="512" height="384" alt="hydralog3" src="https://github.com/user-attachments/assets/6a085083-80b4-4cdc-8fc1-a787eeff722b" />

  - Here we can see the successful login is from kali linux which is the attacker machine

  <img width="512" height="421" alt="hydralog4" src="https://github.com/user-attachments/assets/11a23359-6340-4ad0-9780-5cc449be213a" />

### 2.2. Atomic Red Team (ART) Execution

1 **Setup:** On the Windows 10 machine, turn off Windows Defender's real-time protection and add an exclusion for the C drive. Open PowerShell as an administrator and set the execution policy to bypass with
  
  ```PowerShell
  set-ExecutionPolicy Bypass -CurrentUser`. Install ART by executing: `IEX(IWR   'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
  Install-AtomicRedTeam -getAtomics
  ```

  <img width="512" height="408" alt="redteam" src="https://github.com/user-attachments/assets/5171e837-7eb7-4ce0-b7ac-095d6cae4f41" />

2 **First Technique: Local Account Creation (T1136.001)**
    - **Simulate an Attack:** The ART files are installed on the C drive in a folder containing various technique IDs (TIDs) that correspond to the MITRE ATT&CK framework. Execute a local account creation attack with the command 
        
  ```PowerShell
  Invoke-AtomicTest T1136.001
  ```

  <img width="512" height="407" alt="t1136" src="https://github.com/user-attachments/assets/9fd38cd7-a7f2-40bd-b10f-f40a3c749f12" />
    
This will create a new local user on the system.
    - **Monitor with Splunk:** Go to Splunk and search for the newly created user (e.g., `newlocaluser`). The events that appear confirm that Splunk successfully detected and logged the attack.

  <img width="512" height="364" alt="t1136_log" src="https://github.com/user-attachments/assets/fe3b5c97-8563-4a5b-b0da-6e1395f5fe78" />


      
3 **Second Technique: PowerShell (T1059.001)**
    - **Execution:** To simulate a PowerShell attack, use the command 
        
  ```PowerShell
  Invoke-AtomicTest T1059.001
  ```
  
  <img width="467" height="512" alt="t1059" src="https://github.com/user-attachments/assets/832753f8-cb7c-4905-848f-baf43d2d5edf" />

   - **Monitor with Splunk:** Go to Splunk and search for PowerShell events to confirm that the scripting activity was successfully detected and logged.

  <img width="512" height="440" alt="t1059_log" src="https://github.com/user-attachments/assets/be09bb4a-0097-44d7-9557-66f7dde633b4" />

### Troubleshooting

Navigating potential issues is a crucial part of a hands-on lab. Here are some common problems you might encounter during this project and how to resolve them:

* **Network Connectivity:** If your VMs cannot communicate, double-check your IP address configuration on each machine. Ensure all VMs are on the same subnet and using the correct gateway. Verify that the VirtualBox NAT network is configured with the correct IP range (e.g., `192.168.10.0/24`) and that all machines are assigned to this network adapter. Pinging between the machines is a good way to test connectivity.
* **Splunk Data Ingestion:** If you are not seeing logs in Splunk, confirm that the Splunk Universal Forwarder is installed and running on your Windows machines. Verify that the `input.conf` file is correctly configured to monitor the necessary event logs, such as `Microsoft-Windows-Sysmon/Operational` and `Security`. Additionally, make sure the `endpoint` index has been created in Splunk to receive the forwarded data.
* **Active Directory Domain Join:** If the Windows 10 machine fails to join the domain, ensure the Windows Server is promoted to a Domain Controller and that the domain name is typed correctly. You must use the administrator credentials for the domain, not a local account, to complete the join process. Also, verify that the Windows 10 machine can resolve the domain controller's IP address.
* **Hydra/Brute Force Attacks:** If the brute force attack with Hydra fails, check that Remote Desktop (RDP) is enabled on the target Windows machine and that the service is running. A successful RDP connection requires that port **3389** is open and reachable on the network. Also, verify that your password list and username are correctly specified in the `hydra` command.

***

### Key Lab Learnings

Completing this project provides a holistic understanding of both offensive and defensive cybersecurity principles, from setup to analysis.

* **Active Directory Fundamentals:** You will gain a practical understanding of how Active Directory functions, including the roles of a Domain Controller, how user accounts are managed, and how machines are joined to a domain.
* **Attacker Perspective:** You will learn to use common offensive tools like **Hydra** to perform brute force attacks and **Atomic Red Team** to simulate known attack techniques. This provides insight into how adversaries operate and the indicators they leave behind.
* **Defender Skills:** The core of the lab is developing defensive capabilities. You will practice configuring endpoint monitoring tools like **Sysmon** to capture detailed event logs and a Security Information and Event Management (SIEM) system like **Splunk** to centrally collect and analyze these logs. This hands-on experience is vital for a career in a Security Operations Center (SOC).
* **Log Analysis and Threat Detection:** A key takeaway is the ability to use log data to identify malicious activity. By correlating events in Splunk, you can trace an attack from initial login attempts to the execution of specific commands or scripts, effectively recreating the attack chain. You will be able to recognize failed and successful login attempts (Event ID **4625** and **4624**) from the log data.
* **System Administration and Automation:** This project reinforces system administration skills on multiple operating systems, including network configuration and file transfer. You will also use scripting (e.g., PowerShell and Bash) to automate tasks, a critical skill for both security and IT professionals.

## References

1. [Active Directory Part 1 – MyDFIR](https://www.youtube.com/watch?v=mWqYyl89QaY&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=15&ab_channel=MyDFIR)
2. [Active Directory Part 2 – MyDFIR](https://www.youtube.com/watch?v=2cEj3bS5C0Q&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=16&ab_channel=MyDFIR)
3. [Active Directory Part 3 – MyDFIR](https://www.youtube.com/watch?v=uXRxoPKX65Q&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=17&ab_channel=MyDFIR)
4. [Active Directory Part 4 – MyDFIR](https://www.youtube.com/watch?v=1XeDht_B-bA&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=18&ab_channel=MyDFIR)
5. [Active Directory Part 5 – MyDFIR](https://www.youtube.com/watch?v=orq-OPIdV9M&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=19&ab_channel=MyDFIR)
