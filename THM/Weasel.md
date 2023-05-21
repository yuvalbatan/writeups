# Weasel
### Difficulty: Medium
------------------------------------------------------

Let's run nmap scan for start:
```bash
nmap -sV -sC 10.10.213.247
```

Results:
```bash
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 2b17d88a1e8c99bc5bf53d0a5eff5e5e (RSA)
|   256 3cc0fdb5c157ab75ac8110aee298120d (ECDSA)
|_  256 e9f030bee6cfeffe2d1421a0ac457b70 (ED25519)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: DEV-DATASCI-JUP
|   NetBIOS_Domain_Name: DEV-DATASCI-JUP
|   NetBIOS_Computer_Name: DEV-DATASCI-JUP
|   DNS_Domain_Name: DEV-DATASCI-JUP
|   DNS_Computer_Name: DEV-DATASCI-JUP
|   Product_Version: 10.0.17763
|_  System_Time: 2023-05-21T13:30:10+00:00
| ssl-cert: Subject: commonName=DEV-DATASCI-JUP
| Not valid before: 2023-03-12T11:46:50
|_Not valid after:  2023-09-11T11:46:50
|_ssl-date: 2023-05-21T13:30:17+00:00; +1s from scanner time.
8888/tcp open  http          Tornado httpd 6.0.3
| http-title: Jupyter Notebook
|_Requested resource was /login?next=%2Ftree%3F
| http-robots.txt: 1 disallowed entry 
|_/ 
|_http-server-header: TornadoServer/6.0.3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-05-21T13:30:10
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
```

This machine runs Windows operating system, and the main open services I'll target are probably SMB (445), HTTP (8888) and maybe SSH (22) and RDP (3389).

I'll start with the SMB service - 

Let's use the "crackmapexec" tool to get some information about the shares in there.

```bash
crackmapexec smb 10.10.213.247 -u 'UserNotExist' -p '' --shares
```

Found the following shares:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/2c917ef6-c022-4fb2-9826-3852aa14bb15)

Let's check what's inside "datasci-team" share with "SMBclient" tool:
```bash
smbclient -N //10.10.213.247/datasci-team
```

There are many files and directories here.

![image](https://github.com/yuvalbatan/writeups/assets/114166939/49f377f8-3f76-4975-8243-1a3f3c82c0f6)

I'll enter the "misc" directory:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/27e3d079-c96b-4dd2-9c80-3a16fd697494)

And download the "jupyter-token.txt" file to my computer with the command:
```bash
get jupyter-token.txt
```

Besides this file, there aren't any other interesting ones.

Let's examine this file on my machine:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/6d8d823a-81e0-426d-8e60-54b57f5cf864)


The file appears to only contain a token, which we will use soon, so I'll save it and move forward to the HTTP service.

In my browser, if I enter "10.10.213.247:8888", it redirects me to this page:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/ed14e1fe-e53d-4c8d-9c76-cf4715a656b4)

In here - I can login with password or token. I'll use the token I found in the SMB service.



It works!

![image](https://github.com/yuvalbatan/writeups/assets/114166939/b6b3a4f3-3ea1-4f62-85ab-822fb06c8256)

"Jupyter" is an open-source web application notebook environment for interactive computing.

In the "New" menu there is an option to open a terminal:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/1af72d79-485a-4cb7-beb2-ef7d6cf2b9cf)

Click on it - and you'll get a terminal for the system!

This is very weird because in the nmap scan all the services indicated we have a deal with Windows OS.

Also that's what the terminal displays:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/c62619f3-8026-46d0-b07a-1ac6a6f904af)


It says "GNU/Linux 4.4.0-17763-Microsoft x86_64".

Quick search on Google and a bit of locig tells me that we are dealing with a WSL (Windows Subsystem for Linux), which means that on the Windows OS there is an option to run Linux command line tools and more.


Let's investigate the user we got:

The user is "dev-datasci", it is in the "sudo" group!

I'll check its permission:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/94ff72bc-ee1b-4d8d-9967-786ce93b2719)

Let's see what is the first file we can run with sudo ("/home/dev-datasci/.local/bin/jupyter"):
```bash
cat /home/dev-datasci/.local/bin/jupyter
```

![image](https://github.com/yuvalbatan/writeups/assets/114166939/c57ac311-7848-49c7-9bdc-7b4a1255334e)

There is an error that says there is no file called "jupyter" in this location.

I'll use it for privilege escalation:

First, I'll copy "/bin/bash" to this location and change its name to "jupyter":
```bash
cp /bin/bash /home/dev-datasci/.local/bin/jupyter
```

Next, I'll run this as root with the sudo command:
```bash
sudo /home/dev-datasci/.local/bin/jupyter
```

![image](https://github.com/yuvalbatan/writeups/assets/114166939/277d0d90-15c7-4dec-9a28-b49f1035cbee)

Got the root user!

Let's move forward and explore how to get the files and information from the windows os.

I'll enter the "/mnt" directory:
```bash
cd /mnt && ls -la
```

There is nothing except for this directory:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/155b1ace-47ae-4a0e-a794-709c3217cbbc)

There is nothing in the directory either.

Let's find a way to mount this "c" drive from the WSL.

I found this article "https://www.public-health.uiowa.edu/it/support/kb48568/" - which contains the following command:
```bash
mount -t drvfs M: /mnt/m
```

I'll modify it for my own meeds and run it:
```bash
mount -t drvfs C: /mnt/c
```

Enter the "c" directory again and you will find many files which look like from windows file system!

![image](https://github.com/yuvalbatan/writeups/assets/114166939/58c2ad68-9628-4abc-b841-caa628103f0d)

The "user.txt" file should be on the regular user directory in the "Desktop":
```bash
cd /mnt/c/Users/dev-datasci-lowpriv/Desktop
```

![image](https://github.com/yuvalbatan/writeups/assets/114166939/9e057c2c-9ad4-4d40-8852-9990284c16b7)

Got the first flag!

The next flag is the "root.txt" file located in the "Administrator" directory on the "Desktop":
```bash
cd /mnt/c/Users/Administrator/Desktop
```

![image](https://github.com/yuvalbatan/writeups/assets/114166939/7a275d77-d1aa-4f0b-9616-9c0d5fd72d6c)
