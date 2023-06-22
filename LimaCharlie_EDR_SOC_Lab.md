## LimaCharlie EDR SOC Lab
(use this section with a picture or an introduction to the topic)

## Introduction
In this project, based on the excellent articles from (Eric Capuano), I will simulate real-life cyber attacks on a victim VM, monitor the traffic and activity & finally look to take containment and remediation actions to ensure the attacks are halted. The environment required for this simulation was a Windows 11 machine (Victim) and Ubuntu Server 22.04 (Attack) configured on VMware Workstation 11. 

The Windows 11 machine was configured with Sysmon to monitor and log system activity. This was then filtered into LimaCharlie EDR with their sensor installed on the machine, which allowed visualisation of the simulated attacks through their dashboard (and later on containment and remediation but we'll get onto that). 

The Ubuntu Server, used as the attack machine, would have Sliver-Server C2 installed, which is an 'open source cross-platform adversary emulation / red-team framework' which would allow me to conduct specific activities common with cybercriminals. This activity would specifically look into Credential Gathering techniques, aiming to gather the usernames and passwords in LSASS.exe.     

I will look to conduct a C2 attack against the victim machine, check through the telemetry to confirm the presence of the attack, and then finally look to create 'Detection & Response' rules.

I'd like to take the time now to credit (Eric Capuano) for writing this excellent blog post which was followed for this project. This project was only used for personal educational purposes. 

## PART 1
The first step was to set up the two virtual environments required. Having already installed and used VMware Workstation 11 for previous labs I could import the required .ISO & .OVF files. The Windows machine would be pre-configured so we can head straight over to our Ubuntu Server, which would take some configuration. The below screenshot shows the static IP address set for Ubuntu Server. This was configured by taking the gateway IP & subnet IP of the VMware Workstation NAT network. By executing this we ensure that the IP will remain the same throughout the lab as we will be required to SSH into the Ubuntu Server at a later date. 

<img width="663" alt="Screenshot 2023-06-08 055214" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/e7986d80-e1f2-423d-b039-ce8114ad0ddf">

Now that we have both machines up and running with our specific configurations we'll jump over to the Windows machine to disable some features in Windows Defender permanently, to allow us to conduct our credential-gathering with Sliver-Server later on. First of all, head over to Windows Security and disable all settings under 'Virus & Threat Protection' which includes Real-time Protection, Cloud-delivered Protection, Automatic Sample Submission and Tamper Protection (when turned on, prevents others from tampering with important security features, highlighted in the screenshot below)

<img width="423" alt="Screenshot 2023-06-08 162738" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/1db37d05-744c-4601-95ba-7c302edddf50">

Next, we'll head over to Group Policy Editor and permanently disable Defender by turning off Microsoft Defender Antivirus. After, we'll look to achieve the same goal in Registry via an administrative command prompt (screenshot below)

<img width="462" alt="Screenshot 2023-06-08 164023" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/65d9b7a9-838c-4bf7-a6b9-284d952516db">

Now, we'll Safe Boot (minimal) our Windows machine and when restarted we'll disable some more specific services via the Registry. We'll make our way into Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services to change the value to 4 (Log to a database disabled) of 6 keys including; Sense, WdBoot, WinDefend, WdNisDrv, WinNisSvc and WdFilter. (screenshot below). The final configuration change we'll make is to prevent the Windows machine from going to standby via our administrative command prompt. 

<img width="521" alt="Screenshot 2023-06-08 164857" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/650bdd0c-1c52-4c6c-a764-c8ad0ac18745">

Now, with the Windows machine finely tuned and ready to go, we're going to install Sysmon which gives us more granular telemetry from the Windows endpoint than just LimaCharlie EDR would alone. We'll head over to an administrative Powershell, download and unzip Sysmon and then we'll look to use SwiftOnSecuirty's Sysmon Config [insert link], which the author describes as a 'Microsoft Sysinternals Sysmon configuration file template with high-quality event tracing'. This will help with the telemetry feeding into LimaCharlie. After a quick validation that the service is installed and running, and for the presence of Sysmon Event Logs we can move our attention on to the installation and configuration of LimaCharlie EDR. 


<img width="580" alt="Screenshot 2023-06-08 170336" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/77207bd3-d0a7-4edf-82c5-575c01f02444">

LimaCharlie EDR [possibly a link] is a SIaaS product with free and paid tiers is 'cybersecurity middleware that gives you full control and visibility over your security posture' the company states on their homepage. Whilst logged on the Windows machine and after a simple signup process, we'll look to install their sensor on the machine. Instead of following the step-by-step installer, we'll skip step two, head over to an Administrative Powershell  'cd' into the Downloads file and run 'Invoke-WebRequest -Uri [https://downloads.limacharlie.io/sensor/windows/64] -Outfile C:\Users\User\Downloads\lc_sensor.exe'. After we can fire up a command prompt and start at step 4 which gives us a specific command to copy and paste. This should complete the installation of the LimaCharlie sensor validated on the command line and in the LimaCharlie GUI. (See screenshot below)

<img width="577" alt="Screenshot 2023-06-08 172520" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/d99b3d7b-d0b9-44be-b264-b42fbdbe4746">

Next, we'll have to configure LimaCharlie to import the Sysmon Event Logs parallel to its native telemetry. This is done via the 'Artifact Collection' tab within LimaCharlie with a small amount of configuration. 

<img width="573" alt="Screenshot 2023-06-09 071924" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/d8372acb-12e1-4777-851e-261bc0478fc6">

(Set up attack system)



