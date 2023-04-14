# The Marketplace
## Difficulty: Medium

Let's start with nmap scan:
```bash
nmap -sV -sC 10.10.3.15 | tee nmap/start
```

The results:
```bash
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c83cc56265eb7f5d9224e93b11b523b9 (RSA)
|   256 06b799940b091439e17fbfc75f99d39f (ECDSA)
|_  256 0a75bea260c62b8adf4f457161ab60b7 (ED25519)
80/tcp    open  http    nginx 1.19.2
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: The Marketplace
|_http-server-header: nginx/1.19.2
32768/tcp open  http    Node.js (Express middleware)
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: The Marketplace
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
The SSH version is high enough to not be vulnerable, so I'll start with the http service.
