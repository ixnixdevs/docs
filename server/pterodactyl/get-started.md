#######################################################################
# ✅ DEPENDENCY INSTALLATION – PTERODACTYL PANEL – DEBIAN 12 (BOOKWORM)
#######################################################################

# System aktualisieren & Basis-Tools installieren
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget gnupg2 ca-certificates lsb-release software-properties-common apt-transport-https unzip tar git

# ------------------------------------------
# 1. PHP 8.3 + Extensions (über Sury Repository für Debian)
# ------------------------------------------

# PHP Sury Repo hinzufügen (offiziell für Debian)
sudo wget -qO /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list

# Paketquellen aktualisieren
sudo apt update

# PHP 8.3 + benötigte Extensions installieren
sudo apt install -y php8.3 php8.3-{cli,fpm,common,openssl,gd,mysql,mbstring,tokenizer,bcmath,xml,curl,zip}

# ------------------------------------------
# 2. MariaDB (statt MySQL)
# ------------------------------------------
sudo apt install -y mariadb-server

# ------------------------------------------
# 3. Redis installieren
# (aus offiziellem Redis-Repo für aktuellere Versionen)
# ------------------------------------------
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt update
sudo apt install -y redis-server

# ------------------------------------------
# 4. Webserver: NGINX (alternativ Apache oder Caddy möglich)
# ------------------------------------------
sudo apt install -y nginx

# ------------------------------------------
# 5. Composer v2 installieren (offiziell)
# ------------------------------------------
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer

#######################################################################
# ✅ AB HIER: PANEL SETUP BEGINNEN
#######################################################################

# Beispiel: Pterodactyl-Dateien herunterladen
mkdir -p /var/www/pterodactyl
cd /var/www/pterodactyl

curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
tar -xzvf panel.tar.gz
chmod -R 755 storage/* bootstrap/cache/

#######################################################################
# ✅ SYSTEM IST JETZT BEREIT FÜR PANEL-INSTALLATION
# Datenbank, .env, composer, artisan folgen wie in der Anleitung
#######################################################################
