## How I Bypassed UI Restraints to Leak Organization Roles, Members, and Invites (BAC)
Source: https://medium.com/@omaralgbry1/how-i-bypassed-ui-restraints-to-leak-organization-roles-members-and-invites-bac-9390d994e38f
Please don't forget to follow me there

<img width="720" height="402" alt="1_ZSVso5UAT54THrMm39ndvg" src="https://github.com/user-attachments/assets/8e7c166d-488a-4c88-a592-ed9399f61b88" />

# Introduction
While looking around a platform and doing bug bounty research, I ran into a simple architectural flaw which was: the frontend and the backend were checking permissions in totally different ways.

By using this, I was able to completely bypass UI restrictions using a low-privileged account and extract sensitive organizational data, including custom role definitions, entire member directories,
 and pending user email invitations.
# Discovery
While looking at the organization settings on redacted.com, I was focusing on how user privileges are handled. The application has a highly restricted, low-privileged role called **Trust Sharer (The lowest role)**.

Naturally, a Trust Sharer shouldn’t be allowed to view administrative settings. When I logged into the test account with this restricted role and tried to access the Roles Management interface through the 
browser at: `https://app.redacted.com/settings/organization/roles`

Or when I tried the members at

`https://app.redacted.com/settings/organization/users`

Or pending invites at

`https://app.redacted.com/settings/organization/invitations`

<img width="720" height="406" alt="q" src="https://github.com/user-attachments/assets/e96e2ce8-3997-44d8-9407-f4d8d199f70e" />

The frontend app did exactly what it was supposed to do. It checked my role, blocked the view, and gave back “You don’t have permission to access this page” page .

But as we know, what the UI shows you is not everything, The real question is “Does the server actually back it up?” So let’s test it.
# Investigating

To see what was happening behind the scenes, I needed to capture a valid authorization token from my low-privileged session. I triggered a simple profile update action in the UI, caught the traffic in Burp Suite
, and extracted my ``Authorization: Bearer <trust_sharer_token> header``.

Now since I got my token instead of using the browser, I took that token straight to the backend API responsible for fetching organizational roles:

```
GET /organizations/{organization_id}/roles HTTP/2
Host: auth.redacted.com
Authorization: Bearer <trust_sharer_token>
```
I sent and guess what? **HTTP/2 200 OK**.

The server didn’t care that my token belonged to a restricted Trust Sharer account. It happily dumped the entire list of organization roles, internal IDs, permission mappings, and custom roles. 
You can see the full response payload right here.

<img width="720" height="376" alt="burp1" src="https://github.com/user-attachments/assets/b473112e-b666-4133-b321-62170d0a0cfa" />

Now that I found this I tested it on more than just this endpoint and found 2 more vulnerable endpoints which are
```
/organizations/{organization_id}/users
/organizations/{organization_id}/invitations
```
Sending the same request again for these 2 endpoints gave back 200 OK leaking even more information for a low-privilege user

<img width="720" height="358" alt="burp2" src="https://github.com/user-attachments/assets/677c407c-48be-4d35-b447-c101f961d0d9" />

<img width="720" height="376" alt="burp3" src="https://github.com/user-attachments/assets/818bc793-e1e5-4ea7-bd80-d8aa2d0a3545" />

# Isolating the Bug
To confirm whether this was an authentication breakdown (session issue) or a pure Broken Function Level Authorization (BFLA) flaw, I performed a quick matrix test:
- **No Token:** Requests made without an Authorization header returned a clean `403 Forbidden`. (Authentication is working).
- **Cross-Tenant Token:** Requests made using a valid token from a completely different organization identifier returned a `403 Forbidden`. (Tenant isolation is working).
- **Low-Privileged Internal Token:** Requests made using the target organization’s low-privileged Trust Sharer account returned a `200 OK`.

This proved that while authentication and tenant scoping were handling things fine, the backend was completely skipping role-level validation for these specific endpoints.

# Root Cause Analysis

What went wrong here?

It’s a design habit that a lot of web apps fall into: relying on client-side security. The frontend engineers built robust validation rules to hide menus and block direct URL routing for restricted roles.
But the backend API developers provisioned the `/roles`, `/users`, and `/invitations` data endpoints under the broad assumption that if a token was valid and belonged to the tenant ID, the request was safe to process.

When these layers drift out of sync, an attacker doesn’t need to fight the UI. They just go straight to the backend and replay the query.

# Takeaways for Hunters:

Whenever you’re testing an app that has multi-tiered roles (Admin, Member, Viewer, Guest, etc.):

Don’t stop when a button is grayed out or a page throws an “Access Denied” screen in the browser.
Map out the API routes using your highest-privileged account.
Thanks for reading! Let me know if you have any questions, and happy hunting!
