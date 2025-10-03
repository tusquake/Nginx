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

## SSL with Nginx (Optional)

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
