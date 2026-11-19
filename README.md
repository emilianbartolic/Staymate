# StayMate AI

StayMate is a guest-support agent for three sample holiday properties. It uses the signed-in local Codex CLI on the same computer as its backend. The repository contains illustrative stock photos, not photos of the sample rentals; see [`assets/properties/ATTRIBUTION.md`](assets/properties/ATTRIBUTION.md).

## Start

### Quick start: use your local Codex account

Python and the Codex CLI must run on the same computer. Sign in to Codex once, then:

```bash
codex login
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app:app --host 127.0.0.1 --port 8000
```

Open <http://127.0.0.1:8000>. StayMate starts `codex exec` for each guest question. The Codex login stays on the user's computer and is never committed to GitHub. Do not run another server on port 8000; the backend serves both the page and the chat API. The API rejects non-local clients and applies a per-IP request limit. Keep the server bound to `127.0.0.1`.

Die drei Beispielbilder liegen in `assets/properties/`; Fotografen, Quellseiten und Bildtitel stehen in `assets/properties/ATTRIBUTION.md`. Sie sind illustrative Beispielbilder und zeigen nicht die konkreten Demo-Unterkünfte.

### Alternative: OpenAI API key

The backend can use the OpenAI API instead when `OPENAI_API_KEY` is set in the environment:

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export OPENAI_API_KEY="dein-schluessel"
uvicorn app:app --reload
```

Then open <http://127.0.0.1:8000>. Never commit API keys or `.env` files. Uploaded photos are sent to the configured model for analysis. Do not upload private or sensitive images.

### Important: GitHub Pages and the personal Codex login

GitHub Pages can host the static files, but a page opened there cannot run a visitor's local Codex CLI. To use each person's own Codex login, each person must clone this repo and run the local backend as described above. This repository is designed for that local self-hosted workflow. It is not configured for public production hosting: there is no host account system, durable ticket storage, or secure remote deployment. Escalations are only noted in the current browser and are not sent to a host. Do not bind the API to `0.0.0.0` or expose it to the internet.

### Features

- Three separate property profiles and knowledge bases (Design-Maisonette, City Loft, Haus am See).
- Natural-language guest replies through the local Codex CLI; verified property facts are supplied as context.
- Escalation for host-confirmation cases such as booking, price, pets, exceptions, damage, or safety.
- Photo attachment in chat for local Codex image inspection.
- Responsive property cards with sourced, illustrative stock photos.

## Agentenlogik

- Standardfragen werden über die Fakten in `app.js` beantwortet.
- Bei Storno, Preisen, Buchungsänderungen, Schäden, Sicherheitsproblemen oder unbekannten Fragen wird an den Host eskaliert.
- Die Oberfläche zeigt stets, ob die Antwort automatisch oder als Eskalation behandelt wurde.
- Eskalierte Anliegen erscheinen im Host-Dashboard. Dort können sie als bearbeitet markiert und ein Antwortentwurf kopiert werden.
- Die Unterkunft ist nicht fest verdrahtet: Über den Objektwähler kann zwischen mehreren Unterkünften gewechselt werden. Die Demo enthält drei Beispielobjekte; weitere können in `backend/properties.json` ergänzt werden. Jede Unterkunft verwendet ihre eigene Faktenbasis.
- Die vollständigen, getrennten Objekt-Wissensbasen liegen in `backend/knowledge_bases/`: jeweils eine JSON-Datei pro Unterkunft. `properties.json` enthält die routbaren Suchfakten; die Wissensdateien dokumentieren die fachlichen Antworten je Objekt.
- Die allgemeine Intent-Wissensbasis in `backend/knowledge_base.json` deckt typische sichere Fragen und Host-Bestätigungsfälle ab. Sie dient außerdem als Grundlage für Testfälle und die spätere FAQ-Verwaltung.
- Der sichtbare Agenten-Trace zeigt die drei Arbeitsschritte: Risikoanalyse, Faktenabruf und Antwort bzw. Host-Aktion. Der Host Advisor erkennt zudem wiederkehrende Ticketmuster und empfiehlt Verbesserungen der Wissensbasis.

## Sprachmodell im Backend

Das Sprachmodell kommt nicht in den Browser und erhält niemals einen API-Schlüssel im Frontend. Das FastAPI-Backend (1) prüft die Frage, (2) wählt passende Fakten für die gewählte Unterkunft aus `backend/properties.json` und (3) übergibt ausschließlich diese Fakten mit einem Systemprompt an die Responses API. Kritische oder unbekannte Fragen werden *vor* dem Modellaufruf eskaliert.

Die OpenAI Responses API wird über ein serverseitiges SDK aufgerufen; der API-Schlüssel wird als Umgebungsvariable gespeichert. Siehe die [offizielle Quickstart-Dokumentation](https://developers.openai.com/api/docs/quickstart).
