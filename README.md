# VocabularyCheck

Vokabel-Quiz für die Familie — läuft komplett im Browser (PC, Handy, Tablet). Kein Login, kein Server, Fortschritte bleiben lokal und werden optional per GitHub zwischen allen Geräten der Familie synchronisiert.

## Was die App kann

- **Vokabelsets**: Spanisch 1. Klasse, Latein 1. Klasse, Latein 3. Klasse (vorinstalliert). Eigene Sets per CSV-Import möglich.
- **Lern-Module** (neu): Themen-Module mit drei Modi — **Karteikarten** zum Einprägen, **Quiz** mit Multiple-Choice + Erklärung (falsche Fragen kommen wieder) und **Textanalyse** (z.B. Erzählperspektive bestimmen). Vorinstalliert: „Epik – Deutsch 3. Klasse".
- **Profile**: Jedes Familienmitglied hat sein eigenes Profil mit eigenem Fortschritt.
- **Richtungs-getrennter Fortschritt**: De → Fremdsprache und Fremdsprache → De werden getrennt gezählt.
- **Pro Kapitel / Modul**: Kapitel einzeln auswählbar; Fortschrittsanzeige pro Kapitel.
- **Streaks + Leaderboard**: Meilenstein-Animationen, Bestenliste über alle Profile (Streaks + erledigte Wörter).
- **Hilfsinfos bei richtiger Antwort**: Spezialformen, verwandte Wörter, Fremdwörter — werden nach „richtig" eingeblendet (nicht Teil der Quiz-Prüfung).
- **GitHub-Sync**: Spielstand (Vokabeln + Lern-Module) synct automatisch zwischen allen Geräten der Familie.
- **Auto-Update**: Banner informiert, wenn eine neue Version im Repo liegt.
- **Dark Mode**, Offline-fähig (läuft auch ohne Internet — nur ohne Sync/Update-Check).

## Benutzung

### Variante A — GitHub Pages (empfohlen)

Direkt im Browser öffnen: https://juergenpod.github.io/VocabularyCheck/  
(Pages muss in den Repo-Settings aktiviert sein → Settings → Pages → Source: `main` / `/ (root)`)

### Variante B — Lokal

`index.html` herunterladen (inklusive des `vocab/`-Ordners) und per Doppelklick im Browser öffnen. Funktioniert über `file://`, nur:
- Auto-Update-Banner kann keinen direkten One-Click-Update anbieten (Link zur Raw-Datei statt Reload).
- Sonst identisch.

### Profil anlegen

Beim ersten Start erscheint die Profil-Auswahl. Neues Profil → Name eingeben → los.

## GitHub-Sync für die Familie einrichten

Damit Papa-PC, Mama-Handy und Kinder-Tablet denselben Fortschritt sehen.

### Einmalig (vom Repo-Owner):

1. **Privates Save-Repo anlegen**: `JuergenPod/VocabularyCheck-Saves` (private)
   - Initialdatei `savestates.json` mit Inhalt: `{"lastUpdated":"","profiles":{}}`
2. **Fine-grained Personal Access Token erstellen**:
   - github.com → Settings → Developer Settings → Personal access tokens → Fine-grained
   - Repository access: **Only select repositories** → `VocabularyCheck-Saves`
   - Permissions: `Contents: Read and write`, `Metadata: Read`
   - Ablauf: z.B. 1 Jahr
3. **Token an die Familie weitergeben** (Passwort-Manager oder Signal, nicht per E-Mail)

### Pro Gerät:

1. App öffnen → Profil auswählen oder anlegen
2. Profil-Menü (Zahnrad) → ☁️ GitHub-Sync
3. Token einfügen → „Verbindung testen" → grünes ✓
4. Fertig. Synct automatisch nach jeder Änderung (Debounce 5 Sekunden).

### Token-Leak?

Token widerrufen → github.com → PAT-Liste → Revoke. Neuen Token erzeugen. Familie informieren.

## Eigene Vokabeln hinzufügen

Es gibt zwei Wege:

- **CSV-Import** im Profil-Menü: spontane Zusatzwörter, bleibt im Profil.
- **Festes Set erweitern**: siehe [docs/ENTWICKLUNG.md](docs/ENTWICKLUNG.md).

## Dateien im Repo

| Pfad | Zweck |
|---|---|
| `index.html` | Komplette App (HTML + CSS + JS in einer Datei) |
| `vocab/es-de.js` | Spanisch 1. Klasse |
| `vocab/la-de.js` | Latein 1. Klasse |
| `vocab/la3-de.js` | Latein 3. Klasse |
| `learn/epik-de.js` | Lern-Modul „Epik – Deutsch 3. Klasse" (Karteikarten + Quiz + Textanalyse) |
| `screenshots/` | Quellmaterial für Vokabel-/Lern-Extraktion (nicht im Repo) |
| `docs/ENTWICKLUNG.md` | Entwickler-Doku |

## Fehler / Wünsche

Issue im Repo anlegen oder direkt Juergen fragen.
