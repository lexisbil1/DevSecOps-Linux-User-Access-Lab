# DevSecOps Linux User & Access Control Lab
A hands-on practical exercise implementing user onboarding, password analysis, RBAC group assignments, sudo privilege delegation, service account hardening, and secure user lifecycle procedures in Linux.

---

# Table of Contents
1. [Overview](#overview)
2. [Scenario 1 – Password & User Analysis](#scenario-1--password--user-analysis)
   - Part A: Create a User Without a Password  
   - Part B: Identify Users Without Passwords  
   - Part C: Remediate by Setting a Password  
3. [Scenario 2 – DevSecOps Access Control Configuration](#scenario-2--devsecops-access-control-configuration)
   - Part 1: Create Department Groups  
   - Part 2: Create All Users  
   - Part 3: Assign Users to Groups  
   - Part 4: Passwordless User + Remediation  
   - Part 5: Sudo Access for One Ops User  
   - Part 6: Remove a User but Keep Home Directory  
   - Part 7: Verification & Privilege Boundary Testing  
4. [Key DevSecOps Learnings](#key-devsecops-learnings)
5. [Reflection Write-Up](#reflection-write-up)
6. [Screenshots](#screenshots)

---

# Overview

This lab simulates a real DevSecOps onboarding and RBAC configuration scenario on a shared Linux system used by developers, QA testers, security analysts, and operations engineers.

The goals included:

- User and password state analysis  
- Department-based RBAC (Role-Based Access Control)  
- Least privilege implementation  
- Service account hardening  
- Privilege escalation control  
- User lifecycle management  
- Audit-focused design  

---

# Scenario 1 – Password & User Analysis

## Part A – Create a User Without a Password

### Commands Executed

sudo useradd -m audit_test

getent passwd audit_test

id audit_test

Explanation

audit_test was created using useradd -m, which does not assign a password by default.
getent and id confirmed the account exists with a home directory, but has no usable password yet.

Part B – Identify Passwordless Users & Investigate /etc/shadow

Commands Executed

sudo ls -l /etc/passwd /etc/shadow

sudo grep '^audit_test:' /etc/shadow

sudo awk -F: '($2=="" || $2=="!" || $2=="!!") {print $1, $2}' /etc/shadow

Explanation

/etc/passwd stores user account metadata

/etc/shadow stores sensitive password hashes and password-aging policies

audit_test showed a ! in the password field → no password set

Part C – Remediate by Setting a Password

Commands Executed

sudo passwd audit_test

sudo grep '^audit_test:' /etc/shadow

Explanation

After setting the password, the shadow entry changed from ! to a full hashed password.
This confirms remediation and proper password configuration.

# Scenario 2 – DevSecOps Access Control Configuration

Part 1 – Create Department Groups

Commands Executed

sudo groupadd dev_team

sudo groupadd sec_team

sudo groupadd qa_team

sudo groupadd ops_team

getent group dev_team

getent group sec_team

getent group qa_team

getent group ops_team

Explanation

Four department groups were created to implement clear RBAC separation.
Group existence was verified via getent.

Part 2 – Create All Onboarded Users

Commands Executed

sudo useradd -m lateef

sudo adduser yaa
sudo adduser alex
sudo adduser habiba
sudo adduser aminat
sudo adduser purity
sudo adduser arthur
sudo adduser paula

getent passwd lateef purity arthur paula yaa alex habiba aminat
Explanation

The assignment required both useradd and adduser.
All users were onboarded with home directories and shell environments.

Part 3 – Assign Users to Their Correct Department Groups
Commands Executed
sudo usermod -aG dev_team lateef
sudo usermod -aG dev_team arthur

sudo usermod -aG sec_team purity
sudo usermod -aG sec_team habiba

sudo usermod -aG qa_team paula
sudo usermod -aG qa_team yaa

sudo usermod -aG ops_team alex
sudo usermod -aG ops_team aminat

getent group dev_team
getent group sec_team
getent group qa_team
getent group ops_team

id lateef
id alex
id yaa
Explanation

usermod -aG added each user to their department's supplementary group.
getent and id validated correct RBAC alignment.

Part 4 – Passwordless User + Remediation
Commands Executed
sudo grep '^lateef:' /etc/shadow
sudo passwd lateef
sudo grep '^lateef:' /etc/shadow
Explanation

lateef was intentionally created without a password.
Shadow file analysis confirmed this via the ! lock indicator.
After setting a password, the entry changed to a proper hash.

Part 5 – Sudo Access for ONE ops_team User Only
Commands Executed
sudo visudo
# Added inside sudoers:
# alex ALL=(ALL:ALL) ALL

su - alex
sudo whoami

su - yaa
sudo whoami

Explanation

Only alex (ops_team) received sudo privileges.
All other users correctly received a "not in sudoers" denial.
This confirms enforcement of least privilege and role-based access.

Part 6 – Remove One User but Keep Home Directory
Commands Executed
sudo userdel paula
ls -ld /home/paula
Explanation

userdel without -r removes the user but keeps their home directory for audit, forensic, and compliance needs.

Part 7 – Verification & Privilege Boundary Testing
Commands Executed
getent group dev_team sec_team qa_team ops_team
id alex
id aminat
sudo -l
su - yaa
sudo whoami
Explanation

This final verification confirmed:

Group membership is correctly stored in /etc/group

Users belong to proper supplementary groups

Only the designated ops user (alex) can escalate privileges

Unauthorized escalation attempts are blocked

 Key DevSecOps Learnings

🔹 How Linux Tracks Group Membership

Linux stores group definitions in /etc/group and primary GID mappings in /etc/passwd.
This ensures consistent authorization across the system.

🔹 How Grouping Enforces Access Control

Department groups (dev, sec, qa, ops) create logical separation of duties and enforce RBAC.

🔹 Why Service Accounts Use Non-Login Shells

ci_runner uses /usr/sbin/nologin to prevent human access while still supporting automation.
This protects CI/CD pipelines from misuse.

🔹 Why Home Directories Are Retained After Deletion

Home directories contain logs, configs, and evidence that support forensics and compliance audits.

🔹 How Privilege Escalation Is Controlled

Using sudoers, visudo, and department boundaries ensures:

Only authorized users can escalate

Actions are logged

Least privilege is enforced

 Reflection Write-Up

This lab strengthened my understanding of identity, authentication, RBAC, least privilege, and secure configuration in Linux.
It showcased how foundational system controls support DevSecOps practices and secure operations across engineering teams.

DevSecOps starts with strong access control, audit readiness, and predictable privilege boundaries—and this exercise demonstrated how they work in practice.

🖼 Screenshots

All screenshots (user creation, shadow analysis, sudo tests, group verification, etc.) are stored in the screenshots/ directory.
