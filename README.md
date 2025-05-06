# Final Project: Carrying out DAST with OWASP ZAP

## Introduction

This project focuses on performing Dynamic Application Security Testing using OWASP ZAP on the open-source Damn Vulnerable Web Application (DVWA), a PHP/MySQL-based web app deliberately designed with common security flaws for educational purposes. OWASP ZAP is a widely used DAST tool that automates the detection of vulnerabilities in running web applications. In this project, DVWA was containerized and scanned using ZAP in both unauthenticated and authenticated contexts. Identified vulnerabilities were documented, and several were mitigated through web server hardening, particularly via modifications to Apache configuration files.

## Methods

### DVWA Setup

- **Container**: DVWA was run using Docker:
  ```bash
  docker run -d -p 8081:80 vulnerables/web-dvwa
  ```
- **Environment**: Apache2 on Debian inside the container, default login: `admin / password`

- **Files Modified**:
    - /etc/apache2/apache2.conf - to mitigate Dorectory Browsing
    - /etc/apache2/conf-enabled/security.conf - to mitigate Clickjacking and XSS attack
 
### Testing Methodology




