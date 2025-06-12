### 📄 `README.md` – NGINX-Konfiguration für Pterodactyl unter Debian 12

````markdown
# 🌐 NGINX Konfiguration für Pterodactyl Panel (Debian 12)

Diese Anleitung beschreibt die **richtige Einrichtung von NGINX** als Webserver für das Pterodactyl Panel unter **Debian 12 (Bookworm)**, inklusive **automatischer Weiterleitung auf HTTPS** und **SSL-Zertifikaten via Let's Encrypt**.

---

## 🔹 Schritt 1: Standard-NGINX-Konfiguration entfernen

```bash
sudo rm /etc/nginx/sites-enabled/default
````

---

## 🔹 Schritt 2: Neue Konfigurationsdatei erstellen

Erstelle eine neue Datei namens `pterodactyl.conf`:

```bash
sudo nano /etc/nginx/sites-available/pterodactyl.conf
```

Füge folgenden Inhalt ein. **Ersetze `<domain>` mit deiner tatsächlichen Domain** (z. B. `panel.example.com`):

```nginx
server {
    listen 80;
    server_name <domain>;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name <domain>;

    root /var/www/pterodactyl/public;
    index index.php;

    access_log /var/log/nginx/pterodactyl.app-access.log;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;

    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;

    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header Content-Security-Policy "frame-ancestors 'self'";
    add_header X-Frame-Options DENY;
    add_header Referrer-Policy same-origin;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

---

## 🔹 Schritt 3: Konfiguration aktivieren

Erstelle einen symbolischen Link in das Verzeichnis `sites-enabled`, um die Konfiguration zu aktivieren:

```bash
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf
```

> 🛑 Hinweis: Dieser Schritt ist **nur auf Debian-basierten Systemen** erforderlich – **nicht** bei RHEL, Rocky Linux oder AlmaLinux!

---

## 🔹 Schritt 4: NGINX neu starten

```bash
sudo systemctl restart nginx
```

---

## ✅ Fertig!

NGINX ist jetzt vollständig für Pterodactyl unter Debian 12 eingerichtet – inklusive:

* **HTTPS-Weiterleitung**
* **SSL-Zertifikat via Let's Encrypt**
* **PHP 8.3 FPM**
* **Sicherheits-Header**

Wenn du noch kein SSL-Zertifikat hast, kannst du eines mit [Let's Encrypt](https://certbot.eff.org/instructions?ws=nginx&os=debianbookworm) via Certbot erstellen. Bei Bedarf kann ich dir auch diese Schritte bereitstellen.

---
