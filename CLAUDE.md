# TFD Database – Projektnotizen für Claude

Fan-Tool für **The First Descendant (TFD)**, Name der Website: **TFD Database** (früher „TFD Build-Rechner“): berechnet das beste Modul-Build für jeden Descendant und zeigt, wo man Items und Materialien bekommt. Gestartet mit **Nell**. Inoffiziell, nicht mit NEXON verbunden.

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
Reiter: **Optimierer** (Descendant-Auswahl per Drop-down mit Bild, Ziel-Skill, transzendentes Modul) · **Waffen** · **Descendants** (Übersicht aller Descendants mit Bild und Name; Klick öffnet die Einzelseite mit Portrait, Skills und transzendenten Modulen) · **Item Finder** (Suche über alle Items mit Fundstellen und schematischer Übersichtskarte; die Originalkarte des Spiels ist nicht möglich: NEXON liefert weder Kartenbilder noch Koordinaten). Unten rechts schwebt der **Material-Tracker** (einklappbar). Die Ansicht **Daten & Annahmen** (inkl. Karte „Original-Icons (lokal)“) öffnet sich über das Zahnrad oben rechts (`#gear`); ein zweiter Klick führt zurück.
Sprach-Schalter oben links (DE/EN, gespeichert in `localStorage` "tfdLang").
Design: Sci-Fi-Stil angelehnt an TFD. Inhalt zentriert (`--wrap` 1280 px, `--gut`), Fußzeile „Inoffizielles Fan-Tool · Daten und Bilder: NEXON Open API“. Schriften: **Saira Condensed** (Überschriften, Navigation, Zahlen; `--head`) + **Barlow** (Fließtext; `--body`) von Google Fonts. Hintergrund in den Farben des Charakters (Nell: Anthrazit-Schwarz, Taupe, gedecktes Rosé – CSS-Variablen `--bg`, `--bg-glow1/2`, `--bg-grid`, `--hdr`, `--bar`, `--panel`), Akzente Cyan `--acc`, Bernstein `--acc2`, abgeschrägte Ecken).

### Wichtige JS-Teile
- `I18N` – alle Texte, `t(key)` holt sie.
- `NELL_SKILLS`, `MATS`, `LOC`, `SRC_EXTRA` – Charakter-, Material- und Quellendaten.
- `MODS` – Modul-Datenbank: `n` Name, `s` Sockel, `c` Basiskapazität, `t` Tag (gleiche Tags schließen sich aus), `st` Werte auf Level 0, `sc` Werte die mit Level skalieren, `cond/condSt` bedingte Effekte, `src` Quelle, `surv` Überlebensmodul.
- `CURVE` – Level-Kurve L0–L10 (**geschätzt**, aus „Focus on Non-Attribute“ abgeleitet).
- `GUIDE` – Vergleichs-Builds (AlcastHQ) für Bossing und Mobbing.
- Optimierer (alle Descendants): `parseModStats()` liest die **englischen** Modultexte aus `module.json` (Zeilen „Name +x%“, Schlüssel in `O_LAB`) → echte Werte pro Level. Der Modul-Pool `O_POOL[lang]` entsteht in `slimDescendants()` (allgemeine Descendant-Module ohne Transzendent/Trigger; Klasse heißt im Deutschen „Nachfahre“) und liegt mit im Cache `tfdDesc2_*`. Werte immer aus `O_POOL.en`, Namen/Texte aus `O_POOL[LANG]`. Startet beim Öffnen der Seite (Optimierer ist der Start-Reiter).
- `OSEL` (gewählter Descendant, `localStorage` "tfdOptDesc"), `OSK` (Ziel-Skill), `OTR` (transzendentes Modul); `oSkills()`, `oTrans()`, `cfg()`, `oMods()` (NEXON-Pool oder eingebaute `MODS` als Fallback), `oVec()` (Beitrag zum Ziel-Skill), `oBase()`, `oScore()`, `oEval()`.
- `oOptimize()` – exakte Suche mit Schranke über die 10 freien Slots: Module mit gleichem Tag bilden eine Gruppe; Äste, die selbst mit den besten noch möglichen Werten nicht unter die Top 6 kommen, werden übersprungen. Typisch < 50 ms, mit extremen Gewichten bis ca. 0,3 s.
- Ausrüstung: `EQ_REACTOR` (Element_Arche → Reaktor), `EQ_COMP` (Sets Hunter/Ascending Armory/Slayer), `EQ_TRIGGER` (Power Beyond) – je [Name EN, Name DE, Bild-Hash], einmalig aus NEXONs `reactor.json`, `external-component.json`, `module.json` erzeugt (nur Links). Auswahl als Kachel-Radios (`renderEqPick()`, Zustand `EQ`), im Ergebnis `eqLine()`.
- Drop-down: `renderOptPanel()`, `renderOptList()` (erst normale Descendants, darunter Gruppe „Ultimate“), `ddOpen()/ddClose()/ddMove()/ddPick()` (Listbox mit Tastatur, Tippen springt zum Namen). Liegt bewusst **außerhalb** einer `.card`, weil `clip-path` die Liste sonst abschneidet.
- Ohne NEXON-Daten rechnet der Optimierer nur für Nell (eingebaute `MODS` + `CURVE`).
- `OFFICIAL`, `applyMeta()`, `loadOnline()` – Original-Icons laden (automatisch / mit API-Key / JSON-Dateien manuell).
- Waffen-Reiter: `W_BASE` (eingebaute Fallback-Liste, englische Namen, nur Zahlen + Bild-Links), `slimWeapon()` kürzt NEXONs `weapon.json`, `loadWeapons()`/`ensureWeapons()` laden die aktuellen Daten in der gewählten Sprache (`meta/de/` bzw. `meta/en/`, Cache in `localStorage` "tfdWeap_de"/"tfdWeap_en", 7 Tage). Auswahl (max. 3) in `WSEL`, gespeichert in `localStorage` "tfdWeapons". Seltenheitsfarben über CSS-Klassen `.t1`–`.t4` (Normal blau, Selten violett, Ultimate gold, Transzendent rot). Tooltip: `wTip()`/`showTip()`.
- Descendants-Reiter: `D_BASE` (eingebaute Fallback-Liste: ID, Name, Gruppe, Bild-Link), `slimDescendants()` kürzt NEXONs `descendant.json` + `module.json` (transzendente Module = `module_tier_id` "Tier4", Zuordnung über `available_descendant_id`), `loadDesc()`/`ensureDesc()` laden erst beim Öffnen des Reiters (Cache `localStorage` "tfdDesc_de"/"tfdDesc_en", 7 Tage). `DSEL` = geöffneter Descendant, `renderDescGrid()`/`renderDescDetail()`. Nell hat zusätzlich eigene Texte (Intro, Kern-Kombo, Einsatz-Hinweise `D_USE`) und ohne Verbindung die eigenen Kurztexte aus `NELL_SKILLS`. Der Optimierer rechnet weiterhin nur für Nell.
- Item Finder: `slimItems()` kürzt NEXONs `consumable-material.json`, `acquisition-detail.json` und `research.json`; `loadItems()`/`ensureItems()` laden erst beim Öffnen des Reiters (Cache `localStorage` "tfdItems_de"/"tfdItems_en", 7 Tage, kein eingebauter Fallback). `fBuild()` baut den Suchindex aus Items, Waffen und Descendants; `fSrc()` ordnet jede Fundstelle über das ID-Präfix (`MS`, `VB`, `FO` …) und den Gebietsnamen einem Knoten in `F_REG` und einer Art (Mission, Boss, Gebiet, NPC) zu. `fMap()` zeichnet die Karte als **eigenes Schema** (keine Spielkarte, keine echten Koordinaten – die liefert NEXON nicht). Für Module liefert NEXON keine Fundstellen.
- Material-Tracker: `TRK` (Einträge {k, n, img, have, need}, `localStorage` "tfdTrack", Zustand eingeklappt "tfdTrackOpen"), `renderTracker()`, `trkAdd()`, `trkBase()` (Rezept rekursiv bis zu Grundmaterialien), `trkEdit()` (Zahl anklicken → Eingabefeld). Hinzufügen im Item Finder („+ Tracken“, „+ Rezept tracken“, „+ Grundmaterialien tracken“) und über „Materialien tracken“ auf der Descendant-Seite. Für Teile normaler Descendants hat NEXON keinen Forschungseintrag – sie bleiben Endmaterial.
- Die Waffenauswahl wird beim Build nur angezeigt und fließt **nicht** in den Score ein.

## Rechenmodell (vereinfacht)
- Schadensfaktor = (1 + Skill Power) × (1 + Skill-Power-Modifier) × (1 + Critrate × (Critschaden − 1))
- Skill Power = allgemein + Element des Ziel-Skills; Modifier = allgemein + Arche-Typ des Ziel-Skills.
- Crit: Modulwerte mit [x] wirken multiplikativ – Critrate = (Basis + Shattered Heart 20) × (1 + Σ Module), Critschaden = (Basis + Reaktor 33) × (1 + Σ Module). Critrate mindestens 0 %, Critschaden mindestens 100 %.
- Transzendente Module: Kapazität aus NEXON pro Level; Wirkung nur für Nells Shattered Heart eingerechnet.
- Score = Schadensfaktor × (1/(1 − Cooldown))^GewichtCD × (1 + Reichweite)^GewichtReichweite
- Reaktor: „passend“ (Element + Arche-Typ des Ziel-Skills) oder „anderer Typ“ (×0,6). Guide-Vergleich (AlcastHQ) nur für Nell: Tech-Ziel = Bossing, Fusion-Ziel = Mobbing.
- Kapazität = Basis + Level; passender Sockel halbiert (aufgerundet). Max-Kapazität einstellbar (Standard 85).
- Annahmen (in „Daten & Annahmen“ änderbar): Basis-Crit 5 % / 150 % für alle Descendants, Kapazität Shattered Heart 8 (nur ohne NEXON-Daten), Reaktor-Substats −7,4 % CD / +33 % Critschaden, falscher Reaktor-Typ ×0,6, Cooldown-Grenze 75 %.

## Datenquellen
- Level-0-Modulwerte: game8.co Modulliste
- Builds, Reaktor, Komponenten, Waffen: AlcastHQ Nell Build Guide
- Spielstand: Patch 1.4.8 (November 2026)

## Offene Punkte / Roadmap
1. ~~Echte Modulwerte pro Level aus `module.json`~~ – erledigt (mit Verbindung). Offen: Basis-Critwerte je Descendant (liefert NEXON nicht), Wirkung weiterer transzendenter Module.
2. Sockel-Planung pro Slot (12 Slots wie im Spiel).
3. „Meine Module“: vorhandene Module + Level abhaken, nur baubare Builds vorschlagen, Fehlendes mit Fundort zeigen.
4. Builds speichern und als Code teilen.
5. Waffen in die Rechnung einbeziehen + Waffenmodule (Auswahl-Reiter ist da). Optimierer läuft für alle Descendants.
6. Fundorte für alle Module ergänzen (viele stehen noch auf „Zugangsinfo im Spiel“).

## Veröffentlichung (geplant)
GitHub-Repository (öffentlich) + **GitHub Pages**, damit Updates automatisch bei allen ankommen. Die Hauptdatei heißt dafür bereits `index.html`. Noch nicht eingerichtet – der Nutzer muss GitHub-Konto/Repo anlegen und GitHub mit Claude verbinden.

## Weitere Version
Es gibt zusätzlich eine Online-Version als Claude-Artifact (ohne Original-Icons, noch altes Design). Hauptversion ist die lokale Datei in diesem Ordner.
