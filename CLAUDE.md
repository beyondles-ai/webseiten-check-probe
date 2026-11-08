# Regeln für diesen Bau

Du baust live vor Publikum. Die Leute sehen die fertige Seite gleich auf ihrem Handy und prüfen damit ihre eigene Webseite. Das Wichtigste: Es wird fertig, es läuft, und es ist online.

## Arbeitsweise
- Keine Rückfragen. Wenn etwas unklar ist, triff eine sinnvolle Annahme und bau weiter.
- Schnell fertig werden. Fertig schlägt perfekt. Nicht mehr bauen als beauftragt.
- Kein `npm install`, keine Pakete, kein Build-Schritt, keine Frameworks.
- Die ganze Anwendung ist eine einzige Datei: `docs/index.html` mit eingebettetem CSS und JavaScript. Sonst braucht die Seite nichts.
- Die Adresse des Helfers steht in `HELPER_URL.txt`. Lies sie und setz sie als einzige Konstante ganz oben ins Skript: `const HELPER_URL = "...";` (ohne Schrägstrich am Ende). Nirgends sonst eine Helfer-Adresse hineinschreiben.
- Du darfst den Helfer einmal kurz mit `curl` ausprobieren. Das Datenformat steht aber unten vollständig, du musst nicht suchen.
- Am Ende: `git add -A`, `git commit -m "..."` (englisch), `git push`. Ohne Push ist nichts online. Danach eine Zeile auf Deutsch: was gebaut wurde.
- Nicht anfassen: `.claude/`, `CLAUDE.md`, `AUFTRAG.md`, `ERWEITERUNG.md`, `HELPER_URL.txt`, `README.md`.

## Was die App macht
- Ein Eingabefeld für eine Webadresse und ein Knopf „Prüfen“. Eingaben ohne `https://` sind erlaubt, der Helfer ergänzt das.
- Eine Auswahl: „Nur Startseite“ oder „Startseite + 5 Unterseiten“ (Vorauswahl: mit Unterseiten).
- Aufruf: `GET <HELPER_URL>/fetch?url=<adresse, mit encodeURIComponent>&pages=<0 oder 5>`. Zeitlimit im Browser 45 Sekunden.
- Oben eine Gesamtnote 0 bis 100. Darunter vier Bereiche mit je einer Punktzahl 0 bis 100 und einem farbigen Abzeichen (ab 80 grün, ab 50 gelb, darunter rot):
  1. **KI-Auffindbarkeit (GEO)**. Der wichtigste Bereich. Steht als erster und größter da.
  2. **Vollständigkeit**
  3. **Aussehen & Aufbau**
  4. **Suchmaschinen (SEO)**
- Danach eine Karte für die Startseite und eine Karte pro Unterseite mit eigener kleiner Punktzahl und den wichtigsten Befunden der Seite.
- Jeder Befund ist genau ein deutscher Satz: was da ist oder was fehlt, und eine konkrete Empfehlung. Beispiel: „Die Seite hat keine Beschreibung. Schreib zwei Sätze, was ihr anbietet und für wen.“ Gute Befunde mit grünem Haken, Lücken mit gelbem oder rotem Punkt.
- Die Punkte werden im Browser aus den Daten berechnet, nach festen Regeln:
  - **Vollständigkeit**: Titel da, Beschreibung da, genau eine H1, Sprache (`lang`) gesetzt, `canonical` gesetzt, `viewport` gesetzt, Bilder mit Alternativtext, genug Text (ab 300 Wörtern).
  - **Aussehen & Aufbau**: Überschriften vorhanden, genau eine H1, H2 vorhanden wenn H3 vorhanden, Textlänge, Anteil der Bilder ohne Alternativtext, mobiler `viewport`.
  - **Suchmaschinen (SEO)**: Titel 30 bis 60 Zeichen, Beschreibung 70 bis 160 Zeichen, H1 da, `canonical`, `robotsMeta` enthält nicht `noindex`, Sitemap gefunden, `https`, Open Graph Titel und Bild.
  - **KI-Auffindbarkeit (GEO)**: robots.txt sperrt GPTBot, ClaudeBot, PerplexityBot und Google-Extended nicht (`blocked` ist schlecht, `allowed` und `unspecified` sind in Ordnung), `llms.txt` vorhanden, strukturierte Daten (`jsonLdTypes` nicht leer, Organization oder LocalBusiness ist ein Plus), klare Beschreibung, und die KI-Bewertung unten. Regelteil und KI-Teil zählen je zur Hälfte.
- **KI-Bewertung**: Nach dem Holen ruft die App `POST <HELPER_URL>/ask` auf, Inhaltstyp JSON, Körper `{"prompt": "...", "maxTokens": 600}`. Der Prompt ist deutsch und kurz: Er gibt Titel, Beschreibung, H1, H2 und `textExcerpt` mit und fragt, wie gut ein KI-Assistent wie ChatGPT aus diesem Text versteht, wer die Firma ist, was sie anbietet und für wen, und ob er sie empfehlen würde. Antwort nur als JSON: `{"score": 0-100, "summary": "ein Satz", "missing": ["...", "..."]}`. Die Antwort kommt als `{ok:true, text}`. Im Text kann das JSON in ```-Zeichen eingepackt sein, also das erste `{` bis zum letzten `}` herausschneiden und parsen. Zusammenfassung und fehlende Punkte im GEO-Bereich zeigen.
- Klappt `/ask` nicht (`ok:false`, Fehler oder ungültiges JSON): Nur den Regelteil von GEO werten und den Hinweis „KI-Bewertung gerade nicht verfügbar“ zeigen. Die App läuft trotzdem.
- Die Seite zeigt zuerst das Regel-Ergebnis und ergänzt die KI-Bewertung, sobald sie da ist (mit kleinem Ladehinweis im GEO-Bereich).

## Ausdrücklich NICHT
- KEINE rechtlichen Prüfungen. Kein Impressum, kein Datenschutz, keine Cookie-Banner, keine AGB, nichts Juristisches. Auch nicht als Hinweis, Fußnote oder Tipp. Diese Wörter kommen in der Seite nicht vor.
- Keine Speicherung auf einem Server, keine Statistik, keine Tracker. Die eingegebene Adresse geht nur an den Helfer. Merken nur im Browser (`localStorage`), wenn überhaupt.

## Zustände
- Laden: Drehkreis mit Text „Seite wird geholt …“. Bei Unterseiten dauert es bis zu 20 Sekunden, das ist normal.
- Fehler, Helfer nicht erreichbar: „Der Prüfdienst ist gerade nicht erreichbar. Bitte gleich noch einmal versuchen.“
- Fehler vom Helfer (`ok:false`), je `reason`:
  - `gesperrt`: „Diese Webseite lässt keine automatische Prüfung zu. Das machen manche Seiten mit Schutzdiensten. Probier eine andere Adresse.“
  - `zeitüberschreitung`: „Die Webseite hat zu lange gebraucht.“
  - `nicht erreichbar`: „Diese Webseite ist nicht erreichbar. Stimmt die Adresse?“
  - `ungültige adresse`: „Das ist keine gültige öffentliche Webadresse.“
  - HTTP 429: der Text aus `reason` anzeigen.
- Leer (vor der ersten Prüfung): ein Satz, was die App macht, und ein Beispiel zum Antippen, zum Beispiel `beyondles.ai`.
- Unterseiten mit `ok:false` bekommen eine graue Karte „Nicht geprüft“ mit dem Grund.

## Datenformat des Helfers
`GET /fetch` liefert immer HTTP 200 mit JSON. Erfolg:

```
{ ok: true, url, finalUrl, status, fetchedAt, https: bool,
  title, metaDescription (oder null), lang (oder null), viewport: bool, canonical (oder null), robotsMeta (oder null),
  h1: [], h2: [], h3: [],                      // Texte, je höchstens 30
  images: { total, withoutAlt }, links: { internal, external },
  wordCount, textExcerpt,                      // sichtbarer Text, höchstens 4000 Zeichen
  jsonLdTypes: [],                             // z. B. ["Organization", "WebSite"]
  openGraph: { title, description, image },    // jeweils Text oder null
  robotsTxt: { found: bool, aiCrawlers: { GPTBot, ClaudeBot, PerplexityBot, "Google-Extended", Bingbot, Googlebot } },  // je "allowed" | "blocked" | "unspecified"
  sitemap: { found: bool }, llmsTxt: { found: bool },
  subpages: [ { url, ok, reason?, status, title, metaDescription, h1: [], wordCount, images: { total, withoutAlt } } ],
  cached?: true }
```

Fehler: `{ ok: false, reason: "nicht erreichbar" | "gesperrt" | "zeitüberschreitung" | "ungültige adresse", url }`.

`POST /ask` liefert `{ ok: true, text }` oder `{ ok: false, reason }`.

Unterseiten haben nur die Felder oben. Für sie gelten nur die Regeln, für die Daten da sind (Titel, Beschreibung, H1, Textlänge, Bilder).

## Aussehen
- Mobil zuerst. Eine Spalte, Karten untereinander, große Schrift, Daumen-taugliche Knöpfe. Auf dem Rechner höchstens 720 Pixel breit, mittig.
- Hintergrund `#0A0E1A`. Karten `#161E33` mit Rand `#24304C`, runde Ecken. Text `#FFFFFF`, Nebentext `#C7CEDD`. Akzent `#9652E8` (Knopf, Fokus).
- Abzeichen: grün `#2EBD85`, gelb `#F5B942`, rot `#E5484D`.
- Systemschrift. Keine externen Schriften, Skripte, Bilder oder Tracker. Nur relative Pfade.
- Kopfzeile „Webseiten-Check“. Fußzeile „Gebaut live beim 4. HÄRTING KI-Tag, 19.11.2026“.
- Lange Listen (Überschriften, Befunde) höchstens 6 sichtbar, der Rest zum Aufklappen.

## Sprache
- Alles, was ein Mensch sieht, ist Deutsch. Kurze, klare Sätze, Du-Form, keine Fachwörter ohne Erklärung. „Meta-Beschreibung“ heißt auf der Seite „Beschreibung für Suchergebnisse“.
- Code, Kommentare und Commit-Texte sind Englisch.

## Veröffentlichung
- GitHub Pages zeigt auf `docs/` im Zweig `main`. Nach dem Push dauert es etwa eine Minute.
- Die Platzhalterseite in `docs/index.html` wird vom Bau komplett ersetzt.
