

---

# README.md

## Nautilus PHP Application Deployment (Nginx + PHP-FPM 8.3)

### Project Status

✅ **Successfully completed**

This project documents the successful deployment of a PHP-based application on **Nautilus infrastructure (Stratos DC)** using **Nginx** and **PHP-FPM 8.3**, following production support requirements.

---

## Environment

* **App Server:** stapp01
* **Web Server:** Nginx
* **PHP Version:** PHP-FPM 8.3
* **Socket Type:** UNIX socket
* **Listen Port:** 8093
* **Document Root:** `/var/www/html`

---

## Requirements Implemented

* Nginx installed and configured to listen on **port 8093**
* PHP-FPM **version 8.3** installed
* PHP-FPM configured to use UNIX socket

  ```
  /var/run/php-fpm/default.sock
  ```
* Nginx and PHP-FPM integrated via socket
* Application tested successfully using `curl`

---

## Implementation Steps

### 1. Install Nginx

```bash
sudo yum install -y nginx
```

---

### 2. Configure Nginx

Edit:

```bash
sudo vi /etc/nginx/nginx.conf
```

Server block:

```nginx
server {
    listen 8093;
    server_name localhost;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

---

### 3. Enable and Install PHP-FPM 8.3

```bash
sudo dnf module reset php -y
sudo dnf module enable php:8.3 -y
sudo yum install -y php php-fpm
```

---

### 4. Configure PHP-FPM Socket

Create socket directory:

```bash
sudo mkdir -p /var/run/php-fpm
```

Edit pool config:

```bash
sudo vi /etc/php-fpm.d/www.conf
```

⚠️ **Important: Remove the `;` at the beginning of these lines**

```ini
listen = /var/run/php-fpm/default.sock
user = nginx
group = nginx
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

> Leaving the `;` will comment the configuration and prevent PHP-FPM from creating the socket.

---

### 5. Set Permissions

```bash
sudo chown -R nginx:nginx /var/www/html
```

---

### 6. Start and Enable Services

```bash
sudo systemctl start php-fpm
sudo systemctl enable php-fpm

sudo systemctl start nginx
sudo systemctl enable nginx
```

---

### 7. Validation

Check Nginx configuration:

```bash
sudo nginx -t
```

Verify socket creation:

```bash
ls -l /var/run/php-fpm/default.sock
```

Test application from jump host:

```bash
curl http://stapp01:8093/index.php
```

---
## Screenshots
![Screenshot](screenshots/screenshots_nginx-php-success.png)(https://raw.githubusercontent.com/dr-musa-bala/Nautilus-PHP-Application-Deployment-/main/screenshots/screenshots/screenshots_nginx-php-success.png)   


## Result

* PHP files are processed correctly
* UNIX socket communication confirmed
* Application accessible via required port
* Task completed according to Nautilus standards

---

## Key Lessons Learned

* PHP-FPM configuration lines **must be uncommented** (remove `;`)
* Socket permissions are critical for Nginx ↔ PHP communication
* Correct PHP module version must be enabled before installation

---

## Author

**Deployment completed successfully**
DevOps / Linux Administration Practice (KodeKloud – Nautilus)

---
