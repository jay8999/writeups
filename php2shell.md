# Vulnerability Advisory: Unrestricted File Upload to Remote Code Execution (RCE)
CVE-2026-109618 + CVE-2026-109619

* **Severity:** Critical
* **CVSS Score:** 9.8 (CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
* **Affected Endpoint:** /upload.php

## 1. Overview & Business Impact
During the security assessment, an unrestricted file upload vulnerability was identified within the application's upload functionality. This flaw allows an unauthenticated attacker to upload and execute arbitrary server-side code (PHP). 

**Business Impact:** Successful exploitation grants the attacker full command-line access to the underlying web server. This can lead to complete confidentiality, integrity, and availability loss, including data theft, defacement, or use of the server as a pivot point to attack internal network assets.

---

## 2. Technical Findings & Enumeration
* **Service Discovery:** Enumeration using tools like `nmap` located open web ports (80/443) and identified the underlying stack.
* **Endpoint Discovery:** Directory brute-forcing using tools like `gobuster` revealed hidden file handling and administrative upload endpoints (e.g., `/upload.php`).
* **Vulnerability Identification:** Inspection of the upload form revealed that the application lacked rigorous input validation, relying on weak client-side checks or blacklists rather than validating file properties.

---

## 3. Proof of Concept (PoC) / Exploitation Steps
1. **Payload Crafting:** A malicious PHP payload was crafted to accept system commands via an HTTP GET parameter:
   ```php
   <?php system($_GET['cmd']); ?>

File Upload Bypass: By using a double extension (shell.jpg.php) or bypassing weak client-side filters, the payload was successfully uploaded to the server.

Triggering Execution: Navigating to the uploaded file's web-accessible path allowed remote command execution:

			http://target-ip/uploads/shell.php?cmd=whoami

Session Stabilization: This execution path was leveraged to spawn an interactive reverse shell back to the listener, confirming full OS-level access.

## 4. Remediation & Secure Code Example

To properly secure the application, implement the following controls:

    Whitelist File Validation: Check file headers (magic bytes) rather than relying on user-supplied extensions or Content-Type headers.

    Secure Storage: Store uploaded files outside the public web root directory.

    Disable Execution: Configure the web server to explicitly disable PHP parsing inside any user-upload directories.

    Least Privilege: Run the web server process under a low-privileged service account.

## Secure PHP Implementation Example:
    <?php
    // Configuration
    $uploadDir = '/var/www/secure_uploads/'; // Stored outside the public web root
    $maxFileSize = 2 * 1024 * 1024; // 2MB size limit$allowedMimeTypes = [
        'image/jpeg' => 'jpg',
        'image/png' => 'png'
    ];
    
    if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['userfile'])) {
        $file =$_FILES['userfile'];
    
        // 1. Check for basic upload errors
        if ($file['error'] !== UPLOAD_ERR_OK) {
            die('Upload failed with error code: ' . $file['error']);
        }
    
        // 2. Enforce file size limits
        if ($file['size'] >$maxFileSize) {
            die('File exceeds the 2MB size limit.');
        }
    
        // 3. Verify actual file type using magic bytes (prevent extension spoofing)
        $finfo = new finfo(FILEINFO_MIME_TYPE);$mimeType = $finfo->file($file['tmp_name']);
    
        if (!array_key_exists($mimeType,$allowedMimeTypes)) {
            die('Invalid file type. Only JPEG and PNG are allowed.');
        }
    
        // 4. Generate a cryptographically secure random filename
        $extension = $allowedMimeTypes[$mimeType];
        $safeFilename = bin2hex(random_bytes(16)) . '.' .$extension;
        $destination = $uploadDir .$safeFilename;
    
        // 5. Move the temporary file to the secure destination
        if (move_uploaded_file($file['tmp_name'],$destination)) {
            echo 'File uploaded successfully.';
        } else {
            echo 'Failed to save uploaded file.';
        }
    }
    ?>
