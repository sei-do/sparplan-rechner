# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Regeln

- **Alle Texte auf Deutsch**: Oberfläche, Kommentare, Commit-Nachrichten und Antworten an den Nutzer.
- **Beträge immer im deutschen Format**: `1.234,56 €` (Punkt als Tausendertrenner, Komma als Dezimaltrenner, Leerzeichen vor `€`). Formatierung über `Intl.NumberFormat("de-DE", { style: "currency", currency: "EUR" })`.
- **Nach jeder fertigen Änderung einen Git-Commit machen** – mit einer kurzen deutschen Commit-Nachricht, die beschreibt, was sich für den Nutzer ändert.

## Projekt

Sparplan-Rechner als **eine einzige Datei `index.html`** (HTML + CSS + JavaScript inline). Kein Build, keine Abhängigkeiten, keine Tests. Die Seite muss **ohne Internet** funktionieren: keine CDNs, externen Schriften oder Bibliotheken einbinden – das Diagramm ist handgeschriebenes SVG.

Starten: `index.html` im Browser öffnen. Für Tests im eingebauten Browser einen lokalen Static-Server nutzen (`.claude/launch.json` ist lokal und per `.git/info/exclude` nicht versioniert).

## Aufbau von `index.html`

- **Farben/Theme**: CSS-Variablen in `:root` mit Dark-Mode-Variante über `prefers-color-scheme`. Plan A nutzt `--accent` (Blau), Plan B `--b` (Orange). Das SVG-Diagramm liest die Variablen per `getComputedStyle` und wird bei Theme-Wechsel neu gezeichnet.
- **Eingaben**: Werden aus der Konfiguration `FIELDS` (Startkapital, Sparrate, Laufzeit, Rendite) für jeden Plan in `PLANS` (`a`, `b`) per `renderPlanInputs()` erzeugt; IDs folgen dem Muster `a-rate`, `b-years` usw. Ein neues Eingabefeld wird also in `FIELDS` ergänzt, nicht im HTML.
- **Berechnung**: `simulate(start, rate, years, pct)` liefert eine Zeile pro Jahr (`{year, total, paid}`). Annahmen: Startkapital ab Tag 1 angelegt, Sparrate am Monatsanfang, monatliche Verzinsung mit `(1 + p)^(1/12) − 1` (effektive Jahresrendite). Ohne Steuern, Kosten, Inflation – steht so auch im Hinweistext unter dem Diagramm.
- **Datenfluss**: Jede Eingabe ruft `calc()` → `readPlan()` je aktivem Plan → globales Array `series` (`[{id, rows}]`, ein oder zwei Einträge) → `renderResults()`, `renderLegend()`, `draw()`, `renderTable()`. Alle Render-Funktionen unterscheiden Einzel- und Vergleichsmodus über `isCompare()` (`series.length > 1`).
- **Vergleichsmodus**: Schalter `#compare` → `toggleCompare()` blendet Plan B ein (beim ersten Mal mit Werten von Plan A vorbelegt) und setzt die Klasse `compare` am `<body>`, über die das Layout umschaltet. Ohne Schalter muss der Rechner exakt wie im Einzelmodus aussehen.
- Pläne dürfen unterschiedliche Laufzeiten haben: X-Achse und Tabelle laufen bis `maxYears()`, fehlende Jahre werden als „–“ angezeigt.

## Prüfen

Kontrollwert für die Berechnung: 0 € Startkapital, 200 €/Monat, 20 Jahre, 6 % → Endbetrag rund **91.129 €**, eingezahlt 48.000,00 €. Änderungen im Browser in heller und dunkler Darstellung sowie in Handy-Breite (375 px, kein seitlicher Überlauf) prüfen.
