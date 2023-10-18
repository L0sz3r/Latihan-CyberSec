# WriteUp of Kuya Machine VulnHub

You can download the file ova in <a href=https://www.vulnhub.com/entry/kuya-1,283/> here </a>

# Kuya Machine (Solved)

First, open the ovf file of target machine using VirtualBox and setting network of target machine using host-only adapter. After that, use <strong> sudo arp-scan -l </strong> command in attacker machine to find the ip of target machine.

Next, use nmap to scan the port using this command : nmap -A -sV -sC -p- (ip), i used this command to scan all port with script engine default by nmap and scan the version of all port in target machine. However, i found 2 port (22/ssh and 80/http)

<img src=images/nmap.png>

Let's go to Kuya Website (port 80/http) using browser. After opened i found the website with welcome message and trools ( i dont know how to meaning that). I use gobuster to find the hidden directory in website of target machine and seclists wordlist. Waited quite a long time, i found a suspicious 2 directory of website named /wordpress and /loot

<img src=images/gobuster.png>

In wordpress directory is not working because the php and mysql not installed in target-machine. But in loot directory, i found 5 images with .jpg file format. I think the 5 images should be interesting file or message, so i use strings, exiftool and steghide for the 5 images to find that. The 1.jpeg file embedded file named secret.txt with base64 encoding. After decode that, the message just scammer.

<img src=images/loot1-image.png>

Using the same method for 2.jpg file, i found embedded file named emb.txt with brainfuck cipher. After decode the message i found the flag 1 of Kuya Machine

<img src=images/brainfuck.png>

Now, use the same method for 3.jpg, 4.jpg and images.jpg, i found file pcapng in embedded of 4.jpg but in 3.jpg and images.jpg not interesting. So i opened the pcapng file using wireshark and find the file in http traffic named loot.7z.

<img src=images/wireshark.png>

But the 7z file is locked, i use john and wordlist rockyou to crack the 7z file and find the password, manchester. After that, unlocked the 7z file with cracked password.

<img src=images/john-7z.png>

Open the folder of loot.7z file and find the rsa private key and rsa public key of ssh target machine. I use ssh2john to crack the private key and work it, the passphrase of rsa private key ssh is <strong> hello </strong>. After that, change the permission of rsa private key ssh using command : <strong> chmod 600 id_rsa </strong> and access the ssh machine using test user (read rsa public key ssh) and rsa private key with passphrase hello with command : <strong> ssh -i id_rsa test@ipofmachine -p 55001 </strong> .

<img src=images/ssh-john.png>

After access the shell of target machine, read the file named sshscript.sh to get the flag 2. After that, go to /var/www/html/wordpress (check gobuster tagged) and read the file named wordpress-config-example.php to get the credential of kuya user (to find the kuya user, you can check in /home directory or read file /etc/passwd).

<img src=images/wordpress.png>

Now, change the user from test to kuya. After that go to /home/kuya and read who_is.txt and .bash_history, the interesting about 2 file is password IL0v3C@f3HaV@nA and getcap to read /etc/shadow (exactly, file /etc/shadow only read for root user) and the tar file. I search getcap privellege escalation in google and find this <a href=https://nxnjz.net/2018/08/an-interesting-privilege-escalation-vector-getcap/> blog. </a> I try the tutorial and its working to import the /root directory to /home/kuya directory.

<img src=images/root.png>

Thank You for another writeup i search and my learning about this machine : https://www.hackingarticles.in/vulnhub-kuya-1-walkthrough/
