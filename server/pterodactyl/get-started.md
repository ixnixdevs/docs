````markdown
🧩 Pterodactyl Panel Installation auf Debian 12 (Bookworm)

Diese Anleitung richtet sich an Systemadministratoren, die das Pterodactyl Panel auf einem frischen **Debian 12 Server** installieren möchten. Die Anleitung beinhaltet:

- ✅ Alle notwendigen Abhängigkeiten
- ✅ PHP 8.3 über das Sury-Repository
- ✅ Redis, MariaDB und NGINX
- ✅ Composer (v2)
- ✅ Vorbereitung des Systems für den Panel-Start

---
````
⚙️ System vorbereiten

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget gnupg2 ca-certificates lsb-release software-properties-common apt-transport-https unzip tar git
```

---

## 📦 PHP 8.3 & Extensions installieren

```bash
# PHP Sury Repository hinzufügen
sudo wget -qO /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list

# Paketquellen aktualisieren
sudo apt update

# PHP und Extensions installieren
sudo apt install -y php8.3 php8.3-{cli,fpm,common,openssl,gd,mysql,mbstring,tokenizer,bcmath,xml,curl,zip}
```

---

## 🛢️ MariaDB installieren (MySQL Alternative)

```bash
sudo apt install -y mariadb-server
```

---

## 🧠 Redis installieren

```bash
# Redis GPG Key hinzufügen
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

# Redis Repo hinzufügen
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

# Redis installieren
sudo apt update
sudo apt install -y redis-server
```

---

## 🌐 Webserver installieren (NGINX Beispiel)

```bash
sudo apt install -y nginx
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

# Dann in der MySQL/MariaDB-Konsole:
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'deinStarkesPasswort';
CREATE DATABASE panel;
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
EXIT;
```

---

## 🛠️ Panel vorbereiten

```bash
cp .env.example .env
COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --optimize-autoloader
php artisan key:generate --force
```

---

## ⚙️ Panel konfigurieren

```bash
php artisan p:environment:setup
php artisan p:environment:database
php artisan p:environment:mail
```

---

## 🗃️ Datenbank migrieren

```bash
php artisan migrate --seed --force
```

---

## 👤 Admin-User erstellen

```bash
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

# Füge folgende Zeile ein:
* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1
```

---

## 🔄 Queue Worker via systemd

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
# Worker aktivieren
sudo systemctl daemon-reexec
sudo systemctl enable --now pteroq.service
```

---

## ✅ Fertig!

Das Pterodactyl Panel ist nun installiert und bereit. Rufe nun die Domain oder IP deines Servers im Browser auf, um das Webinterface zu nutzen.

---

## 🔒 Wichtiger Hinweis

Notiere dir den Inhalt der `.env`-Variable `APP_KEY` – dieser ist sicherheitsrelevant. Ohne diesen Schlüssel sind Backups von verschlüsselten Daten unbrauchbar!

---
```
