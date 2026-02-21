# CLAUDE.md — N8N Workflows Repository

## Projektübersicht

Dieses Repository speichert [n8n](https://n8n.io/) Workflow-Automatisierungsdefinitionen. n8n ist ein selbst hostbares, knotenbasiertes Automatisierungstool, das APIs, Dienste und Datenquellen ohne benutzerdefinierten Integrationscode verbindet.

Workflows werden als **JSON-Dateien** aus einer n8n-Instanz exportiert und hier gespeichert, um Versionskontrolle, Zusammenarbeit und Wiederverwendung über Umgebungen hinweg zu ermöglichen.

---

## Aktueller Repository-Status

> **Stand: 2026-02-21** — Das Repository befindet sich in der **Einrichtungsphase**.

Aktuell vorhandene Dateien:

```
N8N_Workflows-/
├── .git/
└── CLAUDE.md          ← einzige Datei bisher
```

Noch nicht vorhandene Verzeichnisse (laut geplanter Struktur):
- `workflows/` — noch keine Workflow-JSONs vorhanden
- `credentials/` — noch keine Credential-Vorlagen vorhanden
- `docs/` — noch keine Workflow-Dokumentation vorhanden
- `README.md` — noch nicht erstellt
- `.gitignore` — noch nicht erstellt (**empfohlen, bald hinzuzufügen**)

---

## Repository-Struktur (Zielzustand)

```
N8N_Workflows-/
├── CLAUDE.md                  # Diese Datei
├── .gitignore                 # Ignoriert .env, temporäre Dateien usw.
├── README.md                  # Projektübersicht für Menschen
├── workflows/                 # Workflow-JSON-Dateien (nach Kategorie oder Funktion)
│   ├── <kategorie>/
│   │   └── <workflow-name>.json
│   └── <workflow-name>.json
├── credentials/               # Credential-Vorlagen (KEINE echten Zugangsdaten — nur Platzhalter)
│   └── <dienst>-vorlage.json
└── docs/                      # Menschenlesbare Dokumentation pro Workflow
    └── <workflow-name>.md
```

> **Hinweis:** Die genaue Verzeichnisstruktur kann sich weiterentwickeln. Workflows sollten logisch nach Integration, Abteilung oder Funktion gruppiert werden.

### Empfohlener .gitignore-Inhalt

```gitignore
# Umgebungsvariablen und Secrets
.env
.env.*
*.env

# n8n lokale Daten
.n8n/

# Betriebssystem-Dateien
.DS_Store
Thumbs.db

# Editor-Konfigurationen
.vscode/
.idea/
*.swp
*.swo

# Temporäre Dateien
*.tmp
*.bak
```

---

## Was ist eine n8n-Workflow-JSON?

Jede `.json`-Datei ist ein vollständiger Workflow-Export aus n8n und enthält:

- **`id`** — Eindeutige ID des Workflows (UUID oder Integer, je nach n8n-Version)
- **`name`** — Anzeigename des Workflows
- **`nodes`** — Array von Node-Objekten (jeder Node ist ein Schritt im Automatisierungsablauf)
- **`connections`** — Verbindungen zwischen Nodes (was in was einfließt)
- **`settings`** — Ausführungseinstellungen (Timeout, Fehlerbehandlung, Zeitzone)
- **`staticData`** — Persistierter Zustand über Ausführungen hinweg (falls vorhanden)
- **`meta`** — Workflow-Metadaten (n8n-Version, Template-ID)
- **`tags`** — Optionale Tags zur Kategorisierung
- **`active`** — Boolean, ob der Workflow aktiviert ist (bei Export meist `false`)

### Vollständiges Beispiel einer Workflow-Struktur

```json
{
  "id": "abc123",
  "name": "Slack Benachrichtigung bei Fehler",
  "active": false,
  "nodes": [
    {
      "id": "uuid-des-nodes",
      "name": "Fehler-Trigger",
      "type": "n8n-nodes-base.errorTrigger",
      "typeVersion": 1,
      "position": [250, 300],
      "parameters": {}
    },
    {
      "id": "uuid-des-zweiten-nodes",
      "name": "Slack Nachricht",
      "type": "n8n-nodes-base.slack",
      "typeVersion": 2,
      "position": [480, 300],
      "parameters": {
        "channel": "#fehler-alerts",
        "text": "={{ $json.execution.id }}"
      },
      "credentials": {
        "slackApi": {
          "id": "ERSETZEN",
          "name": "Slack account"
        }
      }
    }
  ],
  "connections": {
    "Fehler-Trigger": {
      "main": [
        [
          {
            "node": "Slack Nachricht",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "settings": {
    "executionOrder": "v1",
    "saveManualExecutions": true,
    "callerPolicy": "workflowsFromSameOwner",
    "errorWorkflow": ""
  },
  "staticData": null,
  "tags": [],
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "ERSETZEN"
  }
}
```

---

## n8n Expression-Syntax

n8n verwendet eine eigene Template-Syntax für dynamische Werte in Node-Parametern. KI-Assistenten müssen diese Syntax kennen, um Workflows korrekt zu lesen und zu bearbeiten.

### Grundlegende Ausdrücke

```javascript
// Auf Daten des aktuellen Items zugreifen
{{ $json.feldName }}
{{ $json["feld-mit-bindestrich"] }}

// Auf Ausgabe eines bestimmten Nodes zugreifen
{{ $node["Node Name"].json.feldName }}

// Auf alle Items eines Nodes zugreifen
{{ $items("Node Name") }}

// Umgebungsvariablen
{{ $env.VARIABLE_NAME }}

// Aktueller Zeitstempel
{{ $now }}
{{ $today }}

// Workflow-Metadaten
{{ $workflow.id }}
{{ $workflow.name }}
{{ $execution.id }}
```

### Häufige Ausdrucksmuster

```javascript
// Bedingte Werte
{{ $json.status === "aktiv" ? "Ja" : "Nein" }}

// String-Verkettung
{{ "Hallo " + $json.name }}

// Datumformatierung
{{ $now.toFormat("yyyy-MM-dd") }}

// Array-Länge prüfen
{{ $json.items.length > 0 }}

// Auf verschachtelte Objekte zugreifen
{{ $json.benutzer.adresse.stadt }}
```

> **Wichtig:** Ausdrücke werden nur in Node-Parametern ausgewertet, die das `=`-Präfix in der n8n-Oberfläche haben. Im JSON erkennbar daran, dass der Wert mit `=` beginnt: `"={{ expression }}"`.

---

## Wichtige Konventionen

### Dateinamen

- **Kebab-Case** für alle Dateinamen verwenden: `slack-benachrichtigung-bei-fehler.json`
- Bei fehlendem Unterverzeichnis mit einer Kategorie prefixen: `crm-lead-synchronisation.json`
- Keine Leerzeichen oder Sonderzeichen in Dateinamen
- Nur Kleinbuchstaben, Ziffern und Bindestriche

### Workflow-Namen (innerhalb der JSON)

- Das Feld `"name"` soll **menschenlesbar** und beschreibend sein
- Titelschreibweise verwenden: `"Slack Benachrichtigung bei Fehler"`
- Namen auf 60 Zeichen begrenzen

### Kategorisierung (Verzeichnisstruktur)

Workflows im `workflows/`-Verzeichnis nach Funktion oder Integration gruppieren:

```
workflows/
├── crm/                   # CRM-Integrationen (HubSpot, Salesforce)
├── kommunikation/         # Slack, E-Mail, Teams
├── daten/                 # Datenverarbeitung und -transformation
├── monitoring/            # Fehler-Alerts, Health-Checks
├── finanzen/              # Stripe, Buchhaltung
└── integrationen/         # Sonstige API-Verbindungen
```

### Zugangsdaten (Credentials)

- **Niemals echte Zugangsdaten, API-Schlüssel, Tokens oder Passwörter committen**
- Sensible Werte in exportierten JSONs durch Platzhalter ersetzen: `"ERSETZEN"` oder `"<DEIN_API_SCHLÜSSEL>"`
- Im Verzeichnis `credentials/` eine Vorlagendatei bereitstellen, die erklärt, was jedes Feld benötigt
- n8ns eingebauten Credential-Manager auf dem Server nutzen — hier liegen nur Schema-Vorlagen
- Das Feld `credentials[*].id` ebenfalls durch `"ERSETZEN"` ersetzen (enthält interne n8n-IDs)

### Node-IDs

- n8n generiert UUIDs für jeden Node (Feld `"id"` innerhalb der Nodes)
- Node-IDs niemals manuell bearbeiten — sie werden für interne Verbindungsreferenzen verwendet
- Beim Zusammenführen oder Deduplizieren IDs stabil halten
- Die `"id"`-Felder auf Workflow-Ebene ebenfalls beibehalten

### Workflow-Aktivierungsstatus

- Exportierte Workflows haben `"active": false` — das ist korrekt und erwünscht
- Workflows werden nach dem Import manuell in der n8n-Instanz aktiviert
- Den `"active"`-Wert beim Committen nie auf `true` setzen

### Versionskontrolle

- **Ein Workflow pro Datei** — niemals mehrere unzusammenhängende Workflows in eine JSON bündeln
- Export aus n8n über `Workflow → Herunterladen` oder die n8n-CLI/API
- Import in n8n über `Workflow → Aus Datei importieren` oder die n8n-CLI/API
- Workflows immer zuerst in einer **Staging-n8n-Instanz** testen, bevor sie ins Repository aufgenommen werden

---

## Fehlerbehandlung in Workflows

### Fehler-Workflow-Muster

n8n erlaubt das Definieren eines separaten Fehler-Workflows (`settings.errorWorkflow`). Typisches Muster:

1. **Haupt-Workflow** — führt die eigentliche Logik aus
2. **Fehler-Workflow** — wird bei Fehlern im Haupt-Workflow ausgelöst (`errorTrigger`-Node)

### Retry-Logik

Node-Parameter für Wiederholungsversuche (im `parameters`-Objekt eines Nodes):

```json
"onError": "continueErrorOutput",
"retryOnFail": true,
"maxTries": 3,
"waitBetweenTries": 1000
```

### Typische Fehlerbehandlungsknoten

- **`n8n-nodes-base.errorTrigger`** — startet Workflow bei Fehlern in anderen Workflows
- **`n8n-nodes-base.if`** mit Bedingung auf `$json.error` — prüft auf Fehler in HTTP-Antworten
- **`n8n-nodes-base.stopAndError`** — bricht Workflow bewusst mit Fehlermeldung ab

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
   - Bekannte Einschränkungen oder Abhängigkeiten
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
chore: .gitignore hinzugefügt
```

Erlaubte Präfixe: `add`, `update`, `fix`, `remove`, `docs`, `refactor`, `chore`

### Branch-Strategie

- `master` / `main` — stabiler Zustand, nur über Pull Requests aktualisieren
- Feature-Branches für neue Workflows: `feature/<workflow-name>`
- Bugfix-Branches: `fix/<beschreibung>`

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

# Alle Workflows importieren
n8n import:workflow --input=workflows/
```

Über die n8n-REST-API:

```bash
# Alle Workflows auflisten
curl -H "X-N8N-API-KEY: $N8N_API_KEY" \
  "$N8N_BASE_URL/api/v1/workflows" | jq .

# Einzelnen Workflow exportieren
curl -H "X-N8N-API-KEY: $N8N_API_KEY" \
  "$N8N_BASE_URL/api/v1/workflows/<id>" \
  | jq . > workflows/<name>.json

# Workflow importieren (neu erstellen)
curl -X POST \
  -H "X-N8N-API-KEY: $N8N_API_KEY" \
  -H "Content-Type: application/json" \
  -d @workflows/<name>.json \
  "$N8N_BASE_URL/api/v1/workflows"

# Workflow aktualisieren (ersetzen)
curl -X PUT \
  -H "X-N8N-API-KEY: $N8N_API_KEY" \
  -H "Content-Type: application/json" \
  -d @workflows/<name>.json \
  "$N8N_BASE_URL/api/v1/workflows/<id>"
```

---

## Richtlinien für KI-Assistenten

Beim Analysieren oder Bearbeiten von Workflows in diesem Repository:

### Workflows lesen

- Das `nodes`-Array parsen, um die Automatisierungsschritte zu verstehen
- `connections` verfolgen, um den Datenfluss vom Trigger bis zur letzten Aktion nachzuverfolgen
- `settings.executionOrder` prüfen — `"v1"` ist das moderne Ausführungsmodell
- Den Trigger-Node identifizieren (Typ enthält meist `Trigger`, z. B. `n8n-nodes-base.webhookTrigger`, `n8n-nodes-base.scheduleTrigger`)
- Node-Ausdrücke (`={{ ... }}`) erkennen und interpretieren (siehe Expression-Syntax-Abschnitt)
- `typeVersion` beachten — höhere Versionen eines Nodes haben andere Parameter-Strukturen

### Workflows bearbeiten

- Alle bestehenden `"id"`-Felder von Nodes und des Workflows selbst beibehalten
- Die Reihenfolge des `nodes`-Arrays nicht verändern — Positionen sind kosmetisch, die Reihenfolge kann jedoch relevant sein
- `connections` konsistent mit hinzugefügten oder entfernten Nodes halten
- JSON-Wohlgeformtheit vor dem Committen prüfen (`jq . <datei.json>`)
- Node-`position`-Koordinaten beim Hinzufügen neuer Nodes logisch anordnen (kein Überlappen)

### Neue Workflows erstellen

- Von einem exportierten n8n-Workflow-Template ausgehen, nicht von Grund auf neu beginnen
- Datei- und Workflow-Namenskonventionen oben einhalten
- Keine Credential-Werte erfinden — Platzhalter-Strings `"ERSETZEN"` verwenden
- Für neue Nodes plausible UUIDs generieren (Format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
- Den `"active": false`-Status beibehalten

### Connections-Format verstehen

```json
"connections": {
  "<Quell-Node-Name>": {
    "main": [
      [
        { "node": "<Ziel-Node-Name>", "type": "main", "index": 0 }
      ]
    ]
  }
}
```

- Das äußere Array (`main[0]`, `main[1]`) entspricht dem Ausgangs-Index des Quell-Nodes (z. B. True/False bei IF-Nodes)
- Das innere Array enthält alle Ziel-Nodes für diesen Ausgang

### Sicherheit

- Vor jedem Commit auf versehentliche Secrets prüfen:
  ```bash
  grep -rE "(api_key|apikey|password|secret|token|Bearer|Authorization)" workflows/ --include="*.json" -i
  ```
- Falls Secrets gefunden werden, durch `"ERSETZEN"` ersetzen und in der Dokumentation vermerken, welcher Wert benötigt wird
- Keine `.env`-Dateien oder Credential-Exporte in dieses Repository aufnehmen
- `meta.instanceId` und Credential-IDs ebenfalls durch `"ERSETZEN"` ersetzen

### JSON-Validierung

Vor dem Committen jede Workflow-JSON validieren:

```bash
# Einzelne Datei validieren
jq empty workflows/<name>.json && echo "OK" || echo "UNGÜLTIG"

# Alle Workflow-JSON-Dateien validieren
for f in workflows/**/*.json workflows/*.json; do
  [ -f "$f" ] && (jq empty "$f" && echo "OK: $f" || echo "UNGÜLTIG: $f")
done

# Secret-Check
grep -rE "(api_key|password|secret|token|Bearer)" workflows/ --include="*.json" -il
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

### Trigger-Nodes

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.webhook` | HTTP-Webhook-Trigger (empfängt externe Anfragen) |
| `n8n-nodes-base.scheduleTrigger` | Cron-/Intervall-basierter Trigger |
| `n8n-nodes-base.emailReadImap` | E-Mails via IMAP empfangen |
| `n8n-nodes-base.errorTrigger` | Workflow-Ausführungsfehler abfangen |
| `n8n-nodes-base.manualTrigger` | Manueller Start (nur für Tests) |
| `n8n-nodes-base.formTrigger` | n8n-eigenes Formular als Trigger |

### Logik- und Transformations-Nodes

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.if` | Bedingte Verzweigung (True/False) |
| `n8n-nodes-base.switch` | Mehrfach-Routing (mehrere Ausgänge) |
| `n8n-nodes-base.merge` | Daten aus mehreren Zweigen zusammenführen |
| `n8n-nodes-base.set` | Feldwerte transformieren oder setzen |
| `n8n-nodes-base.code` | Benutzerdefiniertes JavaScript/Python ausführen |
| `n8n-nodes-base.function` | Ältere Version von `code` (deprecated) |
| `n8n-nodes-base.noOp` | Durchleitungs-/Platzhalter-Node |
| `n8n-nodes-base.splitInBatches` | Items in Gruppen aufteilen |
| `n8n-nodes-base.aggregate` | Items zusammenfassen |
| `n8n-nodes-base.filter` | Items nach Bedingungen filtern |
| `n8n-nodes-base.sort` | Items sortieren |
| `n8n-nodes-base.limit` | Anzahl der Items begrenzen |
| `n8n-nodes-base.removeDuplicates` | Duplikate entfernen |

### HTTP und Daten-Nodes

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.httpRequest` | HTTP-Anfragen an beliebige APIs senden |
| `n8n-nodes-base.graphql` | GraphQL-Anfragen |
| `n8n-nodes-base.xml` | XML parsen oder erstellen |
| `n8n-nodes-base.html` | HTML extrahieren |
| `n8n-nodes-base.extractFromFile` | Daten aus Dateien extrahieren |
| `n8n-nodes-base.readWriteFile` | Dateien lesen/schreiben |
| `n8n-nodes-base.spreadsheetFile` | Excel/CSV-Dateien verarbeiten |

### Fehlerbehandlungs-Nodes

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.stopAndError` | Workflow mit Fehler abbrechen |
| `n8n-nodes-base.wait` | Workflow pausieren (Zeit oder Webhook) |

---

## Weiterführende Links

- [n8n Dokumentation](https://docs.n8n.io/)
- [n8n Node-Bibliothek](https://n8n.io/integrations/)
- [n8n Expression-Referenz](https://docs.n8n.io/code/expressions/)
- [n8n Community-Forum](https://community.n8n.io/)
- [n8n Workflow-Vorlagen](https://n8n.io/workflows/)
- [n8n REST-API-Referenz](https://docs.n8n.io/api/)
- [n8n CLI-Referenz](https://docs.n8n.io/hosting/cli-commands/)
- [n8n Built-in Variables](https://docs.n8n.io/code/builtin/overview/)
