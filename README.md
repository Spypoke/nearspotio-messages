# Message Center – Dokumentation

## Zweck

Die Datei `messages.json` steuert das **Message Center** der nearspotio-App. Beim App-Start lädt die App diese JSON von GitHub Raw und zeigt ggf. **einen** AlertDialog mit einer wichtigen Nachricht (Info, Warnung oder Update-Hinweis). Der Dialog ist immer wegklickbar – es gibt kein Pflicht-Update.

## Wo liegt die Datei?

Die Datei liegt im eigenen, unabhängigen Repository **`Spypoke/nearspotio-messages`** (Branch `main`), direkt als `messages.json` im Root des Repos:

```
https://raw.githubusercontent.com/Spypoke/nearspotio-messages/refs/heads/main/messages.json
```

Das ist die URL, die die App abruft.

**Wichtig:** Das Repository muss **PUBLIC** sein. GitHub Raw (`raw.githubusercontent.com`) liefert Dateien aus privaten Repos ohne Token nicht aus – die App würde dann immer HTTP 404 erhalten und keine Nachrichten laden.

## Struktur

```json
{
  "android": [ ...Nachrichten... ],
  "ios":     [ ...Nachrichten... ]
}
```

Android und iOS haben **getrennte Listen**. Die `id`-Werte müssen plattformübergreifend eindeutig sein (z. B. Suffix `-android` / `-ios`). Build-Nummern sind pro Plattform zu pflegen.

## Felder pro Nachricht

| Feld        | Typ             | Pflicht | Bedeutung |
|-------------|-----------------|---------|-----------|
| `id`        | string          | ja      | **Eindeutige** ID. Wird beim ersten Anzeigen gespeichert – bei gleicher ID wird die Nachricht nie erneut gezeigt! |
| `show`      | boolean         | ja      | Master-Schalter. `false` = Nachricht deaktiviert (Vorlage, ohne Anzeige). |
| `type`      | string          | nein    | `"info"` (Standard), `"update"` oder `"warning"` |
| `minBuild`  | integer         | nein    | Nachricht erst ab diesem Build anzeigen (inklusiv). Fehlt → keine Untergrenze. |
| `maxBuild`  | integer         | nein    | Nachricht nur anzeigen wenn installierter Build **kleiner** als dieser Wert ist. Ideal für Update-Hinweise: verschwindet automatisch nach dem Update. |
| `showButton`| boolean         | nein    | `true` = zeige zusätzlichen Link-Button (benötigt `url`). Default: `false`. |
| `url`       | string          | nein    | Store-Link oder andere URL, die beim Button-Klick geöffnet wird. |
| `title`     | Sprach-Map      | nein    | Dialog-Titel pro Sprachcode. Fehlt → kein Titel. |
| `message`   | Sprach-Map      | nein    | Dialog-Text pro Sprachcode. |
| `button`    | Sprach-Map      | nein    | Button-Beschriftung pro Sprachcode. Fehlt oder leer → Fallback "Update". |

## Sprach-Maps

Jede Sprach-Map hat App-Sprachcodes als Keys:

```
en, de, fr, it, es, nl, pl, pt, da, cs, sv, ru, hu, sl, hr, no, fi, ro,
sk, bg, uk, tr, el, ptBr, et, lv, lt, sr, bs, ca, ja, zh, ko, he
```

Was in der Map fehlt, fällt auf `"en"` zurück. Ist auch `"en"` nicht vorhanden, bleibt das Feld leer.

## Logik im Detail

1. Die App lädt die JSON beim Start (Timeout: 5 Sekunden; Fehler werden still ignoriert).
2. Pro Plattform wird die Liste von oben nach unten durchgegangen.
3. **Erste** Nachricht, auf die alle Bedingungen zutreffen, wird angezeigt (nur eine pro Start).
4. Beim Anzeigen wird die `id` sofort lokal gespeichert → die Nachricht erscheint in Zukunft nicht mehr.
5. GitHub Raw cached ca. 5 Minuten; ein Cache-Buster ist eingebaut, dennoch kann es kurz dauern, bis Änderungen überall ankommen.

## Hinweis: Android Store-URL

```
https://play.google.com/store/apps/details?id=app.nearspotio
```

## Typische Anwendungsfälle

**Nur Info/Warnung (ohne Update-Button):**
```json
{
  "id": "2026-06-server-info-android",
  "show": true,
  "type": "warning",
  "showButton": false,
  "title": { "en": "Map issues", "de": "Kartenprobleme" },
  "message": { "en": "...", "de": "..." }
}
```

**Update-Hinweis (verschwindet nach Build 150):**
```json
{
  "id": "2026-06-update-hint-android",
  "show": true,
  "type": "update",
  "maxBuild": 150,
  "showButton": true,
  "url": "https://play.google.com/store/apps/details?id=app.nearspotio",
  "title": { "en": "Update recommended", "de": "Update empfohlen" },
  "message": { "en": "...", "de": "..." },
  "button": { "en": "Update now", "de": "Jetzt aktualisieren" }
}
```

**Nachricht deaktivieren (bleibt als Vorlage erhalten):**

`"show": false` setzen – die Nachricht wird nie angezeigt, bis wieder auf `true` gestellt.

## Wichtig

- `id` **muss** bei jeder neuen Nachricht geändert werden. Wird dieselbe `id` wiederverwendet, wird die Nachricht bei Nutzern, die sie schon gesehen haben, nie wieder angezeigt.
- Android und iOS **unbedingt** getrennt halten – eigene IDs, eigene Build-Nummern, eigene Store-URLs.
