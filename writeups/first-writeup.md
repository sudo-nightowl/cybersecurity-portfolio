## How I Got Unlimited “Paid” Resources on a SaaS Platform by Replaying One Request
Source: https://medium.com/@omaralgbry1/while-poking-around-a-saas-platform-for-bug-bounty-research-i-ran-into-a-business-logic-flaw-that-ac52eab4b243
Please don't forget to follow me there

<img width="720" height="393" alt="1_mWB-v1aqawgxSsBFfXycqQ" src="https://github.com/user-attachments/assets/bc8946c2-6af9-42f7-9dfd-db9cf3f72eb4" />

# Introduction

While poking around a SaaS platform for bug bounty research, I ran into a business logic flaw that let me break the free plan API key limit without paying a benny for an upgrade.

What’s wild is that I didn’t need any clever tricks at all, no messing with race conditions or tampering with tokens, not even any payloads.

The whole thing came down to a simple fact:

“The frontend put a cap on API keys, but the backend just took requests at face value even if I replayed them.”

I’ll walk through how I found it, why it worked, and why you can never trust any security built only into the frontend.

# Understanding the Expected Behavior

The service said free users could only make a certain number of API keys.

I created four API keys using the web dashboard just like a normal user. After I hit that limit, the UI blocked me, telling me to upgrade if I wanted more keys.

So at first, it looked like everything was locked down.

But then I asked myself:
“So the UI blocked me…Will the backend block me too?”

Testing the Backend

To find out, I used Burp Suite to catch a real API key creation request.

It was a pretty standard POST, something like:

`POST /v3/resources`

Once I hit the free plan’s key limit, I took that same request and sent it with Burp Repeater and….IT WORKED!!
Surprisingly, the backend went ahead and created another API key even though my account should’ve blocked from any more keys.
No fancy bypassing no race conditions. Just resending the same request.
I checked the dashboard again, and guess what, the new API keys was there and worked just like the first ones.

# Root Cause Analysis

After digging in, it was clear what went wrong:
“the frontend and backend were checking limits in totally different ways.”
The frontend did all the work blocking the creation process, grayed out the UI button, told me to upgrade. But the backend? It didn’t check if and relied on that the frontend did all the work.

So if you went straight to the API and replayed a past request, you could keep generating new resources with no limit.
# Security Impact

People sometimes label this as a “business logic issue,” but it really can hit security and operations hard.

Here’s what can happen:
– Anyone could use paid features without paying.
– The backend could get hammered by extra load.
– Subscription checks basically mean nothing.
– Attackers could even try to exhaust backend resources.

The real danger is how easy it is to create disconnects between what the UI allows and what the server actually enforces.

# Key Takeaways

The big lesson: Frontend checks are NOT security controls.
A lot of SaaS apps break things out the UI tracks subscription status, the backend provisions resources, and entitlement checks float somewhere in between.
But if these pieces drift out of sync, just replaying a valid request can get around rules that look solid from the outside.
So, when you’re testing any app, always ask:
– Does the backend really check quotas?
– Is the server blocking extra resource creation?
– What does it do with old, previously valid requests when your account status has changed?
Sometimes, the UI says “no,” but the server keeps giving out unlimited resources behind the scenes.

# Final Thoughts
The vulnerability drove home something I see all the time:

Frontend validation makes things smooth for users, but only backend validation actually enforces security.

If entitlement checks exist only in the frontend, those restrictions are usually one step away from being useless. Directly calling the backend is often enough to spot broken authorization that the UI tries to hide.
Frontend validation makes things smooth for users, but only backend validation actually enforces security.

If entitlement checks exist only in the frontend, those restrictions are usually one step away from being useless. Directly calling the backend is often enough to spot broken authorization that the UI tries to hide.
