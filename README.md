![osTicket](https://img.shields.io/badge/osTicket-v1.18.1-blue) ![Azure](https://img.shields.io/badge/Microsoft-Azure-0089D6?logo=microsoft-azure) ![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04_LTS-E95420?logo=ubuntu) ![LAMP](https://img.shields.io/badge/Stack-LAMP-green)

# osTicket v1.18.1 Deployment on Microsoft Azure

## Project Overview

I deployed osTicket v1.18.1, an open-source IT help desk ticketing system, on a Microsoft Azure Ubuntu VM using a LAMP stack (Linux, Apache, MySQL, PHP).

I built this project to reflect how a real IT support environment works. I set up a ticketing system with SLA enforcement, tiered agent roles, and department-based routing — which is standard in most IT departments. I chose to deploy on Azure rather than locally so I could work with network security groups, public IPs, and cloud VM management, skills that come up constantly in IT and sysadmin roles.

## Environment

| Component | Detail |
|-----------|--------|
| Platform | Microsoft Azure (Free Tier eligible) |
| OS | Ubuntu Server 22.04 LTS |
| Web Server | Apache2 |
| Database | MySQL Server |
| App | osTicket v1.18.1 |
| Estimated Build Time | 2–3 hours |
| Skill Level | Beginner / Intermediate |

## Architecture


<img width="1408" height="768" alt="Gemini_Generated_Image_5u26cf5u26cf5u26" src="https://github.com/user-attachments/assets/417c6da1-b7c8-4d9e-b0c7-d4bae161895d" />


## Phase 1 — Azure VM Provisioning

### VM Configuration

I provisioned a `Standard_B1s` VM (1 vCPU, 1 GB RAM) running Ubuntu 22.04 LTS, which is free tier eligible.
<img width="1911" height="874" alt="Screenshot 2026-03-06 135708" src="https://github.com/user-attachments/assets/4e3ec611-3782-44e6-a210-72e9eb35d80e" />


### Network Security Group (NSG) Rules

I configured inbound rules to allow SSH (port 22) and HTTP (port 80) traffic to the VM.
<img width="1521" height="600" alt="Screenshot 2026-03-06 135821" src="https://github.com/user-attachments/assets/15e9fd7a-a50d-4526-8d3d-fb4b2aa79fbf" />


## Phase 2 — LAMP Stack Installation

### NSG Confirmed + LAMP Install Started

I confirmed both SSH and HTTP inbound rules were active and then started installing the LAMP stack.

### Apache Verified + MySQL and PHP Install

I verified Apache was running by hitting the VM's public IP in a browser. From there, I installed MySQL and all the required PHP extensions.

<img width="1822" height="964" alt="Screenshot 2026-03-06 141738" src="https://github.com/user-attachments/assets/0126b5bc-3983-4f36-a37a-522b365d92dc" />


## Phase 3 — MySQL Database Setup

### Database and User Created

I created the osTicket database and a dedicated user with full privileges:

```sql
CREATE DATABASE osticket;
CREATE USER 'osticketuser'@'localhost' IDENTIFIED BY '*************!';
GRANT ALL PRIVILEGES ON osticket.* TO 'osticketuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
<img width="806" height="216" alt="Screenshot 2026-03-06 142659" src="https://github.com/user-attachments/assets/5d0d4490-f58e-4eac-85d3-6f94dc347d8e" />


## Phase 4 — osTicket Installation

### Web Installer

I navigated to `http://PUBLIC_IP/osticket/setup/` and went through the setup wizard, filling in system settings, admin account details, and database credentials.

<img width="1777" height="974" alt="Screenshot 2026-03-06 143319" src="https://github.com/user-attachments/assets/9168c60b-3c03-4f6a-8f36-a53559b7ca85" />


### Installation Complete

Once I got the green success screen, I immediately removed the `/setup` directory and locked down `ost-config.php` so it could not be modified through the web:

```bash
sudo rm -rf /var/www/html/osticket/setup
sudo chmod 0644 /var/www/html/osticket/include/ost-config.php
```

## Phase 5 — Help Desk Configuration

### Departments

I went to Admin Panel > Agents > Departments and created four departments for ticket routing:

| Department | Purpose |
|------------|---------|
| IT Support (Tier 1) | Front-line support for general issues |
| Networking | Network connectivity and infrastructure issues |
| Security | Security incidents, access control, account issues |
| Hardware | Physical device issues, replacements, peripherals |

<img width="1205" height="660" alt="Screenshot 2026-03-06 151451" src="https://github.com/user-attachments/assets/96db3189-0457-4a6e-a942-e90a8e3a55cd" />


### SLA Plans

I went to Admin Panel > Manage > SLA Plans and configured three tiers:

| SLA Plan | Grace Period | Schedule |
|----------|-------------|----------|
| High Priority SLA | 4 hours | 24/7 |
| Normal Priority SLA | 24 hours | 24/7 |
| Low Priority SLA | 72 hours | 24/7 |

<img width="1207" height="530" alt="Screenshot 2026-03-06 151656" src="https://github.com/user-attachments/assets/4f6f5073-4f24-4241-bfea-f1bc059b0d6d" />


### Help Topics and Agent Accounts

I created five Help Topics (Password Reset, Network/Wi-Fi Issues, VPN/Remote Access, Hardware Request, Account Lockout) and set up two agent accounts: Jane Smith (Tier 1) and John Doe (Senior/Full Access).

<img width="1192" height="692" alt="Screenshot 2026-03-06 152115" src="https://github.com/user-attachments/assets/5a475482-4f6d-4d68-a154-1221e4274599" />
<img width="1200" height="495" alt="Screenshot 2026-03-06 153320" src="https://github.com/user-attachments/assets/3285e9b8-1aac-46a5-8fdf-9d71f19d27ad" />



## Phase 6 — Ticket Workflow Simulation

To validate the system, I simulated three real-world support scenarios:

### Scenario 1: VPN Not Connecting

**Priority:** High | **SLA:** 4 hours | **Department:** Networking

I submitted a test ticket through the client portal reporting VPN error 800 after a Windows update. Jane Smith (Tier 1) picked up the ticket, set it to High priority, and added an internal note while investigating. She escalated to John Doe, who identified that Windows update `KB5034441` was conflicting with the VPN client. He rolled back the update via remote session and the VPN came back up.

<img width="1010" height="858" alt="Screenshot 2026-03-06 153848" src="https://github.com/user-attachments/assets/a144531d-d7ce-4558-adcf-067be059d26a" />
<img width="1196" height="466" alt="Screenshot 2026-03-06 155608" src="https://github.com/user-attachments/assets/03f49622-227a-497a-9653-fe399a14fc98" />
<img width="1180" height="800" alt="Screenshot 2026-03-06 162435" src="https://github.com/user-attachments/assets/7f8ca6ee-e9a5-49b1-8ea2-709984ef82ef" />


### Scenario 2: Account Lockout

**Priority:** High | **Department:** Security

I created a scenario where a user locked themselves out before a meeting. The agent verified identity through a secondary email, unlocked the AD account, and walked the user through setting up a password manager.

<img width="1203" height="463" alt="Screenshot 2026-03-06 163830" src="https://github.com/user-attachments/assets/c05a7db5-9a51-4893-9704-ab2425375c54" />
<img width="1180" height="816" alt="Screenshot 2026-03-06 164108" src="https://github.com/user-attachments/assets/ab521b1d-143f-415d-8032-565c21430f38" />



### Scenario 3: Email Not Syncing

**Priority:** Normal | **Department:** IT Support (Tier 1)

I simulated an Outlook issue showing "Disconnected" in the status bar. The agent re-entered Exchange credentials, checked server status, and cleared the Outlook cache by deleting the `.ost` file.

<img width="1202" height="473" alt="Screenshot 2026-03-06 164229" src="https://github.com/user-attachments/assets/f5cd2695-450d-4ad1-b424-3c0e5ed6a947" />
<img width="816" height="343" alt="Screenshot 2026-03-06 164300" src="https://github.com/user-attachments/assets/8a70a545-e51e-4880-b8c7-b1982ccd91f3" />


## Challenges and Lessons Learned

- **Apache/PHP not loading osTicket:** I had to enable `mod_rewrite` with `sudo a2enmod rewrite` and restart Apache. I learned that Apache modules need to be explicitly enabled even after PHP is installed.
- **MySQL permissions error during install:** I forgot to run `FLUSH PRIVILEGES` after the `GRANT` statement. MySQL caches privilege tables in memory and the changes simply do not apply until you flush them.
- **Setup directory left open:** I caught this during a self-review pass and made it a checklist item going forward. Leaving `/setup` accessible after install is a real vulnerability.
- **NSG rule not applying:** I found that HTTP traffic was still blocked because the rule got applied to the NIC instead of the NSG. This was a good lesson in understanding the difference between Azure NIC-level and NSG-level rules.

## Real-World Relevance

Most IT support teams run some version of this setup. Ticketing with SLA tracking, tiered agents, and department routing is the baseline for keeping support organized and accountable. I built this on Azure rather than a local machine to add the cloud layer, which is where most of these systems actually live now. Working through the VM provisioning, NSG rules, and network configuration made the underlying infrastructure feel a lot less abstract.
