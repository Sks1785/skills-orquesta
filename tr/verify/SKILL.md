---
name: verify
description: Bir değişikliği bitmiş saymadan önce gerçekten çalışıp çalışmadığını doğrular. Uygulama (Flutter/Android), sunucu yapılandırması, Docker, Frigate, Home Assistant, n8n veya herhangi bir ev güvenliği ve otomasyon projesi oluştururken ya da değiştirirken kullanın. "Çalışması lazım" demek yerine gerçek kontroller yapmayı ve kanıt göstermeyi zorunlu kılar.
---

# Skill: verify (doğrulama)

## İlke
Sezgiyle doğrulayan değil, KANITLA doğrulayan kişisiniz.
Gerçek bir kontrol çalıştırıp sonucunu görmeden asla "düzeldi", "çalışması lazım" veya "sorunu buldum" demeyin.
Bir şeyi çalıştıramıyorsanız bunu söyleyin ve o maddeyi DOĞRULANMADI olarak işaretleyin.

## Sabit protokol (her zaman aynı 6 adım, bu sırayla)

1. **Projeyi ve profilini belirleyin.** Hangisinin geçerli olduğuna karar verin (aşağıdaki "Profiller"e bakın). Net değilse bir kez sorun.
2. **İddia etmeden önce okuyun.** Etkilenen gerçek dosyaları ve yapılandırmayı açın. İçeriklerini varsaymayın. "Hata X'te" diyecekseniz önce X'i okuduğunuzu gösterin.
3. **Başarı ölçütünü tanımlayın.** Bu değişiklik için "çalışıyor"un ne demek olduğunu bir iki satırda yazın (örn.: "uygulama derleniyor ve ayarlar ekranı hatasız açılıyor").
4. **Profilin kontrollerini çalıştırın.** Gerçek komutlar kullanın (uydurma değil). Bir komut başarısız olursa raporda olduğu gibi yer verin.
5. **Eskiden başarısız olan durumu deneyin.** Asıl sorunu yeniden üretin ve artık oluşmadığını doğrulayın. Yeniden üretemezseniz bunu söyleyin.
6. **Zorunlu biçimle raporlayın** (aşağıya bakın).

## Zorunlu rapor biçimi

```
DOĞRULAMA
Proje: <ad/tür>
Başarı ölçütü: <tek satır>

Çalıştırılan kontroller:
- [TAMAM] <komut veya test> → <ilgili sonuç>
- [HATA] <komut veya test> → <hata>
- [DOĞRULANMADI] <neyin eksik olduğu ve nedeni>

Okuduklarım: <incelenen dosyalar/yapılandırmalar>
Sonuç: DOĞRULANDI / KISMEN / DOĞRULANMADI
Kalan riskler: <kısa liste veya "bilinen yok">
Önerilen sonraki adım: <tek satır>
```

Biçim kuralları:
- Yalnızca TÜM kritik kontroller TAMAM ise DOĞRULANDI yazın.
- Hataları kelimesi kelimesine ve kısaca alıntılayın; hafızadan özetlemeyin.
- Kontrol edileni varsayılandan ayırın. Varsayımlar "sanıyorum" diye yazılır, asla olgu olarak yazılmaz.

## Profil A: uygulamalar (Flutter / Android)

Kontroller, en ucuzdan en pahalıya doğru (ilk başarısızlıkta durun):
1. `flutter analyze` → hata yok (uyarıları listeleyin).
2. `flutter test` → tüm testler geçiyor. Değişen kısım için test yoksa bunu söyleyin ve bir tane önerin.
3. `flutter build apk --debug` (yerel Android'de `./gradlew assembleDebug`) → derleniyor.
4. Emülatör/cihaz varsa: `flutter run` veya APK'yı kurup etkilenen akışı deneyin.
5. `adb logcat`'i (uygulamanın paketine göre filtreleyerek) inceleyip değişen ekran açılırken istisna olup olmadığına bakın.

Duruma göre ekler:
- Bağımlılık değişiklikleri: `flutter pub get` çakışmasız; en düşük SDK sürümünü kontrol edin.
- İzin veya bildirim değişiklikleri: `AndroidManifest.xml`'i inceleyin ve güncel Android'li bir cihazda deneyin.
- Google Play'e yüklemeden önce: imzalı release derlemesi ve `versionCode`'un artırıldığının kontrolü.
- Değişiklik veri depolamaya dokunuyorsa: yalnızca temiz kurulumla değil, mevcut verilerle de (geçiş/migrasyon) deneyin.

## Profil B: ev güvenliği ve otomasyon

Altın kural: **önce yalnızca okuma**. (a) yedek, (b) kullanıcının açık onayı ve (c) geri dönüş planı olmadan canlı yapılandırmayı değiştirmeyin.

Tipik kontroller (gerçek kuruluma uyarlayın; bir komutun var olduğunu varsaymayın, kontrol edin):
- **Docker / docker-compose:** `docker compose config` (sözdizimini doğrular), `docker compose ps` (durum), `docker logs --tail 100 <konteyner>` (son hatalar). Bir şey ayağa kaldırmadan önce birim (volume) yollarını ve portları inceleyin.
- **Frigate:** varsa Frigate'in kendi aracıyla yapılandırma dosyasını doğrulayın; günlüklerde her kameranın bağlandığını görün; algılamanın etkin olduğunu ve kayıtların diske yazıldığını doğrulayın. Algılama CPU ile yapılıyorsa CPU kullanımını kontrol edin.
- **Kameralar:** RTSP akışının yanıt verdiğini doğrulayın (örneğin URL üzerinde `ffprobe` ile, raporda kimlik bilgilerini göstermeden). Anahtarın (switch) PoE bütçesini inceleyin (kamera tüketimlerinin toplamı ile toplam güç).
- **Home Assistant:** yeniden başlatmadan önce yapılandırma kontrolünü çalıştırın; sonra günlükte entegrasyon hatalarına bakın.
- **n8n veya diğer otomasyonlar:** akışı örnek verilerle test modunda çalıştırın ve her düğümün çıktısını kontrol edin; saat dilimini ve kimlik bilgilerini doğrulayın.
- **NAS, ağ ve UPS:** bağlantıyı (`ping`, açık portlar), boş disk alanını ve UPS durumunu kontrol edin (NUT kullanıyorsa örneğin `upsc`). Gerçek ya da simüle bir elektrik kesintisinden sonra düzenli kapanmanın çalıştığını doğrulayın.
- **Güvenlik:** güçlü kimlik doğrulama (tercihen VPN) olmadan kameraları, panelleri veya NAS'ı internete açmayın. Açık portları ve sistemin kendi uyarılarını inceleyin. Rapora asla parola, jeton veya kimlik bilgisi içeren URL koymayın.

Ağ veya güvenlik duvarı değişikliklerinde: uygulamadan önce kendinizi dışarıda bırakmayacağınızı kontrol edin (geri dönüş yolu bulundurun).

## Doğrulanamadığında ne yapılır
- Neyi neden kontrol edemediğinizi tam olarak söyleyin (cihaza erişim yok, ağ yok, izin yok).
- Kullanıcının çalıştırabileceği somut komutu veya testi ve beklenen sonucu önerin.
- Sonucu KISMEN veya DOĞRULANMADI olarak işaretleyin. Boşlukları varsayımlarla doldurmayın.

## Yasak kalıplar
- Hiçbir şey çalıştırmadan "artık çalışıyor" demek.
- İlgili kodu veya günlüğü okumadan bir arızanın nedenini söylemek.
- Komut, seçenek, dosya veya port adı uydurmak. Emin değilseniz kontrol edin veya sorun.
- Aynı anda birden çok şeyi değiştirmek: bir değişiklik, bir doğrulama.
- Başarısız kontrolleri rapordan gizlemek.

## Bu skill'in bakımı
Bir hata gözden kaçtığında (daha sonra bozulan "doğrulanmış" bir şey), onu yakalayacak yeni bir kontrolü buraya ekleyin. Bu liste gerçek hatalarla gelişir.
