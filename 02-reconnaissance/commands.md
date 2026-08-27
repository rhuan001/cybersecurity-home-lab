# Reconnaissance Commands

## Host Discovery

Command used:

`sudo nmap -sn 192.168.56.0/24`

Result:
- `192.168.56.1`
- `192.168.56.10`
- `192.168.56.20`
- `192.168.56.30`
- `192.168.56.50`

Wireshark was used on interface `eth1` to observe the ARP traffic generated during host discovery.

![Nmap Host Discovery](../01-setup/screenshots/11-nmap-host-discovery.png)

![Wireshark ARP Discovery](../01-setup/screenshots/12-wireshark-arp-discovery.png)

## TCP SYN Scan

Command used:

`sudo nmap -sS 192.168.56.1 192.168.56.10 192.168.56.30 192.168.56.50`

The scan showed that `192.168.56.30` exposed significantly more services than the other hosts and was selected for deeper enumeration.

![Nmap SYN Scan](../01-setup/screenshots/13-nmap-syn-scan-active-hosts.png)

## Service and Version Detection

Command used:

`sudo nmap -sV 192.168.56.30`

Relevant services identified:

- FTP — `vsftpd 2.3.4`
- SSH — `OpenSSH 4.7p1`
- Telnet
- HTTP — `Apache 2.2.8`
- SMB — `Samba 3.0.20`
- NFS
- MySQL
- PostgreSQL
- VNC
- IRC — `UnrealIRCd`
- Tomcat — `5.5`
- Bind shell — port `1524`

![Nmap Service Version Scan](../01-setup/screenshots/14-nmap-service-version-scan.png)

## Detailed Enumeration

Command used:

`sudo nmap -sC -sV 192.168.56.30`

The detailed scan revealed additional information such as:

- Anonymous FTP login enabled
- SMB signing disabled
- SSLv2 support on SMTP
- Multiple legacy services exposed
- Metasploitable root shell available on port `1524`
- Apache Tomcat available on port `8180`
- UnrealIRCd available on port `6667`

The results confirmed that `192.168.56.30` presents the largest attack surface in the lab and was selected as the primary target for the exploitation phase.

![Metasploitable2 Detailed Enumeration](../01-setup/screenshots/15-metasploitable2-detailed-enumeration.png)
