# CLAUDE.md — N8N Workflows Repository

## Projektübersicht

Dieses Repository speichert [n8n](https://n8n.io/) Workflow-Automatisierungsdefinitionen. n8n ist ein selbst hostbares, knotenbasiertes Automatisierungstool, das APIs, Dienste und Datenquellen ohne benutzerdefinierten Integrationscode verbindet.

Workflows werden als **JSON-Dateien** aus einer n8n-Instanz exportiert und hier gespeichert, um Versionskontrolle, Zusammenarbeit und Wiederverwendung über Umgebungen hinweg zu ermöglichen.

---

## Repository-Struktur

```
N8N_Workflows-/
├── CLAUDE.md                  # Diese Datei
├── workflows/                 # Workflow-JSON-Dateien (nach Kategorie oder Funktion)
│   ├── <kategorie>/
│   │   └── <workflow-name>.json
│   └── <workflow-name>.json
├── credentials/               # Credential-Vorlagen (KEINE echten Zugangsdaten — nur Platzhalter)
│   └── <dienst>-vorlage.json
├── docs/                      # Menschenlesbare Dokumentation pro Workflow
│   └── <workflow-name>.md
└── README.md                  # Projektübersicht für Menschen
```

> **Hinweis:** Die genaue Verzeichnisstruktur kann sich weiterentwickeln. Workflows sollten logisch nach Integration, Abteilung oder Funktion gruppiert werden.

---

## Was ist eine n8n-Workflow-JSON?

Jede `.json`-Datei ist ein vollständiger Workflow-Export aus n8n und enthält:

- **`nodes`** — Array von Node-Objekten (jeder Node ist ein Schritt im Automatisierungsablauf)
- **`connections`** — Verbindungen zwischen Nodes (was in was einfließt)
- **`settings`** — Ausführungseinstellungen (Timeout, Fehlerbehandlung, Zeitzone)
- **`staticData`** — Persistierter Zustand über Ausführungen hinweg (falls vorhanden)
- **`meta`** — Workflow-Metadaten (n8n-Version, Template-ID)

Beispiel einer minimalen Struktur:

```json
{
  "name": "Mein Workflow",
  "nodes": [...],
  "connections": {...},
  "settings": {
    "executionOrder": "v1"
  },
  "staticData": null
}
```

---

## Wichtige Konventionen

### Dateinamen

- **Kebab-Case** für alle Dateinamen verwenden: `slack-benachrichtigung-bei-fehler.json`
- Bei fehlendem Unterverzeichnis mit einer Kategorie prefixen: `crm-lead-synchronisation.json`
- Keine Leerzeichen oder Sonderzeichen in Dateinamen

### Workflow-Namen (innerhalb der JSON)

- Das Feld `"name"` soll **menschenlesbar** und beschreibend sein
- Titelschreibweise verwenden: `"Slack Benachrichtigung bei Fehler"`
- Namen auf 60 Zeichen begrenzen

### Zugangsdaten (Credentials)

- **Niemals echte Zugangsdaten, API-Schlüssel, Tokens oder Passwörter committen**
- Sensible Werte in exportierten JSONs durch Platzhalter ersetzen, z. B. `"<DEIN_API_SCHLÜSSEL>"` oder `"ERSETZEN"`
- Im Verzeichnis `credentials/` eine Vorlagendatei bereitstellen, die erklärt, was jedes Feld benötigt
- n8ns eingebauten Credential-Manager auf dem Server nutzen — hier liegen nur Schema-Vorlagen

### Node-IDs

- n8n generiert UUIDs für jeden Node (Feld `"id"` innerhalb der Nodes)
- Node-IDs niemals manuell bearbeiten — sie werden für interne Verbindungsreferenzen verwendet
- Beim Zusammenführen oder Deduplizieren IDs stabil halten

### Versionskontrolle

- **Ein Workflow pro Datei** — niemals mehrere unzusammenhängende Workflows in eine JSON bündeln
- Export aus n8n über `Workflow → Herunterladen` oder die n8n-CLI/API
- Import in n8n über `Workflow → Aus Datei importieren` oder die n8n-CLI/API
- Workflows immer zuerst in einer **Staging-n8n-Instanz** testen, bevor sie ins Repository aufgenommen werden

---

## Entwicklungsablauf

### Neuen Workflow hinzufügen

1. Workflow in der n8n-Instanz erstellen und testen
2. Exportieren: `Workflow-Menü → Herunterladen`
3. Zugangsdaten/Secrets aus der JSON entfernen oder durch Platzhalter ersetzen
4. Unter `workflows/<kategorie>/<workflow-name>.json` speichern
5. Kurze Dokumentation in `docs/<workflow-name>.md` erstellen mit:
   - Zweck des Workflows
   - Trigger-Typ (Webhook, Zeitplan, manuell)
   - Benötigte Zugangsdaten/Umgebungsvariablen
   - Erwartete Eingaben und Ausgaben
6. Pull Request zur Überprüfung öffnen

### Bestehenden Workflow aktualisieren

1. Vorhandene JSON in die n8n-Instanz importieren
2. Änderungen vornehmen und testen
3. Erneut exportieren und die Datei im Repository ersetzen
4. Zugehörige `docs/`-Datei aktualisieren, falls sich das Verhalten geändert hat
5. Commit mit beschreibender Nachricht erstellen (siehe unten)

### Workflow löschen

- `.json`-Datei und die zugehörige `docs/`-Datei entfernen
- In der Commit-Nachricht begründen, warum er entfernt wurde (veraltet, durch anderen ersetzt usw.)

---

## Git-Commit-Konventionen

Klare, imperativische Commit-Nachrichten verwenden:

```
add: slack-benachrichtigung-bei-fehler Workflow
update: crm-lead-synchronisation - Wiederholungslogik bei HTTP 429 ergänzt
fix: fehlerhafte Verbindung im daten-anreicherung Workflow behoben
remove: legacy-hubspot-sync (ersetzt durch hubspot-v2-sync)
docs: Dokumentation für stripe-zahlungserfassung hinzugefügt
```

Erlaubte Präfixe: `add`, `update`, `fix`, `remove`, `docs`, `refactor`, `chore`

---

## Workflows importieren & exportieren (n8n-CLI)

Falls die n8n-CLI verfügbar ist:

```bash
# Workflow nach ID exportieren
n8n export:workflow --id=<workflow-id> --output=workflows/<name>.json

# Workflow importieren
n8n import:workflow --input=workflows/<name>.json

# Alle Workflows exportieren
n8n export:workflow --all --output=workflows/
```

Über die n8n-REST-API:

```bash
# Export via API
curl -H "X-N8N-API-KEY: $N8N_API_KEY" \
  "http://localhost:5678/api/v1/workflows/<id>" \
  | jq . > workflows/<name>.json

# Import via API
curl -X POST \
  -H "X-N8N-API-KEY: $N8N_API_KEY" \
  -H "Content-Type: application/json" \
  -d @workflows/<name>.json \
  "http://localhost:5678/api/v1/workflows"
```

---

## Richtlinien für KI-Assistenten

Beim Analysieren oder Bearbeiten von Workflows in diesem Repository:

### Workflows lesen

- Das `nodes`-Array parsen, um die Automatisierungsschritte zu verstehen
- `connections` verfolgen, um den Datenfluss vom Trigger bis zur letzten Aktion nachzuverfolgen
- `settings.executionOrder` prüfen — `"v1"` ist das moderne Ausführungsmodell
- Den Trigger-Node identifizieren (Typ enthält meist `Trigger`, z. B. `n8n-nodes-base.webhookTrigger`, `n8n-nodes-base.scheduleTrigger`)

### Workflows bearbeiten

- Alle bestehenden `"id"`-Felder von Nodes und des Workflows selbst beibehalten
- Die Reihenfolge des `nodes`-Arrays nicht verändern — Positionen sind kosmetisch, die Reihenfolge kann jedoch relevant sein
- `connections` konsistent mit hinzugefügten oder entfernten Nodes halten
- JSON-Wohlgeformtheit vor dem Committen prüfen (`jq . <datei.json>`)

### Neue Workflows erstellen

- Von einem exportierten n8n-Workflow-Template ausgehen, nicht von Grund auf neu beginnen
- Datei- und Workflow-Namenskonventionen oben einhalten
- Keine Credential-Werte erfinden — Platzhalter-Strings verwenden

### Sicherheit

- Vor jedem Commit auf versehentliche Secrets prüfen:
  ```bash
  grep -rE "(api_key|apikey|password|secret|token|Bearer)" workflows/ --include="*.json"
  ```
- Falls Secrets gefunden werden, durch `"ERSETZEN"` ersetzen und in der Dokumentation vermerken, welcher Wert benötigt wird
- Keine `.env`-Dateien oder Credential-Exporte in dieses Repository aufnehmen

### JSON-Validierung

Vor dem Committen jede Workflow-JSON validieren:

```bash
# Alle Workflow-JSON-Dateien validieren
for f in workflows/**/*.json; do
  jq empty "$f" && echo "OK: $f" || echo "UNGÜLTIG: $f"
done
```

---

## Umgebungsvariablen / Konfiguration

Falls dieses Repository mit CI/CD oder Skripten verwendet wird, können folgende Umgebungsvariablen referenziert werden:

| Variable | Beschreibung |
|---|---|
| `N8N_API_KEY` | API-Schlüssel zur Authentifizierung an der n8n-Instanz |
| `N8N_BASE_URL` | Basis-URL der n8n-Instanz (z. B. `https://n8n.beispiel.de`) |
| `N8N_ENCRYPTION_KEY` | Verschlüsselungsschlüssel für n8n-Credentials (nur serverseitig) |

Diese Werte müssen in der Umgebung oder in CI-Secrets gesetzt werden — **niemals in dieses Repository committen**.

---

## Referenz: Häufige Node-Typen

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.webhook` | HTTP-Webhook-Trigger |
| `n8n-nodes-base.scheduleTrigger` | Cron-/Intervall-basierter Trigger |
| `n8n-nodes-base.httpRequest` | HTTP-Anfragen an beliebige APIs senden |
| `n8n-nodes-base.set` | Feldwerte transformieren oder setzen |
| `n8n-nodes-base.if` | Bedingte Verzweigung |
| `n8n-nodes-base.switch` | Mehrfach-Routing |
| `n8n-nodes-base.merge` | Daten aus mehreren Zweigen zusammenführen |
| `n8n-nodes-base.code` | Benutzerdefiniertes JavaScript/Python ausführen |
| `n8n-nodes-base.noOp` | Durchleitungs-/Platzhalter-Node |
| `n8n-nodes-base.errorTrigger` | Workflow-Ausführungsfehler abfangen |

---

## Weiterführende Links

- [n8n Dokumentation](https://docs.n8n.io/)
- [n8n Node-Bibliothek](https://n8n.io/integrations/)
- [n8n Community-Forum](https://community.n8n.io/)
- [n8n Workflow-Vorlagen](https://n8n.io/workflows/)
- [n8n REST-API-Referenz](https://docs.n8n.io/api/)
- [n8n CLI-Referenz](https://docs.n8n.io/hosting/cli-commands/)
