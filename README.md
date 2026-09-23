# LeadFlow AI

Ein n8n-Portfolio-Projekt für **AI Automation / Workflow Specialist**. LeadFlow nimmt Interessentenanfragen per Webhook entgegen, validiert und normalisiert sie, erstellt mit einem LLM eine strukturierte Qualifizierung samt Antwortentwurf, speichert den Datensatz in Google Sheets und gibt das Ergebnis mit einem Prüfhinweis zurück.

Der Workflow versendet keine E-Mails automatisch: `reviewRequired` ist `true` und `autoSend` ist `false`.

## Gezeigte Fähigkeiten

- n8n-Webhooks, Verzweigungen und HTTP-Integrationen
- Eingabevalidierung und Normalisierung mit JavaScript
- LLM-Aufruf mit strukturierter JSON-Antwort
- Google-OAuth2 und Google-Sheets-Integration
- Daten-Mapping, Fehlerpfad und Kennzeichnung für menschliche Prüfung
- sichere Trennung von Workflow-Export und Zugangsdaten

## Fallstudie

Die [Fallstudie](docs/portfolio-case-study.md) beschreibt das zugrunde liegende Problem, die Workflow-Entscheidungen, den Test und sinnvolle nächste Ausbaustufen.

## Ablauf

```text
POST /webhook/lead-intake
        │
        ▼
Validate and normalize ── ungültig ──► HTTP 400 mit Validierungsfehlern
        │ gültig
        ▼
Qualify lead and draft reply (OpenAI)
        │
        ▼
Prepare human review payload
        ├─────────────────────────────► Append row in Google Sheets
        │                                      │
        └──────────────────────────────────────┴──► HTTP 200 mit Prüfergebnis
```

## Workflow-Export

Der portable n8n-Workflow liegt in [`workflows/lead-intake.json`](workflows/lead-intake.json). Instanz-IDs und Credential-Verknüpfungen wurden aus dem Export entfernt. Nach dem Import musst du in n8n deine eigenen Credentials auswählen. Der Export enthält keine API-Schlüssel.

## Beispielanfrage

```json
{
  "name": "Alex Example",
  "email": "alex@example.com",
  "company": "Example Studio",
  "message": "We need help automating the intake of support requests.",
  "source": "website"
}
```

Erfolgreiche Antwort: HTTP 200 mit `lead`, `qualification`, `reviewRequired: true` und `autoSend: false`. Ungültige Eingaben erhalten HTTP 400. Kategorie und Priorität sind KI-Einschätzungen. Der Workflow erzeugt einen Antwortentwurf und markiert ihn zur menschlichen Prüfung; eine Freigabe-Oberfläche oder ein E-Mail-Versand ist nicht implementiert.

## Lokal starten

Voraussetzung: Docker Desktop und ein OpenAI-API-Schlüssel. OpenAI-API-Nutzung wird separat nach Verbrauch abgerechnet.

1. Docker Desktop starten.
2. Im Projektordner `docker compose up -d` ausführen.
3. `http://localhost:5678` öffnen und n8n einrichten.
4. [`workflows/lead-intake.json`](workflows/lead-intake.json) in n8n importieren.
5. Ein n8n-Credential vom Typ **Header Auth** anlegen: Header-Name `Authorization`, Wert `Bearer DEIN_OPENAI_API_SCHLÜSSEL`. Dieses Credential im Node **Qualify lead and draft reply** auswählen.
6. Ein Google-Sheets-OAuth2-Credential in n8n einrichten und mit deinem Google-Konto verbinden. Für lokales n8n benötigst du eine OAuth-Client-ID und ein Client-Secret aus der Google Cloud Console. Aktiviere dort die Google Sheets API und Google Drive API und füge die Redirect-URL aus n8n exakt beim OAuth-Webclient ein.
7. Ein Google Sheet mit einer Kopfzeile und diesen Spalten anlegen:

   `Name`, `Email`, `Company`, `Source`, `Summary`, `Category`, `Priority`, `Fit Reason`, `Reply Draft`, `Review Required`, `Auto Send`

8. Im Node **Append row in sheet** Credential, Dokument und Tabellenblatt auswählen. Prüfe, ob alle Spalten auf die passenden Lead- und Qualifizierungsfelder gemappt sind.
9. Den Workflow mit einem Test-Webhook-Aufruf prüfen:

   ```bash
   curl -i -X POST 'http://localhost:5678/webhook-test/lead-intake' \
     -H 'Content-Type: application/json' \
     -d '{"name":"Alex Example","email":"alex@example.com","company":"Example Studio","message":"We need help automating the intake of support requests.","source":"website"}'
   ```

10. Workflow veröffentlichen. Danach ist der Produktions-Webhook `http://localhost:5678/webhook/lead-intake` aktiv, solange n8n läuft.

## Sicherheit und Grenzen

- Die Compose-Datei bindet n8n an `127.0.0.1`; der Webhook ist damit nur lokal erreichbar.
- Vor einer öffentlichen Bereitstellung den Webhook authentifizieren, Zugriff begrenzen und Datenschutzanforderungen prüfen.
- OpenAI- und Google-Zugangsdaten niemals in den Workflow-Export, Beispielpayloads oder ein öffentliches Repository schreiben.
- Lead-Daten und KI-Antwortentwürfe werden in Google Sheets gespeichert. Verwende für Tests synthetische Beispieldaten.
- Das Projekt erstellt nur einen Antwortentwurf. Es verschickt keine E-Mails, ändert keine Kundendaten und löscht keine Datensätze.

## Dateien

- `workflows/lead-intake.json` – n8n-Workflow
- `examples/sample-lead.json` – synthetischer Test-Lead
- `compose.yaml` – lokaler n8n-Dienst mit persistentem Docker-Volume
- `docs/portfolio-case-study.md` – Projektfallstudie für Portfolio und Bewerbung
