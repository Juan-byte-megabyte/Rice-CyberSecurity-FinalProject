# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology


The following machines were identified on the network:

- Kali
  - **Operating System**: Kali Release 2020.1 / Kernel: Linux 5.4.0
  - **Purpose**: Attacker / pen test machine
  - **IP Address**: 192.168.1.90

- ELK
  - **Operating System**: Ubuntu 18.04
  - **Purpose**: Elasticsearch & Kibana Stack
  - **IP Address**: 192.168.1.100

- Target 1
  - **Operating System**: Debian GNU/Linux 8/v3.16.0-6
  - **Purpose**: WordPress host
  - **IP Address**: 192.168.1.110

- Target 2
  - **Operating System**: Debian GNU/Linux 8/v3.16.0-6
  - **Purpose**: WordPress host
  - **IP Address**: 192.168.1.115

- Capstone
  - **Operating System**: Ubuntu 18.04
  - **Purpose**: The Vulnerable Web Server
  - **IP Address**: 192.168.1.105


### Description of Targets

The target of this attack was: `Target 1` (192.168.1.110).

Two VMs on the network were vulnerable to attack due to weak security implementation services and administration: Target 1 (192.168.1.110) and Target 2 (192.168.1.115). However, only Target 1 is covered and was attacked.

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### Excessive HTTP Errors
Excessive HTTP Errors is implemented as follows:
  - **Metric**: 
    WHEN count() GROUPED OVER top 5 ‘http.response.status_code’
  - **Threshold**: IS ABOVE 400
  - **Vulnerability Mitigated**: Enumeration
  - **Reliability**: The alert is highly reliable. Measuring by error codes 400 and above will filter out any normal or successful responses. 400+ codes are client and server errors which are of more concern. Especially when considering these error codes going off at a high rate. It detected our wpscan.
  - ![alt text](https://github.com/Juan-byte-megabyte/Rice-CyberSecurity-FinalProject/blob/62b6033f244f6954663f38c8222f81fa37fe6099/Images/Defense%20Images/ExcessiveHTTPerrors.png)


#### HTTP Request Size Monitor
HTTP Request Size Monitor is implemented as follows:
  - **Metric**: WHEN sum() of http.request.bytes OVER all documents
  - **Threshold**: IS ABOVE 3500
  - **Vulnerability Mitigated**: Measurement of high traffic events; which could be an indicator of attack.
  - **Reliability**: This was highly reliable when John the Ripper was executed.
  - ![alt text](https://github.com/Juan-byte-megabyte/Rice-CyberSecurity-FinalProject/blob/62b6033f244f6954663f38c8222f81fa37fe6099/Images/Defense%20Images/httprequestsizemonitor.png)


#### CPU Usage Monitor
CPU Usage Monitor is implemented as follows:
  - **Metric**: WHEN max() OF system.process.cpu.total.pct OVER all documents
  - **Threshold**:IS ABOVE 0.5
  - **Vulnerability Mitigated**: John the Ripper
  - **Reliability**: Low and unreliable; the cpu usage monitor triggered even when not attacked.
  - ![alt text](https://github.com/Juan-byte-megabyte/Rice-CyberSecurity-FinalProject/blob/62b6033f244f6954663f38c8222f81fa37fe6099/Images/Defense%20Images/cpuusage.png)


### Suggestions for Going Further

- Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. For each vulnerability/exploit identified by the alerts above, suggest a patch. E.g., implementing a blocklist is an effective tactic against brute-force attacks.

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:
- Vulnerability 1: Weak and Easily Brute-Forced Passwords / SSH Login
  - Mitigation Techniques:
    - Set a custom SSH port [1]
    - **Why It Works**: By default, SSH is set to be listening on port 22. Almost all cyber attackers know that. Changing the default port 22, for example 846 offers an additional layer of security by obscurity.
    - Implement SSH Private/Public Keys to login via SSH and disable passwords. [2]
    - **Why It Works**: Each key is a large number with different mathematical properties. The Private Key is stored on the computer you login from, while the public key is stored on the .ssh/authorized_keys file on each computer you want to login to.
    - Allow Only Specific Clients (Connections)
    - **Why It Works**: implementing the following command: "iptables -A INPUT -p tcp -s YourIP --dport 22 -j ACCEPT". By creating a specific filter/rule in your firewall, you are opening SSH port only to your IP address.
- Vulnerability 2: WordPress User Enumeration [3] [4]
  - Mitigation Techniques: 
    - "Texas" 2-Step: Disable Scans and Block User Enumeration via .htcaccess
    - **Why It Works**: This 2 step process will add layers of security by:
    - Adding a code snippet to your theme's functions.php file:
    - ![alt text](https://github.com/Juan-byte-megabyte/Rice-CyberSecurity-FinalProject/blob/62b6033f244f6954663f38c8222f81fa37fe6099/Images/Defense%20Images/WordPress%20UserEnumeration%20Step%202.png)
    - Adding a code snippet to your site's root .htcaccess file (the file will need to be created if you don't originally have it setup)
      - ![alt text](https://github.com/Juan-byte-megabyte/Rice-CyberSecurity-FinalProject/blob/1761fd4ba0d784f4882f97b954fc0f2336e45f84/Images/Defense%20Images/WordPress%20UserEnumeration%20Step%201.png)
- Vulnerability 3: Weak wp-config.php security implementation
  - Mitigation Techniques:
  - Setting up proper file security permissions within user files [4] & Protect the wp-config.php file with .htcaccess file [6]
  - **Why It Works**: Setting up the proper file security permissions could have mitigated reading the file when we performed the cat/nano command. This left the wp-config.php that listed a password in plaintext.
  - ![alt text](https://github.com/Juan-byte-megabyte/Rice-CyberSecurity-FinalProject/blob/62b6033f244f6954663f38c8222f81fa37fe6099/Images/Defense%20Images/Wordpress%20File%20Security%20WP-Configphp.png)
  - Modify wp-config.php [6]
  - **Why It Works**: By creating a new config file outside of WWW access directory it is protected from foreing access or external attackers.
  - Setup the correct file permissions for WP-Config.php to 400 [6]
  - **Why it Works**: By performing chmod 400 on the file; user and groups have permission to only read and others will not be able to access the file.
- Vulnerability 4: Python Privilege Escalation
  - Mitigation Techniques: 
    - Remove sudo access for python for the user.
    - **Why It Works**: If this access is taken away, this method of privilege escalation is no longer susceptible to being vulnerable.

References:
1. https://www.logsign.com/blog/best-practices-for-security-in-ssh/
2. https://blog.devolutions.net/2017/4/10-steps-to-secure-open-ssh
3. https://perishablepress.com/stop-user-enumeration-wordpress/
4. https://www.hackingtutorials.org/web-application-hacking/hack-a-wordpress-website-with-wpscan/
5. https://www.wpwhitesecurity.com/protect-wordpress-wp-config-php-security/
6. https://www.getastra.com/blog/911/secure-wp-config-file/
7. https://www.hackingarticles.in/linux-privilege-escalation-using-exploiting-sudo-rights/


