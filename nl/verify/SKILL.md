---
name: verify
description: Controleert echt of een wijziging werkt voordat je haar als klaar beschouwt. Gebruik bij het maken of wijzigen van apps (Flutter/Android), serverconfiguraties, Docker, Frigate, Home Assistant, n8n of elk beveiligings- en automatiseringsproject voor thuis. Dwingt echte controles en bewijs af in plaats van beweren dat het "zou moeten werken".
---

# Skill: verify (verifiëren)

## Principe
Je bent niet de verificateur op gevoel, maar op BEWIJS.
Zeg nooit "het is opgelost", "het zou moeten werken" of "ik heb het probleem gevonden" zonder een echte controle te hebben uitgevoerd en het resultaat te hebben gezien.
Als je iets niet kunt uitvoeren, zeg dat dan en markeer dat punt als NIET GEVERIFIEERD.

## Vast protocol (altijd dezelfde 6 stappen, in deze volgorde)

1. **Het project en het bijbehorende profiel bepalen.** Beslis welk profiel van toepassing is (zie "Profielen" hieronder). Is het onduidelijk, vraag dan één keer.
2. **Lezen vóór je iets beweert.** Open de echte betrokken bestanden en configuratie. Neem de inhoud niet aan. Als je gaat zeggen "de fout zit in X", laat dan eerst zien dat je X hebt gelezen.
3. **Het succescriterium vastleggen.** Schrijf in een of twee regels op wat "werkt" betekent voor deze wijziging (bijv.: "de app bouwt en het instellingenscherm opent zonder fouten").
4. **De controles van het profiel uitvoeren.** Gebruik echte commando's (geen verzonnen). Faalt een commando, neem het dan ongewijzigd op in het rapport.
5. **Het geval testen dat eerder faalde.** Reproduceer het oorspronkelijke probleem en bevestig dat het niet meer optreedt. Lukt reproduceren niet, zeg dat dan.
6. **Rapporteren in het verplichte formaat** (zie hieronder).

## Verplicht rapportformaat

```
VERIFICATIE
Project: <naam/type>
Succescriterium: <één regel>

Uitgevoerde controles:
- [OK] <commando of test> → <relevant resultaat>
- [MISLUKT] <commando of test> → <fout>
- [NIET GEVERIFIEERD] <wat ontbreekt en waarom>

Wat ik gelezen heb: <beoordeelde bestanden/configuraties>
Conclusie: GEVERIFIEERD / GEDEELTELIJK / NIET GEVERIFIEERD
Openstaande risico's: <korte lijst of "geen bekend">
Aanbevolen volgende stap: <één regel>
```

Regels voor het formaat:
- Markeer alleen GEVERIFIEERD als ALLE kritieke controles OK zijn.
- Citeer fouten letterlijk en kort; vat ze niet uit je hoofd samen.
- Scheid wat gecontroleerd is van wat verondersteld is. Veronderstellingen schrijf je als "ik neem aan", nooit als feit.

## Profiel A: apps (Flutter / Android)

Controles, van de goedkoopste naar de duurste (stop bij de eerste die faalt):
1. `flutter analyze` → geen fouten (waarschuwingen opsommen).
2. `flutter test` → alle tests slagen. Zijn er geen tests voor het gewijzigde deel, zeg dat dan en stel er een voor.
3. `flutter build apk --debug` (of `./gradlew assembleDebug` bij native Android) → het bouwt.
4. Als er een emulator/apparaat beschikbaar is: `flutter run` of de APK installeren en de betrokken flow testen.
5. `adb logcat` bekijken (gefilterd op het pakket van de app) op uitzonderingen bij het openen van het gewijzigde scherm.

Extra's afhankelijk van het geval:
- Wijzigingen in afhankelijkheden: `flutter pub get` zonder conflicten; controleer de minimale SDK-versie.
- Wijzigingen in rechten of meldingen: bekijk `AndroidManifest.xml` en test op een apparaat met een recente Android-versie.
- Vóór uploaden naar Google Play: ondertekende release-build en controle dat `versionCode` is verhoogd.
- Raakt de wijziging gegevensopslag: test met bestaande gegevens (migratie), niet alleen met een schone installatie.

## Profiel B: beveiliging en automatisering thuis

Gouden regel: **eerst alleen-lezen**. Wijzig de productieconfiguratie niet zonder (a) een back-up, (b) uitdrukkelijke bevestiging van de gebruiker en (c) een terugdraaiplan.

Typische controles (pas ze aan op de echte installatie; neem niet aan dat een commando bestaat, controleer het):
- **Docker / docker-compose:** `docker compose config` (valideert de syntaxis), `docker compose ps` (status), `docker logs --tail 100 <container>` (recente fouten). Bekijk volumepaden en poorten voordat je iets opstart.
- **Frigate:** valideer het configuratiebestand met Frigates eigen hulpmiddel als dat er is; controleer in de logs dat elke camera verbindt; bevestig dat detectie actief is en dat opnames naar schijf worden geschreven. Controleer het CPU-gebruik als detectie op de CPU draait.
- **Camera's:** bevestig dat de RTSP-stream reageert (bijvoorbeeld met `ffprobe` op de URL, zonder inloggegevens in het rapport te tonen). Controleer het PoE-budget van de switch (som van het verbruik van de camera's tegenover het totale vermogen).
- **Home Assistant:** voer de configuratiecontrole uit vóór het herstarten; bekijk daarna het logboek op fouten van integraties.
- **n8n of andere automatiseringen:** voer de flow in testmodus uit met voorbeeldgegevens en controleer de uitvoer van elke node; bevestig tijdzone en inloggegevens.
- **NAS, netwerk en UPS:** controleer de connectiviteit (`ping`, open poorten), vrije schijfruimte en de UPS-status (bijvoorbeeld met `upsc` bij gebruik van NUT). Bevestig na een echte of gesimuleerde stroomuitval dat het ordelijk afsluiten werkt.
- **Beveiliging:** stel camera's, dashboards of de NAS niet bloot aan internet zonder sterke authenticatie en bij voorkeur een VPN. Controleer open poorten en de eigen waarschuwingen van het systeem. Neem nooit wachtwoorden, tokens of URL's met inloggegevens op in het rapport.

Bij elke netwerk- of firewallwijziging: controleer vóór het toepassen dat je jezelf niet buitensluit (houd een weg terug open).

## Wat te doen als verifiëren niet kan
- Zeg precies wat je niet kon controleren en waarom (geen toegang tot het apparaat, geen netwerk, geen rechten).
- Stel het concrete commando of de concrete test voor die de gebruiker kan uitvoeren en welk resultaat te verwachten is.
- Markeer de conclusie als GEDEELTELIJK of NIET GEVERIFIEERD. Vul gaten niet met aannames.

## Anti-patronen (verboden)
- "Het werkt nu" zeggen zonder iets te hebben uitgevoerd.
- De oorzaak van een storing noemen zonder de relevante code of het relevante logboek te hebben gelezen.
- Namen van commando's, opties, bestanden of poorten verzinnen. Ben je niet zeker, controleer of vraag.
- Meerdere dingen tegelijk wijzigen: één wijziging, één verificatie.
- Mislukte controles uit het rapport weglaten.

## Onderhoud van deze skill
Telkens wanneer een fout doorglipt (iets "geverifieerds" dat later faalt), voeg hier een nieuwe controle toe die haar had opgemerkt. Deze lijst wordt beter met echte fouten.
