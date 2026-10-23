Linux II Lab 9 — Hardening and Maintenance
Overview

This lab demonstrates practical Linux system hardening and maintenance techniques, focusing on reducing unnecessary attack surface, securing SSH access, configuring bastion-based connections, and enabling automated security updates.

The work covers two areas:

Lab 9.A — Measure and Reduce: Capture system state before and after hardening, audit accounts, and document service and package removal decisions.
Lab 9.B — SSH and Patching: Harden SSH configuration, test password authentication restrictions, configure ProxyJump, and verify automated security updates.
Repository Contents
File or Directory	Description
before.txt	System state captured before hardening
after.txt	System state captured after hardening
diff.txt	Differences between the before and after snapshots
justification.md	Justification for service and package removal decisions
sshd_config	SSH daemon configuration evidence
00-hardening.conf	Additional SSH hardening configuration
sshd-effective.txt	Evidence of effective SSH daemon settings
password-refused.txt	Evidence of password authentication being refused
proxyjump-test.txt	ProxyJump connectivity test evidence
unattended-upgrades.txt	Evidence related to automatic security updates
nmap.txt	Network scan results
accounts-audit.txt	User and account audit evidence
ssh_config_demo/	SSH client configuration demonstration
Lab 9.A — Measure and Reduce

The first part establishes a baseline of the system before hardening and compares it with the resulting system state.

The baseline records:

Listening TCP services and ports.
Running systemd services.
Installed package count.
Files with the SUID permission bit.

The before-and-after snapshots are compared using a unified diff. This makes changes to the system easier to identify and review.

Unnecessary services, packages, and accounts are reviewed to reduce exposure. Removal decisions should explain both the security benefit and the functionality that would be lost.

See before.txt, after.txt, diff.txt, accounts-audit.txt, and justification.md for the corresponding evidence.

Lab 9.B — SSH and Patching
SSH Hardening

SSH configuration is reviewed to reduce the risk of unauthorized access. The evidence includes the SSH daemon configuration, additional hardening settings, and the effective configuration.

Configuration validation should be performed before reloading the SSH service:

sudo sshd -t


The effective settings can be inspected with:

sudo sshd -T


Key-based authentication is tested, and password authentication is checked to confirm that it is refused where configured.

ProxyJump

ProxyJump provides a way to connect to a host on an internal network through an intermediate router or bastion host.

The repository includes demonstration configuration and test evidence for this connection method.

Automated Security Updates

Unattended security updates help keep a system protected against known vulnerabilities by applying eligible updates automatically.

The repository includes evidence related to the update configuration and execution. Service status and logs should be reviewed to verify that the mechanism is operating as expected.

External Exposure Verification

Network scanning is used to identify services exposed by the target system. The results can be compared with the initial baseline to assess changes in the attack surface.

SSH authentication testing provides an additional check that password-based access is refused when disabled.

Evidence and Validation

The repository contains supporting artifacts for the lab's main objectives:

Before-and-after system snapshots.
System state comparison.
Service and package removal justification.
Account audit evidence.
SSH configuration evidence.
Password authentication refusal evidence.
ProxyJump test evidence.
Automated update evidence.
Network scan results.

These files provide a record of the checks performed and support review of the hardening process.

Conclusion

This lab demonstrates a measurement-driven approach to Linux hardening. Recording the initial system state, making justified changes, validating SSH configuration, checking automated updates, and testing exposed services helps make security improvements observable and auditable.

The goal is not simply to remove software or change configuration settings, but to understand the security impact of each change and retain evidence that supports the results.

Lab: Linux II — Lab 9
Topics: Linux hardening, SSH security, ProxyJump, account auditing, automated patching, and network exposure verification.
