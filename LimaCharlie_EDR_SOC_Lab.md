## LimaCharlie EDR SOC Lab
(use this section with a picture or an introduction to the topic)

## Introduction
In this project, based on the excellent articles from (Eric Capuano), I will simulate real-life cyber attacks on a victim VM, monitor the traffic and activity & finally look to take containment and remediation actions to ensure the attacks are halted. The environment required for this simulation was a Windows 11 machine (Victim) and Ubuntu Server 22.04 (Attack) configured on VMware Workstation 11. 

The Windows 11 machine was configured with Sysmon to monitor and log system activity. This was then filtered into LimaCharlie EDR with their sensor installed on the machine, which allowed visualisation of the simulated attacks through their dashboard (and later on containment and remediation but we'll get onto that). 

The Ubuntu Server, used as the attack machine, would have Sliver-Server C2 installed, which is an 'open source cross-platform adversary emulation / red-team framework' which would allow me to conduct specific activities common with cybercriminals. This activity would specifically look into Credential Gathering techniques, aiming to gather the usernames and passwords in LSASS.exe.     

I will look to conduct a C2 attack against the victim machine, check through the telemetry to confirm the presence of the attack, and then finally look to create 'Detection & Response' rules.

I'd like to take the time now to credit (Eric Capuano) for writing this excellent blog post which was followed for this project. This project was only used for personal educational purposes. 

## PART 1
The first step was to set up the two virtual environments required. Having already installed and used VMware Workstation 11 for previous labs I could import the required .ISO & .OVF files. The Windows machine would be pre-configured so this would load up and start by itself with no input but the Ubuntu Server would take some configuration. The two major points that needed addressing were 1) setting the static IP address for the machine 2) Installing OpenSSH. 

[THIS NOW A COULD POINT TO PUT SOME PICTURES IN]
