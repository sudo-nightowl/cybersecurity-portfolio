## How I Found a Race Condition That Broke a Resource Limit (Real-World Case Study)
Source: https://medium.com/@omaralgbry1/how-i-found-a-race-condition-that-broke-a-resource-limit-real-world-case-study-79bd09f29e8f
Please don't forget to follow me there
<img width="720" height="402" alt="1_fIwWiBFIjWvcixLIlscjsA" src="https://github.com/user-attachments/assets/b3845a05-40ac-4a81-b9f5-8236652cbe4c" />

# Introduction

Most beginners in bug bounty focus on things like XSS or SQLI. Those are important, but modern applications are often hardened against them.

What’s much more interesting and often overlooked hmm? Maybe how systems behave under concurrency.

In this article I will walk through a real world case where a simple resource limit could be bypassed using a race condition. No fancy payloads, no complex exploits just understanding how backend logic breaks under pressure.

# The Target

While testing a web application that allows users to create “workspaces” , I noticed something interesting:

1- Free tier users had a strict limit on how many workspaces they could create

2- The UI enforced this limit disallowing you from making more than the allowed “workspaces” for free tier users

But then I asked myself “does the backend do the same?” so I captured the request and in repeater I tried making more than the allowed “workspaces” for a free tier user….didn’t work:

<img width="720" height="746" alt="burp" src="https://github.com/user-attachments/assets/fe910571-c429-4364-8699-e4169e2a8afd" />

# The exploit

Now since it looks like the backend also doesn’t allow free tier users to make more “workspaces” than they are allowed to right? Hmmm not exactly.

Now try adding this request to a group and duplicate it more times than the free tier user is allowed to make “workspaces”

<img width="276" height="144" alt="burp3" src="https://github.com/user-attachments/assets/d6c1bb17-1e74-4d20-b9bf-795116831463" />
<img width="281" height="196" alt="burp2" src="https://github.com/user-attachments/assets/81bced67-6d19-4f7e-bc28-6bd753dcdd37" />

Now since we got everything ready make sure to send the group in parallel (single-packet attack)

<img width="285" height="125" alt="burp4" src="https://github.com/user-attachments/assets/97ad04bb-32b7-45de-90f5-9a8ecad2a754" />


And send and see the responses…..BOOM!! We successfully created more “workspaces” than we are allowed to have as a free tier user and they all work just fine!!

# Conclusion

This wasn’t some crazy exploit or advanced payload. It was just about thinking differently.

Instead of trying to break input validation, I focused on how the system behaves under pressure and that’s where it failed.

Race conditions like this are easy to miss because everything works fine when tested normally. But once you start sending requests in parallel, the backend logic can break in unexpected ways.

If you’re getting into bug bounty, don’t just think about what you send but think about how and when you send it.

Hope yall enjoyed reading this, take care!!
