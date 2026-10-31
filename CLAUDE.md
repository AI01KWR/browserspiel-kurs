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
- Alles steckt in **einer** Datei: `index.html`.
- Three.js **r128** über cdnjs: `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`
- Bibliotheken nur über `<script>`-Tags mit fester Versionsnummer.
- Alle Texturen werden zur Laufzeit auf Canvas gezeichnet, es gibt keine Bilddateien.
- Schrift „Bungee“ (Google Fonts) für Titel und Zahlen.
- localStorage immer über das Objekt `Store` (mit try/catch).
- Comic-Look über `MeshToonMaterial` mit eigener `gradientMap` (Helfer `toon()`).
- Auf Handys (`LOW_Q`) wird Qualität gespart: kleinere Schattenkarte, weniger Teilchen,
  keine Feuer-Lichter, kein Antialiasing.

## Aufbau von index.html (Reihenfolge im Script)
1. **EINSTELLUNGEN** – alle Spielwerte als Konstanten ganz oben.
2. `Store` und `Leaderboard`.
3. Renderer, Szene, Kamera, Licht (Mond mit Schatten, rötlicher Horizont-Schein).
4. Texturen (`TEX`, `facadeTextures`, `shirtTex`) und Materialien (`MAT`).
5. Himmel (Sterne, Mond, Skyline), Straße/Gehwege, Hochhäuser, Deko am Rand.
6. Auto (`carModel`), Zombies und Hindernisse (Fabrikfunktionen `make…`, Pool mit `obtain`/`release`).
7. Teilchen: Splitter/grüne Teilchen (`spawnBits`), Rauch, Asche.
8. Spielzustand `state` und `car`, Spielablauf (`startGame`, `damage`, `gameOver` …).
9. Spawnen (`spawnPattern`, `spawnZombieGroup`), Bewegung und Zusammenstöße.
10. Hauptschleife `update()`, Menüs/Rangliste, Steuerung.

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
- Hindernis-Reihen (`spawnPattern`, `spawning`): Abstand `zufällig(10, 16) - Schwierigkeit * 3`
  Einheiten (nach Rampen zusätzlich Platz). 22 % der Reihen sind Rampen, sonst Hindernis-Reihe:
  3 Hindernisse ab Schwierigkeit > 0.4 mit 15 % Chance, sonst 2 mit Chance
  `0.4 + Schwierigkeit * 0.3`, sonst 1. Beim Start liegen schon Reihen alle 16 Einheiten
  (von z = -40 bis zum Spawn-Punkt).
- Umgefahrene Zombies fliegen comicartig weg, grüne Teilchen, kein Blut.
- Münze: +10 Punkte, +4 Nitro. Zombie: +5 Punkte, +3 Nitro.
- Rote Werkzeugkiste: +25 Leben.
- Springen per Taste oder über gelbe Rampen; im Sprung fliegt man über Hindernisse und
  Zombies. Hinter Rampen stehen oft Hindernisse, darüber Münzen in der Luft.
- Nitro (Leiste voll): 3,5 s, 1,6-faches Tempo, **halber** Schaden (bewusst nicht unverwundbar).
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

## Ideen für später (nur nach Absprache)
- Sounds und Musik
- Shop: Upgrades mit Münzen (Panzerung, besserer Rammbügel), andere Autos
- Weitere Power-ups (z. B. Münzmagnet)
- Einstellung für niedrigere Grafikqualität
- Online-Rangliste

## Hosting
GitHub-Repo `AI01KWR/browserspiel-kurs`, GitHub Pages vom Branch `main` (Ordner `/`).
Adresse: https://ai01kwr.github.io/browserspiel-kurs/
