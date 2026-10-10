---
name: verify
description: Memverifikasi dengan sungguh-sungguh bahwa suatu perubahan berfungsi sebelum dianggap selesai. Gunakan saat membuat atau mengubah aplikasi (Flutter/Android), konfigurasi server, Docker, Frigate, Home Assistant, n8n, atau proyek keamanan dan otomasi rumah apa pun. Mewajibkan pemeriksaan nyata dan bukti, bukan sekadar mengatakan "seharusnya bisa".
---

# Skill: verify (verifikasi)

## Prinsip
Anda bukan pemverifikasi berdasarkan firasat, melainkan berdasarkan BUKTI.
Jangan pernah mengatakan "sudah diperbaiki", "seharusnya berfungsi", atau "saya menemukan masalahnya" tanpa menjalankan pemeriksaan nyata dan melihat hasilnya.
Jika Anda tidak dapat menjalankan sesuatu, katakan begitu dan tandai butir itu sebagai BELUM TERVERIFIKASI.

## Protokol tetap (selalu 6 langkah yang sama, dengan urutan ini)

1. **Tentukan proyek dan profilnya.** Putuskan profil mana yang berlaku (lihat "Profil" di bawah). Jika tidak jelas, tanyakan satu kali saja.
2. **Baca sebelum menyatakan.** Buka berkas dan konfigurasi nyata yang terdampak. Jangan berasumsi isinya. Jika Anda hendak berkata "kesalahannya ada di X", tunjukkan dulu bahwa Anda telah membaca X.
3. **Tetapkan kriteria keberhasilan.** Tulis dalam satu-dua baris apa arti "berfungsi" untuk perubahan ini (mis.: "aplikasi berhasil dibangun dan layar pengaturan terbuka tanpa galat").
4. **Jalankan pemeriksaan profil.** Gunakan perintah nyata (bukan karangan). Jika sebuah perintah gagal, sertakan apa adanya dalam laporan.
5. **Uji kasus yang dulu gagal.** Reproduksi masalah awal dan pastikan masalah itu tidak lagi terjadi. Jika tidak dapat mereproduksinya, katakan.
6. **Laporkan dengan format wajib** (lihat di bawah).

## Format laporan wajib

```
VERIFIKASI
Proyek: <nama/jenis>
Kriteria keberhasilan: <satu baris>

Pemeriksaan yang dijalankan:
- [OK] <perintah atau uji> → <hasil yang relevan>
- [GAGAL] <perintah atau uji> → <galat>
- [BELUM TERVERIFIKASI] <apa yang kurang dan mengapa>

Yang saya baca: <berkas/konfigurasi yang ditinjau>
Kesimpulan: TERVERIFIKASI / SEBAGIAN / BELUM TERVERIFIKASI
Risiko tersisa: <daftar singkat atau "tidak ada yang diketahui">
Langkah berikutnya yang disarankan: <satu baris>
```

Aturan format:
- Tandai TERVERIFIKASI hanya jika SEMUA pemeriksaan kritis berstatus OK.
- Kutip galat secara harfiah dan singkat; jangan meringkasnya dari ingatan.
- Pisahkan yang telah diperiksa dari yang diasumsikan. Asumsi ditulis sebagai "saya kira", jangan pernah sebagai fakta.

## Profil A: aplikasi (Flutter / Android)

Pemeriksaan dari yang paling murah ke paling mahal (berhenti pada yang pertama gagal):
1. `flutter analyze` → tanpa galat (daftar peringatannya).
2. `flutter test` → semua uji lulus. Jika tidak ada uji untuk bagian yang berubah, katakan dan usulkan satu.
3. `flutter build apk --debug` (atau `./gradlew assembleDebug` pada Android native) → berhasil dibangun.
4. Jika ada emulator/perangkat: `flutter run` atau pasang APK dan uji alur yang terdampak.
5. Tinjau `adb logcat` (disaring berdasarkan paket aplikasi) untuk mencari pengecualian saat layar yang diubah dibuka.

Tambahan sesuai kasus:
- Perubahan dependensi: `flutter pub get` tanpa konflik; periksa versi SDK minimum.
- Perubahan izin atau notifikasi: tinjau `AndroidManifest.xml` dan uji pada perangkat dengan Android terbaru.
- Sebelum mengunggah ke Google Play: build rilis yang ditandatangani dan pastikan `versionCode` telah dinaikkan.
- Jika perubahan menyentuh penyimpanan data: uji dengan data yang sudah ada (migrasi), bukan hanya pada instalasi bersih.

## Profil B: keamanan dan otomasi rumah

Aturan emas: **baca-saja dulu**. Jangan ubah konfigurasi produksi tanpa (a) cadangan, (b) konfirmasi eksplisit dari pengguna, dan (c) rencana pemulihan.

Pemeriksaan umum (sesuaikan dengan instalasi nyata; jangan mengira suatu perintah ada, periksa dulu):
- **Docker / docker-compose:** `docker compose config` (memvalidasi sintaks), `docker compose ps` (status), `docker logs --tail 100 <kontainer>` (galat terbaru). Tinjau jalur volume dan port sebelum menjalankan apa pun.
- **Frigate:** validasi berkas konfigurasi dengan alat bawaan Frigate jika ada; pastikan dari log bahwa tiap kamera terhubung; pastikan deteksi aktif dan rekaman tertulis ke disk. Periksa penggunaan CPU jika deteksi berjalan di CPU.
- **Kamera:** pastikan aliran RTSP merespons (misalnya dengan `ffprobe` pada URL, tanpa menampilkan kredensial di laporan). Tinjau anggaran PoE pada switch (jumlah konsumsi kamera dibandingkan daya total).
- **Home Assistant:** jalankan pemeriksaan konfigurasi sebelum memulai ulang; sesudahnya tinjau log untuk galat integrasi.
- **n8n atau otomasi lain:** jalankan alur dalam mode uji dengan data contoh dan periksa keluaran tiap node; pastikan zona waktu dan kredensial.
- **NAS, jaringan, dan UPS:** periksa konektivitas (`ping`, port terbuka), ruang disk yang tersisa, dan status UPS (misalnya dengan `upsc` jika memakai NUT). Setelah pemadaman listrik nyata atau simulasi, pastikan proses shutdown teratur berjalan.
- **Keamanan:** jangan membuka kamera, panel, atau NAS ke internet tanpa autentikasi kuat dan, sebaiknya, VPN. Tinjau port terbuka dan peringatan dari sistem itu sendiri. Jangan pernah memasukkan kata sandi, token, atau URL berisi kredensial ke dalam laporan.

Untuk setiap perubahan jaringan atau firewall: sebelum diterapkan, pastikan Anda tidak akan terkunci dari akses (sediakan jalan kembali).

## Apa yang dilakukan bila tidak dapat diverifikasi
- Katakan dengan tepat apa yang tidak dapat Anda periksa dan mengapa (tidak ada akses ke perangkat, tidak ada jaringan, tidak ada izin).
- Usulkan perintah atau uji konkret yang dapat dijalankan pengguna dan hasil yang diharapkan.
- Tandai kesimpulan sebagai SEBAGIAN atau BELUM TERVERIFIKASI. Jangan mengisi celah dengan asumsi.

## Antipola (dilarang)
- Mengatakan "sekarang sudah berfungsi" tanpa menjalankan apa pun.
- Menyebut penyebab kegagalan tanpa membaca kode atau log yang relevan.
- Mengarang nama perintah, opsi, berkas, atau port. Jika tidak yakin, periksa atau tanyakan.
- Mengubah beberapa hal sekaligus: satu perubahan, satu verifikasi.
- Menyembunyikan pemeriksaan yang gagal dari laporan.

## Pemeliharaan skill ini
Setiap kali ada kesalahan yang lolos (sesuatu yang "terverifikasi" tetapi kemudian gagal), tambahkan di sini pemeriksaan baru yang seharusnya dapat mendeteksinya. Daftar ini membaik melalui kesalahan nyata.
