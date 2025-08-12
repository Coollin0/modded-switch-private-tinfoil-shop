# modded-switch-private-tinfoil-shop
Host your Nintendo Switch game backups with your own private Tinfoil shop.



# What is this?

A minimalist, self-hosted shop serving a shop.json over HTTPS/HTTP. Your modded Switch (Tinfoil) reads this file and you can downloads the listed titles directly from your server.



# Architecture

Nginx: Static host for /titles/* + /shop.json (Range requests, MIME types, SSL)

(optional) Flask: CLI/helper for synchronizing/generating shop.json

/home/<user>/titles/: Folder with your files (e.g., *.nsp, *.nsz, *.xci) and images



# Requirements

Ubuntu/Debian VPS or home server

Nginx (≥ 1.18)

Domain + DNS A/AAAA record

Recommended: Let’s Encrypt certificate (Certbot)

Tinfoil ≥ 20



# Setup (Quick)

1. Prepare system & folders


sudo apt update && sudo apt install -y nginx python3 python3-venv jq
sudo mkdir -p /home/<user>/titles
sudo chown -R <user>:<user> /home/<user>/titles


2. Nginx site – configure /etc/nginx/sites-available/tinfoil with HTTP and HTTPS blocks pointing to /home/<user>/titles.

3. Let’s Encrypt (recommended)


sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d example.com


4. shop.json format – must contain a files array:



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
      "description": "My private backup"
    }
  ]
}



5. Tinfoil config


Name: Private Shop

Protocol: HTTPS (or HTTP for testing)

Host: example.com

Port: 443

Path: /shop.json

Enabled: Yes


# Auto-Sync Script Example



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
print("shop.json updated:", len(files), "entries")



# Contributing

Pull requests are welcome!If you have ideas for improvements or bug fixes, feel free to fork the repo and submit a PR.For major changes, please open an issue first to discuss what you would like to change.


# Security

Use HTTPS whenever possible

Enable firewall: ufw allow 80,443/tcp

Restrict access via Basic Auth/IP whitelist if needed

Keep Nginx and OS updated


# Troubleshooting

SSL Connection failed in Tinfoil → Check and set Switch time/date correctly

Empty list → Ensure shop.json contains { "files": [ ... ] }

Downloads stop → Ensure Nginx sends Accept-Ranges: bytes


# License

This project is licensed under the MIT License.Do not commit copyrighted files.


# Legal

This project is for private, personal use with your own backups. Sharing or offering copyrighted content is illegal in many countries.
