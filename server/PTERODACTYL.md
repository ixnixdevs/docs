#######################################################################
# 🛠️ Pterodactyl v1.x Installation – Debian 12 (Bookworm)
# Vollständiger Installationsguide (Panel & Wings)
# Quelle: https://pterodactyl.io
#######################################################################

# ------------------------------------------
# 1. System aktualisieren und Grundpakete
# ------------------------------------------
apt update && apt upgrade -y
apt install sudo curl wget gnupg software-properties-common apt-transport-https ca-certificates unzip zip git ufw -y

# ------------------------------------------
# 2. MariaDB (MySQL) installieren & sichern
# ------------------------------------------
apt install mariadb-server mariadb-client -y
systemctl start mariadb
systemctl enable mariadb

# MariaDB sicher konfigurieren
mysql_secure_installation

# ------------------------------------------
# 3. Datenbank & Benutzer für Pterodactyl
# ------------------------------------------
mysql -u root -p <<EOF
CREATE DATABASE panel;
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'DeinSicheresPasswort';
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
EOF

# ------------------------------------------
# 4. PHP 8.2 & NGINX installieren
# ------------------------------------------
apt install nginx php8.2 php8.2-fpm php8.2-cli php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl php8.2-zip php8.2-bcmath php8.2-gd php8.2-common php8.2-imagick composer -y
systemctl enable --now php8.2-fpm nginx

# ------------------------------------------
# 5. Pterodactyl Panel herunterladen
# ------------------------------------------
useradd -m -d /var/www/pterodactyl -s /bin/bash pterodactyl
su - pterodactyl <<EOF
curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
tar -xzvf panel.tar.gz
rm panel.tar.gz
composer install --no-dev --optimize-autoloader
cp .env.example .env
php artisan key:generate --force
php artisan p:environment:setup
php artisan p:environment:database
php artisan migrate --seed --force
php artisan p:user:make
exit
EOF

# ------------------------------------------
# 6. Rechte setzen für Webserver
# ------------------------------------------
chown -R www-data:www-data /var/www/pterodactyl
chmod -R 755 /var/www/pterodactyl

# ------------------------------------------
# 7. NGINX vHost einrichten
# ------------------------------------------
cat << 'EOF' > /etc/nginx/sites-available/pterodactyl
server {
    listen 80;
    server_name panel.deinedomain.de;
    root /var/www/pterodactyl/public;

    index index.php;

    access_log /var/log/nginx/pterodactyl.access.log;
    error_log /var/log/nginx/pterodactyl.error.log;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php\$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
EOF

ln -s /etc/nginx/sites-available/pterodactyl /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx

# ------------------------------------------
# 8. SSL-Zertifikat via Let's Encrypt
# ------------------------------------------
apt install certbot python3-certbot-nginx -y
certbot --nginx -d panel.deinedomain.de

# ------------------------------------------
# 9. Cronjob für Task-Scheduler
# ------------------------------------------
(crontab -l ; echo "* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1") | crontab -

# ------------------------------------------
# 10. Docker für Wings installieren
# ------------------------------------------
curl -sSL https://get.docker.com | sh
systemctl enable --now docker

# ------------------------------------------
# 11. Wings (Daemon) installieren
# ------------------------------------------
mkdir -p /etc/pterodactyl
curl -Lo /usr/local/bin/wings https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_amd64
chmod +x /usr/local/bin/wings

# Konfigurationsdatei von deinem Panel runterladen und unter speichern:
# /etc/pterodactyl/config.yml

# ------------------------------------------
# 12. Systemd-Service für Wings erstellen
# ------------------------------------------
cat << 'EOF' > /etc/systemd/system/wings.service
[Unit]
Description=Pterodactyl Wings Daemon
After=docker.service
Requires=docker.service

[Service]
User=root
WorkingDirectory=/etc/pterodactyl
ExecStart=/usr/local/bin/wings
Restart=on-failure
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reexec
systemctl enable --now wings

# ------------------------------------------
# 13. Firewall einrichten (optional)
# ------------------------------------------
ufw allow OpenSSH
ufw allow http
ufw allow https
ufw enable

#######################################################################
# ✅ Pterodactyl Panel: https://panel.deinedomain.de
# Melde dich mit dem erstellten Admin-Account an.
# Jetzt kannst du Nodes, Server und mehr verwalten.
#######################################################################
