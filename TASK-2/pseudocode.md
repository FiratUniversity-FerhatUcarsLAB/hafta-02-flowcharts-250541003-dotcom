// ** VERİ YAPILARI (Global/Sistem Tanımları) **
YAPI URUN
    ID: TamSayı
    Ad: Metin
    Fiyat: OndalıkSayı // Tek birim fiyatı
SON_YAPI

YAPI SEPET_KALEMI
    UrunID: TamSayı
    Miktar: TamSayı
    BirimFiyat: OndalıkSayı // O anki fiyatı kaydeder
SON_YAPI

// Sepet, SEPET_KALEMI nesnelerinden oluşan bir liste/dizi olacaktır.
SEPET: Liste<SEPET_KALEMI> = BOŞ_LİSTE
URUN_KATALOGU: Sözlük<TamSayı, URUN> = ... // Sistemdeki tüm ürünlerin listesi (ID -> URUN)

// ** FONKSİYONLAR VE PROSEDÜRLER **

// PROSEDÜR 1: Ürünü Sepete Ekleme
PROSEDÜR Sepete_Urun_Ekle(UrunID: TamSayı, Miktar: TamSayı)
BASLA
    // 1. Ürünün katalogda varlığını kontrol et
    EĞER URUN_KATALOGU.ANAHTAR_VAR_MI(UrunID) İSE
        SECILEN_URUN = URUN_KATALOGU.AL(UrunID)

        // 2. Sepette zaten var mı kontrol et
        KALEM_BULUNDU = YANLIŞ
        HER_BIR KALEM IÇIN SEPET'TE TEKRARLA
            EĞER KALEM.UrunID == UrunID İSE
                // 3. Varsa miktarı güncelle
                KALEM.Miktar = KALEM.Miktar + Miktar
                KALEM_BULUNDU = DOĞRU
                CIK
            SON_EĞER
        SON_TEKRARLA

        EĞER KALEM_BULUNDU == YANLIŞ İSE
            // 4. Yoksa yeni sepet kalemi oluştur ve ekle
            YENI_KALEM = YENI SEPET_KALEMI()
            YENI_KALEM.UrunID = UrunID
            YENI_KALEM.Miktar = Miktar
            YENI_KALEM.BirimFiyat = SECILEN_URUN.Fiyat // Fiyatı anlık olarak kaydet
            SEPET.EKLE(YENI_KALEM)
        SON_EĞER

        EKRANA_YAZ(SECILEN_URUN.Ad + " ürünü sepetinize başarıyla eklendi/güncellendi. Yeni miktar: " + YENI_KALEM.Miktar)
    DEĞİLSE
        EKRANA_YAZ("HATA: Belirtilen ID'ye sahip ürün katalogda bulunamadı.")
    SON_EĞER
BITIR

// PROSEDÜR 2: Sepetteki Ürünün Miktarını Güncelleme
PROSEDÜR Miktari_Guncelle(UrunID: TamSayı, YeniMiktar: TamSayı)
BASLA
    EĞER YeniMiktar <= 0 İSE
        // Miktar 0 veya altına inerse ürünü sepetten çıkar
        Sepetten_Urun_Cikar(UrunID)
        GERI_DON
    SON_EĞER

    KALEM_BULUNDU = YANLIŞ
    HER_BIR KALEM IÇIN SEPET'TE TEKRARLA
        EĞER KALEM.UrunID == UrunID İSE
            // 1. Miktarı doğrudan yeni değerle değiştir
            KALEM.Miktar = YeniMiktar
            KALEM_BULUNDU = DOĞRU
            EKRANA_YAZ("Ürün miktarı başarıyla " + YeniMiktar + " olarak güncellendi.")
            CIK
        SON_EĞER
    SON_TEKRARLA

    EĞER KALEM_BULUNDU == YANLIŞ İSE
        EKRANA_YAZ("HATA: Belirtilen ID'ye sahip ürün sepetinizde bulunmamaktadır.")
    SON_EĞER
BITIR

// PROSEDÜR 3: Sepetten Ürün Çıkarma (Silme)
PROSEDÜR Sepetten_Urun_Cikar(UrunID: TamSayı)
BASLA
    KALDIRILACAK_INDEKS = -1
    INDEKS = 0

    // 1. Çıkarılacak ürünün indeksini bul
    HER_BIR KALEM IÇIN SEPET'TE TEKRARLA
        EĞER KALEM.UrunID == UrunID İSE
            KALDIRILACAK_INDEKS = INDEKS
            CIK
        SON_EĞER
        INDEKS = INDEKS + 1
    SON_TEKRARLA

    // 2. Ürünü listeden çıkar
    EĞER KALDIRILACAK_INDEKS != -1 İSE
        SEPET.KALDIR_INDEKS(KALDIRILACAK_INDEKS)
        EKRANA_YAZ("Ürün sepetinizden başarıyla kaldırıldı.")
    DEĞİLSE
        EKRANA_YAZ("HATA: Belirtilen ID'ye sahip ürün sepetinizde bulunmamaktadır.")
    SON_EĞER
BITIR

// FONKSİYON 4: Sepet Toplam Tutarını Hesaplama
FONKSİYON Sepet_Toplam_Tutari_Hesapla() : OndalıkSayı
BASLA
    GENEL_TOPLAM = 0.0

    EĞER SEPET.BOYUT == 0 İSE
        EKRANA_YAZ("Sepetiniz boş.")
        GERI_DON 0.0
    SON_EĞER

    // 1. Her bir kalem için Tutar = Miktar * Birim Fiyat hesapla
    HER_BIR KALEM IÇIN SEPET'TE TEKRARLA
        KALEM_TOPLAM = KALEM.Miktar * KALEM.BirimFiyat
        GENEL_TOPLAM = GENEL_TOPLAM + KALEM_TOPLAM
    SON_TEKRARLA

    // 2. (Opsiyonel: KDV, İndirim vb. ekle/çıkar)
    // INDIRIM_ORANI = 0.10
    // GENEL_TOPLAM = GENEL_TOPLAM * (1 - INDIRIM_ORANI)
    
    GERI_DON GENEL_TOPLAM
BITIR

// PROSEDÜR 5: Sepet İçeriğini Görüntüleme
PROSEDÜR Sepeti_Goster()
BASLA
    EKRANA_YAZ("--- ALIŞVERİŞ SEPETİNİZ ---")
    EĞER SEPET.BOYUT == 0 İSE
        EKRANA_YAZ("Sepetinizde ürün bulunmamaktadır.")
        GERI_DON
    SON_EĞER

    HER_BIR KALEM IÇIN SEPET'TE TEKRARLA
        // Ürün adını katalogdan al
        URUN_AD = URUN_KATALOGU.AL(KALEM.UrunID).Ad
        
        // Kalem Toplamını hesapla
        KALEM_TOPLAM = KALEM.Miktar * KALEM.BirimFiyat

        // Görüntüle
        EKRANA_YAZ("Ürün: " + URUN_AD + 
                   " | Miktar: " + KALEM.Miktar + 
                   " | Birim Fiyat: " + KALEM.BirimFiyat + 
                   " TL | Ara Toplam: " + KALEM_TOPLAM + " TL")
    SON_TEKRARLA
    
    // Genel toplamı göster
    EKRANA_YAZ("-----------------------------")
    EKRANA_YAZ("GENEL TOPLAM: " + Sepet_Toplam_Tutari_Hesapla() + " TL")
    EKRANA_YAZ("-----------------------------")
BITIR
