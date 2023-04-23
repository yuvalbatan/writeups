# Inject - HTB
### Difficulty level: Easy
------------------------------------------------------

I'll start with nmap scan:
```bash
nmap -sV -sC 10.10.11.204 | tee nmap/first_scan
```

Results:
```bash
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 caf10c515a596277f0a80c5c7c8ddaf8 (RSA)
|   256 d51c81c97b076b1cc1b429254b52219f (ECDSA)
|_  256 db1d8ceb9472b0d3ed44b96c93a7f91d (ED25519)
8080/tcp open  nagios-nsca Nagios NSCA
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

This is a linux machine, there are 2 open services - web app on 8080 and ssh on 22.

I'll start with the web app service on port 8080 by testing its directories:
```bash
gobuster dir -u http://10.10.11.204:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee gobuster/dir_scan
```

Results:
```bash
/register             (Status: 200) [Size: 5654]
/blogs                (Status: 200) [Size: 5371]
/upload               (Status: 200) [Size: 1857]
/environment          (Status: 500) [Size: 712]
/error                (Status: 500) [Size: 106]
/release_notes        (Status: 200) [Size: 1086]
```

The most interesting is "/upload" directory, let's open it:

![image](https://user-images.githubusercontent.com/114166939/233862706-dbd2a3ce-0356-4cb9-b0c2-990778ce6a91.png)

I'll make an empty PNG file (so it won't take much time to upload it) with the command:
```bash
touch empty.png
```
Also, I'll power-up Foxy-Proxy tool:

![image](https://user-images.githubusercontent.com/114166939/233863036-986a4feb-828f-4ebf-a77f-57eb4a614e9b.png)

And open interception on Burp-Suite to examine the request:

![image](https://user-images.githubusercontent.com/114166939/233863114-5ca16b2a-c330-4719-bf61-3ae0a0fac1d4.png)

Click on upload:

![image](https://user-images.githubusercontent.com/114166939/233863161-33199fc6-50c1-40b9-a454-2f9e376e89b5.png)

Caught the request! Now I'll send it to the repeater with ctrl + R,

Because the image is empty added some strings,

Click on send and the image uploaded:

![image](https://user-images.githubusercontent.com/114166939/233863419-2691fb6b-2bf0-4a95-832c-14830aaf7678.png)


On the response there is the html code:
```html
<a class="text-success" href="/show_image?img=empty.png">View your Image</a>
```

Let's try show the image with the repeater:




