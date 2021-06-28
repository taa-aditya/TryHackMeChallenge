
# Bounty Hunter

You were boasting on and on about your elite hacker skills in the bar and a few Bounty Hunters decided they'd take you up on claims! Prove your status is more than just a few glasses at the bar. I sense bell peppers & beef in your future!

## Task 1
**Q1 - No answer required.**  
On visiting the IP address of the machine, we are greeted with this page.  
![firstpage](https://user-images.githubusercontent.com/74425235/123645180-ba9a1280-d843-11eb-814d-a0d87c100dc8.png)

Checking page source gives no clue.  
![sourcecode](https://user-images.githubusercontent.com/74425235/123645211-c4237a80-d843-11eb-9b6d-56d402d17f5d.png)

**Q2 : Find open ports on the machine.**  
For this, I’ll use the nmap command.
```bash
nmap -sV -vv -T4 10.10.81.42
```
-sV probes the open port to determine service/version info.  
-vv gives an increased verbose output (displays the steps it performs on the screen)  
-T sets the timing template, it can range from 0 - 5, 5 being the highest.  
Result of the nmap scan:  
![nmapscan](https://user-images.githubusercontent.com/74425235/123645334-e0bfb280-d843-11eb-9112-1eaa2ad72fc8.png)

We can see that there are 3 ports open:  
* Port 21 : ftp version 3.0.3  
* Port 22 : ssh version 7.2p2  
* Port 80 : http, apache version 2.4.18 - This shows that there is a web server running on the machine.

**Q3 : Who wrote the task list?**  
Now let’s try connecting to the ftp server.
On connecting, we get a prompt to enter the username. I left it blank and pressed enter.
Received the following message:  
![foundanon](https://user-images.githubusercontent.com/74425235/123645425-f6cd7300-d843-11eb-94aa-26544fd4c4c7.png)

This shows that it supports anonymous login, let’s try that out.  
Once the login is successful, use the `ls -al` command to view all the files including the hidden files in the directory. We find two files, **task.txt** and **locks.txt.**  
I copy the files to my local directory using the `lcd` command to set the download directory and then the `get` command with the filename.  
![copyftp](https://user-images.githubusercontent.com/74425235/123645511-0947ac80-d844-11eb-8e0e-8fb112dca10a.png)

Checking the contents of the **task.txt** file gives us the answer.  
![username](https://user-images.githubusercontent.com/74425235/123645818-4b70ee00-d844-11eb-91b7-9c12f2553674.png)

**Q4 : What service can you bruteforce with the text file found? & Q5 : What is the users password?**    
Next question asks us which service we can bruteforce with the text file found. This refers to the **locks.txt** file which looks like a wordlist of passwords.
We use this list and try to bruteforce our way into the SSH server.
To do so, we use the hydra command.  
`hydra -l <username> -P locks.txt ssh://<Machine_IP>`  
-l uses the specified login name.  
-P uses the wordlist provided.  
![brutessh](https://user-images.githubusercontent.com/74425235/123645865-56c41980-d844-11eb-9b11-a0fc14e8277e.png)

**Q6 : user.txt**  
Using the credentials we login to the SSH server.  
Once we are in, use the same command `ls -al` to view the files. We find the **user.txt** file.  
`cat user.txt` gives us the flag.  
![userflag](https://user-images.githubusercontent.com/74425235/123645765-3d22d200-d844-11eb-9ece-fb375fa5c111.png)

**Q7 : root.txt**  
Assuming the root.txt will be placed in the root folder, I try to access it.  
![permdeny](https://user-images.githubusercontent.com/74425235/123645938-63e10880-d844-11eb-9dfa-a3dd388912a0.png) but permission is denied.

Now, I use the “sudo -l” command to check for sudo commands which can be used by our user.  
![sudol](https://user-images.githubusercontent.com/74425235/123645985-6d6a7080-d844-11eb-8f24-2cef3330e4d3.png)

Notice that the root user can use the “/bin/tar” command. Interesting, let’s check if we can use this for privilege escalation on GTFObins.  
YES! There is a way we can use this for our benefit.  
![gtfobins](https://user-images.githubusercontent.com/74425235/123646017-78bd9c00-d844-11eb-9f67-717e60c36350.png)

I use the command  
`sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`
Using  the `id` command, we see that now we are root!  
I locate the **root.txt** file and we get our flag.  
![rootflag](https://user-images.githubusercontent.com/74425235/123646056-81ae6d80-d844-11eb-9b95-6332c35acabd.png)

# References:  
- https://nmap.org/book/man-briefoptions.html
- https://www.howtoforge.com/tutorial/how-to-use-ftp-on-the-linux-shell/
- https://gtfobins.github.io/gtfobins/tar/
