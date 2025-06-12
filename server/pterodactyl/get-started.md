# 🧩 Pterodactyl Panel Installation auf Debian 12 (Bookworm)

Diese Anleitung beschreibt die vollständige Installation des Pterodactyl Panels auf einem frischen Debian 12 Server.

---

## 🛠️ System vorbereiten

```bash
sudo apt update
sudo apt install -y curl ca-certificates apt-transport-https lsb-release gnupg unzip git software-properties-common
```

---

## 🐘 PHP 8.3 installieren (via Sury Repository)

Debian 12 nutzt das **Sury** Repository für aktuelle PHP-Versionen:

```bash
# Sury PHP Repository hinzufügen
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
curl -fsSL https://packages.sury.org/php/apt.gpg | sudo gpg --dearmor -o /usr/share/keyrings/sury.gpg
echo 'deb [signed-by=/usr/share/keyrings/sury.gpg] https://packages.sury.org/php/ bookworm main' | sudo tee /etc/apt/sources.list.d/php.list

# Repositories aktualisieren
sudo apt update

# PHP 8.3 + Extensions installieren
sudo apt install -y php8.3 php8.3-{cli,common,gd,mysql,mbstring,bcmath,xml,fpm,curl,zip}
```

---

## 🐬 MariaDB (MySQL-kompatibel) installieren

```bash
curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | sudo bash
sudo apt update
sudo apt install -y mariadb-server
```

---

## 🚀 Redis installieren

```bash
# Redis GPG Key und Repo hinzufügen
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

# Redis installieren
sudo apt update
sudo apt install -y redis-server
```

---

## 🌐 NGINX & Tools installieren

```bash
sudo apt install -y nginx tar unzip git
```

---

## 🎼 Composer v2 installieren

```bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

---

## 📁 Pterodactyl Panel herunterladen

```bash
mkdir -p /var/www/pterodactyl
cd /var/www/pterodactyl

curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
tar -xzvf panel.tar.gz
chmod -R 755 storage/* bootstrap/cache/
```

---

## 🧩 Datenbank einrichten

```bash
sudo mysql -u root -p

# In der MariaDB-Konsole:
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'deinStarkesPasswort';
CREATE DATABASE panel;
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
EXIT;
```

---

## ⚙️ Panel konfigurieren

```bash
cp .env.example .env
COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --optimize-autoloader
php artisan key:generate --force
php artisan p:environment:setup
php artisan p:environment:database
php artisan p:environment:mail
```

---

## 🗃️ Migration & Admin-User

```bash
php artisan migrate --seed --force
php artisan p:user:make
```

---

## 🔒 Datei-Rechte setzen

```bash
chown -R www-data:www-data /var/www/pterodactyl/*
```

---

## ⏱️ Cronjob einrichten

```bash
sudo crontab -e

# Füge folgendes ein:
* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1
```

---

## 🔄 Queue Worker (systemd-Service)

```bash
sudo nano /etc/systemd/system/pteroq.service
```

```ini
[Unit]
Description=Pterodactyl Queue Worker
After=redis-server.service

[Service]
User=www-data
Group=www-data
Restart=always
ExecStart=/usr/bin/php /var/www/pterodactyl/artisan queue:work --queue=high,standard,low --sleep=3 --tries=3
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reexec
sudo systemctl enable --now pteroq.service
```

---

## ✅ Fertig!

Das Pterodactyl Panel ist nun vollständig installiert. Rufe jetzt deine Server-IP oder Domain im Browser auf und melde dich mit deinem Admin-Konto an.

---

## 🔐 Wichtiger Hinweis

Deine `.env`-Datei enthält den `APP_KEY`, der für alle verschlüsselten Daten entscheidend ist. Sichere diesen separat – z. B. in einem Passwortmanager.

---
