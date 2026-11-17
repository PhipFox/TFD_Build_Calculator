# TFD Build-Rechner – Projektnotizen für Claude

Fan-Tool für **The First Descendant (TFD)**: berechnet das beste Modul-Build für einen Descendant und zeigt, wo man Items und Materialien bekommt. Gestartet mit **Nell**. Inoffiziell, nicht mit NEXON verbunden.

## Arbeitsweise
- Mit dem Nutzer auf **Deutsch** sprechen.
- Die Website ist **zweisprachig (DE/EN)**: jeder neue Text braucht einen Eintrag in `I18N.de` **und** `I18N.en`.
- Hauptdatei: `index.html` – eine einzelne, eigenständige HTML-Datei (HTML + CSS + JS inline). Beim Wachsen des Projekts darf sie aufgeteilt werden (z. B. `data/`, `css/`, `js/`).
- Nach Änderungen die Seite im Browser prüfen (alle fünf Reiter, DE und EN).

## Regeln (wichtig)
- **Original-Icons nur per Link** von NEXONs Servern laden (Bild-URLs aus der NEXON Open API, Metadaten `https://open.api.nexon.com/static/tfd/meta/en/descendant.json` und `module.json`). Bilder **niemals** herunterladen und ins Projekt kopieren. Fehlt ein Bild → eigenes SVG-Symbol als Fallback (ist eingebaut).
- **Kein API-Key im Code oder Repo.** Der Key wird nur im Browser des Nutzers gespeichert (`localStorage` "tfdKey").
- Keine Logos/Schriftzüge des Spiels nachbauen – nur den Stil.
- Vor einer Veröffentlichung: Hinweis „Inoffizielles Fan-Tool · Daten: NEXON Open API“ einbauen und die Nutzungsbedingungen der NEXON Open API beachten.

## Aufbau der Seite
Reiter: **Optimierer** · **Waffen** · **Nell & Skills** · **Wo bekomme ich was?** · **Daten & Annahmen** (inkl. Karte „Original-Icons (lokal)“).
Sprach-Schalter oben links (DE/EN, gespeichert in `localStorage` "tfdLang").
Design: Sci-Fi-Stil angelehnt an TFD (dunkles Blauschwarz, Cyan `--acc`, Bernstein `--acc2`, abgeschrägte Ecken, Schriften Rajdhani + Exo 2 von Google Fonts).

### Wichtige JS-Teile
- `I18N` – alle Texte, `t(key)` holt sie.
- `NELL_SKILLS`, `MATS`, `LOC`, `SRC_EXTRA` – Charakter-, Material- und Quellendaten.
- `MODS` – Modul-Datenbank: `n` Name, `s` Sockel, `c` Basiskapazität, `t` Tag (gleiche Tags schließen sich aus), `st` Werte auf Level 0, `sc` Werte die mit Level skalieren, `cond/condSt` bedingte Effekte, `src` Quelle, `surv` Überlebensmodul.
- `CURVE` – Level-Kurve L0–L10 (**geschätzt**, aus „Focus on Non-Attribute“ abgeleitet).
- `GUIDE` – Vergleichs-Builds (AlcastHQ) für Bossing und Mobbing.
- `optimize()` – probiert alle erlaubten Kombinationen für die 10 freien Slots (Slot 1 = Shattered Heart, Mid-Air Maneuvering fest).
- `evaluate()` – Rechenformel (siehe unten).
- `OFFICIAL`, `applyMeta()`, `loadOnline()` – Original-Icons laden (automatisch / mit API-Key / JSON-Dateien manuell).
- Waffen-Reiter: `W_BASE` (eingebaute Fallback-Liste, englische Namen, nur Zahlen + Bild-Links), `slimWeapon()` kürzt NEXONs `weapon.json`, `loadWeapons()`/`ensureWeapons()` laden die aktuellen Daten in der gewählten Sprache (`meta/de/` bzw. `meta/en/`, Cache in `localStorage` "tfdWeap_de"/"tfdWeap_en", 7 Tage). Auswahl (max. 3) in `WSEL`, gespeichert in `localStorage` "tfdWeapons". Seltenheitsfarben über CSS-Klassen `.t1`–`.t4` (Normal blau, Selten violett, Ultimate gold, Transzendent rot). Tooltip: `wTip()`/`showTip()`.
- Die Waffenauswahl wird beim Build nur angezeigt und fließt **nicht** in den Score ein.

## Rechenmodell (vereinfacht)
- Schadensfaktor = (1 + Skill Power) × (1 + Skill-Power-Modifier) × (1 + Critrate × (Critschaden − 1))
- Score = Schadensfaktor × (1/(1 − Cooldown))^GewichtCD × (1 + Reichweite)^GewichtReichweite
- Bossing = Skill 1/4 (Non-Attribute + **Tech**), Mobbing = Skill 2/3 (Non-Attribute + **Fusion**).
- Kapazität = Basis + Level; passender Sockel halbiert (aufgerundet). Max-Kapazität einstellbar (Standard 85).
- Annahmen (im Reiter „Daten“ änderbar): Basis-Crit 5 % / 150 %, Kapazität Shattered Heart 8, Reaktor-Substats −7,4 % CD / +33 % Critschaden, falscher Reaktor-Typ ×0,6.

## Datenquellen
- Level-0-Modulwerte: game8.co Modulliste
- Builds, Reaktor, Komponenten, Waffen: AlcastHQ Nell Build Guide
- Nell freischalten & Materialien: Sportskeeda Farming Guide
- Spielstand: Patch 1.4.8 (November 2026)

## Offene Punkte / Roadmap
1. **Echte Modulwerte pro Level** aus `module.json` (NEXON) auslesen statt `CURVE`-Schätzung – höchste Priorität.
2. Sockel-Planung pro Slot (12 Slots wie im Spiel).
3. „Meine Module“: vorhandene Module + Level abhaken, nur baubare Builds vorschlagen, Fehlendes mit Fundort zeigen.
4. Builds speichern und als Code teilen.
5. Waffen in die Rechnung einbeziehen + Waffenmodule (Auswahl-Reiter ist da), danach weitere Descendants.
6. Fundorte für alle Module ergänzen (viele stehen noch auf „Zugangsinfo im Spiel“).

## Veröffentlichung (geplant)
GitHub-Repository (öffentlich) + **GitHub Pages**, damit Updates automatisch bei allen ankommen. Die Hauptdatei heißt dafür bereits `index.html`. Noch nicht eingerichtet – der Nutzer muss GitHub-Konto/Repo anlegen und GitHub mit Claude verbinden.

## Weitere Version
Es gibt zusätzlich eine Online-Version als Claude-Artifact (ohne Original-Icons, noch altes Design). Hauptversion ist die lokale Datei in diesem Ordner.
