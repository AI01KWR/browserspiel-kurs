# 🧟 Zombie Highway

Ein 3D-Browserspiel: Du fährst nachts ein gepanzertes rotes Auto durch eine verlassene
Stadt voller Zombies. Dein Auto hält einiges aus. Du kannst durch Zäune, Kisten, Wracks
und Betonsperren brechen, aber jedes Mal kostet es Leben. Weichst du aus, oder nimmst du
den Schaden in Kauf?

**Jetzt spielen:** https://ai01kwr.github.io/browserspiel-kurs/

## So funktioniert's
- Sammle Münzen und fahr Zombies um, damit die Nitro-Leiste voll wird.
- Nitro macht dich schneller, und du bekommst nur halben Schaden.
- Rote Werkzeugkisten reparieren dein Auto (+25 Leben).
- Über gelbe Rampen springst du über Hindernisse.
- Punkte = Strecke + Münzen × 10 + Zombies × 5. Die besten 10 landen in der Rangliste.

## Steuerung

| Aktion        | Tastatur                    | Handy                       |
|---------------|-----------------------------|-----------------------------|
| Spur wechseln | ← / → oder A / D            | nach links / rechts wischen |
| Springen      | Leertaste, ↑ oder W         | nach oben wischen           |
| Nitro         | E                           | nach unten wischen oder Nitro-Knopf |

## Lokal starten
Das Spiel besteht nur aus der Datei `index.html`. Ein Build-Schritt ist nicht nötig.

1. Einfach `index.html` doppelt anklicken, dann öffnet sich das Spiel im Browser.
2. Damit die Rangliste gespeichert wird, besser über einen kleinen lokalen Server starten.
   Dazu im Projektordner ausführen:
   ```bash
   python -m http.server 8000
   ```
   Danach im Browser http://localhost:8000 öffnen.

Eine Internetverbindung braucht man trotzdem, weil Three.js und die Schriftart aus dem Netz
geladen werden.

## Technik
- HTML, CSS und JavaScript in einer einzigen Datei
- 3D-Grafik mit [Three.js](https://threejs.org/) r128
- Alle Texturen werden beim Start mit Canvas gezeichnet, es gibt keine Bilddateien.
- Gehostet mit GitHub Pages
