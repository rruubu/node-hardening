Linux II Lab 9 — Hardening and Maintenance

This lab introduces practical skills for hardening a Linux system, reducing unnecessary exposure, and maintaining a secure baseline through SSH hardening and automated patching.

The lab is split into two parts:

Lab 9.A — Measure and Reduce
Lab 9.B — SSH and Patching
Objectives

By the end of this lab, you should be able to:

Capture the before state of a system and compare it with the after state.
Identify and safely remove unnecessary services, packages, and accounts.
Justify each removal by explaining what the service or package exposed and what would break without it.
Harden SSH configuration and validate it before reloading services.
Configure ProxyJump through a router VM to reach a second host on the LAN.
Enable unattended security updates on Ubuntu and Rocky Linux.
Prove that automatic updates are functioning.
Verify the attack surface externally using port scanning and SSH authentication testing.
Environment

This lab is designed for a Linux environment containing at least:

A deliberately bloated and insecure VM.
A second VM or host for testing SSH access.
A router VM for bastion-style access.
Ubuntu 24.04 or Rocky Linux 9 systems, as appropriate.

You are expected to build an intentionally bloated VM yourself.

Deliverables

The following evidence must be submitted or retained:

before.txt
after.txt
A diff between before.txt and after.txt
A justification table for each removal
Both SSH daemon configuration files
A working ProxyJump configuration
Evidence that automatic updates ran successfully
Lab 9.A — Measure and Reduce
Purpose

This snapshot is required evidence for the Week 11 report.

The objective is to measure the initial system state, reduce unnecessary software and services, and document the resulting differences.

Step 1: Capture the Pre-Hardening Snapshot

Run the following command and save the output to ~/before.txt:

{
  ss -tlnp
  systemctl list-units --type=service --state=running
  dpkg -l | wc -l
  find / -perm -4000 -type f 2>/dev/null
} > ~/before.txt


This captures:

Listening network services.
Running system services.
Installed package count.
SUID binaries.

This evidence provides the baseline for comparison after remediation.

Step 2: Remove Unnecessary Services and Packages

The deliberately bloated VM may include:

A desktop environment.
Samba.
An FTP server.
telnetd.
An unused database.
Avahi.
CUPS.
Stale user accounts.

Example cleanup commands:

sudo systemctl disable --now cups avahi-daemon
sudo apt purge telnetd vsftpd samba
sudo apt autoremove --purge


Review installed packages and dependencies before confirming removals.

For every removal, document the answers to two questions:

What did this expose?
What would break without it?

A removal that cannot be justified is considered incorrect, even if it appears to be a sensible cleanup.

Step 3: Audit Accounts

Check for:

Accounts with UID greater than or equal to 1000.
Accounts with a valid shell that should not have one.
Accounts with no password set.
Expired accounts.

Use:

sudo passwd -S -a | grep -v " P "
sudo lastlog


Also review user accounts and shell assignments carefully. Remove or correct stale or unnecessary accounts where appropriate.

Step 4: Measure the Post-Hardening State

After cleanup, save the new state to ~/after.txt:

{
  ss -tlnp
  systemctl list-units --type=service --state=running
  dpkg -l | wc -l
  find / -perm -4000 -type f 2>/dev/null
} > ~/after.txt


Compare the before and after states:

diff -u ~/before.txt ~/after.txt

Checkpoint

You should be able to demonstrate:

A shorter ss -tlnp output, where applicable.
A clear diff between the two snapshots.
A justification table containing a row for each change.
Justification Table

Use a table like the following in your report or notes.

Change	What did this expose?	What would break without it?
Removed CUPS	Unused printing service increased the network-printing attack surface.	Local or remote printing could stop working.
Removed Avahi	Multicast service discovery exposed additional network functionality.	Local service discovery could fail in some LAN workflows.
Purged Telnet	Cleartext remote login could expose credentials and session data.	Legacy Telnet clients could no longer connect.
Purged vsftpd	An unnecessary FTP service increased file-transfer exposure.	FTP-based file transfers would stop working.
Purged Samba	SMB services could expose file-sharing functionality to the network.	Windows file sharing and SMB integration would stop working.
Disabled desktop environment	Unnecessary graphical software increased the installed software footprint.	GUI login and desktop workflows would be unavailable.

Add one row for every service or package actually removed. Explain both the security benefit and the functionality that could be lost.

Lab 9.B — SSH and Patching
Purpose

This part focuses on SSH hardening, secure remote access, and automated patching.

Important Rule: Never Test a Change With Your Only Active SSH Session

Before changing SSH configuration:

Never close the only working SSH connection.
Open a second SSH session before making changes.
Validate the configuration before reloading the service.
Test from the second session before disconnecting the original.

This is a required operational security practice.

Step 1: Harden sshd_config

Configure the SSH daemon securely on both VMs.

Validate the configuration before reloading:

sudo sshd -t


Reload the SSH daemon rather than restarting it:

sudo systemctl reload sshd


On Ubuntu, the SSH service may be named ssh instead of sshd. If necessary, use:

sudo systemctl reload ssh


Reloading applies configuration changes while normally preserving existing connections.

Step 2: Disable Password Authentication

After confirming that key-based login works, disable password authentication.

Ubuntu 24.04 may contain configuration drop-in files under /etc/ssh/sshd_config.d/ that affect the effective settings. Editing only /etc/ssh/sshd_config may not produce the expected result.

Inspect the relevant files:

ls /etc/ssh/sshd_config.d/
sudo grep -R "PasswordAuthentication" \
  /etc/ssh/sshd_config \
  /etc/ssh/sshd_config.d/


If the effective configuration allows password authentication, correct the conflicting setting.

Verify the effective value:

sudo sshd -T | grep passwordauthentication


The expected result is:

passwordauthentication no


Then test password authentication from a separate terminal:

ssh -o PreferredAuthentications=password \
    -o PubkeyAuthentication=no user@host


Replace user@host with the actual target account and hostname or IP address. The password-only authentication attempt should be refused.

Step 3: Configure ProxyJump Through the Router VM

Use the Windows SSH client configuration file at %USERPROFILE%\.ssh\config to create a bastion configuration that routes access to an internal LAN host through the router VM.

Example:

Host router
    HostName 10.10.10.1
    User admin
    IdentityFile ~/.ssh/id_ed25519

Host lan-b
    HostName 10.10.10.20
    User user
    ProxyJump router
    IdentityFile ~/.ssh/id_ed25519


Replace the example IP addresses, usernames, and key paths with the actual lab environment values.

Test the connection:

ssh lan-b


This demonstrates access to an internal host through an intermediate SSH bastion.

Step 4: Configure Unattended Security Upgrades
Ubuntu

Install and enable unattended upgrades:

sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades


Inspect the configuration:

sudo grep -R "Unattended-Upgrade" /etc/apt/apt.conf.d/

Rocky Linux

Install and enable automatic updates:

sudo dnf install -y dnf-automatic
sudo systemctl enable --now dnf-automatic.timer


Review the configuration file:

/etc/dnf/automatic.conf


Verify that the configured update behavior matches the intended security-update policy.

Step 5: Force an Update and Prove It Works

Do not assume that the service is working simply because it is enabled. Run the update mechanism and inspect the logs.

Ubuntu
sudo unattended-upgrade --dry-run
sudo unattended-upgrade
sudo journalctl -u unattended-upgrades -n 100 --no-pager

Rocky Linux
sudo dnf-automatic
sudo journalctl -u dnf-automatic -n 100 --no-pager


Retain evidence showing that the automatic update mechanism ran successfully. A dry run alone demonstrates a test, not necessarily a successful real update.

Step 6: Verify Hardening From the Outside

From the router VM, scan the target host:

nmap -sV 10.10.10.20


Replace the example address with the actual target IP.

Compare the discovered open ports with the baseline in before.txt.

Then test password authentication externally:

ssh -o PreferredAuthentications=password \
    -o PubkeyAuthentication=no user@host


The password-only attempt should be refused when password authentication is disabled.

Evidence to Keep

Retain evidence for all of the following:

before.txt
after.txt
Diff output
Service and package removal justifications
Account audit results
SSH configuration and relevant drop-in files
ProxyJump configuration
Automatic update logs or journal output
Nmap scan results
SSH password authentication refusal output
Final Notes

The key lesson is that security is not just about adding rules or toggling switches. It is about understanding what is exposed, identifying what is unnecessary, and validating the system before and after hardening.

A good final submission should demonstrate:

A measured before-and-after comparison.
Justified service and package removals.
Secure SSH configuration.
Working remote access through a bastion.
Automated patching with supporting evidence.
External verification of network exposure and authentication restrictions.
Submission Checklist

Before submitting, confirm that you have completed the following:

Captured before.txt.
Captured after.txt.
Recorded the diff.
Documented each removal with justification.
Hardened SSH on both VMs.
Validated the configuration with sshd -t.
Reloaded SSH without closing the active session.
Disabled password authentication.
Configured and tested ProxyJump.
Enabled unattended updates.
Demonstrated that automatic updates ran.
Verified external port exposure and password authentication refusal.
