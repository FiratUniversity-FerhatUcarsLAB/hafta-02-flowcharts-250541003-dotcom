FONKSIYON ATM_Para_Cekme_Islemi()
BASLA

    // 1. Kart Girişi ve Başlangıç
    EKRANA_YAZ("Lütfen kartınızı takın.")
    KART = KART_OKU()

    EĞER KART_GECERLI_MI(KART) İSE
        EKRANA_YAZ("Lütfen 4 haneli şifrenizi girin.")
        PIN_GIRIS_HAKKI = 3

        TEKRARLA
            PIN_GIRIS = KULLANICIDAN_GIRIS_AL()
            EĞER PIN_DOGRU_MU(KART, PIN_GIRIS) İSE
                DONGUDEN_CIK
            DEĞİLSE
                PIN_GIRIS_HAKKI = PIN_GIRIS_HAKKI - 1
                EĞER PIN_GIRIS_HAKKI > 0 İSE
                    EKRANA_YAZ("Hatalı şifre. Kalan hak: " + PIN_GIRIS_HAKKI)
                DEĞİLSE
                    EKRANA_YAZ("Çok fazla hatalı giriş. Kartınız bloke edildi.")
                    KARTI_BLOKE_ET(KART)
                    KARTI_IADE_ET(KART)
                    BITIR // İşlemi sonlandır
                SON_EĞER
            SON_EĞER
        PIN_GIRIS_HAKKI > 0 İKEN

        // 2. İşlem Seçimi
        EKRANA_YAZ("Lütfen yapmak istediğiniz işlemi seçin:")
        EKRANA_YAZ("1: Para Çekme, 2: Bakiye Sorgulama, 3: Diğer İşlemler")
        ISLEM_SECIM = KULLANICIDAN_GIRIS_AL()

        EĞER ISLEM_SECIM == 1 İSE
            // 3. Hesap Seçimi (Birden fazla hesap varsa)
            HESAP = HESAP_SEC(KART) // Örneğin, Vadesiz, Tasarruf, Kredi Kartı

            // 4. Tutar Girişi
            EKRANA_YAZ("Lütfen çekmek istediğiniz tutarı girin (TL).")
            CEKILECEK_TUTAR = KULLANICIDAN_GIRIS_AL()

            // 5. Geçerlilik Kontrolleri
            BAKIYE = HESAP_BAKIYESINI_SORGULA(HESAP)
            ATM_LIMIT = ATM_GUNLUK_CEKIM_LIMITINI_AL()
            ATM_NAKIT_DURUMU = ATM_KASA_DURUMUNU_SORGULA()

            EĞER CEKILECEK_TUTAR > BAKIYE İSE
                EKRANA_YAZ("Yetersiz bakiye.")
            DEĞİLSE EĞER CEKILECEK_TUTAR > ATM_LIMIT İSE
                EKRANA_YAZ("İstenen tutar günlük/tek işlem limitini aşıyor.")
            DEĞİLSE EĞER CEKILECEK_TUTAR > ATM_NAKIT_DURUMU İSE
                EKRANA_YAZ("ATM'de yeterli nakit bulunmamaktadır.")
            DEĞİLSE EĞER CEKILECEK_TUTAR MOD ATM_PARA_KATLARI != 0 İSE // Örn: 10, 20, 50
                EKRANA_YAZ("İstenen tutar ATM'nin verebileceği banknot katlarında değildir.")
            DEĞİLSE
                // 6. İşlemi Gerçekleştirme
                EKRANA_YAZ("İşleminiz gerçekleştiriliyor, lütfen bekleyin...")

                // Veritabanı İşlemleri: Bakiyeyi Güncelleme
                YENI_BAKIYE = BAKIYE - CEKILECEK_TUTAR
                HESAP_BAKIYESINI_GUNCELLE(HESAP, YENI_BAKIYE)

                // ATM İşlemleri: Nakiti Verme
                NAKITI_VER(CEKILECEK_TUTAR)

                // ATM Kasa Durumunu Güncelle
                ATM_KASA_DURUMUNU_GUNCELLE(CEKILECEK_TUTAR)

                // 7. Sonuç
                EKRANA_YAZ("Paranızı alabilirsiniz. Yeni bakiyeniz: " + YENI_BAKIYE)
                EKRANA_YAZ("Makbuz ister misiniz? (E/H)")
                MAKBUZ_SECIM = KULLANICIDAN_GIRIS_AL()
                EĞER MAKBUZ_SECIM == 'E' İSE
                    MAKBUZ_BAS(CEKILECEK_TUTAR, YENI_BAKIYE, ISLEM_TARIHI)
                SON_EĞER
            SON_EĞER

        DEĞİLSE EĞER ISLEM_SECIM == 2 İSE
            // Bakiye sorgulama işlemi...
            EKRANA_YAZ("Mevcut bakiyeniz: " + HESAP_BAKIYESINI_SORGULA(HESAP))
        DEĞİLSE
            EKRANA_YAZ("Geçersiz işlem seçimi.")
        SON_EĞER

        // 8. Kart İadesi ve Oturum Sonu
        KARTI_IADE_ET(KART)
        EKRANA_YAZ("İyi günler dileriz.")

    DEĞİLSE
        // Kart okunamadı veya geçerli değil
        EKRANA_YAZ("Kartınız okunamadı veya geçerli bir kart değil.")
        KARTI_IADE_ET(KART)
    SON_EĞER

BITIR
