## How I Found a Bug That Let Me Change My Account Email to Any Email Address Without Owning It
Source: https://medium.com/@omaralgbry1/how-i-found-a-bug-that-let-me-change-my-account-email-to-any-email-address-without-owning-it-6e4a6d797330
Please don't forget to follow me there
<img width="720" height="402" alt="1_9g691oyYITS_x6ki9Jtx-A" src="https://github.com/user-attachments/assets/c5b3d049-c4b7-4ab0-9e5d-d51c877df071" />

# Introduction

While testing redacted.com, I discovered a bug in the email change functionality that allowed users to change their account email address to a new email and verify it without actually owning it.

The bug existed because the application exposed the email verification hash directly inside the API response during the email change workflow. Since the verification token was supposed to act as proof of inbox ownership, exposing it to the authenticated user completely broke the verification process.

For responsible disclosure reasons, the target will be referred to as `redacted.com` throughout this writeup.

# Understanding the Intended Flow

Normally, changing an account email should work like this:

User submits a new email address
Server generates a secret verification token
Token is sent only to the target inbox
User clicks the verification link from the email
Server verifies ownership and updates the account email
The whole thing is built on the idea of :

“Only the owner of the inbox should have access to the verification token.”

# Discovery and Exploitation

While exploring how the website updates the email I intercepted the request responsible for changing the account email using Burp Suite.

<img width="720" height="706" alt="burp1" src="https://github.com/user-attachments/assets/97357770-a94c-4833-a37f-53b87b8a16e0" />

Hmm…everything looks normal here and it’s waiting to be verified

<img width="720" height="59" alt="img1" src="https://github.com/user-attachments/assets/7bd0aeb0-8334-4b4a-9077-9104d4fb56f4" />

Now what happens when we click on the resend email? let’s find out.

<img width="720" height="395" alt="burp2" src="https://github.com/user-attachments/assets/77b8e3e3-922a-4d79-90b5-6c679886c8e2" />

Now that’s what you get once you click on resend email…interesting isn’t it?

If you haven’t connected the dots yet lemme help you out. This is the hash that’s sent to the email you wanna change to.

You can now go ahead and verify your email using this hash:

`https://api.redacted.com/user/confirm-change-email?email=<hash>`

# Impact

This vulnerability permitted authenticated users to completely bypass email ownership verification.

An attacker may:

Replace their account email with random emails
Email verification without inbox access
Tie accounts to email addresses not owned
Break systems using verified email trust

Verified email addresses are often regarded as trusted identity attributes across applications. If email ownership validation fails, other account and authorisation issues could potentially arise depending on how the platform internally leverages verified emails.

# Root cause

The root cause was a broken trust boundary in the e-mail change workflow.

The verification hash, which should have been available only through the target inbox, was exposed directly to the authenticated client in the API response.

# Final Thoughts

This bug was a simple reminder even simple logic flaws can completely mess up a website’s security mechanisms. Sometimes all you gotta do is watch carefully and observe any weird behaviour.
