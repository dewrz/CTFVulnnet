# CTF: VulnNet

In this CTF we’ll be taking advantage of Linux, PHP misconfigurations, and LFI. To start off, I'm going to run an NMAP scan followed by a directory scan. We know from the start that there will be a web server so we'll dive right in and see what we can dig up. 

The NMAP scan discovered port 22 and 80 open. The directory scan displayed nothing out of the ordinary, an /img, /js/ and /css directories. 
<br>
<br>
<a href="https://imgur.com/9Kdtpid"><img src="https://i.imgur.com/9Kdtpid.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/Qugrton"><img src="https://i.imgur.com/Qugrton.jpg" title="source: imgur.com" /></a>
<br>
<br>
I opened the site in a browser and looked at the page source. At the end of the source there are two JS scripts. I looked at them individually in the /js directory and one of the scripts displays a subdomain. In the other JS script, we find a referrer link, we will investigate both
<br>
<br>
<a href="https://imgur.com/CztKaNE"><img src="https://i.imgur.com/CztKaNE.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/lmTCHL9"><img src="https://i.imgur.com/lmTCHL9.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/juBYpqv"><img src="https://i.imgur.com/juBYpqv.jpg" title="source: imgur.com" /></a>
<br>
<br>
Ok, so it looks like we can take advantage of local file inclusion to try and enumerate users so later I can try to bruteforce the html login. Running "curl curl http://vulnnet.thm/index.php?referer=/etc/passwd" lets us know indeed LFI works, but it did not reveal any usernames. We know there is an Apache server, so we can search for the config file to further enumerate. I'm not exactly sure which directory I should be looking in, so I searched google for the information and tried several different paths. This was a bit of a time sink, but I found a tutorial on the DigitalOcean website about setting up Apache virtual hosts and that the config file is stored in "/etc/apache2/sites-enabled/000-default.conf". Even though this took forever, but this is how you learn so I added this information to my notes and soldiered on. 
<br>
<br>
It looks like we have an AuthUserFile to investigate.
<br>
<br>
<a href="https://imgur.com/Ghi0JGD"><img src="https://i.imgur.com/Ghi0JGD.jpg" title="source: imgur.com" /></a>
<br>
<br>
We receive a login and password hash which we'll crack. 
<br>
<br>
<a href="https://imgur.com/RBPFQiQ"><img src="https://i.imgur.com/RBPFQiQ.jpg" title="source: imgur.com" /></a>
<br>
<br>
Cracking the hash with John reveals a password. 
<br>
<br>
<a href="https://imgur.com/LP4tsgD"><img src="https://i.imgur.com/LP4tsgD.jpg" title="source: imgur.com" /></a>
<br>
<br>
Going to broadcast.vulnnet.thm and logging in with the credentials brings us to a ClipBucket page. I will create an account and go on from there. 
<br>
<br>
<a href="https://imgur.com/Z56Inx5"><img src="https://i.imgur.com/Z56Inx5.jpg" title="source: imgur.com" /></a>
<br>
<br>
I created an account, but couldn’t find any exploitable surfaces from that angle, so I went to exploit DB to see if there were any vulnerabilities for ClipBucket v4.0. There are some serous vulnerabilities, "ClipBucket < 4.0.0 - Release 4902 - Command Injection / File Upload / SQL Injection" and we will take advantage of the file upload and upload a reverse shell. *credit to www.sec-consult.com. 
<br>
After uploading the .php file we will navigate to the directory and file specified and trigger the shell via netcat. 
<br>
<br>
<a href="https://imgur.com/XweGBm8"><img src="https://i.imgur.com/XweGBm8.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/8UlEZ25"><img src="https://i.imgur.com/8UlEZ25.jpg" title="source: imgur.com" /></a>
<br>
<br>
I tried to access the server-management directory but did not have permission. So, I searched for files and folders that may be accessible/writeable and discovered this ssh backup file. I started a server and downloaded that file to my attack machine to authenticate via ssh.
<br>
<br>
<a href="https://imgur.com/uvBHBvP"><img src="https://i.imgur.com/uvBHBvP.jpg" title="source: imgur.com" /></a>
<br>
<br>
I tried to login with the id_rsa via ssh, but it revealed that a password for the key is required. I then used ssh2john and john to find the keys password. 
<br>
<br>
<a href="https://imgur.com/bhTd4dm"><img src="https://i.imgur.com/bhTd4dm.jpg" title="source: imgur.com" /></a>
<br>
<br>
I then logged in via SSH and searched the users home directory revealing the contents of the user flag.
<br>
<br>
<a href="https://imgur.com/7IEYVVb"><img src="https://i.imgur.com/7IEYVVb.jpg" title="source: imgur.com" /></a>
<br>
<br>
I uploaded linpeas to the /tmp folder to do a quick scan of possible vulnerabilities and the only thing that looked interesting was a cronjob for a backup file.  
<br>
<br>
<a href="https://imgur.com/K0pkR3Y"><img src="https://i.imgur.com/K0pkR3Y.jpg" title="source: imgur.com" /></a>
<br>
<br>
<a href="https://imgur.com/q75WbIp"><img src="https://i.imgur.com/q75WbIp.jpg" title="source: imgur.com" /></a>
<br>
<br>
We do not have permission to write to the file, so we have to find another way. I spent some time googling it and utilized a previous walkthrough and GTFObins to help me break this barrier. *credit to Aldeid and GTFObins. 
<br>
<br>
<a href="https://imgur.com/4R2xpJ7"><img src="https://i.imgur.com/4R2xpJ7.jpg" title="source: imgur.com" /></a>
<br>
<br>
We will set up a netcat listener and a reverse shell that will be executed with privilege when the cronjob is triggered. 
<br>
<br>
<a href="https://imgur.com/0hiHLoR"><img src="https://i.imgur.com/0hiHLoR.jpg" title="source: imgur.com" /></a>
<br>
<br>
After the shell is triggered, we receive root privileges and can capture the root flag. 
<br>
<br>
<a href="https://imgur.com/YPDTmVG"><img src="https://i.imgur.com/YPDTmVG.jpg" title="source: imgur.com" /></a>


































































