// Üniversite Ders Kayıt Sistemi Pseudocode

İŞLEM DersKayitSistemiBaslat
    
    // 1. Öğrenci Kimlik Doğrulama
    DÖNGÜ:
        GİRİŞ: OgrenciNumarasi, Sifre
        
        EĞER KimlikDogrulamaBasarili(OgrenciNumarasi, Sifre) İSE:
            CIK DÖNGÜDEN // Başarılı giriş
        DEĞİLSE:
            MESAJ_GOSTER("Hata: Öğrenci numarası veya şifre yanlış. Lütfen tekrar deneyin.")
        SON_EĞER
    SON_DÖNGÜ
    
    // Geçerli Öğrencinin Bilgilerini Yükle
    OgrenciBilgisi = OgrenciVerileriniGetir(OgrenciNumarasi)
    
    // 2. Kayıt Dönemi Kontrolü
    EĞER KayitDonemiAcik() İSE:
        MESAJ_GOSTER("Ders kayıt sistemi aktiftir. Hoş geldiniz, " + OgrenciBilgisi.Ad + ".")
    DEĞİLSE:
        MESAJ_GOSTER("Hata: Ders kayıt dönemi şu anda kapalıdır.")
        İŞLEM_SONLANDIR
    SON_EĞER

    // 3. Ders Seçim Arayüzü
    SEÇİLEN_DERSLER_LİSTESİ = BOS_LİSTE
    KONTROL_TAMAM = YANLIŞ
    
    DÖNGÜ_SÜRÜNCE (KONTROL_TAMAM == YANLIŞ):
        
        // a. Mevcut dersleri göster
        DersListesiniGoster(OgrenciBilgisi.Bolum, OgrenciBilgisi.Sinif)
        
        MESAJ_GOSTER("Seçmek istediğiniz dersin kodunu giriniz (Bitirmek için 'TAMAM' yazınız):")
        GİRİŞ: DersKodu

        EĞER DersKodu == "TAMAM" İSE:
            KONTROL_TAMAM = DOĞRU
            ATLA SONRAKİ ADIMA
        SON_EĞER
        
        SEÇİLEN_DERS = DersBilgileriniGetir(DersKodu)

        EĞER SEÇİLEN_DERS BOŞ DEĞİL İSE:
            
            // b. Önkoşul Kontrolü
            EĞER OnkosulKontrolu(OgrenciBilgisi, SEÇİLEN_DERS) İSE:
            
                // c. Kontenjan Kontrolü
                EĞER SEÇİLEN_DERS.Kontenjan > SEÇİLEN_DERS.KayitliOgrenciSayisi İSE:
                    
                    // d. Saat Çakışması Kontrolü
                    EĞER SaatCakismasiYok(SEÇİLEN_DERSLER_LİSTESİ, SEÇİLEN_DERS) İSE:
                        
                        // e. Ders Ekleme
                        LİSTEYE_EKLE(SEÇİLEN_DERSLER_LİSTESİ, SEÇİLEN_DERS)
                        MESAJ_GOSTER(SEÇİLEN_DERS.Ad + " dersi geçici listeye eklendi.")
                        
                    DEĞİLSE:
                        MESAJ_GOSTER("Hata: " + SEÇİLEN_DERS.Ad + " dersi, listedeki başka bir ders ile saat çakışması yaşamaktadır.")
                    SON_EĞER
                    
                DEĞİLSE:
                    MESAJ_GOSTER("Hata: " + SEÇİLEN_DERS.Ad + " dersinin kontenjanı dolmuştur.")
                SON_EĞER
                
            DEĞİLSE:
                MESAJ_GOSTER("Hata: " + SEÇİLEN_DERS.Ad + " dersini alabilmek için gerekli önkoşulları (başarılmış dersler) sağlamıyorsunuz.")
            SON_EĞER
            
        DEĞİLSE:
            MESAJ_GOSTER("Hata: Girilen ders kodu geçersizdir.")
        SON_EĞER
        
    SON_DÖNGÜ
    
    // 4. Seçilen Derslerin Gözden Geçirilmesi
    MESAJ_GOSTER("----- Seçilen Dersleriniz -----")
    LİSTEYİ_GOSTER(SEÇİLEN_DERSLER_LİSTESİ)
    MESAJ_GOSTER("Toplam Kredi: " + ToplamKrediHesapla(SEÇİLEN_DERSLER_LİSTESİ))
    
    // 5. Kayıt Onaylama
    GİRİŞ: Onay (Evet/Hayır)
    
    EĞER Onay == "Evet" İSE:
        
        // Kayıt işlemini veritabanına kaydet
        DERSLERİ_KAYDET(OgrenciNumarasi, SEÇİLEN_DERSLER_LİSTESİ)
        
        // Kontenjan bilgisini güncelle (kayıtlı öğrenci sayısını artır)
        KontenjanGuncelle(SEÇİLEN_DERSLER_LİSTESİ)
        
        MESAJ_GOSTER("Tebrikler! Ders kaydınız başarıyla tamamlanmıştır.")
        
    DEĞİLSE:
        MESAJ_GOSTER("Ders kaydı öğrenci isteğiyle iptal edildi.")
    SON_EĞER
    
    // 6. Sistemi Kapatma
    İŞLEM_SONLANDIR

SON_İŞLEM

// Yardımcı Fonksiyonlar (Ayrıntıları burada gösterilmemiştir)
FONKSİYON KimlikDogrulamaBasarili(Numara, Sifre) -> MANTIKSAL
FONKSİYON OgrenciVerileriniGetir(Numara) -> OgrenciNESNESİ
FONKSİYON KayitDonemiAcik() -> MANTIKSAL
FONKSİYON DersListesiniGoster(Bolum, Sinif)
FONKSİYON DersBilgileriniGetir(DersKodu) -> DersNESNESİ
FONKSİYON OnkosulKontrolu(Ogrenci, Ders) -> MANTIKSAL
FONKSİYON SaatCakismasiYok(MevcutListe, YeniDers) -> MANTIKSAL
FONKSİYON ToplamKrediHesapla(DersListesi) -> SAYISAL
FONKSİYON DERSLERİ_KAYDET(Numara, DersListesi)
FONKSİYON KontenjanGuncelle(DersListesi)
