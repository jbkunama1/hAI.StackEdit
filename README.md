
# Highfish StackEdit Workspace (mit eigener OAuth-Konfiguration)

![StackEdit](https://img.shields.io/badge/stackedit-self--hosted-blue)
![Docker](https://img.shields.io/badge/docker-compose-blue)
![Portainer](https://img.shields.io/badge/portainer-stack-13BEF9)
![Markdown](https://img.shields.io/badge/markdown-editor-success)
![OAuth](https://img.shields.io/badge/OAuth-Google%20%26%20GitHub-orange)
![License](https://img.shields.io/badge/license-MIT-green)

Dieses Repo stellt einen selbst gehosteten StackEdit-Workspace bereit, der mit **eigenen OAuth-Credentials** für Google Drive und GitHub arbeitet. Ziel: deine eigene Domain/Subdomain (z. B. über Cloudflare-Tunnel) nutzt StackEdit mit funktionierender Cloud-Synchronisation.

## Überblick

- Image: `qmcgaw/stackedit` – leichtgewichtige Server-Variante von StackEdit.[web:47][web:53]
- OAuth über Umgebungsvariablen:
  - `GOOGLE_CLIENT_ID`, `GOOGLE_API_KEY`
  - `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`[web:53]
- Auslieferung über deine Domain/Subdomain (z. B. `https://notes.deinedomain.tld`) hinter Cloudflare.
- Betrieb wahlweise per Docker Compose oder direkt als **Portainer Stack**.[web:67][web:72]

## Voraussetzungen

- Docker und Docker Compose sind installiert.
- Portainer läuft bereits auf deinem Host oder in deiner Docker-Umgebung.
- Externes Docker-Netzwerk (Standard: `highfishNetwork`).
- Port `3030` (oder Wunschport) auf dem Host frei.
- Eigene OAuth-Clients bei Google und GitHub sind eingerichtet.

## Schritt 1: Google OAuth-Client anlegen

1. In der **Google Cloud Console** ein Projekt anlegen.
2. Die **Google Drive API** und ggf. weitere benötigte APIs aktivieren.[web:60]
3. Unter "APIs & Services" → "Credentials" einen **OAuth 2.0 Client ID** vom Typ "Web application" erstellen.[web:60]
4. Als **Authorized JavaScript origins** deine Domain/Subdomain eintragen, z. B.:
   - `https://notes.deinedomain.tld`
5. Als **Authorized redirect URIs** den Pfad verwenden, den StackEdit erwartet (z. B. `https://notes.deinedomain.tld/auth/google/callback`).
6. Die Werte **Client ID** und **API Key** notieren.[web:60]

## Schritt 2: GitHub OAuth-App anlegen

1. Auf GitHub unter **Settings → Developer settings → OAuth Apps** eine neue OAuth-App erstellen.[web:54]
2. Als **Homepage URL** deine StackEdit-URL eintragen, z. B. `https://notes.deinedomain.tld`.[web:54]
3. Als **Authorization callback URL** den passenden Callback-Pfad eintragen, z. B. `https://notes.deinedomain.tld/auth/github/callback`.[web:54]
4. Nach dem Speichern erhältst du **Client ID** und **Client Secret**.[web:54][web:57]

## Schritt 3: .env ausfüllen

Nutze `.env.example` als Vorlage und lege eine `.env` im Repo an:

- `STACKEDIT_IMAGE` – Standard: `qmcgaw/stackedit`
- `STACKEDIT_CONTAINER_NAME` – z. B. `stackedit`
- `STACKEDIT_PORT` – Host-Port, Standard: `3030`
- `STACKEDIT_NETWORK` – Name des externen Docker-Netzwerks
- `STACKEDIT_ROOT_URL` – Root-URL, i. d. R. `/`
- `STACKEDIT_USER_BUCKET_NAME` – interner Bucket-Name, Standard: `stackedit-users`
- `GOOGLE_CLIENT_ID` – aus der Google Cloud Console
- `GOOGLE_API_KEY` – aus der Google Cloud Console
- `GITHUB_CLIENT_ID` – aus den GitHub OAuth-Einstellungen
- `GITHUB_CLIENT_SECRET` – aus den GitHub OAuth-Einstellungen

**Wichtig:** `.env` gehört nicht ins öffentliche Repo; halte Client Secrets privat.[web:63]

## Docker Compose

```bash
cp .env.example .env

docker network create highfishNetwork

docker compose up -d
```

Stoppen:

```bash
docker compose down
```

## Portainer Stack

Portainer unterstützt das Anlegen von Stacks direkt aus einer Compose-Datei über den Web Editor oder aus einem Git-Repository.[web:67][web:72]

### Variante A: Web Editor in Portainer

1. In Portainer links **Stacks** öffnen.[web:67]
2. Auf **Add stack** klicken.[web:67]
3. Einen Namen vergeben, z. B. `stackedit-oauth`.[web:67]
4. **Web editor** wählen.[web:72]
5. Den Inhalt aus `docker-compose.yml` einfügen.
6. Im Bereich für Umgebungsvariablen die Werte für `GOOGLE_CLIENT_ID`, `GOOGLE_API_KEY`, `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` sowie optional Port/Netzwerk setzen.[web:68][web:72]
7. Vorher sicherstellen, dass das externe Netzwerk `highfishNetwork` bereits existiert.
8. Auf **Deploy the stack** klicken.[web:67][web:72]

### Variante B: Git-Repository in Portainer

1. Repo zu GitHub oder Gitea hochladen.
2. In Portainer **Stacks → Add stack** öffnen.[web:67]
3. **Repository** bzw. Git-Variante auswählen.[web:72]
4. Repo-URL eintragen.
5. Als Compose-Pfad `docker-compose.yml` angeben.[web:72]
6. Benötigte Umgebungsvariablen in Portainer setzen; alternativ bei Docker Standalone eine `.env` im gleichen Repo-Verzeichnis mitführen.[web:68]
7. Stack deployen.[web:72]

### Wichtige Hinweise für Portainer

- Portainer kann Umgebungsvariablen direkt im UI setzen; das ist oft sauberer als Secrets im Compose-Text zu hinterlegen.[web:68]
- Bei Git-basierten Stacks ist das Verhalten rund um `.env` bzw. `stack.env` versionsabhängig; UI-Variablen sind meist der robustere Weg.[web:68][web:75]
- Das externe Docker-Netzwerk muss vor dem Deploy existieren, sonst schlägt der Stack fehl.

## Cloudflare / Reverse Proxy

- Dein Cloudflare-Tunnel/Reverse Proxy zeigt auf `http://<dein-server>:STACKEDIT_PORT`.
- SSL/TLS-Modus so wählen, dass keine Redirect-Schleifen entstehen.
- Die Domain/Subdomain muss exakt zu den in Google/GitHub hinterlegten OAuth-Redirect-URLs passen.[web:19][web:54]

## Cloud-Integration in StackEdit

Mit korrekt gesetzten OAuth-ENV-Variablen und passender Domain kannst du in der StackEdit-Weboberfläche:

- **Google Drive**-Workspaces verknüpfen und Ordner als Workspace nutzen.[web:33][web:35]
- Dokumente nach **GitHub/Gist** veröffentlichen und aktualisieren.[web:33][web:35]

Die Anbindung läuft dann über deine eigenen OAuth-Clients, nicht über die Standard-Clients von stackedit.io.

## Repository-Struktur

- `docker-compose.yml` – Docker-Setup für `qmcgaw/stackedit` mit OAuth-ENV-Variablen
- `.env.example` – Vorlage für deine Konfiguration
- `README.md` – Doku für Docker Compose und Portainer
- `index.html` – einfache Projektseite
- `LICENSE` – MIT-Lizenz

## Sicherheitshinweise

- Client Secrets nicht im Repo commiten.
- `.env` lokal oder in Portainer-/Host-seitigem Env-Management halten.[web:68][web:63]
- Nur HTTPS für produktiven Zugriff verwenden.

## Lizenz

Dieses Projekt steht unter der **MIT-Lizenz**.
