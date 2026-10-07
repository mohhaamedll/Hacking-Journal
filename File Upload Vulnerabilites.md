## What Is File Upload

File Upload is a functionality that allows users to upload files such as PDFs, documents, JPGs, and PNGs to an application or the server when needed.

A **File Upload Vulnerability** can arise when an attacker is able to upload a file that is malicious or with an attacker controlled content into the server that do not implement proper security validations that restrict the safe upload of files.

File Upload Vulnerabilities are a **Server-Side Vulnerabilities** that can have an impact that varies from low to critical severity depending on what the attacker can upload, where it is stored, whether an attacker can access it and whether the server do execute it.

In the most severe cases, a file-upload vulnerability can potentially lead to **Remote Code Execution (RCE)**, but file upload vulnerabilities do not necessarily result in code execution.

----
## How Secure File Upload Validation Happens on the Server

```php 
<?php

	if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
	    http_response_code(405);
	    exit;
	}
	
	$file = $_FILES['image'] ?? null;
	
	if (!$file || $file['error'] !== UPLOAD_ERR_OK) {
	    http_response_code(400);
	    exit('Invalid upload');
	}
	
	/* 1. Limit size */
	$maxSize = 5 * 1024 * 1024; // 5 MB
	
	if ($file['size'] > $maxSize) {
	    http_response_code(400);
	    exit('File too large');
	}
	
	/* 2. Detect the actual file type */
	$finfo = new finfo(FILEINFO_MIME_TYPE);
	$mime = $finfo->file($file['tmp_name']);
	
	$allowed = [
	    'image/jpeg' => 'jpg',
	    'image/png'  => 'png',
	    'image/gif'  => 'gif'
	];
	
	if (!isset($allowed[$mime])) {
	    http_response_code(400);
	    exit('Invalid file type');
	}
	
	/* 3. Verify that it is actually a valid image */
	if (@getimagesize($file['tmp_name']) === false) {
	    http_response_code(400);
	    exit('Not a valid image');
	}
	
	/* 4. Generate our own filename */
	$extension = $allowed[$mime];
	$filename = bin2hex(random_bytes(16)) . '.' . $extension;
	
	/* 5. Store outside the web root */
	$destination = '/var/appdata/uploads/' . $filename;
	
	if (!move_uploaded_file($file['tmp_name'], $destination)) {
	    http_response_code(500);
	    exit('Upload failed');
	}
	
	echo 'Upload successful';
?>
```

#### Check Sequence:

1. Check for the file size.
2. Then use MIME (Multipurpose Internet Mail Extensions) to check for the actual file type uploaded, so if a file is uploaded as PNG, it can detect if it is really a PNG.
3. Then check if the detected MIME type is actually in the allowed list.
4. Check if the image is a valid image.

```
               uploaded file
                    │
                    ▼
            finfo examines it
                    │
                    ▼
             image/jpeg ?
               /       \
             YES        NO
              │          │
              ▼          ▼
            allow      reject
              │
              ▼
        server chooses
           ".jpg"
```
### File Upload payload example:

1. PHP one-liner could be used to read arbitrary files from the server's filesystem:
```php
	<?php echo file_get_contents('/path/to/target/file');?>
```
- **Impact** after uploading to the profile upload function a PHP file with this code when you load the page the image is fetched so if the server runs the PHP file as PHP and do not return it as an text then that is RCE from file upload.   

2. An attacker could upload an HTML file containing an XSS payload such as:

```html
<script>alert(document.domain)</script>
```

- **Impact:** If the application allows the uploaded file to be accessed and the browser renders its contents as HTML instead of treating it as a download or another non-executable content type, the JavaScript can execute in the context of the application's origin. This can result in a **Stored XSS** vulnerability.

---- 
## Common File Upload Vulnerabilities

- **Content-Type restriction bypass:** _The `Content-type:` can be blindly trusted by the server without checking on the actual data sent allowing an attacker to change the type to image `Content-Type: image/jpeg` bypassing the security mechanism implemented._
    
    For Practical implementation check the **Portswigger lab** on this scenario: [Portswigger](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-content-type-restriction-bypass)
    
- **Web shell upload via path traversal:** _If the file upload functionality is accepting the PHP malicious file but do not run it and only return it as `Content-Type: text/plain` because this endpoint do not execute code then you can look for another endpoint the execute scripts and use path traversal to fetch the malicious file you uploaded to run on this endpoint that execute scripts._
    
    For Practical implementation check the **Portswigger lab** on this scenario: [Portswigger](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-path-traversal)
    
- If developers did not block the attacker from uploading a `.htaccess` file and making it not overridable, an attacker might upload a file `.htaccess` that the **Apache server** sees it and apply the code in side which can be:
    
    - `AddType application/x-httpd-php .php` --> This code makes the **Apache server** execute `.php` files as a **PHP** in the server side for the directory that `.htaccess` was uploaded to.
        
    - Also an attacker can upload a code to `.htaccess` that that make a custom extensions be treated as PHP `AddType application/x-httpd-php .x` --> `.x` files will be treated as PHP.
        
    - Another file exist on **IIS** (Internet Information Services) named `web.config`.
        
  For Practical implementation check the **Portswigger lab** on this scenario: [Portswigger](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-extension-blacklist-bypass)
        
- If an application only accepts `.jpg` or `.png`, you can bypass it by **Obfuscating** the extension like:
    
    1. Provide multiple extensions. Depending on the algorithm used to parse the filename, the following file may be interpreted as either a PHP file or JPG image: `exploit.php.jpg`.
        
    2. Add trailing characters. Some components will strip or ignore trailing whitespaces, dots, and suchlike: `exploit.php.`
        
    3. Try using the URL encoding (or double URL encoding) for dots, forward slashes, and backward slashes. If the value isn't decoded when validating the file extension, but is later decoded server-side, this can also allow you to upload malicious files that would otherwise be blocked: `exploit%2Ephp`.
        
    4. Add semicolons or URL-encoded null byte characters before the file extension. If validation is written in a high-level language like PHP or Java, but the server processes the file using lower-level functions in C/C++, for example, this can cause discrepancies in what is treated as the end of the filename: `exploit.asp;.jpg` or `exploit.asp%00.jpg`.
        
    5. Try using multibyte unicode characters, which may be converted to null bytes and dots after unicode conversion or normalization. Sequences like `xC0 x2E`, `xC4 xAE` or `xC0 xAE` may be translated to `x2E` if the filename parsed as a UTF-8 string, but then converted to ASCII characters before being used in a path.
        
    6. if the application uses striping dangerous extensions if it is not recursive you can add a prohibited extension in site another one that it's removing make this reconnect like this `.p.phphp` -> `.php`
        
  For Practical implementation check the **Portswigger lab** on this scenario: [Portswigger](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-obfuscated-file-extension)
        
- If the application know the MIME from the file content you can use a tool like **EXIFTOOL** to make a PHP file have an JEPG data with this command: `exiftool -Comment="<?php echo file_get_contents('your-command')?>" original.jpg -o polyglot.php` so the when server read the file thinks it's a JEPG by passing the security but the server will see the file extension as a PHP and runs it as PHP.
    
    For Practical implementation check the **Portswigger lab** on this scenario: [Portswigger](https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-polyglot-web-shell-upload)
    
- Modern frameworks are more battle-hardened against these kinds of attacks. They generally don't upload files directly to their intended destination on the filesystem. Instead, they take precautions like uploading to a temporary, sandboxed directory first and randomizing the name to avoid overwriting existing files. They then perform validation on this temporary file and only transfer it to its destination once it is deemed safe to do so. That said, developers sometimes implement their own processing of file uploads independently of any framework. Not only is this fairly complex to do well, it can also introduce dangerous race conditions that enable an attacker to completely bypass even the most robust validation. For example, some websites upload the file directly to the main filesystem and then remove it again if it doesn't pass validation. This kind of behavior is typical in websites that rely on anti-virus software and the like to check for malware. This may only take a few milliseconds, but for the short time that the file exists on the server, the attacker can potentially still execute it. These vulnerabilities are often extremely subtle, making them difficult to detect during blackbox testing unless you can find a way to leak the relevant source code.
    
    For Practical implementation check the **Portswigger lab** on this scenario: [Portswigger](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-race-condition)
    
