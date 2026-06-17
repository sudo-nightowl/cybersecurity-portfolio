## TryHackMe Lo-Fi Writeup
<img width="600" height="453" alt="image" src="https://github.com/user-attachments/assets/50669120-6ee8-4e8d-ae6d-f23240c1bb5a" />

Want to hear some lo-fi beats, to relax or study to? We’ve got you covered!

# Introduction

Lo-Fi is an easy TryHackMe room focused on identifying and exploiting a Local File Inclusion (LFI) vulnerability. 
In this writeup, I’ll walk through the enumeration, discovery, and exploitation process used to obtain the flag.

# Nmap scan

First thing you do in any CTF is an nmap scan
<img width="720" height="453" alt="image" src="https://github.com/user-attachments/assets/ce56c81c-bd48-4d78-b604-25b0ba64716d" />

Now we know that there is a website running on port 80

# Discovery
Opening the website I get welcomed with a Lo-Fi page
<img width="720" height="345" alt="image" src="https://github.com/user-attachments/assets/89303fac-78c6-4d57-9145-32cc46ed4710" />

Looking around I watch the pattern and see how the URL changes depending on the page you’re at
<img width="720" height="345" alt="image" src="https://github.com/user-attachments/assets/fd6fcb24-b7fa-491e-b26a-3e7607d6305f" />

I noticed the application used a `page` parameter to dynamically load PHP files. Since user controlled file inclusion is a common source of Local File Inclusion (LFI) vulnerabilities, 
I tested whether directory traversal sequences could escape the intended directory.

# Exploiting
I tried a basic ../../../etc/passwd and got this back:
<img width="720" height="316" alt="image" src="https://github.com/user-attachments/assets/d67f4700-2109-4e23-bbe0-6e73b8fbf8dc" />

Perfect I have successfully exploited a LFI vulnerability here

# Flag
Since you could get the content of /etc/passwd using ../../../etc/passwd you should be able to get the content of flag.txt using ../../../flag.txt
<img width="720" height="304" alt="image" src="https://github.com/user-attachments/assets/509b8d41-4c2e-4204-9223-79586499c87a" />

# What you learn from this CTF

This room demonstrates how unsafe file inclusion can lead to Local File Inclusion vulnerabilities. By manipulating the page parameter with directory traversal sequences, 
an attacker can access files outside the intended web directory, potentially exposing sensitive information.
