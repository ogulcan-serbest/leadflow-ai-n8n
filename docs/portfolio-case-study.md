# LeadFlow AI – Fallstudie

## Kurzbeschreibung

LeadFlow AI automatisiert die erste Bearbeitung eingehender Geschäftsanfragen. Ein n8n-Webhook nimmt Leads an, prüft die Eingaben, lässt sie von einem Sprachmodell strukturieren, speichert das Ergebnis in Google Sheets und gibt es als JSON zurück.

**Rolle:** persönliches Portfolio-Projekt  
**Technologien:** n8n, Docker Compose, JavaScript, OpenAI API (`gpt-4.1-mini`), Google Sheets API und OAuth2

## Ausgangsproblem

Kleine Teams sortieren neue Anfragen oft manuell. Dabei müssen sie fehlende Angaben erkennen, den passenden Bereich bestimmen und eine erste Antwort vorbereiten. LeadFlow demonstriert, wie dieser Eingang standardisiert werden kann, ohne die Kommunikation ungeprüft zu automatisieren.

## Umsetzung

1. Ein Webhook nimmt Name, E-Mail, Unternehmen, Nachricht und Quelle als JSON entgegen.
2. Ein JavaScript-Schritt normalisiert die Eingaben und prüft Pflichtfelder, E-Mail-Format sowie Nachrichtenlänge.
3. Ungültige Anfragen erhalten eine HTTP-400-Antwort mit Validierungsfehlern.
4. Gültige Anfragen werden an `gpt-4.1-mini` gesendet. Das Modell liefert Zusammenfassung, Kategorie, Priorität, Begründung und einen Antwortentwurf als JSON.
5. n8n baut daraus einen Prüfdatensatz und hängt ihn an eine Google-Sheets-Tabelle an.
6. Der Webhook gibt den Prüfdatensatz zurück. `reviewRequired` ist `true`; `autoSend` ist `false`.

## Sicherheits- und Produktentscheidungen

- Der Workflow verschickt keine E-Mails und behauptet nicht, dass eine Antwort versendet wurde.
- Die KI-Ausgabe bleibt ein Entwurf. Kategorie und Priorität sind Vorschläge und benötigen menschliche Prüfung.
- OpenAI- und Google-Zugangsdaten liegen in n8n-Credentials und nicht im Git-Repository.
- Der lokale Docker-Compose-Port ist an `127.0.0.1` gebunden. Der Workflow ist damit nicht öffentlich erreichbar.
- Die Tests nutzen synthetische Beispieldaten.

## Überprüfung

Der Test-Webhook und anschließend der veröffentlichte Produktions-Webhook wurden mit synthetischen Leads aufgerufen. Beide Aufrufe lieferten HTTP 200 und ein strukturiertes JSON-Ergebnis. Die Google-Sheets-Integration hat die Testdatensätze angehängt. Ein Testdatensatz wurde als Systemtest mit Kategorie `other` und Priorität `low` klassifiziert; automatische Aktionen blieben deaktiviert.

Es wurden keine Last-, Sicherheits- oder Genauigkeitsbenchmarks durchgeführt. Das Projekt ist ein lokaler Prototyp und noch kein öffentlich bereitgestellter Produktivdienst.

## Mögliche nächste Schritte

- Eine echte menschliche Freigabe mit Approve/Reject-Schritt ergänzen.
- Dubletten erkennen und Anfragen anhand von Kategorie oder Priorität routen.
- Eingehende Webhooks authentifizieren und Rate-Limits ergänzen, bevor der Dienst öffentlich erreichbar wird.
- Fehlerbenachrichtigungen und strukturierte Betriebslogs ergänzen.
