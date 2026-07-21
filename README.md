
# Highfish StackEdit Workspace (mit eigener OAuth-Konfiguration)

![StackEdit](https://img.shields.io/badge/stackedit-self--hosted-blue)
![Docker](https://img.shields.io/badge/docker-compose-blue)
![Markdown](https://img.shields.io/badge/markdown-editor-success)
![OAuth](https://img.shields.io/badge/OAuth-Google%20%26%20GitHub-orange)

Dieses Repo stellt einen selbst gehosteten StackEdit-Workspace bereit, der mit **eigenen OAuth-Credentials** für Google Drive und GitHub arbeitet. Ziel: deine eigene Domain/Subdomain (z. B. über Cloudflare-Tunnel) nutzt StackEdit mit funktionierender Cloud-Synchronisation.

## Überblick

- Image: `qmcgaw/stackedit` – leichtgewichtige Server-Variante von StackEdit.[web:47][web:53]
- OAuth über Umgebungsvariablen:
  - `GOOGLE_CLIENT_ID`, `GOOGLE_API_KEY`
  - `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`[web:53]
- Auslieferung über deine Domain/Subdomain (z. B. `https://notes.deinedomain.tld`) hinter Cloudflare.

## Voraussetzungen

- Docker und Docker Compose sind installiert.
- Externes Docker-Netzwerk (Standard: `highfishNetwork`).
- Port `3030` (oder Wunschport) auf dem Host frei.
- Eigene OAuth-Clients bei Google und GitHub sind eingerichtet.

## Schritt 1: Google OAuth-Client anlegen

1. In der **Google Cloud Console** ein Projekt anlegen.
2. Die **Google Drive API** und ggf. weitere benötigte APIs aktivieren.[web:60]
3. Unter "APIs & Services" → "Credentials" einen **OAuth 2.0 Client ID** vom Typ "Web application" erstellen.[web:60]
4. Als **Authorized JavaScript origins** deine Domain/Subdomain eintragen, z. B.:
   - `https://notes.deinedomain.tld`
5. Als **Authorized redirect URIs** den Pfad verwenden, den StackEdit erwartet (siehe qmcgaw-Doku, z. B. `https://notes.deinedomain.tld/auth/google/callback`).
6. Die Werte **Client ID** und ggf. **API Key** notieren.[web:60]

## Schritt 2: GitHub OAuth-App anlegen

1. Auf GitHub unter **Settings → Developer settings → OAuth Apps** eine neue OAuth-App erstellen.[web:54]
2. Als **Homepage URL** deine StackEdit-URL eintragen, z. B. `https://notes.deinedomain.tld`.
3. Als **Authorization callback URL** den passenden Callback-Pfad eintragen (siehe qmcgaw-Doku, z. B. `https://notes.deinedomain.tld/auth/github/callback`).[web:54]
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

## Schritt 4: Docker-Compose starten

```bash
cp .env.example .env   # falls noch nicht vorhanden, dann Werte anpassen

# externes Netzwerk anlegen (falls nötig)
docker network create highfishNetwork

# StackEdit starten
docker compose up -d

# StackEdit stoppen
docker compose down
```

Zugriff im Browser:

- `https://notes.deinedomain.tld` (über Cloudflare/Reverse Proxy auf Port 3030 → 8000 im Container).

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
- `README.md` – diese Dokumentation
- `index.html` – einfache Projektseite (lokale Doku, kein spezielles GitHub-Pages-Setup)

## Sicherheitshinweise

- Client Secrets nicht im Repo commiten (GitHub public).
- `.env` lokal oder in Host-spezifischen Secrets/Env-Management (z.. B. Portainer) halten.[web:61][web:63]
- Nur HTTPS für produktiven Zugriff verwenden.

## Lizenz

Dieses Projekt steht unter der **MIT-Lizenz**. Du kannst es frei verwenden, anpassen und weitergeben, solange der Lizenztext beigefügt bleibt.
