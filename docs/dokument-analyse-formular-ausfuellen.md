# Schuldnerberatung: Dokument-Analyse und Formular-Ausfüllung

## Zweck

Dieser Workflow automatisiert die Verarbeitung von Schuldnerberatungs-Dokumenten. Sobald eine neue Datei in einem definierten Google Drive Ordner abgelegt wird, analysiert Claude (Anthropic) das Dokument, extrahiert alle relevanten Daten und füllt automatisch das Schuldnerberatungs-Webformular aus. Gleichzeitig werden die Daten in Google Sheets gespeichert.

---

## Ablauf

```
Google Drive (neue Datei)
        │
        ▼
Dokument herunterladen
        │
        ▼
Datei vorbereiten (Metadaten + Binary)
        │
        ▼
Claude (Anthropic) analysiert Dokument
        │
        ▼
Daten extrahieren (JSON parsen)
        │
        ▼
Analyse erfolgreich?
    ├─── JA ──► Google Sheets speichern
    │            + Webformular ausfüllen
    │
    └─── NEIN ► Fehlerprotokoll in Google Sheets
```

---

## Trigger

- **Typ:** Google Drive Trigger (Polling, jede Minute)
- **Ereignis:** Neue Datei in einem bestimmten Ordner
- **Unterstützte Formate:** PDF, Bilder (JPG, PNG), Google Docs, Word-Dokumente

---

## Extrahierte Felder

| Feld | Beschreibung |
|---|---|
| `vorname` | Vorname des Schuldners |
| `nachname` | Nachname des Schuldners |
| `geburtsdatum` | Geburtsdatum (TT.MM.JJJJ) |
| `strasse` | Straße und Hausnummer |
| `plz` | Postleitzahl |
| `ort` | Wohnort |
| `telefon` | Telefonnummer |
| `email` | E-Mail-Adresse |
| `familienstand` | ledig / verheiratet / geschieden / verwitwet |
| `anzahl_kinder` | Anzahl unterhaltsberechtigter Kinder |
| `beschaeftigung` | angestellt / selbständig / arbeitslos / rentner / sonstiges |
| `arbeitgeber` | Name des Arbeitgebers |
| `netto_einkommen` | Monatliches Nettoeinkommen (€) |
| `sonstige_einnahmen` | Sonstige Einnahmen pro Monat (€) |
| `miete` | Monatliche Wohnkosten (€) |
| `lebenshaltung` | Monatliche Lebenshaltungskosten (€) |
| `gesamtschulden` | Gesamtschuldenbetrag (€) |
| `bank` | Hausbank |
| `glaeubiger` | Array: `[{name, betrag, art}]` |
| `besonderheiten` | Sonstige Anmerkungen |

---

## Benötigte Zugangsdaten

### 1. Google Drive OAuth2
- **Credential-Typ in n8n:** `Google Drive OAuth2`
- **Vorlage:** `credentials/google-drive-vorlage.json`
- **Benötigt:**
  - Google Cloud Projekt mit aktivierter Drive API
  - OAuth2 Client ID und Secret
  - Scopes: `https://www.googleapis.com/auth/drive`

### 2. Google Sheets OAuth2
- **Credential-Typ in n8n:** `Google Sheets OAuth2`
- **Benötigt:**
  - Gleiche OAuth2-Credentials wie Drive oder separate
  - Scopes: `https://www.googleapis.com/auth/spreadsheets`

### 3. Anthropic API Key
- **Credential-Typ in n8n:** `Header Auth`
- **Vorlage:** `credentials/anthropic-api-vorlage.json`
- **Header Name:** `x-api-key`
- **Wert:** Anthropic API Key (`sk-ant-...`)
- **API Konsole:** https://console.anthropic.com/settings/keys

---

## Konfiguration vor dem Import

Folgende Werte müssen nach dem Import in n8n angepasst werden:

| Platzhalter | Node | Beschreibung |
|---|---|---|
| `ERSETZEN_GOOGLE_DRIVE_ORDNER_ID` | Google Drive Trigger | ID des Ordners, der überwacht werden soll |
| `ERSETZEN_GOOGLE_SHEET_ID` | Google Sheets speichern / Fehler protokollieren | ID des Google Sheets Dokuments |
| `ERSETZEN_WEBFORMULAR_URL` | Webformular ausfüllen | URL des Schuldnerberatungs-Webformulars |
| Alle `ERSETZEN_CREDENTIAL_ID` | Verschiedene Nodes | Credential-IDs nach dem Anlegen in n8n |

**Google Drive Ordner-ID finden:**
Die Ordner-ID steht in der URL von Google Drive:
`https://drive.google.com/drive/folders/<ORDNER_ID>`

**Google Sheets ID finden:**
Die Sheet-ID steht in der URL:
`https://docs.google.com/spreadsheets/d/<SHEET_ID>/edit`

---

## Google Sheets Struktur

Erstelle ein Google Sheets Dokument mit zwei Tabellenblättern:

### Blatt 1: "Schuldnerberatung"
Spalten: Datum, Quelldatei, Status, Vorname, Nachname, Geburtsdatum, Straße, PLZ, Ort, Telefon, Email, Familienstand, Anzahl Kinder, Beschäftigung, Arbeitgeber, Netto Einkommen (€), Sonstige Einnahmen (€), Miete (€), Lebenshaltung (€), Gesamtschulden (€), Bank, Besonderheiten, Gläubiger (JSON)

### Blatt 2: "Fehlerprotokoll"
Spalten: Datum, Datei, Fehler, Status

---

## Webformular-Integration

Der Node "Webformular ausfüllen" sendet einen **HTTP POST** mit allen extrahierten Daten als JSON an die Formular-URL. Falls das Zielformular:
- **Andere Feldnamen** verwendet → Code im Node `jsonBody` anpassen
- **Authentifizierung** benötigt → Header Auth im Node ergänzen
- **Multipart/Form-Data** erwartet → Body-Typ im Node ändern

---

## Datenschutz-Hinweise

- Alle verarbeiteten Dokumente enthalten sensible personenbezogene Daten (DSGVO-relevant)
- Google Drive, Google Sheets und Anthropic müssen den DSGVO-Anforderungen entsprechen
- Anthropic-Datenverarbeitungsvertrag (DPA) prüfen und ggf. abschließen
- Zugriff auf den Google Drive Ordner und das Sheets-Dokument auf autorisierte Personen beschränken
- Verarbeitungsprotokoll gemäß Art. 30 DSGVO führen

---

## Import & Aktivierung

```bash
# Workflow importieren
n8n import:workflow --input=workflows/schuldnerberatung/dokument-analyse-formular-ausfuellen.json

# Oder über die n8n-API
curl -X POST \
  -H "X-N8N-API-KEY: $N8N_API_KEY" \
  -H "Content-Type: application/json" \
  -d @workflows/schuldnerberatung/dokument-analyse-formular-ausfuellen.json \
  "$N8N_BASE_URL/api/v1/workflows"
```

Nach dem Import in n8n:
1. Credentials für Google Drive, Google Sheets und Anthropic anlegen
2. Platzhalter-Werte in den Nodes ersetzen
3. Workflow in einer **Staging-Umgebung** testen
4. Workflow aktivieren (Toggle oben rechts in n8n)
