---
title: CTF Writeup - mkingdom
date: 2024-12-03 14:18:00 +0800
categories: [TryHackMe, CTF]
tags: [ctf]     # TAG names should always be lowercase
image:
  path: /img/writeup/thm/mkingdom/thumbnail.png
  alt: CTF Writeup
description: Writeup of the CTF "mkingdom".
---

Hello everyone! This is my first write-up. I've done a couple of CTFs before, but due to university, I stopped practicing. Now, I've decided to document my thought process and show how I managed to grab the flag.

The first step is reconnaissance so we can understand what we are working with. I started with an Nmap scan, which revealed an HTTP service running on port 85, as detailed in the image below

![img-description](/img/writeup/thm/mkingdom/nmap.png)

After accessing the HTTP service in the browser, nothing caught my eye.

![img-description](/img/writeup/thm/mkingdom/mainpage.png)

Decided to inspect the page source code but nothing either.

![img-description](/img/writeup/thm/mkingdom/mainpagesource.png)

Since I didn't find anything, it can only mean one thing, we need to explore deeper, so I ran some good ol'gobuster to discover any other pages, and lucky us we found one.

![img-description](/img/writeup/thm/mkingdom/appjump.png)

Clicking the "JUMP" button redirected us to a blog website.

![img-description](/img/writeup/thm/mkingdom/blog.png)

Using Wappalyzer, I gathered information about the website and found that it uses Concrete CMS, a Content Management System (CMS), similar to WordPress.

![img-description](/img/writeup/thm/mkingdom/wappalyzer.png)

After searching for vulnerabilities in this version of Concrete CMS, I found a website detailing an exploit for a reverse shell:
https://hackerone.com/reports/768322

The exploit's steps are as follows:
1. Log in as Admin.
2. Visit "Allow File Types" and add the PHP file type if necessary.
3. Use the File Manager to upload the reverse shell.

Obviously, to use this exploit, we first need to obtain admin access. After researching the CMS, I learned that the default login page is index.php/login, and luckily, it's the same for us. The default username is "admin," but the password is set during setup. Since we know the username, all that's left is to find the password.

I used Burp Suite's Intruder attack to discover the password. As shown in the image below, we found it—ironically, the password is "password."

![img-description](/img/writeup/thm/mkingdom/burpsuitebrute.png)

After logging in, I followed the exploit steps listed above.
To generate the reverse shell, I used the website revshells.com and selected one suitable for the job.

After uploading our reverse shell, all that remained was to execute it by opening the file.

![img-description](/img/writeup/thm/mkingdom/upload.png)

**It's ALIVE!** (hope someone gets the reference)

![img-description](/img/writeup/thm/mkingdom/nc.png)

Once we obtained a reverse shell, we logged in as the www-data user. After further inspection, we found the following users:

* toad
* mario

Our objective was now to escalate privileges and obtain the flags.

I attempted privilege escalation through SUID and binaries, but nothing worked. So, I resorted to using LinPEAS.

As shown below, LinPEAS found a password that appears to belong to Toad.

![img-description](/img/writeup/thm/mkingdom/toadpassword.png)

It worked!
Looking through Toad's files, I found a .txt file, but it contained nothing interesting.

![img-description](/img/writeup/thm/mkingdom/toadtxt.png)

Continuing the search, I checked the environment variables and found an encrypted password.

![img-description](/img/writeup/thm/mkingdom/toadlog.png)

Using the website cyberchef we are able to decrypt it and we got a password.

![img-description](/img/writeup/thm/mkingdom/cyberchef.png)

I tried this password on the user Mario, and it worked!

![img-description](/img/writeup/thm/mkingdom/mariolog.png)

We found one of the flags in Mario's directory, but when attempting to read it, we lacked the necessary permissions as it was owned by root.

![img-description](/img/writeup/thm/mkingdom/cyberchef.png)

After trying various privilege escalation techniques without success, I checked a write-up for tips.
Using LinPEAS again, I found an interesting file we could modify: /etc/hosts. The write-up also mentioned a background process.

To identify the process, I used a tool called pspy64, which analyzes running processes without needing root permissions.

![img-description](/img/writeup/thm/mkingdom/pspy64inicial.png)

As shown below, we discovered an interesting process.

![img-description](/img/writeup/thm/mkingdom/counteraux.png)

This process runs a curl command to fetch a script named counter.sh. Since we have write permissions for the /etc/hosts file, we could redirect the script's fetch path to our machine, where we serve a modified counter.sh containing a reverse shell.

**/app/castle/application**

I modify the host file and change the IP for the website to our machine.

To recap how the attack works

1. The curl process fetches counter.sh from the web server.
2. We modify the /etc/hosts file to redirect the web server's address to our machine.
3. On our machine, we create a malicious counter.sh file with a reverse shell.
4. When the curl command executes, it fetches our malicious script and triggers the reverse shell.

As shown below, after a successful reverse shell, we gained root access and retrieved the final flag!

![img-description](/img/writeup/thm/mkingdom/rootflag.png)

Thank you for reading my first writeup, overall in the end I was a bit dissapointed in myself since I ended up checking the writeup for help but nonetheless I learned something new :) 

![img-description](/img/writeup/thm/mkingdom/meme1.png){: width="400" height="200" }
