**CTF Write-up: TryHackMe - LFI Challenge**
-------------------------------------------

### **1\. Initial Discovery & Enumeration**

The challenge began by identifying a web application that used a URL parameter to load different pages. The URL structure looked like this: `http://<target-ip>/index.php?page=home.php`

This "page" parameter is a classic entry point for File Inclusion vulnerabilities. To test if the application was properly sanitizing inputs, I attempted to read the `/etc/passwd` file, which exists on all Linux systems and is world-readable.

### **2\. Exploitation: Testing for LFI**

I used **Directory Traversal** characters (`../`) to escape the web server's root directory (`/var/www/html`). By "over-traversing," I ensured that I would reach the system root regardless of the current folder's depth.

**Payload:** `?page=../../../../../../../../etc/passwd`

**Result:** The server successfully returned the contents of the `/etc/passwd` file:

Plaintext

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
...
www-data:x:33:33:www-data:/var/www:/bin/sh
...

```

This confirmed the **Local File Inclusion (LFI)** vulnerability was present.

### **3\. Path Traversal Mechanics**

In Linux, the filesystem is a tree starting at `/`. Using `..` moves the path up one level. Because the system prevents moving "above" the root, a long chain of `../` will always land you at `/`, no matter how deep the starting directory is.

-   **Starting Point:** `/var/www/html/`

-   **Step 1 (`../`):** `/var/www/`

-   **Step 2 (`../`):** `/var/`

-   **Step 3 (`../`):** `/` (Root)

-   **Step 4+ (`../`):** Remains at `/`

### **4\. Capturing the Flag**

With the ability to read files from the root of the filesystem, I targeted common flag locations. In many TryHackMe challenges, the flag is located at the root directory for simplicity.

**Final Payload:** `?page=../../../../../../../../flag.txt`

By sending this request, the application included the `flag.txt` file from the system root and rendered its contents in the browser, successfully completing the challenge.

* * * * *

### **Key Takeaways**

-   **Vulnerability:** Insecure implementation of PHP `include()` or `file_get_contents()`.

-   **Bypass:** Using directory traversal (`../`) to break out of the intended directory.

-   **Mitigation:** Developers should use **Allow-listing** (only allow specific filenames) or use `basename()` to strip directory paths from user input.
