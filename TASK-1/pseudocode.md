FONKSİYON ATM_ParaCekme_Islemi():

  EKRANA_YAZ("Kartınızı takınız...")
  kart = KART_OKUYUCU_OKU()

  EĞER kart GEÇERSİZSE:
    EKRANA_YAZ("Geçersiz kart. Lütfen tekrar deneyin.")
    KART_IADE_ET()
    ÇIKIŞ

  // PIN doğrulama
  denemeSayısı = 0
  DOĞRULANDI = FALSE

  TEKRAR PIN doğrulanana kadar VEYA denemeSayısı < MAX_PIN_DENEMESİ:
    girilenPIN = KULLANICIDAN_AL("PIN giriniz:")
    EĞER PIN_DOGRULA(kart, girilenPIN):
      DOĞRULANDI = TRUE
      DUR
    DEĞİLSE:
      denemeSayısı = denemeSayısı + 1
      EKRANA_YAZ("Hatalı PIN. Kalan deneme: " + (MAX_PIN_DENEMESİ - denemeSayısı))

  EĞER DOĞRULANDI == FALSE:
    EKRANA_YAZ("3 kez hatalı PIN. Kart bloke edildi.")
    KART_BLOKE_ET(kart)
    KART_IADE_ET()
    ÇIKIŞ

  // Hesap seçimi
  hesap = KULLANICIDAN_AL("İşlem yapmak istediğiniz hesabı seçin:")

  // Menü
  işlem = KULLANICIDAN_AL("1. Para Çekme\n2. Bakiye Sorgulama\n3. Çıkış")

  EĞER işlem == 1:
    PARA_CEKME_ISLEMI(hesap)

  EĞER işlem == 2:
    bakiye = HESAP_BAKİYE_GETIR(hesap)
    EKRANA_YAZ("Bakiyeniz: " + bakiye + " TL")

  EĞER işlem == 3:
    EKRANA_YAZ("İyi günler.")
    KART_IADE_ET()
    ÇIKIŞ

  KART_IADE_ET()


