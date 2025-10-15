BASLA SÜREKLİ_CALISMA

    // 1. Sensör Verilerini Oku ve Güncelle
    HareketAlgilandi = SENSÖR_VERISI_OKU(HareketSensorleri)
    KapiPencereAcik = SENSÖR_VERISI_OKU(KapiPencereSensorleri)
    DumanAlgilandi = SENSÖR_VERISI_OKU(DumanSensorleri)
    KullaniciGirdisi = KULLANICI_ARAYÜZÜNDEN_KOMUT_AL()

    // 2. Kullanıcı Komutlarını İşle
    EĞER KullaniciGirdisi BOŞ DEĞİLSE
        KOMUT = KullaniciGirdisi.KOMUT()
        PARAMETRE = KullaniciGirdisi.PARAMETRE()

        EĞER KOMUT EŞİTTİR "KUR" VE PARAMETRE EŞİTTİR Sifre İSE
            SistemDurumu = "DEVREDE"
            KULLANICIYA_BILDIRIM_GONDER("Güvenlik sistemi kuruldu.")

        YOKSA EĞER KOMUT EŞİTTİR "KALDIR" VE PARAMETRE EŞİTTİR Sifre İSE
            SistemDurumu = "DEVRE_DISI"
            ALARM_KAPAT()
            KULLANICIYA_BILDIRIM_GONDER("Güvenlik sistemi devre dışı bırakıldı.")

        YOKSA EĞER KOMUT EŞİTTİR "ŞİFRE_DEĞİŞTİR" VE PARAMETRE.ESKI EŞİTTİR Sifre İSE
            Sifre = PARAMETRE.YENI
            KULLANICIYA_BILDIRIM_GONDER("Şifre başarıyla değiştirildi.")

        YOKSA EĞER KOMUT EŞİTTİR "KUR" VEYA "KALDIR" İSE
            KULLANICIYA_BILDIRIM_GONDER("Hatalı şifre. İşlem başarısız.")
        SON_EĞER
    SON_EĞER

    // 3. Güvenlik İhlali Kontrolü
    EĞER SistemDurumu EŞİTTİR "DEVREDE" İSE
        EĞER HareketAlgilandi VEYA KapiPencereAcik İSE
            TEHLIKE_YONETIMI("İZİNSİZ_GİRİŞ")
        SON_EĞER

    YOKSA EĞER SistemDurumu EŞİTTİR "DEVRE_DISI" İSE
        // Devre dışı iken sadece Yangın/Duman izlenir
        EĞER DumanAlgilandi İSE
            TEHLIKE_YONETIMI("YANGIN_TEHLİKESİ")
        SON_EĞER
    SON_EĞER

    // 4. Sistem Durumu Kontrolü (Alarm Durumunda Tekrarlı Bildirim)
    EĞER SistemDurumu EŞİTTİR "ALARM" İSE
        BEKLE(5 saniye) // Alarm çalarken bekleme
        ACIL_DURUM_BILDIRIM_GÖNDER() // Güvenlik şirketi/polise bildirim
    SON_EĞER

    BEKLE(1 saniye) // Kontrol döngüsü arasındaki bekleme süresi

SON_SÜREKLİ_CALISMA

#### Fonksiyonlar (Alt Programlar)

```pseudocode
FONKSIYON TEHLIKE_YONETIMI(TehlikeTipi)
    YAZ("--- TEHLİKE TESPİT EDİLDİ: " + TehlikeTipi + " ---")
    SistemDurumu = "ALARM"
    ALARM_SES_CAL() // Sireni etkinleştir
    KAMERA_KAYIT_BASLAT() // Kaydı yüksek çözünürlükte başlat
    KULLANICIYA_BILDIRIM_GONDER(TehlikeTipi + " algılandı!")
    ACIL_DURUM_BILDIRIM_GÖNDER() // Acil servislere otomatik bildirim
SON_FONKSIYON

FONKSIYON KULLANICIYA_BILDIRIM_GONDER(Mesaj)
    // Mobil uygulama veya e-posta ile bildirim gönderme simülasyonu
    YAZ("Kullanıcı Bildirimi: " + Mesaj)
SON_FONKSIYON

FONKSIYON ACIL_DURUM_BILDIRIM_GÖNDER()
    // Güvenlik şirketi veya itfaiye/polise otomatik çağrı/mesaj gönderme
    YAZ("Acil durum hizmetlerine otomatik bildirim gönderiliyor...")
SON_FONKSIYON

FONKSIYON ALARM_SES_CAL()
    YAZ("YÜKSEK SESLİ ALARM ÇALIYOR!")
SON_FONKSIYON

FONKSIYON ALARM_KAPAT()
    // Alarm sesini durdur
    // Kamera kaydını durdur (isteğe bağlı)
    YAZ("Alarm sesi kapatıldı.")
SON_FONKSIYON

FONKSIYON SENSÖR_VERISI_OKU(SensorTuru)
    // Gerçek sensörden veri okuma işlemini simüle eder
    // Burası, donanımdan (örneğin IoT modülü) gelen verinin okunduğu yerdir.
    DÖN (Rastgele bir DOĞRU veya YANLIŞ değer, test amaçlı)
SON_FONKSIYON

FONKSIYON KULLANICI_ARAYÜZÜNDEN_KOMUT_AL()
    // Mobil uygulama, tuş takımı vb. arayüzden gelen komutu alır.
    // Örn: "KUR:1234", "KALDIR:1234"
    DÖN (Gelen komut veya BOŞ)
SON_FONKSIYON
