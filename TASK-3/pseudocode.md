// ** VERİ YAPILARI (Global Tanımlar) **

YAPI HASTA
    ID: TamSayı
    Ad: Metin
    Soyad: Metin
    TCKimlikNo: Metin
    Sifre: Metin
SON_YAPI

YAPI DOKTOR
    ID: TamSayı
    Ad: Metin
    Soyad: Metin
    UzmanlikAlani: Metin // Örn: Dahiliye, Kardiyoloji
    CalismaSaatleri: Liste<TarihSaat> // Haftalık veya günlük müsait saatler
SON_YAPI

YAPI RANDEVU
    ID: TamSayı
    HastaID: TamSayı
    DoktorID: TamSayı
    TarihSaat: TarihSaat
SON_YAPI

// Veritabanı yerine geçecek statik listeler
HASTALAR: Liste<HASTA>
DOKTORLAR: Liste<DOKTOR>
RANDEVULAR: Liste<RANDEVU> = BOŞ_LİSTE

// Oturum değişkeni
GirisYapmisKullaniciID: TamSayı = -1

// ** FONKSİYONLAR VE PROSEDÜRLER **

// PROSEDÜR 1: Kullanıcı Girişi
PROSEDÜR Giris_Yap()
BASLA
    EKRANA_YAZ("Lütfen TC Kimlik Numaranızı ve şifrenizi girin.")
    TCKNO_GIRIS = KULLANICIDAN_GIRIS_AL()
    SIFRE_GIRIS = KULLANICIDAN_GIRIS_AL()

    BULUNAN_HASTA = HASTALAR.BUL_KOSUL(h -> h.TCKimlikNo == TCKNO_GIRIS)

    EĞER BULUNAN_HASTA != BOŞ İSE
        EĞER BULUNAN_HASTA.Sifre == SIFRE_GIRIS İSE
            GirisYapmisKullaniciID = BULUNAN_HASTA.ID
            EKRANA_YAZ("Başarıyla giriş yapıldı. Hoş geldiniz, " + BULUNAN_HASTA.Ad + "!")
        DEĞİLSE
            EKRANA_YAZ("Hatalı şifre.")
        SON_EĞER
    DEĞİLSE
        EKRANA_YAZ("Kullanıcı bulunamadı.")
    SON_EĞER
BITIR

// PROSEDÜR 2: Randevu Alma
PROSEDÜR Randevu_Al()
BASLA
    EĞER GirisYapmisKullaniciID == -1 İSE
        EKRANA_YAZ("Lütfen önce giriş yapın.")
        GERI_DON
    SON_EĞER

    // Adım 1: Uzmanlık Alanı Seçimi
    EKRANA_YAZ("Lütfen randevu almak istediğiniz uzmanlık alanını seçin.")
    UZMANLIK_ALANI_GIRIS = KULLANICIDAN_GIRIS_AL()

    // Adım 2: Uygun Doktorları Listele
    UYGUN_DOKTORLAR = DOKTORLAR.FILTRELE(d -> d.UzmanlikAlani == UZMANLIK_ALANI_GIRIS)
    EĞER UYGUN_DOKTORLAR.BOYUT == 0 İSE
        EKRANA_YAZ("Bu alanda uygun doktor bulunmamaktadır.")
        GERI_DON
    SON_EĞER
    EKRANA_YAZ("Uygun Doktorlar:")
    HER_BIR DOKTOR IÇIN UYGUN_DOKTORLAR'DA TEKRARLA
        EKRANA_YAZ("ID: " + DOKTOR.ID + ", Ad: " + DOKTOR.Ad + " " + DOKTOR.Soyad)
    SON_TEKRARLA

    // Adım 3: Doktor Seçimi
    EKRANA_YAZ("Lütfen bir doktor ID'si girin.")
    DOKTOR_ID_SECIM = KULLANICIDAN_GIRIS_AL()
    SECILEN_DOKTOR = UYGUN_DOKTORLAR.BUL_KOSUL(d -> d.ID == DOKTOR_ID_SECIM)

    EĞER SECILEN_DOKTOR != BOŞ İSE
        // Adım 4: Müsait Saatleri Listele
        MUS_SAATLER = Mevcut_Musait_Randevu_Saatlerini_Bul(DOKTOR_ID_SECIM)
        EKRANA_YAZ("Lütfen aşağıdaki müsait saatlerden birini seçin:")
        HER_BIR SAAT IÇIN MUS_SAATLER'DE TEKRARLA
            EKRANA_YAZ(SAAT.FORMAT_TARIH_SAAT())
        SON_TEKRARLA

        // Adım 5: Saat Seçimi
        EKRANA_YAZ("Lütfen randevu saatini (GG.AA.YYYY SS:DD) girin.")
        SECILEN_TARIH_SAAT = KULLANICIDAN_GIRIS_AL()
        
        // Adım 6: Randevu müsait mi kontrol et ve oluştur
        EĞER Randevu_Musait_Mi(DOKTOR_ID_SECIM, SECILEN_TARIH_SAAT) İSE
            YENI_RANDEVU = YENI RANDEVU()
            YENI_RANDEVU.ID = RANDEVULAR.SON_ID() + 1
            YENI_RANDEVU.HastaID = GirisYapmisKullaniciID
            YENI_RANDEVU.DoktorID = DOKTOR_ID_SECIM
            YENI_RANDEVU.TarihSaat = SECILEN_TARIH_SAAT
            
            RANDEVULAR.EKLE(YENI_RANDEVU)
            EKRANA_YAZ("Randevunuz başarıyla oluşturuldu.")
            EKRANA_YAZ("Randevu Detayı: " + SECILEN_DOKTOR.Ad + " " + SECILEN_DOKTOR.Soyad + " ile " + SECILEN_TARIH_SAAT.FORMAT_TARIH_SAAT())
        DEĞİLSE
            EKRANA_YAZ("Bu saat dolu. Lütfen başka bir saat seçin.")
        SON_EĞER
    DEĞİLSE
        EKRANA_YAZ("Geçersiz doktor ID'si.")
    SON_EĞER
BITIR

// FONKSİYON 3: Randevunun Müsait Olup Olmadığını Kontrol Etme
FONKSIYON Randevu_Musait_Mi(DoktorID: TamSayı, TarihSaat: TarihSaat) : Mantıksal
BASLA
    // 1. Doktorun çalışma saatlerinde mi?
    SECILEN_DOKTOR = DOKTORLAR.BUL_KOSUL(d -> d.ID == DoktorID)
    EĞER SECILEN_DOKTOR.CalismaSaatleri.ICERIYOR_MU(TarihSaat) DEĞİLSE
        GERI_DON YANLIŞ
    SON_EĞER
    
    // 2. Bu saatte başka bir randevu var mı?
    BULUNAN_RANDEVU = RANDEVULAR.BUL_KOSUL(r -> r.DoktorID == DoktorID VE r.TarihSaat == TarihSaat)
    EĞER BULUNAN_RANDEVU != BOŞ İSE
        GERI_DON YANLIŞ
    SON_EĞER

    GERI_DON DOĞRU
BITIR

// FONKSİYON 4: Bir Doktorun Müsait Saatlerini Bulma
FONKSIYON Mevcut_Musait_Randevu_Saatlerini_Bul(DoktorID: TamSayı) : Liste<TarihSaat>
BASLA
    SECILEN_DOKTOR = DOKTORLAR.BUL_KOSUL(d -> d.ID == DoktorID)
    MUS_SAATLER = BOŞ_LİSTE
    
    EĞER SECILEN_DOKTOR != BOŞ İSE
        // Doktorun tüm çalışma saatlerini gez
        HER_BIR SAAT IÇIN SECILEN_DOKTOR.CalismaSaatleri'DE TEKRARLA
            // Eğer bu saatte randevu yoksa
            EĞER RANDEVULAR.BUL_KOSUL(r -> r.DoktorID == DoktorID VE r.TarihSaat == SAAT) == BOŞ İSE
                MUS_SAATLER.EKLE(SAAT)
            SON_EĞER
        SON_TEKRARLA
    SON_EĞER
    GERI_DON MUS_SAATLER
BITIR

// PROSEDÜR 5: Randevuları Görüntüleme
PROSEDÜR Randevularimi_Goruntule()
BASLA
    EĞER GirisYapmisKullaniciID == -1 İSE
        EKRANA_YAZ("Lütfen önce giriş yapın.")
        GERI_DON
    SON_EĞER

    HASTA_RANDEVULARI = RANDEVULAR.FILTRELE(r -> r.HastaID == GirisYapmisKullaniciID)
    
    EĞER HASTA_RANDEVULARI.BOYUT == 0 İSE
        EKRANA_YAZ("Yaklaşan bir randevunuz bulunmamaktadır.")
        GERI_DON
    SON_EĞER

    EKRANA_YAZ("--- Yaklaşan Randevularınız ---")
    HER_BIR RANDEVU IÇIN HASTA_RANDEVULARI'NDA TEKRARLA
        DOKTOR_AD = DOKTORLAR.BUL_KOSUL(d -> d.ID == RANDEVU.DoktorID).Ad
        DOKTOR_SOYAD = DOKTORLAR.BUL_KOSUL(d -> d.ID == RANDEVU.DoktorID).Soyad
        EKRANA_YAZ("Randevu ID: " + RANDEVU.ID +
                   " | Doktor: " + DOKTOR_AD + " " + DOKTOR_SOYAD + 
                   " | Tarih: " + RANDEVU.TarihSaat.FORMAT_TARIH_SAAT())
    SON_TEKRARLA
BITIR

// PROSEDÜR 6: Randevu İptal Etme
PROSEDÜR Randevu_Iptal_Et()
BASLA
    EĞER GirisYapmisKullaniciID == -1 İSE
        EKRANA_YAZ("Lütfen önce giriş yapın.")
        GERI_DON
    SON_EĞER

    Randevularimi_Goruntule() // Önce mevcut randevuları göster
    EKRANA_YAZ("Lütfen iptal etmek istediğiniz randevunun ID'sini girin.")
    IPTAL_EDILECEK_ID = KULLANICIDAN_GIRIS_AL()

    BULUNAN_RANDEVU = RANDEVULAR.BUL_KOSUL(r -> r.ID == IPTAL_EDILECEK_ID)

    EĞER BULUNAN_RANDEVU != BOŞ İSE
        EĞER BULUNAN_RANDEVU.HastaID == GirisYapmisKullaniciID İSE // Randevu sahibini kontrol et
            RANDEVULAR.KALDIR(BULUNAN_RANDEVU)
            EKRANA_YAZ("Randevunuz başarıyla iptal edilmiştir.")
        DEĞİLSE
            EKRANA_YAZ("Bu randevuyu iptal etme yetkiniz yoktur.")
        SON_EĞER
    DEĞİLSE
        EKRANA_YAZ("Geçersiz randevu ID'si.")
    SON_EĞER
BITIR

// ** ANA PROGRAM AKIŞI **

PROGRAM_CALIS()
BASLA
    // Sistemin açılışında veri tabanı simülasyonu
    // Örnek Hasta ve Doktor verileri oluştur
    // Bu kısım gerçek bir uygulamada veritabanından veri çekecektir.

    TEKRARLA
        EKRANA_YAZ("--- ANA MENÜ ---")
        EKRANA_YAZ("1: Giriş Yap")
        EKRANA_YAZ("2: Randevu Al")
        EKRANA_YAZ("3: Randevularımı Görüntüle")
        EKRANA_YAZ("4: Randevu İptal Et")
        EKRANA_YAZ("5: Çıkış")
        SECIM = KULLANICIDAN_GIRIS_AL()

        EĞER SECIM == 1 İSE
            Giris_Yap()
        DEĞİLSE EĞER SECIM == 2 İSE
            Randevu_Al()
        DEĞİLSE EĞER SECIM == 3 İSE
            Randevularimi_Goruntule()
        DEĞİLSE EĞER SECIM == 4 İSE
            Randevu_Iptal_Et()
        DEĞİLSE EĞER SECIM == 5 İSE
            EKRANA_YAZ("Sistemden çıkılıyor. İyi günler dileriz.")
            DONGUDEN_CIK
        DEĞİLSE
            EKRANA_YAZ("Geçersiz seçim. Lütfen tekrar deneyin.")
        SON_EĞER
    SON_TEKRARLA
BITIR
