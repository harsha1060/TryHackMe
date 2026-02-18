🚩 Writeup: Exploiting MD2PDF (Local File Read & SSRF)
======================================================

📝 Executive Summary
--------------------

In this challenge, I explored a web application designed to convert Markdown text into PDF files. By exploiting the way the server-side rendering engine handles HTML tags, I was able to perform a **Local File Read (LFR)** to leak system files and a **Server-Side Request Forgery (SSRF)** to access an internal-only admin panel.

* * * * *

🔍 Phase 1: Reconnaissance
--------------------------

The application is a simple, modern interface with a `CodeMirror` editor and a "Convert to PDF" button.

Looking at the source code, I saw the frontend sends the Markdown content to a `/convert` endpoint via a POST request. The server then returns a `blob` (the PDF).

### The Internal Hint

During the initial reconnaissance, I identified an internal IP address associated with the environment: **`10.49.139.86`**. In a CTF context, internal IPs like this usually point toward an SSRF target.

* * * * *

🛠️ Phase 2: Exploitation (Local File Read)
-------------------------------------------

Markdown is often rendered into HTML before being converted to PDF. If the PDF engine is misconfigured, it may allow the use of dangerous HTML tags like `<iframe>`, `<object>`, or `<img>`.

### The Payload

I tested for file inclusion by injecting an HTML `<iframe>` pointing to the Linux password file:

HTML

```
<iframe src="file:///etc/passwd" width="100%" height="500px"></iframe>

```

### The Result

The PDF generated successfully. When I opened it, it contained the full contents of the server's `/etc/passwd` file!

> **File Snippet:** `root:x:0:0:root:/root:/bin/bash` `www-data:x:33:33:www-data:/var/www:/bin/sh`

This confirmed that the backend (likely a tool like `wkhtmltopdf`) has read access to the local filesystem.

* * * * *

🌐 Phase 3: Pivoting (SSRF)
---------------------------

Now that I knew I could make the server "render" whatever I wanted, I turned my attention to the internal IP: **`10.49.139.86`**.

External users cannot visit this IP, but the **MD2PDF server can**.

### The SSRF Payload

I used another `<iframe>` to force the server to fetch its own internal resource:

HTML

```
<iframe src="http://10.49.139.86/admin" width="1000px" height="1000px"></iframe>

```

* * * * *

🏆 Conclusion & Remediation
---------------------------

By abusing the PDF rendering engine's ability to process local file protocols (`file://`) and internal network requests, I was able to bypass network restrictions.

### How to Fix This:

1.  **Disable Local File Access:** Configure the PDF engine (e.g., `--disable-local-file-access` in `wkhtmltopdf`) to prevent `file://` lookups.

2.  **Network Sandboxing:** The PDF generator should reside in an isolated network environment with no access to internal management IPs or metadata services.

3.  **Input Sanitization:** Strip dangerous HTML tags like `<iframe>`, `<script>`, and `<object>` before passing the Markdown to the renderer.
