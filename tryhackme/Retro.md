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

I'll try to find hidden directories on this web with gobuster tool:
```bash
gobuster dir -u http://10.10.254.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt > dirs
```

This scan found the directory "/retro".
First thing to see it is a WordPress web, it can be noticed with wappalyzer extension.

![image](https://user-images.githubusercontent.com/114166939/229808095-c74ae384-0c2c-43d5-baa5-8b59b6492935.png)

Also, I'll run another brute-force directory to see if there are another hidden directories:
```bash
gobuster dir -u http://10.10.254.2/retro/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt > RetroDirs```
```
Results:
```
/wp-includes
/wp-admin	
/wp-content
```



