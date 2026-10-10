---
name: verify
description: Prüft wirklich, dass eine Änderung funktioniert, bevor sie als erledigt gilt. Verwenden beim Erstellen oder Ändern von Apps (Flutter/Android), Serverkonfigurationen, Docker, Frigate, Home Assistant, n8n oder jedem Smart-Home-Sicherheits- und Automatisierungsprojekt. Erzwingt echte Prüfungen und Belege, statt zu behaupten, es „sollte funktionieren".
---

# Skill: verify (prüfen)

## Grundsatz
Du bist nicht der Prüfer aus Intuition, sondern der Prüfer aus BELEGEN.
Sage niemals „ist behoben", „sollte funktionieren" oder „ich habe das Problem gefunden", ohne eine echte Prüfung ausgeführt und ihr Ergebnis gesehen zu haben.
Wenn du etwas nicht ausführen kannst, sag es und markiere diesen Punkt als NICHT GEPRÜFT.

## Festes Protokoll (immer dieselben 6 Schritte, in dieser Reihenfolge)

1. **Projekt und Profil bestimmen.** Entscheide, welches gilt (siehe „Profile" unten). Wenn unklar, frage einmal nach.
2. **Lesen, bevor du behauptest.** Öffne die tatsächlich betroffenen Dateien und Konfigurationen. Nimm ihren Inhalt nicht an. Wenn du sagen willst „der Fehler liegt in X", zeige zuerst, dass du X gelesen hast.
3. **Erfolgskriterium festlegen.** Schreibe in ein bis zwei Zeilen, was „funktioniert" für diese Änderung bedeutet (z. B.: „Die App baut, und der Einstellungsbildschirm öffnet sich ohne Fehler").
4. **Die Prüfungen des Profils ausführen.** Verwende echte Befehle (keine erfundenen). Schlägt ein Befehl fehl, nimm ihn unverändert in den Bericht auf.
5. **Den früher fehlerhaften Fall testen.** Reproduziere das ursprüngliche Problem und bestätige, dass es nicht mehr auftritt. Kannst du es nicht reproduzieren, sag es.
6. **Mit dem Pflichtformat berichten** (siehe unten).

## Pflichtformat des Berichts

```
PRÜFUNG
Projekt: <Name/Art>
Erfolgskriterium: <eine Zeile>

Ausgeführte Prüfungen:
- [OK] <Befehl oder Test> → <relevantes Ergebnis>
- [FEHLER] <Befehl oder Test> → <Fehler>
- [NICHT GEPRÜFT] <was fehlt und warum>

Was ich gelesen habe: <geprüfte Dateien/Konfigurationen>
Fazit: GEPRÜFT / TEILWEISE / NICHT GEPRÜFT
Offene Risiken: <kurze Liste oder „keine bekannt">
Empfohlener nächster Schritt: <eine Zeile>
```

Formatregeln:
- Markiere nur dann GEPRÜFT, wenn ALLE kritischen Prüfungen OK sind.
- Zitiere Fehler wörtlich und kurz; fasse sie nicht aus dem Gedächtnis zusammen.
- Trenne Geprüftes von Angenommenem. Angenommenes steht als „ich vermute", niemals als Tatsache.

## Profil A: Apps (Flutter / Android)

Prüfungen von der günstigsten zur teuersten (beim ersten Fehlschlag anhalten):
1. `flutter analyze` → keine Fehler (Warnungen auflisten).
2. `flutter test` → alle Tests bestehen. Gibt es keine Tests für das Geänderte, sag es und schlage einen vor.
3. `flutter build apk --debug` (oder `./gradlew assembleDebug` bei nativem Android) → es baut.
4. Wenn ein Emulator/Gerät verfügbar ist: `flutter run` oder die APK installieren und den betroffenen Ablauf testen.
5. `adb logcat` (nach dem Paket der App gefiltert) auf Ausnahmen beim Öffnen des geänderten Bildschirms prüfen.

Zusätze je nach Fall:
- Änderungen an Abhängigkeiten: `flutter pub get` ohne Konflikte; die minimale SDK-Version prüfen.
- Änderungen an Berechtigungen oder Benachrichtigungen: `AndroidManifest.xml` prüfen und auf einem Gerät mit aktuellem Android testen.
- Vor dem Hochladen zu Google Play: signierter Release-Build und Kontrolle, dass `versionCode` erhöht wurde.
- Betrifft die Änderung die Datenspeicherung: mit vorhandenen Daten testen (Migration), nicht nur mit sauberer Installation.

## Profil B: Sicherheit und Automatisierung im Haus

Goldene Regel: **zuerst nur lesen**. Ändere die Produktivkonfiguration nicht ohne (a) Backup, (b) ausdrückliche Bestätigung des Nutzers und (c) Rückfallplan.

Typische Prüfungen (an die reale Installation anpassen; nimm nicht an, dass ein Befehl existiert, prüfe es):
- **Docker / docker-compose:** `docker compose config` (prüft die Syntax), `docker compose ps` (Status), `docker logs --tail 100 <Container>` (jüngste Fehler). Volume-Pfade und Ports prüfen, bevor etwas gestartet wird.
- **Frigate:** die Konfigurationsdatei mit Frigates eigenem Werkzeug validieren, falls vorhanden; in den Logs prüfen, dass jede Kamera verbindet; bestätigen, dass die Erkennung aktiv ist und Aufnahmen auf die Festplatte geschrieben werden. CPU-Auslastung prüfen, wenn die Erkennung über die CPU läuft.
- **Kameras:** bestätigen, dass der RTSP-Stream antwortet (zum Beispiel mit `ffprobe` auf die URL, ohne Zugangsdaten im Bericht zu zeigen). PoE-Budget des Switches prüfen (Summe des Kameraverbrauchs gegenüber der Gesamtleistung).
- **Home Assistant:** vor dem Neustart die Konfigurationsprüfung ausführen; danach das Log auf Integrationsfehler prüfen.
- **n8n oder andere Automatisierungen:** den Ablauf im Testmodus mit Beispieldaten ausführen und die Ausgabe jedes Knotens prüfen; Zeitzone und Zugangsdaten bestätigen.
- **NAS, Netzwerk und USV:** Konnektivität (`ping`, offene Ports), freien Speicherplatz und USV-Status prüfen (zum Beispiel mit `upsc`, falls NUT verwendet wird). Nach einem echten oder simulierten Stromausfall bestätigen, dass das geordnete Herunterfahren funktioniert.
- **Sicherheit:** Kameras, Oberflächen oder das NAS nicht ohne starke Authentifizierung und vorzugsweise VPN ins Internet stellen. Offene Ports und die Warnungen des Systems selbst prüfen. Niemals Passwörter, Token oder URLs mit Zugangsdaten in den Bericht aufnehmen.

Bei jeder Netzwerk- oder Firewalländerung: vor dem Anwenden prüfen, dass du dich nicht aussperrst (einen Rückweg bereithalten).

## Was tun, wenn nicht geprüft werden kann
- Sage genau, was du nicht prüfen konntest und warum (kein Zugriff auf das Gerät, kein Netz, keine Berechtigungen).
- Schlage den konkreten Befehl oder Test vor, den der Nutzer ausführen kann, und welches Ergebnis zu erwarten ist.
- Markiere das Fazit als TEILWEISE oder NICHT GEPRÜFT. Fülle Lücken nicht mit Annahmen.

## Anti-Muster (verboten)
- „Jetzt funktioniert es" sagen, ohne etwas ausgeführt zu haben.
- Die Ursache eines Fehlers nennen, ohne den betreffenden Code oder das Log gelesen zu haben.
- Namen von Befehlen, Optionen, Dateien oder Ports erfinden. Bei Unsicherheit prüfen oder nachfragen.
- Mehrere Dinge gleichzeitig ändern: eine Änderung, eine Prüfung.
- Fehlgeschlagene Prüfungen im Bericht verschweigen.

## Pflege dieses Skills
Jedes Mal, wenn ein Fehler durchrutscht (etwas „Geprüftes", das später fehlschlägt), füge hier eine neue Prüfung hinzu, die ihn erkannt hätte. Diese Liste wird mit echten Fehlern besser.
