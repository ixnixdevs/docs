---

# 🐦 Wings Installation (Pterodactyl Daemon) – Debian

## 📦 Voraussetzungen

* Debian 10, 11 oder 12
* Root-Zugriff oder `sudo`-Rechte
* Pterodactyl Panel bereits installiert

---

## 🐳 Docker installieren

```bash
curl -sSL https://get.docker.com/ | CHANNEL=stable bash
sudo systemctl enable --now docker
```

---

## 📂 Wings herunterladen & installieren

```bash
sudo mkdir -p /etc/pterodactyl

curl -L -o /usr/local/bin/wings "https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_$([[ \"$(uname -m)\" == \"x86_64\" ]] && echo \"amd64\" || echo \"arm64\")"

sudo chmod u+x /usr/local/bin/wings
```

> **OVH/SYS-Server-Hinweis:** Wenn dein Root-Dateisystem klein ist, verwende `/home/daemon-data` als Speicherort für Serverdaten.

---

## ⚙️ Wings konfigurieren

1. Logge dich in dein Panel ein.
2. Gehe zu: `Nodes > [Dein Node] > Configuration`.
3. Kopiere den Inhalt.
4. Füge ihn ein unter:

```bash
sudo nano /etc/pterodactyl/config.yml
```

> Alternativ: Nutze den Button „Generate Token“ im Panel und führe den vorgeschlagenen Befehl im Terminal aus.

---

## 🧪 Wings testen (Debug-Modus)

```bash
sudo wings --debug
```

> Bei Erfolg mit `CTRL+C` abbrechen und fortfahren.

---

## 📌 Wings als Dienst einrichten (systemd)

Erstelle:

```bash
sudo nano /etc/systemd/system/wings.service
```

Füge Folgendes ein:

```ini
[Unit]
Description=Pterodactyl Wings Daemon
After=docker.service
Requires=docker.service
PartOf=docker.service

[Service]
User=root
WorkingDirectory=/etc/pterodactyl
LimitNOFILE=4096
PIDFile=/var/run/wings/daemon.pid
ExecStart=/usr/local/bin/wings
Restart=on-failure
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Dann:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wings
```

---

## 🌐 IP & Ports zuweisen (Allocations)

Finde deine öffentliche IP:

```bash
hostname -I | awk '{print $1}'
```

Oder:

```bash
ip addr | grep "inet "
```

> ⚠️ **Hinweis:** Verwende **nicht** `127.0.0.1` als IP.

Zuweisung erfolgt im Panel:
**Nodes > Dein Node > Allocation**

---

## ✅ Wings läuft!

Prüfen mit:

```bash
sudo systemctl status wings
```

Fertig! Wings läuft nun als stabiler Hintergrunddienst.

---
