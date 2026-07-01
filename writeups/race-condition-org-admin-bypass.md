# How I Found a Race Condition That Broke Organization Administration

<img width="720" height="402" alt="image" src="https://github.com/user-attachments/assets/e5c66130-8d66-4457-b013-c6e0c81d7b46" />

## Introduction

Many SaaS applications enforce a simple security rule “an organization should always have at least one administrator”. Without an administrator, 
no one can manage members, permissions, or organization settings which makes it pretty important right?

During testing, I discovered that this rule could be bypassed through a race condition. 
By sending two concurrent requests, it was possible to leave an organization while simultaneously removing the last remaining administrator, resulting in an organization with zero administrators.

## Understanding the Permission Model

```
Administrator
├── Manage members
├── Remove members
├── Promote users
└── Leave organization

Member
├── View resources
└── Cannot manage organization
```

Normally the application prevents the final administrator from leaving.

```
Admin A
Admin B
Member C
```

If Admin A attempts to leave while Admin B exists:

✅ Allowed

If Admin A is the last administrator:

🚫Blocked

This protection is what the race condition bypasses.

## Looking at the API

Now looking at the leaving organization request and deleting other member they were the same and they were:

```
DELETE /organizations/{org}/members/{member}
```

Now we have the request let’s exploit the race condition

## Exploiting

We said before that we have both admins which are user A and user B, now sending both these requests in group in parallel

```
DELETE /organizations/{org}/members/{User_A}
DELETE /organizations/{org}/members/{User_B}
```

and both requests worked!!

Looking back at our organization we will only find User C which is still a member who can’t do anything 😓and not just that, 
if User C tried to delete their account they can’t cuz the website sees them in the org and assumes they are the owner and 
tells them to delete it first but still User C doesn’t have permission to which increases the impact!!

## Impact

The organization enters an orphaned state.

Remaining members cannot:

- promote themselves
- add administrators
- manage organization settings
- transfer ownership
- recover control
- leave the organization
- Additionally, if account deletion requires leaving every organization first, affected users cannot delete their accounts because they are permanently trapped inside the orphaned organization.

This creates an operational denial-of-service affecting organization management and account lifecycle functionality.

## Why This Happens

The application validates authorization against a stale snapshot of organization membership. 
Because both requests execute concurrently, each observes two administrators and independently concludes that removing one administrator is safe.

Neither request is aware that the other is about to commit, allowing the invariant of “at least one administrator” to be violated.
