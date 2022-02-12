# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services

A netdiscover scan revealed the below IP addresses to be on the network:

```bash
$ netdiscover -r 192.168.1.255-16
```

![Pen test - Kali netdiscover -r 192 168 1 255-16](https://user-images.githubusercontent.com/88017838/153582890-fdd92bcc-9d61-4bb7-9081-01d59983567f.PNG)

Nmap scan results for each machine reveal the below services and OS details:

```bash
$ nmap -sV 192.168.1.110
```

![Pen test - Kali nmap -sV 192 168 1 110](https://user-images.githubusercontent.com/88017838/153583803-da04ca70-fe0a-477d-8b35-e5a1054eeaa5.PNG)


This scan identifies the services below as potential points of entry:
- Target 1
  - Port 22/tcp   open  ssh OpenSSH 6.7p1 Debian5+deb8u4                     
  - Port 80/tcp   open  http Apache httpd 2.4.10 ((Debian))
  - Port 111/tcp  open  rpcbind 2-4 (RPC #100000)
  - Port 139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
  - Port 445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X

Common Vulnerabilites and Exposures:

  - CVE-2021-28041 open SSH (Severity-7.1 High)
  - CVE-2017-15710 Apache httpd 2.4.10 (Severity-7.5 High)
  - CVE-2017-8779 rpcbind able to be exploited to initiate denial of service attack (Severity-7.5 High)
  - CVE-2017-7494 Samba remote code execution vulnerability (Severity-10 Critical)

### Critical Vulnerabilities:

The following vulnerabilities were found on Target 1:

- User Enumeration (Wordpress)
  - Able to obtain username details for michael and steven via 'wpscan'
  - Can then initiate brute force attacks to crack passwords for these users
  
- Weak Password
  - Brute force tools are able to easily crack user's password (password for user michael was 'michael')
  - Once the password was cracked, the attacker is able to SSH into the server

- Sensitive Data Exposure
  - Able to find a file with root user access details to MySQL database
 
- Unsalted Password Hashes
  - Password hashes located by viewing MySQL table data
  - These were unsalted and able to be cracked using 'John the Ripper' tool

- User Privilege Escalation
  - Able to utilise user steven's sudo privileges for python to escalate to root


### Exploitation

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:

- Target 1

  - `flag1.txt`:{b9bbcb33e11b80be759c4e844862482d}
  
    - **Exploit Used**
      - WPScan was used to enumerate users on the Wordpress site for Target 1
      - wpscan --url 192.168.1.110/wordpress --enumerate vp,u

![Pen test - Kali WPS scan command](https://user-images.githubusercontent.com/88017838/153717305-6f93c63e-b549-4056-b2ce-85e0a36f9ae6.PNG)
![Pen test - Kali WPScan Users identified](https://user-images.githubusercontent.com/88017838/153717542-e785b810-839e-471f-9ee7-4b5890003b9e.PNG)

- SSH using michael's credentials:
  - Focusing on the user michael, I was able to run a Hydra brute force to quickly obtain his password, which as indicated below is a weak and easily crackable one
  - hydra -l michael -P /usr/share/wordlists/rockyou.txt 192.168.1.110 ssh -t 4
  - Using michael's credentials we can then SSH into 192.168.1.110 and located flag1 by following below steps:
  - cd /var/www/html
  - ls -l
  - after investigating multiple files, the flag1 was located via nano services.html
      
![Pen test - Kali Hydra michael](https://user-images.githubusercontent.com/88017838/153717654-754970ea-af9a-4c40-8590-47abebbd7194.PNG)
![image](https://user-images.githubusercontent.com/88017838/153718079-8aa99bb7-b2c8-4a1a-9f90-d25bed0ae648.png)


  - `flag2.txt`:{fc3fd58dcdad9ab23faca6e9a36e581c}
  
    - **Exploit Used**
      - While still logged in as michael via SSH I was able to locate flag2 by following the below
      - cd /var/www
      - ls -l
      - cat flag2.txt

![image](https://user-images.githubusercontent.com/88017838/153718231-699b0a91-8495-4810-923e-253e78175acc.png)


   - `flag3.txt`:{afc01ab56b50591e7dccf93122770cd2}

