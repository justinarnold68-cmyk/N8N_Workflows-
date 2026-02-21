# CLAUDE.md — N8N Workflows Repository

## Projektübersicht

Dieses Repository speichert [n8n](https://n8n.io/) Workflow-Automatisierungsdefinitionen. n8n ist ein selbst hostbares, knotenbasiertes Automatisierungstool, das APIs, Dienste und Datenquellen ohne benutzerdefinierten Integrationscode verbindet.

Workflows werden als **JSON-Dateien** aus einer n8n-Instanz exportiert und hier gespeichert, um Versionskontrolle, Zusammenarbeit und Wiederverwendung über Umgebungen hinweg zu ermöglichen.

---

## Aktueller Status (Stand: 2026-02-21)

Das Repository befindet sich in der **Initialisierungsphase**. Bisher wurde nur `CLAUDE.md` hinzugefügt. Die folgenden Verzeichnisse und Dateien müssen noch angelegt werden:

| Pfad | Status |
|---|---|
| `workflows/` | Noch nicht erstellt |
| `docs/` | Noch nicht erstellt |
| `credentials/` | Noch nicht erstellt |
| `README.md` | Noch nicht erstellt |

Beim Hinzufügen des ersten Workflows bitte die gesamte beschriebene Struktur anlegen.

---

## Mein Setup & Präferenzen

### n8n Instanz

- **Typ:** n8n Cloud (nicht self-hosted)
- **Version:** 1.123.14
- **Sprache:** Deutsch — alle Kommunikation, Dateinamen in Docs und Commit-Nachrichten auf Deutsch

### Aktive Credentials / Integrationen

| Dienst | Verwendungszweck |
|---|---|
| Google Sheets | Dateneingabe, einfache Listen, temporäre Zwischenspeicherung |
| Gmail | E-Mail-Versand und -Empfang |
| *(weitere hier ergänzen)* | *(Beschreibung)* |

### Kundendaten — Speicherung & Sicherheit

- **Ziel:** Kundendaten DSGVO-konform und sicher speichern
- **Empfohlene Lösung:** Supabase (PostgreSQL, EU-Server, kostenlos bis ~500 MB) oder Airtable
- **Status:** Noch nicht entschieden — beim ersten Kunden-Workflow gemeinsam festlegen
- Google Sheets **nicht** für sensible Kundendaten verwenden
- Keine echten Kundendaten in Workflow-JSONs committen — nur Platzhalter

### Workflow-Präferenzen

- Lieber **native n8n-Nodes** als Code-Nodes (einfacher zu warten)
- Node-Namen sollen **beschreibend** sein, damit der Workflow selbsterklärend ist
- Jeder Workflow bekommt eine **Docs-Datei** in `docs/`
- Beim Erstellen eines Workflows immer zuerst fragen: Trigger → Logik → Aktion

### Wie ich mit Claude arbeite

- Claude liest diese Datei automatisch — hier notieren was er "wissen" soll
- Neue Erkenntnisse, Entscheidungen oder Tools hier ergänzen
- Workflow-Status in der Tabelle unter "Aktueller Status" aktuell halten
- Einfach sagen: *"Baue mir einen Workflow der X macht"* — Claude liefert die JSON

---

## Repository-Struktur

```
N8N_Workflows-/
├── CLAUDE.md                  # Diese Datei
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

> **Hinweis:** Workflows sollten logisch nach Integration, Abteilung oder Funktion gruppiert werden. Beispielkategorien: `crm/`, `marketing/`, `finance/`, `devops/`, `notifications/`.

---

## Was ist eine n8n-Workflow-JSON?

Jede `.json`-Datei ist ein vollständiger Workflow-Export aus n8n und enthält:

- **`name`** — Anzeigename des Workflows
- **`nodes`** — Array von Node-Objekten (jeder Node ist ein Schritt im Automatisierungsablauf)
- **`connections`** — Verbindungen zwischen Nodes (was in was einfließt)
- **`settings`** — Ausführungseinstellungen (Timeout, Fehlerbehandlung, Zeitzone)
- **`staticData`** — Persistierter Zustand über Ausführungen hinweg (falls vorhanden)
- **`meta`** — Workflow-Metadaten (n8n-Version, Template-ID)
- **`pinData`** — Festgepinnte Testdaten für Nodes (optional)

Beispiel einer minimalen Struktur:

```json
{
  "name": "Mein Workflow",
  "nodes": [
    {
      "id": "uuid-hier",
      "name": "Start",
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [240, 300],
      "parameters": {}
    }
  ],
  "connections": {},
  "settings": {
    "executionOrder": "v1"
  },
  "staticData": null,
  "meta": {
    "templateCredsSetupCompleted": true
  }
}
```

---

## Wichtige Konventionen

### Dateinamen

- **Kebab-Case** für alle Dateinamen: `slack-benachrichtigung-bei-fehler.json`
- Bei fehlendem Unterverzeichnis mit einer Kategorie prefixen: `crm-lead-synchronisation.json`
- Keine Leerzeichen, Umlaute oder Sonderzeichen in Dateinamen
- Nur ASCII-Zeichen und Bindestriche verwenden

### Workflow-Namen (innerhalb der JSON)

- Das Feld `"name"` soll **menschenlesbar** und beschreibend sein
- Titelschreibweise verwenden: `"Slack Benachrichtigung bei Fehler"`
- Namen auf 60 Zeichen begrenzen

### Zugangsdaten (Credentials)

- **Niemals echte Zugangsdaten, API-Schlüssel, Tokens oder Passwörter committen**
- Sensible Werte in exportierten JSONs durch Platzhalter ersetzen: `"<DEIN_API_SCHLÜSSEL>"` oder `"ERSETZEN"`
- Im Verzeichnis `credentials/` eine Vorlagendatei bereitstellen, die erklärt, was jedes Feld benötigt
- n8ns eingebauten Credential-Manager auf dem Server nutzen — hier liegen nur Schema-Vorlagen

### Node-IDs

- n8n generiert UUIDs für jeden Node (Feld `"id"` innerhalb der Nodes)
- Node-IDs **niemals manuell bearbeiten** — sie werden für interne Verbindungsreferenzen verwendet
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

## Git-Konventionen

### Branching

- `master` — stabiler Hauptbranch; nur getestete Workflows
- Feature-Branches: `add/<workflow-name>`, `update/<workflow-name>`, `fix/<workflow-name>`
- Claude-Branches: `claude/<beschreibung>-<session-id>` (automatisch generiert)

### Commit-Nachrichten

Klare, imperativische Commit-Nachrichten mit festem Präfix verwenden:

```
add: slack-benachrichtigung-bei-fehler Workflow
update: crm-lead-synchronisation - Wiederholungslogik bei HTTP 429 ergänzt
fix: fehlerhafte Verbindung im daten-anreicherung Workflow behoben
remove: legacy-hubspot-sync (ersetzt durch hubspot-v2-sync)
docs: Dokumentation für stripe-zahlungserfassung hinzugefügt
refactor: daten-anreicherung in Unterverzeichnis verschoben
chore: CLAUDE.md aktualisiert
```

Erlaubte Präfixe: `add`, `update`, `fix`, `remove`, `docs`, `refactor`, `chore`

---

## Workflows importieren & exportieren

### n8n-CLI

```bash
# Workflow nach ID exportieren
n8n export:workflow --id=<workflow-id> --output=workflows/<name>.json

# Workflow importieren
n8n import:workflow --input=workflows/<name>.json

# Alle Workflows exportieren
n8n export:workflow --all --output=workflows/
```

### n8n-REST-API

```bash
# Export via API
curl -H "X-N8N-API-KEY: $N8N_API_KEY" \
  "$N8N_BASE_URL/api/v1/workflows/<id>" \
  | jq . > workflows/<name>.json

# Import via API
curl -X POST \
  -H "X-N8N-API-KEY: $N8N_API_KEY" \
  -H "Content-Type: application/json" \
  -d @workflows/<name>.json \
  "$N8N_BASE_URL/api/v1/workflows"

# Alle Workflows auflisten
curl -H "X-N8N-API-KEY: $N8N_API_KEY" \
  "$N8N_BASE_URL/api/v1/workflows" | jq '.data[].name'
```

---

## Richtlinien für KI-Assistenten

### Workflows lesen

- Das `nodes`-Array parsen, um die Automatisierungsschritte zu verstehen
- `connections` verfolgen, um den Datenfluss vom Trigger bis zur letzten Aktion nachzuverfolgen
- `settings.executionOrder` prüfen — `"v1"` ist das moderne Ausführungsmodell
- Den Trigger-Node identifizieren (Typ enthält meist `Trigger`)
- `pinData` beachten — festgepinnte Daten überschreiben echte Node-Ausgaben beim Testen

### Workflows bearbeiten

- Alle bestehenden `"id"`-Felder von Nodes und des Workflows selbst **beibehalten**
- Die Reihenfolge des `nodes`-Arrays nicht verändern
- `connections` konsistent mit hinzugefügten oder entfernten Nodes halten
- `typeVersion`-Werte nicht ohne Grund ändern — sie steuern das Node-Verhalten
- JSON-Wohlgeformtheit vor dem Committen prüfen: `jq . <datei.json>`
- `position`-Felder in Nodes können angepasst werden (nur kosmetisch)

### Neue Workflows erstellen

- Von einem exportierten n8n-Workflow-Template ausgehen, nicht von Grund auf neu beginnen
- Datei- und Workflow-Namenskonventionen einhalten
- Keine Credential-Werte erfinden — Platzhalter-Strings verwenden
- Sicherstellen, dass jeder Workflow genau einen Trigger-Node hat

### Sicherheit

Vor jedem Commit auf versehentliche Secrets prüfen:

```bash
grep -rE "(api_key|apikey|password|secret|token|Bearer|Authorization)" \
  workflows/ --include="*.json" -i
```

Falls Secrets gefunden werden:
1. Durch `"ERSETZEN"` ersetzen
2. In der `docs/`-Datei vermerken, welcher Wert benötigt wird
3. In `credentials/<dienst>-vorlage.json` eine Vorlage bereitstellen

Niemals hinzufügen:
- `.env`-Dateien
- Credential-Exporte aus n8n
- Private Schlüssel oder Zertifikate

### JSON-Validierung

Vor dem Committen alle Workflow-JSONs validieren:

```bash
# Einzelne Datei
jq empty workflows/mein-workflow.json && echo "OK"

# Alle Dateien rekursiv
find workflows/ -name "*.json" -exec sh -c \
  'jq empty "$1" && echo "OK: $1" || echo "UNGÜLTIG: $1"' _ {} \;
```

### Häufige Fehlerquellen

| Problem | Ursache | Lösung |
|---|---|---|
| Workflow importiert nicht | Ungültige JSON-Syntax | `jq empty` ausführen |
| Node verbindet sich nicht | Falsche Node-ID in `connections` | Node-IDs aus `nodes[].id` prüfen |
| Credential fehlt | Platzhalter nicht ersetzt | Echte Credentials in n8n eintragen |
| Falsches Ausführungsverhalten | `executionOrder` nicht `"v1"` | `settings.executionOrder` auf `"v1"` setzen |

---

## Umgebungsvariablen / Konfiguration

| Variable | Beschreibung |
|---|---|
| `N8N_API_KEY` | API-Schlüssel zur Authentifizierung an der n8n-Instanz |
| `N8N_BASE_URL` | Basis-URL der n8n-Instanz (z. B. `https://n8n.beispiel.de`) |
| `N8N_ENCRYPTION_KEY` | Verschlüsselungsschlüssel für n8n-Credentials (nur serverseitig) |

Diese Werte müssen in der Umgebung oder in CI-Secrets gesetzt werden — **niemals in dieses Repository committen**.

---

## Referenz: Node-Typen

### Trigger-Nodes

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.manualTrigger` | Manuell ausgelöste Ausführung |
| `n8n-nodes-base.webhook` | HTTP-Webhook-Trigger (POST/GET/etc.) |
| `n8n-nodes-base.scheduleTrigger` | Cron-/Intervall-basierter Trigger |
| `n8n-nodes-base.emailTrigger` | Ausgelöst durch eingehende E-Mail (IMAP) |
| `n8n-nodes-base.errorTrigger` | Workflow-Ausführungsfehler abfangen |

### Logik & Datenverarbeitung

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.set` | Feldwerte transformieren oder setzen |
| `n8n-nodes-base.if` | Bedingte Verzweigung (wahr/falsch) |
| `n8n-nodes-base.switch` | Mehrfach-Routing nach Wert |
| `n8n-nodes-base.merge` | Daten aus mehreren Zweigen zusammenführen |
| `n8n-nodes-base.splitInBatches` | Große Datensätze in Batches aufteilen |
| `n8n-nodes-base.filter` | Items nach Bedingung herausfiltern |
| `n8n-nodes-base.aggregate` | Items zu einem einzelnen zusammenfassen |
| `n8n-nodes-base.sort` | Items sortieren |
| `n8n-nodes-base.limit` | Anzahl der Items begrenzen |
| `n8n-nodes-base.removeDuplicates` | Doppelte Items entfernen |
| `n8n-nodes-base.code` | Benutzerdefiniertes JavaScript/Python ausführen |
| `n8n-nodes-base.noOp` | Durchleitungs-/Platzhalter-Node |
| `n8n-nodes-base.wait` | Ausführung pausieren (Zeitverzögerung oder Webhook-Fortsetzung) |

### Kommunikation & Integration

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.httpRequest` | HTTP-Anfragen an beliebige APIs |
| `n8n-nodes-base.emailSend` | E-Mail versenden (SMTP) |
| `n8n-nodes-base.slack` | Slack-Nachrichten und -Aktionen |
| `n8n-nodes-base.telegram` | Telegram-Nachrichten |
| `n8n-nodes-base.gmail` | Gmail-Integration |
| `n8n-nodes-base.googleSheets` | Google Sheets lesen/schreiben |
| `n8n-nodes-base.airtable` | Airtable-Datenbankoperationen |
| `n8n-nodes-base.notion` | Notion-Seiten und -Datenbanken |
| `n8n-nodes-base.github` | GitHub-Repositories und -Issues |

### Datenbank & Datei

| Node-Typ | Zweck |
|---|---|
| `n8n-nodes-base.postgres` | PostgreSQL-Datenbankoperationen |
| `n8n-nodes-base.mysql` | MySQL-Datenbankoperationen |
| `n8n-nodes-base.mongodb` | MongoDB-Operationen |
| `n8n-nodes-base.redis` | Redis-Schlüssel-Wert-Speicher |
| `n8n-nodes-base.readWriteFile` | Lokale Dateien lesen/schreiben |
| `n8n-nodes-base.xml` | XML parsen und konvertieren |
| `n8n-nodes-base.spreadsheetFile` | Excel/CSV-Dateien verarbeiten |

---

## Weiterführende Links

- [n8n Dokumentation](https://docs.n8n.io/)
- [n8n Node-Bibliothek](https://n8n.io/integrations/)
- [n8n Community-Forum](https://community.n8n.io/)
- [n8n Workflow-Vorlagen](https://n8n.io/workflows/)
- [n8n REST-API-Referenz](https://docs.n8n.io/api/)
- [n8n CLI-Referenz](https://docs.n8n.io/hosting/cli-commands/)
- [n8n Self-Hosting Guide](https://docs.n8n.io/hosting/)
