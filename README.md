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

The security testing process was performed in three stages using the OWASP ZAP tool to identify and assess vulnerabilities in the DVWA. All scans were executed locally against the DVWA instance running at http://127.0.0.1:8081.

1. Initial Default Scan (Unauthenticated)

The first test was a default scan with minimal configuration. Only the target URL was entered, and no authentication or custom Automation plan was used. This quick passive and active scan revealed 10 vulnerabilities, including:

- Missing Content Security Policy (CSP) header
- Missing X-Frame-Options header
- Directory browsing enabled (/dvwa/css/)
- Server version disclosure via the Server header
- Missing cookie attributes (HttpOnly, SameSite)
- Absence of X-Content-Type-Options
- Informational alerts related to session and authentication behavior

This served as a baseline scan and confirmed that DVWA was exposed to common web application misconfigurations. Full report can be found in `2025-05-03-ZAP-Report-.html`.

2. Authenticated Scan with Defined Automation Plan

To access authenticated sections of DVWA, the ZAP Authentication Tester was used. ZAP context was created to maintain the authenticated session. Video can be found in `img/authenticated_tester.mov`

A test plan was then defined with the following steps:

- Authentication context setup
- Spider
- AJAX Spider
- Passive Scan

More details about ZAP scan plan can be found in `zap_plan.yaml` file. This improved scan revealed 23 vulnerabilities, many of which were present in authenticated-only sections, including:

- CSP configuration issues (wildcard directives, unsafe inline styles)
- Application error disclosures
- Absence of Anti-CSRF tokens
- Debug error messages, private IP disclosure
- Cross-domain script inclusions
- More instances of missing security headers

Full report can be found in `2025-05-03-ZAP-Report-plan.html`. Video result of running ZAP plan can be found in `zap_before_fix.mov`

3. Mitigation and Re-Scanning

After addressing several vulnerabilities, including:
- Disabling directory browsing via Apache configuration
- Adding secure response headers (Content-Security-Policy, X-Frame-Options)
- Restricting access to static asset directories

The results showed a reduced set of 17 vulnerabilities, confirming that mitigation steps were effective. Full report can be found in `2025-05-03-ZAP-Report-less_vuln.html`. Video result of running ZAP plan can be found in `zap_after_fix.mov`

## Results and Discussion

### Mitigations Applied

The following vulnerabilities were identified using OWASP ZAP and fixed.

1. Application Error Disclosure & Directory Browsing 

To prevent users from listing directory contents and reduce error-based information leaks, the `apache2.conf` file was updated with stricter access rules:

```yaml
<Directory /var/www/>
	Options -Indexes 
	AllowOverride None
	Require all granted
</Directory>
```
This disables directory indexing and ensures that only explicitly allowed directories are accessible.

![](./img/Снимок%20экрана%202025-05-04%20в%2012.30.30.png)

2. Missing Anti-clickjacking Header

To defend against clickjacking attacks, the `security.conf` file was configured to include the X-Frame-Options header:

```yaml
Header set X-Frame-Options: "sameorigin"
```
This helps ensure that the application is not embedded in external sites via `<iframe>`, reducing the risk of UI redress attacks.

![](./img/Снимок%20экрана%202025-05-03%20в%2022.20.53.png)

3. CSP: style-src unsafe-inline & Content Security Policy (CSP) Header Not Set

A strict CSP header was introduced in `security.conf`:

```yaml
Header always set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'; frame-ancestors 'none';"
```

This significantly strengthens the application’s defense against XSS by:
- Blocking inline JavaScript and styles (unsafe-inline is removed).
- Preventing external scripts and objects from being loaded unless from the same origin.
- Disallowing the application to be framed (frame-ancestors 'none').

![](./img/Снимок%20экрана%202025-05-03%20в%2022.25.29.png)

## Conclusion
This project successfully demonstrated the process of running DAST using OWASP ZAP, identifying multiple real vulnerabilities in DVWA, and applying practical mitigations to make app more secure. While DVWA remains intentionally insecure for educational purposes, key risks like clickjacking, xss and information leakage were resolved through server hardening.