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
For beginning I'll try the simplest in the title:
```html
<script>alert(1)</script>
```
Didn't work:

![image](https://user-images.githubusercontent.com/114166939/232100940-7a66c85d-2dbd-4908-b04c-aad6c5327ceb.png)

Let's try the description:

![image](https://user-images.githubusercontent.com/114166939/232101091-70b6a894-ece1-420a-a82d-63718944d837.png)

It works!

![image](https://user-images.githubusercontent.com/114166939/232101158-1cf33e18-0bbb-4ecc-bc20-45d2601fb543.png)

Also when I'm going to the home page and clicking on this listing the alert pops out.
It is a reflected and stored xss!

The 2 options for this listing are: "Contact the listing author" and 
"Report listing to admins" - when I report the listing 
