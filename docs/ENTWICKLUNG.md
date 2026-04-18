# Entwickler-Doku

Kurze Orientierung für Änderungen am Code. Die App ist bewusst eine **einzelne HTML-Datei** plus ausgelagerte Vokabel-JS-Dateien — kein Build-Schritt, keine Dependencies.

## Architektur auf einen Blick

```
index.html                  ← HTML + CSS + JS in einer Datei (~2.300 Zeilen)
vocab/
  es-de.js                  ← registriert sich in window.VOCAB_REGISTRY
  la-de.js
  la3-de.js
learn/
  epik-de.js                ← registriert sich in window.LEARN_REGISTRY
```

**Warum `<script src>` statt `fetch` für die Vokabeln?**  
`fetch()` ist bei `file://`-Origins in Chrome blockiert. `<script src>` funktioniert überall: GitHub Pages, `file://`, lokaler Server.

## Vokabel-Set hinzufügen

1. Neue Datei `vocab/<id>.js` anlegen (z.B. `vocab/fr-de.js`):

   ```js
   (window.VOCAB_REGISTRY=window.VOCAB_REGISTRY||[]).push({
     id:"fr-de",
     name:"Französisch 1. Klasse",
     langA:"Französisch",
     langB:"Deutsch",
     flagA:"🇫🇷",
     flagB:"🇩🇪",
     stripArticlesOnB:true,     // "das Haus" → "Haus" beim Matching
     stripArticlesOnA:false,    // "le jardin" bleibt wie es ist
     vocab:[
       {id:1, a:"bonjour",     b:"hallo",     u:"1"},
       {id:2, a:"la maison",   b:"das Haus",  u:"1", forms:"les maisons"},
       // ...
     ]
   });
   ```

2. In `index.html` nach den anderen vocab-Scripts einbinden:

   ```html
   <script src="vocab/fr-de.js"></script>
   ```

3. Fertig — die App findet das Set automatisch über `window.VOCAB_REGISTRY`.

### Feldliste pro Vokabel-Eintrag

| Feld | Pflicht | Zweck |
|---|---|---|
| `id` | ja | **Niemals** nachträglich ändern — User-Fortschritt hängt daran |
| `a` | ja | Wort in Sprache A (Prompt) |
| `b` | ja | Wort in Sprache B (Antwort) |
| `u` | ja | Einheit/Kapitel/Modul — z.B. `"1"`, `"2"`, `"X"` |
| `forms` | optional | Spezialformen (z.B. Stammformen). Nur Anzeige. |
| `rel` | optional | Verwandte Wörter (orange Tag). Nur Anzeige. |
| `loan` | optional | Fremd-/Lehnwörter (grüner Tag). Nur Anzeige. |

`forms` / `rel` / `loan` werden **nicht** beim Quiz abgeglichen — sie erscheinen nur nach einer richtigen Antwort als Lernhilfe.

### Set-Metadaten

| Feld | Zweck |
|---|---|
| `stripArticlesOnA/B` | Entfernt `der/die/das/el/la/los/las/le/la/les` vor dem Match |

## Lern-Modul hinzufügen

Lern-Module sind themenbasiert (nicht vokabelbasiert) und haben drei Modi: Karteikarten, Quiz, Textanalyse. Jeder Modus ist optional — leeres Array = Tab wird versteckt.

1. Neue Datei `learn/<id>.js` anlegen:

   ```js
   (window.LEARN_REGISTRY=window.LEARN_REGISTRY||[]).push({
     id:"epik-de",
     name:"Epik – Deutsch 3. Klasse",
     subject:"Deutsch",
     icon:"📖",
     flashcards:[
       {id:1, term:"Epik", def:"Erzählende Literatur in Prosa oder Versen.", ex:"Roman, Novelle, Kurzgeschichte"},
       // ...
     ],
     quiz:[
       {id:1, q:"Was ist ein Ich-Erzähler?", opts:["A","B","C","D"], correct:0, expl:"Erzählt aus der Ich-Perspektive, ist selbst Figur."},
       // ...
     ],
     texts:[
       {id:1, author:"E.T.A. Hoffmann", work:"Der Sandmann", year:1816,
        kind:"Erzählperspektive",
        choices:["Ich-Erzähler","Auktorial","Personal","Neutral"],
        correct:0,
        text:"„Gewiß werdet Ihr alle voll Unruhe sein ...",
        expl:"Die Ich-Form + direkte Ansprache des Lesers kennzeichnen den Ich-Erzähler."},
       // ...
     ]
   });
   ```

2. In `index.html` nach den anderen learn-Scripts einbinden:

   ```html
   <script src="learn/epik-de.js"></script>
   ```

3. Fertig — die App listet das Modul automatisch im Profil-Menü.

### Feldlisten

**flashcards**: `id` (Pflicht, stabil), `term`, `def`, `ex` (optional).

**quiz**: `id`, `q` (Frage), `opts` (Array mit 2+ Antworten), `correct` (Index in `opts`), `expl` (Erklärung für Richtig + Falsch). Optionen werden zur Laufzeit gemischt; `correct` wird intern umgerechnet.

**texts**: `id`, `author`, `work`, `year`, `kind` (was zu bestimmen ist, z.B. „Erzählperspektive"), `choices`, `correct`, `text` (der Textausschnitt selbst), `expl`.

Falsche Antworten in Quiz/Textanalyse werden zurück in die Queue gestellt (idx+3/+4) und kommen später nochmal.

### Fortschritts-Keys (pro Profil)

- `<profil>:learn:<moduleId>:fc:<itemId>` — Karteikarte gelernt (`"1"`)
- `<profil>:learn:<moduleId>:q:<itemId>` — Quiz-Frage korrekt beantwortet
- `<profil>:learn:<moduleId>:t:<itemId>` — Textanalyse korrekt bestimmt

Werden automatisch vom GitHub-Sync mit erfasst (scannt alle `<profil>:*`).

## Wichtige Code-Orte in `index.html`

| Zeile (ca.) | Was |
|---|---|
| `~495` | `<script src>` Tags für Vokabelsets + Lern-Module |
| `~540` | `BUILTIN_SETS`, `getActiveSet()`, `getAllUnits()` |
| `~820` | `renderProfileMenu()` — listet Vokabelsets + Lern-Module |
| `~840` | `save()` — localStorage-Write, triggert `schedulePush()` |
| `~1033` | `renderExtraInfo()` — zeigt forms/rel/loan nach richtiger Antwort |
| `~1580` | `checkForUpdate()` — Auto-Update-Banner via GitHub-API |
| `~1700` | Leaderboard (`computeLeaderboardData`, `renderLeaderboard`) |
| `~1800` | GitHub-Sync (`pullFromGitHub`, `pushToGitHub`, `schedulePush`) |
| `~1907` | Lern-Modul JS (`showLearn`, Karteikarten/Quiz/Textanalyse-Renderer) |
| `~2150` | Init-Block |

## localStorage-Keys

- `sq_profiles` — JSON-Array der Profilnamen
- `sq_active_profile` — Aktives Profil
- `<profil>:<setId>:<dir>:d` — Done-IDs (Array), `dir` ∈ `a-b` / `b-a`
- `<profil>:<setId>:<dir>:h` — Streak-History (Array von `{date, max}`)
- `<profil>:learn:<modId>:{fc|q|t}:<itemId>` — Lern-Fortschritt pro Modul/Modus/Item
- `<profil>:mtime` — ISO-Timestamp letzter Änderung (für Sync-Merge)
- `sq_gh_token` — GitHub PAT (pro Gerät)
- `sq_gh_device` — frei wählbarer Gerätename
- `sq_gh_last_sync` — Letzter erfolgreicher Sync
- `sq_gh_file_sha` — Letzter bekannter SHA von `savestates.json` (Konflikt-Check)
- `sq_seen_sha` — Letzter als „gesehen" markierter Repo-SHA (Update-Banner)
- `sq_theme` — `light` / `dark`

## Sync-Protokoll

- **Pull beim Start**: `GET /repos/.../contents/savestates.json` → Merge per Profil, neuerer `lastModified` gewinnt.
- **Push nach `save()`**: Debounced 5s → `PUT` mit letztem bekanntem File-SHA. Bei `409 Conflict` re-pull + ein Retry.
- **Pusht immer alle Profile**, damit keine Profile verloren gehen, wenn ein Gerät zwischen Profilen wechselt.

Datenformat in `savestates.json`:

```json
{
  "lastUpdated": "2026-04-18T14:30:00.000Z",
  "profiles": {
    "Jonas": {
      "lastModified": "2026-04-18T14:29:55.000Z",
      "device": "Papa-PC",
      "state": { "Jonas:es-de:a-b:d": "[1,2,3]", "...": "..." }
    }
  }
}
```

## Auto-Update-Banner

- Beim Start: `GET /repos/JuergenPod/VocabularyCheck/commits/main` → 7-Zeichen-SHA
- Vergleich mit `sq_seen_sha` in localStorage
- Bei Abweichung: oranger Banner mit „Aktualisieren"-Button (Reload mit `?v=<sha>` Cache-Bust)
- **Kein** `APP_BUILD_SHA` im File selbst — die App merkt sich, was der User zuletzt „gesehen" hat.

## Screenshots zu Vokabeln

Screenshots sind im `screenshots/`-Ordner (per `.gitignore` ausgeschlossen). Typischer Workflow:

1. Screenshots pro Klasse/Jahr in `screenshots/<Klasse>/`
2. Wenn Bilder zu hoch sind (>760px), mit Python/PIL splitten:
   ```python
   from PIL import Image
   img = Image.open("bild.png"); w, h = img.size
   mid = h // 2 + 20
   img.crop((0, 0, w, mid)).save("bild_a.png")
   img.crop((0, mid - 40, w, h)).save("bild_b.png")
   ```
3. Per Vision-Modell (oder manuell) transkribieren → `vocab/<id>.js`
4. Bei Deduplikation über Module: **erste Nennung gewinnt**, damit jede ID nur einmal existiert.

## Keine Build-Pipeline

Änderung → Browser neu laden (bei `file://`) bzw. Git push → Banner triggert Reload (bei Pages). Punkt.
