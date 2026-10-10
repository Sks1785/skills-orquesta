---
name: verify
description: Naprawdę sprawdza, czy zmiana działa, zanim uzna się ją za zakończoną. Używaj przy tworzeniu lub modyfikowaniu aplikacji (Flutter/Android), konfiguracji serwerów, Dockera, Frigate, Home Assistant, n8n oraz każdego projektu domowego monitoringu i automatyzacji. Wymusza prawdziwe kontrole i dowody zamiast twierdzenia, że „powinno działać".
---

# Skill: verify (weryfikacja)

## Zasada
Nie weryfikujesz na wyczucie, tylko na podstawie DOWODÓW.
Nigdy nie mów „naprawione", „powinno działać" ani „znalazłem problem", jeśli nie wykonałeś prawdziwej kontroli i nie zobaczyłeś jej wyniku.
Jeśli czegoś nie możesz wykonać, powiedz to i oznacz ten punkt jako NIEZWERYFIKOWANE.

## Stały protokół (zawsze te same 6 kroków, w tej kolejności)

1. **Ustal projekt i jego profil.** Zdecyduj, który ma zastosowanie (patrz „Profile" poniżej). Jeśli to niejasne, zapytaj jeden raz.
2. **Czytaj, zanim coś stwierdzisz.** Otwórz prawdziwe, których dotyczy zmiana, pliki i konfigurację. Nie zakładaj ich zawartości. Jeśli masz powiedzieć „usterka jest w X", najpierw pokaż, że przeczytałeś X.
3. **Zdefiniuj kryterium sukcesu.** Napisz w jednym–dwóch wierszach, co oznacza „działa" dla tej zmiany (np.: „aplikacja się buduje, a ekran ustawień otwiera się bez błędów").
4. **Wykonaj kontrole profilu.** Używaj prawdziwych poleceń (nie zmyślonych). Jeśli polecenie zawiedzie, umieść je w raporcie bez zmian.
5. **Przetestuj przypadek, który wcześniej zawodził.** Odtwórz pierwotny problem i potwierdź, że już nie występuje. Jeśli nie możesz go odtworzyć, powiedz to.
6. **Zrelacjonuj w obowiązkowym formacie** (patrz niżej).

## Obowiązkowy format raportu

```
WERYFIKACJA
Projekt: <nazwa/typ>
Kryterium sukcesu: <jeden wiersz>

Wykonane kontrole:
- [OK] <polecenie lub test> → <istotny wynik>
- [BŁĄD] <polecenie lub test> → <błąd>
- [NIEZWERYFIKOWANE] <czego brakuje i dlaczego>

Co przeczytałem: <przejrzane pliki/konfiguracje>
Wniosek: ZWERYFIKOWANE / CZĘŚCIOWO / NIEZWERYFIKOWANE
Pozostałe ryzyka: <krótka lista lub „brak znanych">
Zalecany następny krok: <jeden wiersz>
```

Zasady formatu:
- Oznacz ZWERYFIKOWANE tylko wtedy, gdy WSZYSTKIE krytyczne kontrole są w porządku (OK).
- Cytuj błędy dosłownie i krótko; nie streszczaj ich z pamięci.
- Oddziel to, co sprawdzone, od tego, co założone. Założenia zapisuj jako „zakładam", nigdy jako fakt.

## Profil A: aplikacje (Flutter / Android)

Kontrole od najtańszych do najdroższych (zatrzymaj się na pierwszej, która zawiedzie):
1. `flutter analyze` → brak błędów (wypisz ostrzeżenia).
2. `flutter test` → wszystkie testy przechodzą. Jeśli dla zmienionej części nie ma testów, powiedz to i zaproponuj jeden.
3. `flutter build apk --debug` (lub `./gradlew assembleDebug` w natywnym Androidzie) → buduje się.
4. Jeśli dostępny jest emulator/urządzenie: `flutter run` albo zainstaluj APK i przetestuj dotknięty przepływ.
5. Przejrzyj `adb logcat` (z filtrem na pakiet aplikacji) pod kątem wyjątków przy otwieraniu zmienionego ekranu.

Dodatki zależnie od przypadku:
- Zmiany zależności: `flutter pub get` bez konfliktów; sprawdź minimalną wersję SDK.
- Zmiany uprawnień lub powiadomień: przejrzyj `AndroidManifest.xml` i przetestuj na urządzeniu z nowym Androidem.
- Przed wysłaniem do Google Play: podpisany build release i sprawdzenie, czy `versionCode` został zwiększony.
- Jeśli zmiana dotyczy przechowywania danych: przetestuj na istniejących danych (migracja), a nie tylko na czystej instalacji.

## Profil B: bezpieczeństwo i automatyzacja domowa

Złota zasada: **najpierw tylko odczyt**. Nie zmieniaj konfiguracji produkcyjnej bez (a) kopii zapasowej, (b) wyraźnego potwierdzenia użytkownika i (c) planu wycofania zmian.

Typowe kontrole (dostosuj do prawdziwej instalacji; nie zakładaj, że polecenie istnieje, sprawdź to):
- **Docker / docker-compose:** `docker compose config` (waliduje składnię), `docker compose ps` (stan), `docker logs --tail 100 <kontener>` (ostatnie błędy). Przejrzyj ścieżki woluminów i porty, zanim cokolwiek uruchomisz.
- **Frigate:** zwaliduj plik konfiguracyjny narzędziem samego Frigate, jeśli takie istnieje; sprawdź w logach, że każda kamera się łączy; potwierdź, że detekcja jest aktywna, a nagrania zapisują się na dysku. Sprawdź użycie CPU, jeśli detekcja działa na CPU.
- **Kamery:** potwierdź, że strumień RTSP odpowiada (np. za pomocą `ffprobe` na adresie URL, bez pokazywania danych logowania w raporcie). Sprawdź budżet PoE przełącznika (suma poboru kamer wobec całkowitej mocy).
- **Home Assistant:** uruchom kontrolę konfiguracji przed restartem; potem przejrzyj dziennik pod kątem błędów integracji.
- **n8n i inne automatyzacje:** uruchom przepływ w trybie testowym na danych przykładowych i sprawdź wyjście każdego węzła; potwierdź strefę czasową i dane uwierzytelniające.
- **NAS, sieć i UPS (zasilacz awaryjny):** sprawdź łączność (`ping`, otwarte porty), wolne miejsce na dysku i stan UPS (np. `upsc`, jeśli używany jest NUT). Po prawdziwej lub symulowanej przerwie w zasilaniu potwierdź, że uporządkowane wyłączanie działa.
- **Bezpieczeństwo:** nie wystawiaj kamer, paneli ani NAS do internetu bez silnego uwierzytelniania i, najlepiej, VPN. Sprawdź otwarte porty i własne alerty systemu. Nigdy nie umieszczaj w raporcie haseł, tokenów ani adresów URL z danymi logowania.

Przy każdej zmianie sieci lub zapory: przed zastosowaniem sprawdź, czy nie zablokujesz sobie dostępu (zostaw drogę powrotu).

## Co robić, gdy nie można zweryfikować
- Powiedz dokładnie, czego nie mogłeś sprawdzić i dlaczego (brak dostępu do urządzenia, brak sieci, brak uprawnień).
- Zaproponuj konkretne polecenie lub test, które użytkownik może wykonać, i jakiego wyniku się spodziewać.
- Oznacz wniosek jako CZĘŚCIOWO lub NIEZWERYFIKOWANE. Nie wypełniaj luk założeniami.

## Antywzorce (zakazane)
- Mówienie „już działa" bez wykonania czegokolwiek.
- Podawanie przyczyny usterki bez przeczytania odpowiedniego kodu lub logu.
- Zmyślanie nazw poleceń, opcji, plików lub portów. Jeśli nie jesteś pewien, sprawdź lub zapytaj.
- Zmienianie kilku rzeczy naraz: jedna zmiana, jedna weryfikacja.
- Ukrywanie nieudanych kontroli w raporcie.

## Utrzymanie tego skilla
Za każdym razem, gdy jakiś błąd się prześliźnie (coś „zweryfikowanego", co później zawodzi), dodaj tu nową kontrolę, która by go wykryła. Ta lista poprawia się dzięki prawdziwym błędom.
