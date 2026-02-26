# Schuldnerberatung: Dokument-Analyse und Formular-Ausfüllung

## Zweck

Dieser Workflow automatisiert die Verarbeitung von **Schuldenbriefen, Mahnungen, Inkassoschreiben und Gerichtsbescheiden**. Sobald ein neues Dokument in den Google Drive Ordner hochgeladen wird, analysiert Claude (Anthropic) den Inhalt, extrahiert alle relevanten Daten und füllt automatisch das Standard-Schuldnerberatungs-Webformular aus. Parallel werden alle Daten strukturiert in Google Sheets gespeichert.

---

## Welche Dokumente werden verarbeitet?

| Dokumenttyp | Erkannte Mahnstufen |
|---|---|
| Zahlungserinnerung | `ERSTE_MAHNUNG` |
| 2. / 3. Mahnung | `ZWEITE_MAHNUNG`, `DRITTE_MAHNUNG` |
| Inkassoschreiben | `INKASSO` |
| Anwaltsschreiben | `KLAGE` |
| Mahnbescheid (Gericht) | `MAHNBESCHEID` |
| Vollstreckungsbescheid | `VOLLSTRECKUNG` |
| Pfändungsbescheid | `PFAENDUNG` |

---

## Was wird aus den Dokumenten extrahiert?

### Schuldner (Empfänger des Schreibens)
| Feld | Beschreibung |
|---|---|
| `schuldner_vorname` | Vorname |
| `schuldner_nachname` | Nachname |
| `schuldner_geburtsdatum` | Geburtsdatum (TT.MM.JJJJ) |
| `schuldner_strasse` | Straße und Hausnummer |
| `schuldner_plz` | Postleitzahl |
| `schuldner_ort` | Wohnort |

### Absender / Gläubiger
| Feld | Beschreibung |
|---|---|
| `absender_name` | Name des Gläubigers, Inkassobüros oder der Kanzlei |
| `absender_typ` | `GLAEUBIGER` / `INKASSO` / `ANWALT` / `GERICHT` |
| `urspruenglicher_glaeubiger` | Bei Inkasso/Anwalt: Ursprünglicher Gläubiger |
| `art_der_schuld` | `KREDIT` / `MIETE` / `STROM_GAS` / `TELEKOMMUNIKATION` / `VERSICHERUNG` / `KAUFVERTRAG` / `KRANKENVERSICHERUNG` / `SONSTIGES` |
| `ursprungsvertrag` | z.B. „Ratenkredit", „Mobilfunkvertrag" |
| `aktenzeichen` | Aktenzeichen / Referenznummer |
| `vertragsnummer` | Vertragsnummer oder Kontonummer |

### Forderungsbeträge
| Feld | Beschreibung |
|---|---|
| `hauptforderung` | Hauptforderung in € |
| `zinsen` | Zinsbetrag in € |
| `mahngebuehren` | Mahngebühren in € |
| `inkassokosten` | Inkasso- / Anwaltskosten in € |
| `gesamtforderung` | **Gesamtbetrag in €** |
| `faellig_seit` | Datum seit wann fällig |

### Mahnung / Fristen
| Feld | Beschreibung |
|---|---|
| `mahndatum` | Datum des Schreibens |
| `mahnstufe` | Stufe der Mahnung (s. Tabelle oben) |
| `zahlungsfrist` | Datum bis wann gezahlt werden muss |
| `zahlungsfrist_tage` | Anzahl der verbleibenden Tage |

### Zahlungsdaten des Gläubigers
| Feld | Beschreibung |
|---|---|
| `iban_glaeubiger` | IBAN für die Zahlung |
| `bic_glaeubiger` | BIC |
| `bank_glaeubiger` | Bankname des Gläubigers |
| `verwendungszweck` | Anzugebender Verwendungszweck |

### Rechtliche Situation
| Feld | Beschreibung |
|---|---|
| `rechtliche_schritte_angedroht` | `true` / `false` |
| `art_rechtliche_schritte` | z.B. „Klage", „Pfändung", „Schufa-Meldung" |
| `bereits_tituliert` | Ob Vollstreckungstitel vorliegt |
| `aktenzeichen_gericht` | Gerichtliches Aktenzeichen |

### Sonderoptionen
| Feld | Beschreibung |
|---|---|
| `ratenzahlung_angeboten` | Ob Ratenzahlung angeboten wurde |
| `ratenzahlung_betrag` | Angebotener Ratenbetrag in € |
| `vergleich_angeboten` | Ob ein Vergleich angeboten wurde |
| `vergleich_betrag` | Angebotener Vergleichsbetrag |
| `besonderheiten` | Weitere wichtige Informationen |

---

## Ablauf

```
Google Drive (neue Datei hochgeladen)
        │
        ▼
Dokument herunterladen
        │
        ▼
Datei vorbereiten (Metadaten ermitteln)
        │
        ▼
Claude analysiert Schuldenbrief/Mahnung
(extrahiert alle Felder oben)
        │
        ▼
Daten extrahieren (JSON parsen)
        │
        ▼
Analyse erfolgreich?
    ├─── JA ──► Google Sheets speichern (Blatt: "Schuldnerberatung")
    │            + Schuldnerberatungs-Webformular ausfüllen
    │
    └─── NEIN ► Fehler protokollieren (Blatt: "Fehlerprotokoll")
```

---

## Trigger

- **Typ:** Google Drive Trigger (Polling, jede Minute)
- **Ereignis:** Neue Datei in einem bestimmten Ordner
- **Empfohlene Formate:** PDF, JPG, PNG, TIFF (Scans von Briefen)

---

## Benötigte Zugangsdaten

### 1. Google Drive OAuth2
- **Credential-Typ in n8n:** `Google Drive OAuth2`
- **Vorlage:** `credentials/google-drive-vorlage.json`
- **Scopes:** `https://www.googleapis.com/auth/drive`

### 2. Google Sheets OAuth2
- **Credential-Typ in n8n:** `Google Sheets OAuth2`
- **Scopes:** `https://www.googleapis.com/auth/spreadsheets`

### 3. Anthropic API Key
- **Credential-Typ in n8n:** `Header Auth`
- **Vorlage:** `credentials/anthropic-api-vorlage.json`
- **Header Name:** `x-api-key`
- **Wert:** Anthropic API Key (`sk-ant-...`)
- **API Konsole:** https://console.anthropic.com/settings/keys

---

## Konfiguration vor dem Import

| Platzhalter | Node | Was eintragen |
|---|---|---|
| `ERSETZEN_GOOGLE_DRIVE_ORDNER_ID` | Google Drive Trigger | ID des überwachten Ordners |
| `ERSETZEN_GOOGLE_SHEET_ID` | Google Sheets speichern & Fehler protokollieren | ID des Google Sheets |
| `ERSETZEN_WEBFORMULAR_URL` | Webformular ausfüllen | URL des Schuldnerberatungsformulars |
| `ERSETZEN_CREDENTIAL_ID` | Alle Google Nodes | Credential-ID nach dem Anlegen in n8n |
| `ERSETZEN_ANTHROPIC_CREDENTIAL_ID` | Claude Dokument analysieren | Anthropic Credential-ID |

---

## Google Sheets Struktur

### Blatt 1: „Schuldnerberatung"
Spalten: Verarbeitet am, Quelldatei, Status, Schuldner Vorname, Schuldner Nachname, Schuldner Geburtsdatum, Schuldner Straße, Schuldner PLZ, Schuldner Ort, Absender Name, Absender Typ, Urspr. Gläubiger, Art der Schuld, Ursprungsvertrag, Aktenzeichen, Mahndatum, Mahnstufe, Zahlungsfrist, Hauptforderung (€), Zinsen (€), Mahngebühren (€), Inkassokosten (€), Gesamtforderung (€), IBAN Gläubiger, Bank Gläubiger, Rechtl. Schritte angedroht, Art rechtl. Schritte, Bereits tituliert, Ratenzahlung angeboten, Ratenzahlung Betrag (€), Vergleich angeboten, Besonderheiten

### Blatt 2: „Fehlerprotokoll"
Spalten: Datum, Datei, Fehler, Status

---

## Datenschutz-Hinweise (DSGVO)

- Schuldenbriefe enthalten **besonders sensible personenbezogene Daten**
- Anthropic DPA (Datenverarbeitungsvertrag) prüfen und abschließen
- Zugriff auf Google Drive Ordner und Sheets auf autorisierte Personen beschränken
- Verarbeitungsverzeichnis gemäß Art. 30 DSGVO führen
- Aufbewahrungsfristen für die Daten festlegen und einhalten

---

## Import & Aktivierung

```bash
# Workflow importieren
n8n import:workflow --input=workflows/schuldnerberatung/dokument-analyse-formular-ausfuellen.json
```

Nach dem Import in n8n:
1. Credentials anlegen (Google Drive, Google Sheets, Anthropic)
2. Platzhalter in den Nodes ersetzen
3. Google Sheets Dokument mit beiden Blättern erstellen
4. In **Staging-Umgebung** testen
5. Workflow aktivieren
