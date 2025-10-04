# Hands-on
```index.html
<html>
	<head>
		<link rel="stylesheet" href="style.css" />
		<title> Tushar Seth </title>
	</head>

	<body>
		<h1> hello From Nginx </h1>
		<p> This is sample Nginx WebPage </p>
	</body>
</html>

```
```style.css
body{	
	background-color: #ff0000;
}
```

<img width="1366" height="607" alt="image" src="https://github.com/user-attachments/assets/be9daba4-3885-4e6a-a2d3-92074a02d6d1" />



# Nginx Guide

This document explains important concepts, installation steps, and commands for working with **Nginx** on Ubuntu and Docker(WSL).

# 1.Using Windows Powershell
<img width="1366" height="720" alt="image" src="https://github.com/user-attachments/assets/872a9289-348a-46b2-aab4-e9fd2a42dce1" />

# 2.Using WSL
<img width="1350" height="708" alt="image" src="https://github.com/user-attachments/assets/fffa69b3-6ccf-409b-b62c-55bf140d4594" />


# About Nginx
NGINX (pronounced "engine-x") is an open-source software that functions as a high-performance web server, reverse proxy, load balancer, and email proxy (IMAP, POP3, and SMTP). It is renowned for its ability to handle a large number of concurrent connections with minimal resource consumption, making it a popular choice for high-traffic websites and applications.

**->Key Features and Functions:**
**->Web Server**: NGINX excels at serving static content like HTML pages, images, and other files efficiently.

**->Reverse Proxy**: It acts as an intermediary for requests from clients to backend servers, enhancing security, performance, and flexibility.

**->Load Balancer**: NGINX distributes incoming network traffic across multiple backend servers to optimize resource utilization, maximize throughput, and ensure high availability.

**->Caching**: It can cache frequently accessed content to reduce server load and improve website speed.

**->Email Proxy**: NGINX can function as a proxy server for email protocols like IMAP, POP3, and SMTP. 

**->Event-driven Architecture**: Its non-threaded, event-driven architecture enables efficient handling of numerous concurrent requests, reducing CPU computation per request and client waiting times.

**->Scalability**: NGINX is highly scalable and can be configured to manage extensive connections and traffic demands.

**->Zero-Downtime Upgrades**: It allows for upgrades and configuration changes without requiring server downtime.

---

## Installation

###  On Ubuntu

1. Update package index:
   ```bash
   sudo apt update
   ```

2. Install Nginx:
   ```bash
   sudo apt install nginx -y
   ```

3. Verify installation:
   ```bash
   nginx -v
   ```
   Example output:
   ```
   nginx version: nginx/1.24.0 (Ubuntu)
   ```

4. Enable Nginx service:
   ```bash
   sudo systemctl enable nginx
   sudo systemctl start nginx
   ```

---

###  Using Docker

Run Nginx in a container (default port 80 mapped to host):

```bash
docker run --name my-nginx -p 80:80 -d nginx
```

To mount a custom HTML site:

```bash
docker run --name my-nginx -p 80:80 -v /path/to/html:/usr/share/nginx/html:ro -d nginx
```

Check logs:

```bash
docker logs my-nginx
```

Stop container:

```bash
docker stop my-nginx
```

Remove container:

```bash
docker rm my-nginx
```

---

##  Nginx File Structure (Ubuntu)

After installing Nginx, configuration files are located in `/etc/nginx/`:

```
/etc/nginx/
├── nginx.conf          # Main Nginx configuration file
├── sites-available/    # Available site configs
├── sites-enabled/      # Enabled site configs (symlinks to sites-available)
├── conf.d/             # Extra configs loaded automatically
├── snippets/           # Config snippets (reusable parts)
├── mime.types          # MIME type definitions
```

---

## Basic Configuration

Minimal `nginx.conf`:

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

## Virtual Hosts (Server Blocks)

Example site config in `/etc/nginx/sites-available/example`:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/example;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable with symlink:

```bash
sudo ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/
```

Reload:

```bash
sudo nginx -s reload
```

---

## Common Commands

```bash
# Check Nginx version
nginx -v

# Test configuration syntax
sudo nginx -t

# Start Nginx
sudo service nginx start

# Stop Nginx
sudo service nginx stop

# Restart Nginx
sudo service nginx restart

# Reload configuration without downtime
sudo nginx -s reload

# Check which process is using port 80
sudo lsof -i :80
```

---

## Troubleshooting

- **Port already in use**  
  ```bash
  sudo lsof -i :80
  sudo kill -9 <PID>
  ```

- **Invalid configuration**  
  ```bash
  sudo nginx -t
  ```

- **Logs**  
  ```
  /var/log/nginx/error.log
  /var/log/nginx/access.log
  ```

---

## SSL with Nginx

# Understanding SSL Certificates and the Encryption Flow

##  What is SSL?

**SSL (Secure Sockets Layer)** is a standard security technology used to establish an **encrypted link** between a web server and a browser.  
It ensures that all data transferred between the server and client remains **private, integral, and authenticated**.

Today, SSL has been replaced by a more secure version called **TLS (Transport Layer Security)** — but the term "SSL" is still widely used.

---

##  Why SSL is Important

- ✅ Protects sensitive data (login credentials, payment info, etc.)  
- ✅ Verifies the identity of the website (via a trusted certificate authority)  
- ✅ Prevents attackers from intercepting or altering data (man-in-the-middle attacks)  
- ✅ Builds user trust (the “padlock” icon in browsers)

---

## Key Concepts: Public Key & Private Key

SSL uses a concept called **Public Key Cryptography** (Asymmetric Encryption).

| Key Type | Description | Used For |
|-----------|--------------|----------|
| **Public Key** | Shared openly with anyone (in the SSL certificate). | Encrypting data or verifying digital signatures. |
| **Private Key** | Kept secret by the server. | Decrypting data or creating digital signatures. |

🧩 These two keys are **mathematically linked** — what one key encrypts, only the other can decrypt.

---

## Components of an SSL Certificate

An SSL certificate includes:
- The **domain name** it is issued for  
- The **organization** or **owner** details  
- The **public key**  
- The **issuer (Certificate Authority)**  
- The **validity period**  
- The **digital signature** of the CA  

---

## How SSL/TLS Works (Step-by-Step Flow)

Here’s the full handshake and data encryption process:

### **1. Client Hello**
- The browser (client) connects to a secure website (e.g., `https://example.com`).
- The browser sends a **ClientHello** message to the server.
- This includes the SSL/TLS version, supported encryption algorithms (cipher suites), and a random string of bytes.

### **2. Server Hello**
- The server responds with a **ServerHello**, selecting the strongest common encryption algorithm.
- It sends back its **SSL Certificate**, which contains:
  - The server’s **public key**
  - The **CA’s signature**
  - Domain and identity details

### **3. Certificate Verification**
- The browser verifies:
  - The certificate’s validity (date, CA, domain)
  - That it’s issued by a **trusted Certificate Authority (CA)**
  - The **digital signature** on the certificate

If all checks pass , the connection proceeds. Otherwise, the browser shows a **“Not Secure”** warning.

### **4. Key Exchange**
- The client generates a **session key** (used for symmetric encryption — faster than asymmetric).
- The session key is **encrypted with the server’s public key** and sent to the server.

### **5. Decryption by Server**
- The server uses its **private key** to decrypt the session key.
- Now both the client and server share the **same session key**.

### **6. Secure Communication Begins**
- Both sides now use the **shared session key** for symmetric encryption of all subsequent communication.
- Data exchanged (like login info, payments, etc.) is securely encrypted.

---

## Symmetric vs Asymmetric Encryption in SSL

| Type | Description | Used In |
|------|--------------|---------|
| **Asymmetric (Public/Private Keys)** | Uses two keys; slower but secure. | Used during handshake for key exchange. |
| **Symmetric (Session Key)** | Uses one shared key; faster. | Used after handshake for bulk data encryption. |

---

## Example Real-World Analogy

Imagine:
- The **Public Key** is a **locked mailbox** anyone can drop a message into.
- The **Private Key** is the **only key** that can open that mailbox.
- Once the secure session begins, both parties agree on a **shared lock (session key)** for faster communication.

---

## Certificate Authorities (CA)

A **Certificate Authority (CA)** is a trusted entity that issues SSL certificates.  
Examples: **DigiCert, Let’s Encrypt, GoDaddy, GlobalSign, Comodo**

They verify the organization’s identity before signing the certificate with their private key — so browsers can trust it.

---

##  Summary of SSL Flow

[1] Browser → Server: "ClientHello"
[2] Server → Browser: "ServerHello + SSL Certificate (Public Key)"
[3] Browser → CA: Verify certificate validity
[4] Browser → Server: Send Encrypted Session Key (with Public Key)
[5] Server: Decrypt Session Key using Private Key
[6] Secure Data Exchange: All communication encrypted with Session Key


---

## Types of SSL Certificates

| Type | Description |
|------|--------------|
| **Domain Validated (DV)** | Verifies domain ownership only. |
| **Organization Validated (OV)** | Verifies organization identity. |
| **Extended Validation (EV)** | Highest level of validation; shows company name in address bar. |
| **Wildcard SSL** | Secures a domain and all its subdomains. |
| **Multi-Domain SSL** | Secures multiple domains with a single certificate. |

---

## Summary

| Concept | Purpose |
|----------|----------|
| **SSL/TLS** | Secure communication protocol |
| **Public Key** | Shared with everyone; encrypts data |
| **Private Key** | Kept secret; decrypts data |
| **Session Key** | Used for faster symmetric encryption |
| **CA** | Verifies and issues SSL certificates |

---

## Example: HTTPS in Action

When you visit `https://yourbank.com`:
- Your browser and the bank’s server perform the SSL handshake.
- A secure encrypted tunnel is created.
- All data (like passwords, transactions) stays private and protected.

---

> **In short:** SSL Certificates make the internet safe by encrypting communication, verifying authenticity, and ensuring that your data reaches only the intended recipient.

---

Install Let’s Encrypt and configure HTTPS:

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d example.com -d www.example.com
```

---

## Summary

- Install via `apt` (Ubuntu) or `docker run nginx`.  
- Config files in `/etc/nginx/`.  
- `nginx.conf` → global config, `sites-available/` → virtual hosts.  
- Always test configs with `nginx -t` before reloads.  
- Logs are in `/var/log/nginx/`.  

---

With this guide, you can **install, configure, run, and troubleshoot Nginx** on both **Ubuntu and Docker**.
