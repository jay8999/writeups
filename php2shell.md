# Reconnaissance & Enumeration

Service Discovery: Attackers scan the target host using tools like nmap to locate open web ports (such as 80 or 443) and identify the underlying web server software (like Apache or Nginx) and scripting languages (such as PHP).

Directory Brute-Forcing: Tools like gobuster or dirsearch locate hidden administrative panels, upload endpoints, or backup files (e.g., upload.php, admin/, config.php.bak).

Vulnerability Identification: Attackers inspect input fields that handle file uploads or file inclusions (include, require, move_uploaded_file). Unrestricted file upload points or local file inclusion (LFI) endpoints lacking proper input validation indicate a high risk of remote code execution.

# Exploitation

Malicious Payload Crafting: The attacker creates a small PHP script containing command-execution functions, commonly:

    <?php system($_GET['cmd']); ?>

File Upload Bypass: If the application relies solely on client-side validation or weak blacklisting (e.g., only blocking .php extensions), the attacker bypasses filters using alternative extensions (.php5, .phtml) or double extensions (shell.jpg.php).

Triggering Execution: Once the payload is successfully uploaded and its web-accessible path is discovered, the attacker executes system commands by issuing an HTTP request containing the command parameter:
Plaintext

    http://target-ip/uploads/shell.php?cmd=whoami

Establishing a Reverse Shell: The attacker leverages the command execution to spawn an interactive reverse shell back to their listening machine (using netcat, bash, or python scripts), securing persistent command-line access to the underlying operating system.

# Remediation

Strict File Type Validation: Implement robust whitelist-based validation checking file headers (magic bytes) rather than relying on file extensions or user-supplied Content-Type headers.

Secure Storage Configuration: Store uploaded files outside the web root directory or configure the web server (Apache/Nginx) to explicitly disable script execution (PHP parsing) inside user-upload directories.

Input Sanitization and Disabling Dangerous Functions: Sanitize all user inputs and disable dangerous execution functions (system, exec, passthru, shell_exec) in the php.ini configuration file if they are not required by the application.

Principle of Least Privilege: Run the web server process under a low-privileged service account to limit the impact and prevent complete system compromise if an upload vulnerability is exploited.

# Secure Example:

    <?php
    // Configuration
    $uploadDir = '/var/www/secure_uploads/'; // Stored outside the public web root
    $maxFileSize = 2 * 1024 * 1024; // 2MB size limit
    $allowedMimeTypes = [
        'image/jpeg' => 'jpg',
        'image/png' => 'png'
    ];
    
    if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['userfile'])) {
        $file = $_FILES['userfile'];
    
        // 1. Check for basic upload errors
        if ($file['error'] !== UPLOAD_ERR_OK) {
            die('Upload failed with error code: ' . $file['error']);
        }
    
        // 2. Enforce file size limits
        if ($file['size'] > $maxFileSize) {
            die('File exceeds the 2MB size limit.');
        }
    
        // 3. Verify actual file type using magic bytes (prevent extension spoofing)
        $finfo = new finfo(FILEINFO_MIME_TYPE);
        $mimeType = $finfo->file($file['tmp_name']);
    
        if (!array_key_exists($mimeType, $allowedMimeTypes)) {
            die('Invalid file type. Only JPEG and PNG are allowed.');
        }
    
        // 4. Generate a cryptographically secure random filename
        $extension = $allowedMimeTypes[$mimeType];
        $safeFilename = bin2hex(random_bytes(16)) . '.' . $extension;
        $destination = $uploadDir . $safeFilename;
    
        // 5. Move the temporary file to the secure destination
        if (move_uploaded_file($file['tmp_name'], $destination)) {
            echo 'File uploaded successfully.';
        } else {
            echo 'Failed to save uploaded file.';
        }
    }
    ?>
