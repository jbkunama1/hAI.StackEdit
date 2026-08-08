

[![Buy me a coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/highfish)
# Highfish StackEdit Workspace (benweet-original)

[![StackEdit](https://img.shields.io/badge/stackedit-self--hosted-blue)](https://github.com/jbkunama1/hAI.StackEdit)
[![Docker](https://img.shields.io/badge/docker-compose-blue)](https://github.com/jbkunama1/hAI.StackEdit)
[![Portainer](https://img.shields.io/badge/portainer-stack-13BEF9)](https://github.com/jbkunama1/hAI.StackEdit)
[![Markdown](https://img.shields.io/badge/markdown-editor-success)](https://github.com/jbkunama1/hAI.StackEdit)
[![License](https://img.shields.io/badge/license-MIT-green)](https://github.com/jbkunama1/hAI.StackEdit)

Dieses Repo bildet dein bestehendes Setup mit `benweet/stackedit:latest` möglichst nah am Original ab. Es ist für einen stabilen Self-Hosting-Betrieb mit Docker Compose oder als Portainer Stack gedacht.[web:4][web:67]

## Überblick

- Image: `benweet/stackedit:latest`.[web:4]
- Port-Mapping: Host `3030` auf Container `8080`.
- Persistenz über Docker-Volume `stackedit_data`.
- Externes Docker-Netzwerk `highfishNetwork`.
- Nutzung lokal, hinter Reverse Proxy oder über Cloudflare-Tunnel.

## Wichtiger Hinweis zur Cloud-Anbindung

Dieses Repo bleibt bewusst bei `benweet/stackedit:latest`.[web:4] Für dieses Image sind in den recherchierten Quellen keine dokumentierten Docker-Umgebungsvariablen für eigene Google- oder GitHub-OAuth-Clients ersichtlich, anders als bei alternativen Images.[web:4][web:53]

Das bedeutet:

- Das Setup bildet deinen aktuellen Stand sauber ab.
- Eine funktionierende Self-Host-OAuth-Anbindung für Google Drive/GitHub ist damit **nicht** Bestandteil dieses Repos.
- Wenn du Cloud-Anbindungen über eigene OAuth-Clients brauchst, wäre dafür ein anderes Setup bzw. ein anderes Image nötig.[web:53]

## Voraussetzungen

- Docker und Docker Compose sind installiert.
- Optional: Portainer läuft bereits.
- Externes Docker-Netzwerk `highfishNetwork` existiert oder wird angelegt.
- Port `3030` ist auf dem Host frei.

## Netzwerk anlegen

```bash
docker network create highfishNetwork
```

## Konfiguration

Nutze `.env.example` als Vorlage:

- `STACKEDIT_IMAGE` – Standard: `benweet/stackedit:latest`
- `STACKEDIT_CONTAINER_NAME` – Standard: `stackedit`
- `STACKEDIT_PORT` – Standard: `3030`
- `STACKEDIT_NETWORK` – Standard: `highfishNetwork`

## Docker Compose

Start:

```bash
cp .env.example .env
docker compose up -d
```

Stop:

```bash
docker compose down
```

Aufruf im Browser:

- lokal: `http://<server-ip>:3030`
- über Reverse Proxy / Tunnel: mit deiner gewünschten Domain/Subdomain

## Portainer Stack

Portainer kann Stacks direkt aus einer Compose-Datei im Web Editor oder aus einem Git-Repository anlegen.[web:67]

### Variante A: Portainer Web Editor

1. In Portainer **Stacks** öffnen.[web:67]
2. **Add stack** anklicken.[web:67]
3. Stack-Namen vergeben, z. B. `stackedit-benweet`.
4. **Web editor** wählen.
5. Inhalt aus `docker-compose.yml` einfügen.
6. Optional Umgebungsvariablen im Portainer-UI setzen (`STACKEDIT_IMAGE`, `STACKEDIT_CONTAINER_NAME`, `STACKEDIT_PORT`, `STACKEDIT_NETWORK`).[web:68]
7. Sicherstellen, dass das externe Netzwerk bereits existiert.
8. **Deploy the stack** klicken.[web:67]

### Variante B: Portainer über Git-Repo

1. Repo zu GitHub/Gitea hochladen.
2. In Portainer **Stacks → Add stack** öffnen.[web:67]
3. Git-/Repository-Variante auswählen.
4. Repo-URL eintragen.
5. Compose-Pfad: `docker-compose.yml`.[web:67]
6. ENV-Werte bei Bedarf in Portainer setzen; Portainer arbeitet dabei je nach Version mit eigenem Variablenhandling.[web:68]
7. Stack deployen.

## Cloudflare / Reverse Proxy

Wenn du StackEdit über Cloudflare-Tunnel oder Reverse Proxy erreichbar machst:

- Tunnel/Proxy auf `http://<dein-server>:3030` zeigen lassen.
- HTTPS nur über die externe URL erzwingen.
- Bei Problemen mit Google Drive/GitHub im UI beachten, dass dieses Repo keine spezielle OAuth-Self-Host-Konfiguration enthält.[web:4][web:53]

## Persistenz und Backup

Daten liegen im Docker-Volume `stackedit_data`. Das Volume bleibt erhalten, auch wenn du den Stack stoppst oder neu startest.

Beispiel für ein Backup:

```bash
docker run --rm   -v stackedit_data:/data   -v "$PWD:/backup"   busybox sh -c "tar czf /backup/stackedit_data.tar.gz /data"
```

## Repository-Struktur

- `docker-compose.yml` – Compose-Setup nahe an deinem Original
- `.env.example` – Beispielwerte für Port, Name und Netzwerk
- `README.md` – Dokumentation für Docker Compose und Portainer
- `index.html` – einfache Projektseite
- `LICENSE` – MIT-Lizenz

## Lizenz

Dieses Projekt steht unter der **MIT-Lizenz**.
