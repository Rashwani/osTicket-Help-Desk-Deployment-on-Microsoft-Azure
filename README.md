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

```
Client Browser
    | HTTP (Port 80)
Azure NSG (Network Security Group)
    | Allowed: SSH (22), HTTP (80)
Ubuntu Server 22.04 LTS (Azure VM)
    |
Apache2 Web Server
    |
osTicket (PHP Application)
    |
MySQL Database
```

## Phase 1 — Azure VM Provisioning

### VM Configuration

I provisioned a `Standard_B1s` VM (1 vCPU, 1 GB RAM) running Ubuntu 22.04 LTS, which is free tier eligible.

### Network Security Group (NSG) Rules

I configured inbound rules to allow SSH (port 22) and HTTP (port 80) traffic to the VM.

## Phase 2 — LAMP Stack Installation

### NSG Confirmed + LAMP Install Started

I confirmed both SSH and HTTP inbound rules were active and then started installing the LAMP stack.

### Apache Verified + MySQL and PHP Install

I verified Apache was running by hitting the VM's public IP in a browser. From there, I installed MySQL and all the required PHP extensions.

## Phase 3 — MySQL Database Setup

### Database and User Created

I created the osTicket database and a dedicated user with full privileges:

```sql
CREATE DATABASE osticket;
CREATE USER 'osticketuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON osticket.* TO 'osticketuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Phase 4 — osTicket Installation

### Web Installer

I navigated to `http://YOUR_PUBLIC_IP/osticket/setup/` and went through the setup wizard, filling in system settings, admin account details, and database credentials.

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

### SLA Plans

I went to Admin Panel > Manage > SLA Plans and configured three tiers:

| SLA Plan | Grace Period | Schedule |
|----------|-------------|----------|
| High Priority SLA | 4 hours | 24/7 |
| Normal Priority SLA | 24 hours | 24/7 |
| Low Priority SLA | 72 hours | 24/7 |

### Help Topics and Agent Accounts

I created five Help Topics (Password Reset, Network/Wi-Fi Issues, VPN/Remote Access, Hardware Request, Account Lockout) and set up two agent accounts: Jane Smith (Tier 1) and John Doe (Senior/Full Access).

## Phase 6 — Ticket Workflow Simulation

To validate the system, I simulated three real-world support scenarios:

### Scenario 1: VPN Not Connecting

**Priority:** High | **SLA:** 4 hours | **Department:** Networking

I submitted a test ticket through the client portal reporting VPN error 800 after a Windows update. Jane Smith (Tier 1) picked up the ticket, set it to High priority, and added an internal note while investigating. She escalated to John Doe, who identified that Windows update `KB5034441` was conflicting with the VPN client. He rolled back the update via remote session and the VPN came back up.

### Scenario 2: Account Lockout

**Priority:** High | **Department:** Security

I created a scenario where a user locked themselves out before a meeting. The agent verified identity through a secondary email, unlocked the AD account, and walked the user through setting up a password manager.

### Scenario 3: Email Not Syncing

**Priority:** Normal | **Department:** IT Support (Tier 1)

I simulated an Outlook issue showing "Disconnected" in the status bar. The agent re-entered Exchange credentials, checked server status, and cleared the Outlook cache by deleting the `.ost` file.

## Challenges and Lessons Learned

- **Apache/PHP not loading osTicket:** I had to enable `mod_rewrite` with `sudo a2enmod rewrite` and restart Apache. I learned that Apache modules need to be explicitly enabled even after PHP is installed.
- **MySQL permissions error during install:** I forgot to run `FLUSH PRIVILEGES` after the `GRANT` statement. MySQL caches privilege tables in memory and the changes simply do not apply until you flush them.
- **Setup directory left open:** I caught this during a self-review pass and made it a checklist item going forward. Leaving `/setup` accessible after install is a real vulnerability.
- **NSG rule not applying:** I found that HTTP traffic was still blocked because the rule got applied to the NIC instead of the NSG. This was a good lesson in understanding the difference between Azure NIC-level and NSG-level rules.

## Real-World Relevance

Most IT support teams run some version of this setup. Ticketing with SLA tracking, tiered agents, and department routing is the baseline for keeping support organized and accountable. I built this on Azure rather than a local machine to add the cloud layer, which is where most of these systems actually live now. Working through the VM provisioning, NSG rules, and network configuration made the underlying infrastructure feel a lot less abstract.
