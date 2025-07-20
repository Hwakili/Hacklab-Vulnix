VULNHUB: VULNIX 

<img width="1578" height="937" alt="Screenshot 2025-07-20 151742" src="https://github.com/user-attachments/assets/916daa27-adf7-4e47-be22-3f02533af4c8" />

Objective

The goal of this challenge was to conduct a comprehensive penetration test against the Vulnix vulnerable machine available on VulnHub. This process included discovering open ports and running services, taking advantage of poorly configured network services, and ultimately escalating privileges to achieve root access. The purpose of the exercise was to replicate real-world scenarios involving enumeration and exploitation on a Linux environment, while strengthening hands-on skills in reconnaissance, service enumeration, privilege escalation, and post-exploitation activities.
The objective of HackLAB: Vulnix on VulnHub is straightforward yet classic: it's a "boot‑to‑root" virtual machine. The  mission:
Boot up the Ubuntu 12.04‑based VM.


Discover its IP on your network.


Explore and exploit misconfigurations—there are no intentionally vulnerable software versions, but plenty of configuration flaws.


Gain an initial foothold (commonly via NFS‑exported /home/vulnix, accessible by aligning UIDs, mounting it, dropping your SSH key, and logging in as the vulnix user).


Privilege escalate to root, often by modifying /etc/exports, remounting root’s directory, and planting another SSH key.


Finally, capture the flag: the trophy file hidden in /root.

Tools Used 

1. Kali Linux Terminal 
2. Nmap 
3. Metasploit Framework 
4. Showmount/Rpcinfo
5. SSH 
6. Hydra 
7. Linux Privilege Escalation Scripts


Attack Summary

Note: Before you start attacking this machine, configure the network setting for both the machines to be the same. 

Discover IP address of machine using ifconfig


<img width="641" height="433" alt="Screenshot 2025-06-04 164747" src="https://github.com/user-attachments/assets/be3b8d45-567c-433c-87b0-df513556f273" />

Enumeration

Network Scanning (Nmap) 

  1. Used nmap scan (nmap -sP) to perform a Ping Scan with Nmap. This scan discovered the  live hosts on the network without conducting a full port scan.


   <img width="692" height="641" alt="Screenshot 2025-06-04 170726" src="https://github.com/user-attachments/assets/199575f0-a96e-4f31-9d07-9292cfbe6a1b" />

 As seen from the result, it is identified that the IP address for the running VirtualBox which is the vulnix machine is 192.168.1.65 


  2. Used Nmap scan to find open ports and services running on 192.168.1.65 
  
  <img width="729" height="742" alt="Screenshot 2025-06-04 170747" src="https://github.com/user-attachments/assets/d6c7e191-0f7c-48d5-8411-9682edec1917" />

The Nmap scan reveals numerous open ports and active services on the target machine.

For instance, port 22 is open, indicating that SSH is available — potentially allowing direct login or a brute-force attack to uncover valid credentials.
Additionally, port 25 is open and running the Postfix SMTP (Simple Mail Transfer Protocol) daemon. This service supports commands like VRFY, which can be used to interact with the mail server and check the validity of specific user data. We'll explore this further during the information-gathering stage

SMTP Enumeration (Metasploit) 

1. Used Metasploit to exploit weaknesses in the SMTP service, enabling the discovery of valid system users through enumeration. In certain cases, this vulnerability could also be escalated to achieve remote code execution.

<img width="936" height="759" alt="Screenshot 2025-06-04 172335" src="https://github.com/user-attachments/assets/c674e489-7fa5-47da-b047-aad807490663" />

2. Finger Exploitation: Searched for in msfconsole using “searh smtp” and used Auxilary Module for exploit

   This enumeration finds a list of users that exists in the mail server.

   <img width="942" height="799" alt="Screenshot 2025-06-04 172930" src="https://github.com/user-attachments/assets/c1412f54-a042-4f53-8bfc-8c7c71fc147c" />
   <img width="951" height="386" alt="Screenshot 2025-06-04 173848" src="https://github.com/user-attachments/assets/675a88ee-76ce-4588-bba4-044718287443" />
   <img width="724" height="69" alt="Screenshot 2025-06-04 174856" src="https://github.com/user-attachments/assets/1063a478-2dff-4679-abef-d0aef2363bc3" />
   <img width="940" height="170" alt="Screenshot 2025-06-04 175644" src="https://github.com/user-attachments/assets/3d970f77-b635-4e74-ab17-1d67c024247e" />
   <img width="946" height="534" alt="Screenshot 2025-06-04 181102" src="https://github.com/user-attachments/assets/c39e1df5-1c59-4e11-893e-677637318cb4" />
   <img width="923" height="881" alt="Screenshot 2025-06-04 181049" src="https://github.com/user-attachments/assets/ff55da13-805d-48b2-bcf4-e94c1e115086" />
   <img width="945" height="324" alt="Screenshot 2025-06-04 181755" src="https://github.com/user-attachments/assets/11ce8112-35f5-4311-a097-8568c2972d1e" />
   <img width="950" height="577" alt="Screenshot 2025-06-04 182732" src="https://github.com/user-attachments/assets/580bb43d-d461-4bd4-94e7-e61735bb56a9" />

Successfully retrieved list of users after using metaploit auxiliary module (Finger service enumeration) on the attack IP Address 
Save all users to a file (usernames.txt) 


Brute Force Attack (Hydra) 

Performed a brute-force attack on the SSH service with Hydra, utilizing the previously gathered list of usernames to attempt authentication and gain access.

<img width="621" height="387" alt="Screenshot 2025-06-04 192424" src="https://github.com/user-attachments/assets/620bcbdf-ffcd-49e1-94dd-e62c35c23d71" />

NFS Enumeration and Mounting Step:

After successfully performing the brute-force attack on SSH and gathering valid credentials, the next phase involved enumerating the NFS (Network File System) shares exposed by the target machine at 192.168.1.65. Using the showmount -e command, it was identified that /home/vulnix was available for mounting.

Attempts were made to create a local directory /mnt/vulnix for mounting the share. Although the directory already existed, the mounting process was retried using the mount command. After correcting the syntax and removing the unnecessary options, the remote NFS share was successfully mounted locally.

This allowed access to the /home/vulnix directory from the attacking machine, paving the way for further exploitation, such as adding an SSH key for persistent access or exploring sensitive files for privilege escalation opportunities.

<img width="560" height="417" alt="Screenshot 2025-06-04 200450" src="https://github.com/user-attachments/assets/d6d6f640-995e-4d2f-b884-43376d15e55c" />

After successfully mounting the NFS share to /mnt/vulnix, the contents of the vulnix user's home directory were examined. Standard user files such as .bashrc, .profile, and .bash_logout were observed, confirming access to the user's environment.

An initial attempt was made to switch to the vulnix user locally using su vulnix, but this failed due to missing or invalid credentials.

To prepare for persistence and establish future access, the .ssh directory within /mnt/vulnix/ was removed to allow the attacker to place a new authorized SSH key later. This action ensures that once a new SSH key is placed, the attacker will be able to access the target system directly as the vulnix user without further brute-force attempts.

This step sets up the groundwork for achieving persistent SSH access, which is a common technique in post-exploitation scenarios to maintain a foothold on the compromised system.

<img width="606" height="580" alt="Screenshot 2025-06-04 204735" src="https://github.com/user-attachments/assets/dd7a2c3e-1ade-4f9d-98e8-7e843aa07ebf" />

Shell Access (SSH)

Configured SSH key-based authentication to enable passwordless access to the vulnix account. The public SSH key (id_rsa.pub) was copied into the /mnt/vulnix/.ssh/authorised_keys file, allowing the system to recognize the key during authentication. After copying the key, verified its successful placement by checking the contents of both the original public key and the authorised_keys file. The setup was completed correctly, facilitating secure and seamless SSH access to the vulnix account.

Logged into the system via SSH with valid credentials obtained through brute-forcing.

<img width="626" height="438" alt="Screenshot 2025-06-04 204944" src="https://github.com/user-attachments/assets/098ff762-c008-437c-95dc-c3c60ee49aab" />
<img width="492" height="150" alt="Screenshot 2025-06-04 205041" src="https://github.com/user-attachments/assets/c4cf61a8-dd6c-41e8-ae99-1f18049bca34" />
<img width="681" height="543" alt="Screenshot 2025-06-04 211000" src="https://github.com/user-attachments/assets/d0676533-04d2-4c24-a023-b99632f7eaee" />

Now that I had remote write access as vulnix, I created a new SSH key pair, and copied the public key into .ssh/authorized_keys, which then allowed me to SSH in to the box as vulnix

<img width="663" height="430" alt="Screenshot 2025-06-04 221224" src="https://github.com/user-attachments/assets/f645944f-533f-41c3-99fa-3d01dd61a7f6" />

Root Access

Upon examining the sudo privileges for the vulnix user, it was evident that they could edit the NFS exports file without requiring a password. By leveraging sudoedit /etc/exports, the user could add a new export entry that includes the no_root_squash option, allowing root users to retain their privileges instead of being mapped to the nobody user.

<img width="795" height="732" alt="Screenshot 2025-06-04 224027" src="https://github.com/user-attachments/assets/bdc3e9b1-5684-4718-b19e-d13f3be76448" />

After rebooting the VM, the new share into the /root directory can be seen:

<img width="602" height="406" alt="Screenshot 2025-06-04 230938" src="https://github.com/user-attachments/assets/76b8de66-4764-453b-af54-e7f444a63752" />

Following the same steps as before, it is now possible to add an SSH key into /root/.ssh/authorized_keys and gain root access:

<img width="597" height="442" alt="Screenshot 2025-06-04 235808" src="https://github.com/user-attachments/assets/83b1b207-0008-4432-a329-045e2931a4ee" />

<img width="730" height="751" alt="Screenshot 2025-06-05 000810" src="https://github.com/user-attachments/assets/4e560830-8fdf-4288-8c68-f62aa9d50078" />

Captured the Flag

Gained root access and captured the final flag, completing the challenge.
<img width="513" height="64" alt="Screenshot 2025-06-05 000826" src="https://github.com/user-attachments/assets/012b138f-c07a-46d9-9347-86a343a7a53f" />

Screenshot unavailable but after gaining access and seeing the files in the Vulnix box, the cat trophy.txt command was ran and it should return the value of the txt file which is the flag

root@vulnix:~# cat trophy.txt
cc614640424f5bd60ce5d5264899c3be

To confirm the id and the root access I ran the id in the vulnix box 

<img width="381" height="105" alt="Screenshot 2025-06-05 000842" src="https://github.com/user-attachments/assets/e70119f5-bc91-4d31-8e93-401e760fc9ba" />

The command indicates that the current user is operating as the root user. Here's what each part means:
uid=0(root): The user ID is 0, which is always assigned to the root (administrator) account in Unix/Linux systems.
gid=0(root): The group ID is also 0, meaning the user belongs to the root group.
groups=0(root): Confirms that the user is part of the root group.

This output verifies that the user has full administrative privileges, granting unrestricted access to all files, commands, and system resources on the machine.

Conclusion

This Vulnix challenge provided a comprehensive and practical opportunity to simulate a real-world penetration test within a controlled environment. Starting with network reconnaissance and service enumeration, the process involved identifying vulnerable configurations, particularly within the NFS and SSH services. Through careful analysis and methodical exploitation, access was initially gained to the vulnix user account. By leveraging weak sudo configurations, it was then possible to escalate privileges and obtain full root access. The final goal, retrieving the trophy.txt flag was successfully achieved, confirming control over the target system. This exercise reinforced key offensive security skills such as manual exploitation, privilege escalation, and CTF methodology, while deepening familiarity with Linux environments and misconfiguration-based attack vectors.



