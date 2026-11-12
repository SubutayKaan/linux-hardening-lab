# Linux Hardening Lab: Automated Security Baseline

## 📌 Purpose
This project demonstrates a complete security hardening process for an Ubuntu Server 24.04 LTS environment. The goal was to measure the baseline security posture, apply industry-standard hardening controls (CIS Benchmark aligned), and automate the entire process using Ansible (Infrastructure as Code).

## 🏗️ Architecture & Environment
- **Host:** macOS (Intel)
- **Hypervisor:** VirtualBox
- **Guest OS:** Ubuntu Server 24.04.5 LTS (Headless)
- **Automation:** Ansible Core (via pipx)

## 🛠️ Tools & Technologies Used
- **Lynis:** Security auditing and vulnerability scanning.
- **Ansible:** Configuration management and automation.
- **Auditd:** System auditing and logging.
- **AIDE:** File integrity monitoring.
- **UFW:** Uncomplicated Firewall.
- **PAM (libpam-pwquality):** Password policy enforcement.

## 📊 Results (Before & After)
| Phase | Lynis Hardening Index | Description |
| :--- | :--- | :--- |
| **Baseline (Clean Install)** | **57** | Default Ubuntu installation, no firewall, password SSH enabled, vulnerable packages present. |
| **Manual Hardening** | **70** | Patches applied, UFW enabled, SSH key-only auth, Sysctl, Auditd, PAM, and AIDE configured manually. |
| **Automated (Ansible)** | **70** | All manual steps codified into idempotent Ansible playbooks. |

## 🔍 Key Findings & Remediations
| Finding (Lynis Code) | Root Cause | Remediation |
| :--- | :--- | :--- |
| **PKGS-7392** (Vulnerable packages) | 5 packages (bind9, libpcre2, etc.) had pending security updates. | Applied `apt upgrade`, enabled `unattended-upgrades`. |
| **FIRE-4512** (No firewall) | UFW was inactive. | Enabled UFW with default deny incoming, allowing only SSH. |
| **SSH-7408** (Weak SSH config) | Password authentication enabled, root login allowed. | Deployed `/etc/ssh/sshd_config.d/01-hardening.conf` via Ansible. |
| **AUTH-9262** (Weak passwords) | No password quality module installed. | Installed `libpam-pwquality`, enforced 12+ char length and complexity. |
| **ACCT-9628** (No auditing) | Auditd not configured. | Deployed custom rules to monitor `/etc/passwd`, `sudoers`, and `execve`. |

## 🚫 Intentionally Not Applied (Engineering Decisions)
*   **GRUB Password (BOOT-5122):** Omitted to prevent lockout scenarios in headless remote management.
*   **LUKS Disk Encryption:** Omitted for remote reboot automation capabilities.
*   **SSH Port Change:** Omitted as "security by obscurity" provides no real defense against port scanning.

## 🚀 How to Run (Ansible)
To apply the full security baseline to a new Ubuntu server:
```bash
ansible-playbook site.yml -K

## 🧠 Lessons Learned & Challenges Overcome

- **Infrastructure as Code (IaC) Transition:** Moving from manual, ad-hoc terminal commands to idempotent Ansible playbooks. I learned how Ansible's declarative nature prevents configuration drift.
- **Ansible `copy` vs `blockinfile`:** I initially used the `copy` module to deploy the AIDE configuration, which overwrote the default Ubuntu configuration file and caused a `return code 15` error. I learned to use `blockinfile` to safely append rules without destroying default settings.
- **Handling Modern OS Quirks:** Ubuntu 24.04 changed the AIDE package structure. I had to adapt my playbook to use `dpkg -L` to locate the default configuration file and restore it using `remote_src`.
- **Ansible Handlers:** Learned the importance of `notify` and `handlers`. Restarting the SSH service only when its configuration file actually changes prevents unnecessary connection drops.
- **Legacy Hardware Constraints:** Ran into a build error with the `cryptography` Python package on an Intel Mac. Solved it by pinning the package version (`cryptography<49`) during the Ansible installation.
