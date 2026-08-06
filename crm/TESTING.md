# Manuelle QA-Checkliste

Die App kommt bewusst ohne Test-Framework aus (Single-File, null Abhängigkeiten).
Stattdessen diese Checkliste — einmal vor jedem Release komplett durchgehen.

Start: `python3 -m http.server 8125 -d crm`, dann <http://localhost:8125>.
Immer denselben Host+Port verwenden — IndexedDB ist origin-gebunden.

---

## 1. Nutzerwege (Reihenfolge einhalten)

| # | Schritt | Erwartet |
|---|---|---|
| 1 | Leere DB öffnen | Willkommens-Screen mit „Beispieldaten laden“ / „Ersten Kunden anlegen“ |
| 2 | Kunde anlegen | Erscheint sofort in `#/clients`, Status-Badge stimmt |
| 3 | Kontaktperson anlegen, als Primär markieren | Genau **ein** Kontakt trägt „Primär“ |
| 4 | „+ Interaktion“ → Kurzfassung + Häkchen „Wiedervorlage setzen“ + Chip „+1 Woche“ → **ein** Submit | Interaktion in der Timeline **und** Follow-up in „Offene Wiedervorlagen“, Toast mit „Rückgängig“ |
| 5 | Toast „Rückgängig“ klicken | Beide Datensätze wieder weg |
| 6 | Schritt 4 wiederholen, Follow-up am Dashboard abhaken | Verschwindet aus „Offen“, taucht unter „Erledigt“ auf |
| 7 | Quick-Link mit GitHub-URL anlegen | Repo-Icon 🐙 wird automatisch erkannt |
| 8 | Quick-Link mit `file:///…` anlegen | Kein Link, sondern Button „Pfad kopieren“ |
| 9 | Quick-Link mit `javascript:alert(1)` | Wird beim Speichern mit Fehlermeldung abgelehnt |
| 10 | Projekt anlegen | Erscheint in `#/projects` unter seinem Kunden und im Kundendetail |
| 11 | 3 Aufgaben per Inline-Zeile anlegen (nur Enter) | Fokus bleibt im Feld, alle 3 landen in der Liste |
| 12 | Aufgabe per ↑/↓ verschieben | Reihenfolge ändert sich |
| 13 | Timer starten | Chip in der Topbar läuft sekündlich |
| 14 | **Reload** bei laufendem Timer | Chip läuft nahtlos weiter (Recovery aus `meta.activeTimer`) |
| 15 | Timer stoppen | Genau **ein** Zeiteintrag mit Quelle „Timer“ |
| 16 | Manuelle Zeit `1,5h`, `1:30`, `90m` erfassen | Jeweils 90 Minuten, Live-Vorschau unter dem Feld |
| 17 | Manuelle Zeit `abc` erfassen | Speichern blockiert, Fehlermeldung am Feld |
| 18 | CSV exportieren | Datei öffnet in Excel (de) spaltenrichtig, `1,50` ist eine Zahl, Umlaute stimmen |
| 19 | Backup exportieren | `crm-backup-JJJJ-MM-TT.json`, „Letztes Backup“ aktualisiert sich |
| 20 | Daten → Alle Daten löschen (Wort `LÖSCHEN` tippen) | Leere, funktionsfähige App |
| 21 | Backup importieren → **Ersetzen** | Exakt der Stand des Backups |
| 22 | Etwas ändern, Backup erneut importieren → **Zusammenführen** | Keine Duplikate, lokale Extra-Datensätze bleiben |
| 23 | „Rückgängig“ nach dem Import | Vor-Import-Zustand vollständig zurück |

## 2. Persistenz-Matrix

Nach **jedem** Schritt aus Abschnitt 1 einmal neu laden — der Zustand muss unverändert sein.
Zusätzlich:

- Änderung + Tab sofort schließen (< 400 ms) → nach erneutem Öffnen vorhanden (Flush bei `pagehide`).
- Zwei Tabs offen, in Tab A etwas ändern → Tab B lädt ohne manuellen Reload nach.
- Tab B mit offenem Formular, Tab A ändert → Formular bleibt erhalten, Nachladen passiert erst beim Schließen.

## 3. Grenzfälle

- [ ] Leere Datenbank: alle sechs Screens ohne Fehler, jeder mit Empty State
- [ ] Kunde ohne jede Interaktion → erscheint unter „Stille Kunden“
- [ ] Follow-up ohne Kunde und ohne Projekt → wird korrekt dargestellt
- [ ] Projekt ohne Zeiteinträge → „Noch keine Zeit erfasst“
- [ ] Follow-up mit heutigem Datum → gilt als *fällig*, nicht als *überfällig*
- [ ] Archivierte/pausierte Kunden erscheinen **nicht** unter „Stille Kunden“
- [ ] Backup mit `schemaVersion: 999` → Ablehnung mit Meldung, Daten unverändert
- [ ] Absichtlich beschädigtes JSON → Meldung, Daten unverändert
- [ ] Kundenname `<script>alert(1)</script>` → wird als Text angezeigt, nichts wird ausgeführt
- [ ] Notiz mit `;`, `"` und Zeilenumbruch → CSV bleibt spaltenrichtig
- [ ] Umlaute/ß in allen Feldern → Anzeige, Export und Re-Import korrekt

## 4. Darstellung & Bedienung

- [ ] 320 / 390 / 768 / 1440 px: kein horizontales Scrollen auf keiner Seite
- [ ] Mobil: Navigation unten, Modale als Bottom-Sheet, Tastatur verdeckt keine Felder
- [ ] Hell **und** Dunkel: alle Screens lesbar, Theme-Wechsel ohne Flackern beim Laden
- [ ] Nur Tastatur: Kunde anlegen · Interaktion + Follow-up · Follow-up abhaken · Timer start/stop
- [ ] `Esc` schließt jeden Dialog, `⌘/Strg + Enter` schickt jedes Formular ab
- [ ] `n` in einem Notizfeld öffnet **keinen** Dialog
- [ ] `?` zeigt die Shortcut-Übersicht

## 5. Browser-Matrix

| Browser | Durchlauf 1–4 | Bemerkung |
|---|---|---|
| Safari (macOS) | ☐ | |
| Chrome (macOS) | ☐ | |
| Firefox (macOS) | ☐ | |
| Safari (iOS) | ☐ | Bottom-Sheet, Safe-Area, kein Zoom beim Fokus |

## 6. Technik-Check

- [ ] Netzwerk-Tab: beim Start **genau eine** Anfrage (das HTML) — keine CDNs, Fonts oder Bilder
- [ ] Konsole beim Durchlauf frei von Fehlern
- [ ] 20 schnelle Änderungen erzeugen genau **einen** Schreibvorgang (Debounce)
- [ ] Mit dem 10-fachen Beispieldatensatz (~500 Interaktionen, ~1.000 Zeiteinträge) bleibt jeder Screenwechsel unter 100 ms
