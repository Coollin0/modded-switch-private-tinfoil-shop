# modded-switch-private-tinfoil-shop
Hoste deine Nintendo-Switch-Spiel-Backups mit deinem eigenen privaten Tinfoil-Shop.
# ✨ Was ist das?
Ein minimalistischer, selbst gehosteter Shop, der eine shop.json über HTTPS/HTTP bereitstellt.  
Deine gemoddete Switch (Tinfoil) liest diese Datei ein und kann die gelisteten Titel direkt von deinem Server herunterladen.
# 🧱 Architektur
Nginx: Statischer Host für /titles/* + /shop.json (Range-Requests, MIME-Typen, SSL)  
(optional) Flask: CLI/Helper zum Synchronisieren/Erstellen der shop.json  
/home/<user>/titles/: Ordner mit deinen Dateien (z. B. *.nsp, *.nsz, *.xci) und Bildern  
# 🧩 Voraussetzungen
Ubuntu/Debian VPS oder Heimserver  
Nginx (≥ 1.18)  
Domain + DNS A/AAAA-Record  
Empfohlen: Let’s Encrypt-Zertifikat (Certbot)  
Tinfoil (≥ 20.0)  
# ⚙️ Einrichtung (Schnellstart)
1. System & Ordner vorbereiten
```bash
sudo apt update && sudo apt install -y nginx python3 python3-venv jq
sudo mkdir -p /home/<user>/titles
sudo chown -R <user>:<user> /home/<user>/titles
```
2. Nginx-Site einrichten – /etc/nginx/sites-available/tinfoil mit HTTP- und HTTPS-Blöcken konfigurieren, die auf /home/<user>/titles zeigen.
3. Let’s Encrypt (empfohlen)
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d example.com
```
4. shop.json-Format – muss ein files-Array enthalten:
```json
{
  "files": [
    {
      "name": "Test Game",
      "url": "https://example.com/titles/TestGame.nsp",
      "size": 123456789,
      "nsuId": 70010000000000,
      "titleId": "0100000000000000",
      "region": "EU",
      "iconUrl": "https://example.com/titles/icon.png",
      "description": "Mein privates Backup"
    }
  ]
}
```
5. Tinfoil-Konfiguration
Name: Private Shop  
Protokoll: HTTPS (oder HTTP zum Testen)  
Host: example.com  
Port: 443  
Pfad: /shop.json  
Aktiviert: Ja  
# 🔁 Auto-Sync-Skript-Beispiel
```python
#!/usr/bin/env python3
import os, json
TITLES = "/home/<user>/titles"
BASE = "https://example.com/titles/"
ALLOWED = {".nsp", ".nsz", ".xci"}

files = []
for fn in sorted(os.listdir(TITLES)):
    ext = os.path.splitext(fn)[1].lower()
    if ext in ALLOWED:
        p = os.path.join(TITLES, fn)
        files.append({
            "name": os.path.splitext(fn)[0],
            "url": BASE + fn,
            "size": os.path.getsize(p),
            "nsuId": 0,
            "titleId": "0000000000000000",
            "region": "EU",
            "iconUrl": BASE + "icon.png",
            "description": "Auto-Sync"
        })

with open(os.path.join(TITLES, "shop.json"), "w") as f:
    json.dump({"files": files}, f, indent=2)
print("shop.json aktualisiert:", len(files), "Einträge")
```
# 🤝 Mitwirken
Pull Requests sind willkommen!  
Wenn du Ideen für Verbesserungen oder Bugfixes hast, kannst du das Repo forken und einen PR einreichen.  
Für größere Änderungen bitte zuerst ein Issue eröffnen, um zu besprechen, was du ändern möchtest.  
# 🔒 Sicherheit
Verwende nach Möglichkeit HTTPS  
Firewall aktivieren:
```bash
ufw allow 80,443/tcp
```
Zugriff bei Bedarf per Basic Auth oder IP-Whitelist einschränken  
Nginx und das Betriebssystem aktuell halten  
# 🧰 Fehlerbehebung
SSL-Verbindung in Tinfoil fehlgeschlagen → Zeit/Datum der Switch korrekt einstellen  
Leere Liste → Sicherstellen, dass shop.json { "files": [ ... ] } enthält  
Downloads brechen ab → Sicherstellen, dass Nginx den Header Accept-Ranges: bytes sendet
# 📜 Lizenz
Dieses Projekt ist unter der MIT License lizenziert.  
Bitte keine urheberrechtlich geschützten Dateien committen.  
⚠️ Rechtlicher Hinweis
Dieses Projekt ist ausschließlich für den privaten, persönlichen Gebrauch mit deinen eigenen Backups gedacht.  
Das Teilen oder Anbieten urheberrechtlich geschützter Inhalte ist in vielen Ländern illegal.
