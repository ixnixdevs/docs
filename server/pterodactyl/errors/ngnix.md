# 🌐 NGINX Setup für Pterodactyl Panel (Debian 12 - Bookworm)

Diese Anleitung richtet sich an Benutzer, die Pterodactyl unter **Debian 12 (Bookworm)** mit **NGINX** und **SSL via Let's Encrypt** betreiben möchten.

---

## 📦 1. NGINX installieren

```bash
sudo apt update
sudo apt install -y nginx
```

---

## 🧹 2. Standardseite deaktivieren

```bash
sudo rm /etc/nginx/sites-enabled/default
```

---

## 🛠️ 3. NGINX-Konfiguration erstellen

Erstelle die Datei:

```bash
sudo nano /etc/nginx/sites-available/pterodactyl.conf
```

Füge folgenden Inhalt ein und **ersetze `<domain>` mit deiner Domain** (z. B. `panel.example.com`):

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
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

---

## 🔗 4. Konfiguration aktivieren

```bash
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf
```

---

## 🔐 5. Let's Encrypt SSL-Zertifikat einrichten

Falls du noch kein Zertifikat hast:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d <domain>
```

> Ersetze `<domain>` mit deiner echten Domain. Beispiel: `panel.example.com`

---

## ✅ 6. Konfiguration testen und NGINX neustarten

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Wenn alles grün ist: ✅ Pterodactyl Panel ist nun über HTTPS erreichbar.

---

## 🧯 Fehlerbehebung

* Fehler beim Starten? → `sudo nginx -t` zeigt dir, wo der Fehler liegt.
* Zertifikat fehlt? → Stelle sicher, dass `certbot` erfolgreich ausgeführt wurde.
* PHP-FPM Socket stimmt nicht? → Prüfe, ob `/run/php/php8.3-fpm.sock` existiert.

---
