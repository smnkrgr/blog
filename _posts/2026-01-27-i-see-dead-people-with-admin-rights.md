---
layout: post
title: "I see dead people with Admin Rights"
date: 2026-01-27
---

![iseedeadpeople_nokrugsec 2.png](/assets/images/iseedeadpeople_nokrugsec 2.png)
On a recent Active Directory assessment, a customer showed large amounts of disabled user and computer accounts.
These amass over time when people leave the company or hardware gets rotated. Once disabled they pose no risk to the organisation, right? 
Wrong. In this article I list reasons of why having zombie objects can come back to bite you in your administrative backside.

## Reanimate-Tombstones Extended Right
This one plays right into the zombie theme. The reanimate-tombstones extended right allows restoration of deleted AD objects within the tombstone lifetime (180 days default), preserving their original SID, group memberships, and permissions. If the Active Directory recycle bin feature is enabled, the restoration even retains all of the objects properties. 
Attackers that have gained access to this right can resurrect disabled accounts that might have access to more resources thus facilitating lateral movement. Group memberships are especially crucial as attackers immediately gain all previously assigned permissions without additional privilege escalation. Reanimating user objects essentially grants control over the account because the attacker must set a password for the restored account - all of this without triggering an account creation.
To remediate audit and restrict reanimate-tombstones permissions to only essential recovery administrators. Furthermore, implement automated group membership removal upon disablement. Re-enabling might not create account creation events, but you can still implement monitoring for object restoration events (Event ID 5138 and 5139).

![deadadmincard.jpeg](/assets/images/deadadmincard.jpeg)
## Kerberoasting on Disabled Service Accounts
Assume you try to be a good admin and disable an account with Service Principal Names (SPNs) that you do not need anymore to adhere to the least-privilege-principle. Turns out they can still be queried for Kerberos service tickets when disabled.
Attackers can still utilize the kerberoasting attack vector on disabled accounts with SPNs to crack and recover passwords, which can be used elsewhere (e.g. in Password Spraying attacks).
To not fall into this trap and give the attackers zombies to roast, remove SPNs from disabled accounts.
## Resource-Based Constrained Delegation (RBCD)
We spoke a lot about dead users, but what about dead computers?
RBCD is an AD feature introduced in Windows Server 2012 that allows a target computer object to specify which other accounts (users or computers) can impersonate users to access it. You guessed it - disabled computer objects can still be configured to allow delegation attacks.
Attackers modify the msDS-AllowedToActOnBehalfOfOtherIdentity attribute on disabled computers they control to impersonate users and move laterally across the network.
To remediate remove delegation attributes from disabled computers and monitor for modifications to the msDS-AllowedToActOnBehalfOfOtherIdentity attribute on non-active objects.

# Conclusion
This is not an exhaustive list as there are more aspects to disabled objects that can increase your attack surface (e.g. bypassing password policy changes on disabled accounts, AdminSDHolder residual permissions, stale Kerberos tickets, ...), but these three give you enough of a reason to take a look at the disabled objects you might still have in your environment serving no purpose other than make life easier for the adversary.

![removedeadcard.jpeg](/assets/images/removedeadcard.jpeg)