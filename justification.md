| Change | What did this expose? | What would break without it? |
|---|---|---|
| Disabled/purged CUPS (+cups-browsed) | Print service and network printer discovery (631/tcp, UDP browsing) | Local/network printing |
| Disabled/purged Avahi | mDNS/multicast service discovery (5353/udp), information leakage on LAN | .local name resolution and auto-discovery |
| Purged telnetd | Cleartext remote login, credential sniffing | Legacy telnet clients can't connect |
| Purged vsftpd | Cleartext FTP service (21/tcp), anonymous/credential attack surface | FTP file transfers |
| Purged Samba | SMB/NetBIOS exposure (139/445), historically wormable | Windows file/printer sharing |
| Deleted account `staleuser` | Unused account with login shell and no owner (stale credential) | Nothing, account was unused |
| Desktop environment | (not present in WSL; on a real VM: X/display manager, many extra packages) | GUI login |
