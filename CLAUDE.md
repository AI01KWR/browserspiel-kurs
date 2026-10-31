# Zombie Highway – Hinweise für Claude

Endless Runner im Stil von Subway Surfers, aber mit eigenem Thema: Man fährt nachts ein
gepanzertes Auto durch eine verlassene Zombie-Stadt. Das Auto hat Lebenspunkte, alles kann
durchbrochen werden. Die ständige Entscheidung: ausweichen oder Schaden in Kauf nehmen?
Keine Figuren, Namen oder Grafiken von Subway Surfers verwenden.

## Zusammenarbeit
- Der Besitzer ist Programmier-Anfänger: kurz und einfach auf Deutsch erklären, was und warum.
- **Vor Änderungen am Spielgefühl fragen** (Schaden, Tempo, Zombie-Anzahl, Steuerung).
- Betroffene Stellen in `index.html` lesen, bevor etwas geändert wird.
- Nach jeder Änderung prüfen, dass das Spiel ohne Fehler in der Browser-Konsole startet
  (lokal mit `python -m http.server 8000`, siehe `.claude/launch.json`; unter `file://` bzw.
  `data:` ist localStorage gesperrt).
- Nach jeder fertigen Änderung ein Git-Commit mit verständlicher deutscher Nachricht.
- Diese Datei aktuell halten, wenn sich am Spiel etwas Wichtiges ändert.

## Technik
- Nur statische Dateien für GitHub Pages: kein Server, kein Build-Schritt.
- Alles steckt in **einer** Datei: `index.html` (der Branch `grafik` ist in `main` übernommen).
- Three.js **0.169.0** als ES-Modul über eine **Import-Map** von jsdelivr
  (`three` und `three/addons/` für Zusatzmodule). Das Spiel-Script ist `<script type="module">`.
  Weil Module keine globalen Variablen haben, gibt es zum Testen `window.ZH`
  (state, car, startGame, damage …).
- Bibliotheken nur mit fester Versionsnummer, keine Build-Tools.
- Grafik: sRGB-Ausgabe, ACES-Filmic-Tone-Mapping, eigene gemalte Nacht-Environment-Map
  (PMREM), `MeshStandardMaterial` überall (Autolack: `MeshPhysicalMaterial` mit Klarlack).
  Lichtstärken sind physikalisch (Spot/Point mit decay 2), daher große Zahlen.
  Canvas-Texturen mit Farben brauchen `colorSpace = SRGBColorSpace` (macht `canvasTex()`).
- Nachbearbeitung über `composer` (EffectComposer): RenderPass → UnrealBloomPass
  (threshold 1.0) → `fxPass` (Vignette + Bewegungsunschärfe bei Nitro) → OutputPass.
  Es wird mit `composer.render()` gezeichnet, nicht mit `renderer.render()`.
- Was leuchten soll, bekommt eine Farbe heller als Weiß über `glow(farbe, stärke)`
  (Scheinwerfer, Laternen, Feuer, Zombie-Augen, Nitro). Fenster bleiben knapp unter der Schwelle.
- Nasse Straße: `roughnessMap` mit Pfützen (scrollt mit der Straßen-Textur mit),
  Lichtflecken der Laternen auf dem Boden (`MAT.pool`).
- Schatten: VSM mit Weichzeichner am PC, PCF auf dem Handy.
- Kein Licht von vorne/vom Horizont hinzufügen: Es spiegelt sich in der nassen Straße als
  greller Bloom-Fleck direkt vor dem Auto (wurde deshalb entfernt).
- Alle Texturen werden zur Laufzeit auf Canvas gezeichnet, es gibt keine Bilddateien.
- Schrift „Bungee“ (Google Fonts) für Titel und Zahlen.
- localStorage immer über das Objekt `Store` (mit try/catch).
- Grafik-Ziel (Branch `grafik`): möglichst realistisch Richtung GTA, Gameplay bleibt gleich.
  Materialien über Helfer `std()`.
- Auf Handys (`LOW_Q`) wird Qualität gespart: kleinere Schattenkarte, weniger Teilchen,
  keine Feuer-Lichter, kein Antialiasing.

## Aufbau von index.html (Reihenfolge im Script)
1. **EINSTELLUNGEN** – alle Spielwerte als Konstanten ganz oben.
2. `Store`, `Leaderboard`, `Wallet` (Münzkonto), `Garage`, `UPGRADES`/`Upgrades`.
3. Renderer, Szene, Kamera, Licht (Mond mit Schatten), Nachbearbeitung (`composer`, `renderPass`).
4. Texturen (`TEX`, `facadeTextures`, `shirtTex`) und Materialien (`MAT`).
5. Himmel (Sterne, Mond, Skyline), Straße/Gehwege, Hochhäuser, Deko am Rand.
6. Autos (`build…`-Funktionen, `CARS`, `RARITY`, `buildCar`, `carModel`, `setPlayerCar`), Zombies und Hindernisse (Fabrikfunktionen `make…`, Pool mit `obtain`/`release`).
7. Teilchen: Splitter/grüne Teilchen (`spawnBits`), Rauch, Asche.
8. Spielzustand `state` und `car`, Spielablauf (`startGame`, `damage`, `gameOver` …).
9. Spawnen (`spawnPattern`, `spawnZombieGroup`), Bewegung und Zusammenstöße.
10. Hauptschleife `update()`/`frame()`, Menüs/Rangliste, Showroom (`showScene`, Garage, Shop,
    Kisten-Animation), Upgrades-Fenster, Steuerung.

Die Welt bewegt sich auf das Auto zu (Auto bleibt bei z = 0, fährt nach -z). Neue Dinge
erscheinen bei `SPAWN_Z = -130` im Nebel und werden hinter der Kamera aufgeräumt.

## Spielmechanik (genau so eingestellt – nicht ohne Rückfrage ändern)
- 3 Spuren bei x = -2.6, 0, 2.6. Das Auto fährt automatisch.
- Tempo 20 → max. 48, Schwierigkeit steigt über 120 Sekunden.
- 100 Leben. Schaden: Zombie 10, Holzzaun 15, Kisten 20, Autowrack 35, Betonsperre 50.
- Zombies laufen auf das Auto zu und lenken in seine Spur; manche kommen vom Straßenrand,
  manchmal eine Horde aus 2.
- Zombie-Takt: `(1.5 - Schwierigkeit * 0.65) * zufällig(0.7, 1.3)` Sekunden, erster Zombie
  nach ca. 1 Sekunde. Horden-Chance `0.18 + Schwierigkeit * 0.12`, Horde = 2 bis 3 Zombies.
  (Wurde mehrfach angepasst: erst zu viele, dann zu wenige, dann zu viele – jetzt Mittelweg.)
- Hindernis-Reihen (`spawnPattern`, `spawning`): Abstand `zufällig(12, 16) - Schwierigkeit * 3`
  Einheiten (nach Rampen `Tempo * 0.3` extra zum Landen). 22 % der Reihen sind Rampen, sonst
  Hindernis-Reihe mit 1–3 Hindernissen. Der Durchschnitt pro Reihe entspricht den Chancen
  „3 ab Schwierigkeit > 0.4 mit 15 %, sonst 2 mit `0.4 + Schwierigkeit * 0.3`, sonst 1“,
  wird aber über ein Konto (`obstacleCount`, `state.obstacleBudget`) **gleichmäßig verteilt**
  (Wunsch: nicht mal sehr viele und dann gar keine). Beim Start liegen schon Reihen alle
  16 Einheiten (von z = -40 bis zum Spawn-Punkt). Zuletzt auf Wunsch „ganz wenig weniger“
  (vorher 11–15).
- Autolicht: SpotLights Stärke 700, decay 1.2, Winkel 0.42 – sichtbar, aber nicht grell
  (110 war zu schwach, man sah gar kein Licht mehr; 220 mit decay 2 war zu stark).
- Umgefahrene Zombies fliegen comicartig weg, grüne Teilchen, kein Blut.
- Münze: +10 Punkte, +4 Nitro. Zombie: +5 Punkte, +3 Nitro.
- Rote Werkzeugkiste: +25 Leben.
- Springen per Taste oder über gelbe Rampen; im Sprung fliegt man über Hindernisse und
  Zombies. Hinter Rampen stehen oft Hindernisse, darüber Münzen in der Luft.
- Nitro (Leiste voll): 3,5 s, 1,6-faches Tempo, **unverwundbar** (auf Wunsch geändert, vorher
  halber Schaden). Hindernisse werden trotzdem durchbrochen, nur ohne Schaden und ohne Abbremsen.
- Nach Treffer: kurz langsamer, Bildschirm wackelt und blinkt rot.
- Unter 50 Leben wird das Auto dunkler und raucht, unter 25 stärker. Bei 0 Game Over.
- Punkte = Strecke + Münzen × 10 + Zombies × 5.

## Steuerung
- Spur: ←/→ oder A/D · Springen: Leertaste, ↑ oder W · Nitro: E
- Handy: links/rechts wischen, hoch = springen, runter = Nitro, plus Nitro-Knopf.

## Rangliste
- Lokal: Top 10 pro Gerät in localStorage, Schlüssel `zombie-highway-board`.
  Letzter Name unter `zombie-highway-name`.
- Namensfeld nach Game Over bei Top-10-Platz (max. 12 Zeichen). Namen werden nur per
  `textContent` angezeigt.
- Objekt `Leaderboard` mit `load()` und `submit(entry)`. Beide sind schon `async`, damit
  später eine Online-Rangliste (z. B. Supabase oder Firebase) nur diese zwei Funktionen ersetzt.
  **Vorher mit dem Besitzer besprechen**, wie Schummeln und unpassende Namen verhindert werden.

## Münzen, Shop, Garage, Upgrades (nur Spielmünzen, kein echtes Geld)
- Nach jeder Fahrt werden die Münzen aufs Konto addiert (`zombie-highway-coins`).
  Anzeige im Hauptmenü, im Shop, bei den Upgrades und beim Game Over („+45 Münzen“).
- Hauptmenü: Losfahren, Garage, Shop, Upgrades, Rangliste.
- Shop: Kiste für 1000 Münzen. Seltenheiten: Gewöhnlich 60 % (grau), Selten 25 % (blau),
  Episch 12 % (lila), Legendär 3 % (gold), Mythisch 0,01 % (pink; dafür Gewöhnlich 59,99 %).
  Doppeltes Auto → Münzen zurück: 200 / 400 / 700 / 1000 / 5000. Animation: Kiste wackelt, springt auf, Auto erscheint und dreht sich.
- Autos: Startauto „Zombie-Brecher“ (rot) + Pickup, Alter Kombi, Taxi (gewöhnlich),
  Polizeiauto, Geländewagen (selten), Monstertruck, Panzer-Transporter (episch),
  Flammen-Muscle-Car (legendär), Phantom-Hoverauto (mythisch, schwebt, Neon). Alle aus eigenen Bauteilen (keine Modelldateien).
  Sie unterscheiden sich **nur im Aussehen**; Zusammenstöße nutzen immer dieselben Maße.
- Garage (`zombie-highway-garage`: `owned`, `selected`): eigene Autos farbig, fehlende als
  Silhouette mit „?“ und Seltenheit. Das gezeigte Auto dreht sich in der Showroom-Szene.
- Showroom: eigene Szene `showScene`, wird über denselben Composer gezeichnet
  (`renderPass.scene` wird umgeschaltet). Kamera-Abstand passt sich dem Hochformat an.
- Upgrades (`zombie-highway-upgrades`), je 3 Stufen:
  Panzerung −5 % Schaden/Stufe (300/600/1000), Rammbügel −2 Zombie-Schaden/Stufe
  (250/500/800), Nitro-Tank +0,5 s/Stufe (300/600/1000), Werkzeugkasten +5 Leben pro
  Werkzeugkiste/Stufe (200/400/700). Umsetzung in `damageFor`, `nitroDuration`, `repairAmount`.
- Balance-Schätzung: ca. 100–250 Münzen pro Fahrt → 4–10 Fahrten pro Kiste.

## Ideen für später (nur nach Absprache)
- Sounds und Musik
- Kleine Unterschiede zwischen den Autos (z. B. etwas mehr Leben, längeres Nitro, max. 10–20 %)
- Echte 3D-Modelle (CC0) – Schritt 3 des Grafik-Umbaus, noch offen; vor Downloads fragen
- Weitere Power-ups (z. B. Münzmagnet)
- Einstellung für niedrigere Grafikqualität
- Online-Rangliste

## Hosting
GitHub-Repo `AI01KWR/browserspiel-kurs`, GitHub Pages vom Branch `main` (Ordner `/`).
Adresse: https://ai01kwr.github.io/browserspiel-kurs/
