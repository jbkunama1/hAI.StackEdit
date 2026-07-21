
# Highfish StackEdit Workspace

![StackEdit](https://img.shields.io/badge/stackedit-self--hosted-blue)
![Docker](https://img.shields.io/badge/docker-compose-blue)
![Markdown](https://img.shields.io/badge/markdown-editor-success)

Dieses Repo stellt deinen selbst gehosteten StackEdit-Workspace als Docker-Setup bereit. Es ist so aufgebaut, dass du es einfach auf einen Server klonen, starten und über GitHub Pages dokumentieren kannst.

## Überblick

StackEdit ist ein In-Browser-Markdown-Editor, der Dateien lokal im Browser speichert und mit Diensten wie Google Drive, Dropbox und GitHub synchronisieren kann.[web:33][web:35]

Dieses Repo kapselt deinen StackEdit-Container inklusive Volume und externem Docker-Netzwerk in einer `docker-compose.yml` und ergänzt eine kleine Projektseite (`index.html`) für GitHub Pages.

## Voraussetzungen

- Docker und Docker Compose sind installiert.
- Externes Docker-Netzwerk mit dem Namen `highfishNetwork` (oder ein eigener Name, siehe `.env.example`).
- Port `3030` (oder dein Wunschport) ist auf dem Host frei.

### Netzwerk anlegen (falls noch nicht vorhanden)

```bash
docker network create highfishNetwork
```

## Konfiguration über Umgebungsvariablen

Über die Datei `.env` (bzw. `.env.example` als Vorlage) kannst du zentrale Einstellungen anpassen:

- `STACKEDIT_IMAGE` – Docker-Image, Standard: `benweet/stackedit:latest`
- `STACKEDIT_CONTAINER_NAME` – Container-Name, Standard: `stackedit`
- `STACKEDIT_PORT` – Host-Port, über den StackEdit erreichbar ist, Standard: `3030`
- `STACKEDIT_NETWORK` – Name des externen Docker-Netzwerks, Standard: `highfishNetwork`

## Start und Stop

```bash
# im Repo-Verzeichnis
cp .env.example .env   # Werte bei Bedarf anpassen
docker compose up -d   # StackEdit starten

docker compose down    # StackEdit stoppen
```

Zugriff im Browser:

- `http://<dein-server>:STACKEDIT_PORT` – z. B. `http://192.168.1.10:3030`

Falls du einen Reverse Proxy (z. B. über Cloudflare, Nginx Proxy Manager, Traefik) nutzt, leitest du diesen Port einfach auf die gewünschte Domain/Subdomain weiter.

## Persistenz

Serverseitige Daten werden im Docker-Volume `stackedit_data` gespeichert. Dieses Volume bleibt auch nach `docker compose down` erhalten, solange du es nicht explizit löschst.

### Backup-Idee (lokal als Tarball)

```bash
docker run --rm   -v stackedit_data:/data   -v "$PWD:/backup"   busybox sh -c "tar czf /backup/stackedit_data.tar.gz /data"
```

Die Datei `stackedit_data.tar.gz` kannst du dann extern sichern.

## Cloud-Integration (Google Drive & GitHub)

Die Anbindung an Google Drive, Dropbox und GitHub wird in StackEdit selbst über die Weboberfläche verwaltet, nicht über zusätzliche Docker-Parameter.[web:33][web:35]

### Google Drive

1. StackEdit im Browser öffnen.
2. Im Menü einen Google-Workspace bzw. ein Google-Drive-Konto verknüpfen.
3. Einen Ordner in Google Drive als Workspace auswählen.
4. Dokumente in StackEdit mit diesem Workspace synchronisieren.

### GitHub / Gist

1. In StackEdit ein Dokument öffnen.
2. Über **Publish** GitHub oder Gist als Ziel auswählen.
3. Repo/Pfad bzw. Gist angeben und die OAuth-Freigabe durchführen.
4. Änderungen aus StackEdit heraus nach GitHub pushen.

## Repository-Struktur

- `docker-compose.yml` – Docker-Setup für den StackEdit-Container
- `.env.example` – Beispielkonfiguration für Port, Container-Namen und Netzwerk
- `README.md` – diese Dokumentation
- `index.html` – einfache Projektseite für GitHub Pages

## GitHub Pages

Die Datei `index.html` ist so vorbereitet, dass du sie direkt über GitHub Pages ausliefern kannst.

Typischer Ablauf:

1. Repo zu GitHub pushen.
2. In den Repo-Einstellungen GitHub Pages aktivieren (z. B. Branch `main`, Root `/`).
3. `https://<dein-user>.github.io/<dein-repo>/` aufrufen.

## Hinweise zu Cloudflare / Reverse Proxy

- Achte darauf, dass deine Domain/Subdomain im Browser exakt zu der Redirect-URI passt, die Google/GitHub für OAuth erwartet.
- Falls du einen Cloudflare-Tunnel nutzt, sollte der Tunnel auf den Host/Port zeigen, auf dem dein StackEdit-Container läuft.

## Lizenz

Dieses Setup ist als persönliche Arbeitsgrundlage gedacht. Du kannst später eine Lizenz deiner Wahl ergänzen (z. B. MIT), wenn du das Repo öffentlich teilen möchtest.
