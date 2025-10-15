fonksiyon PINiDogrula(kart, girilenPIN):
  kayitliHash = kartIcinPinHashGetir(kart)
  geriDondur hash(girilenPIN) == kayitliHash

fonksiyon ParaDagitimPlaniniHesapla(nakitEnvanteri, miktar):
  // Açgözlü algoritma: büyükten küçüğe sayarak, envanteri dikkate al
  kalan = miktar
  plan = boş harita (banknot -> adet)

  için banknot in NOMINASYONLAR: // büyükten küçüğe sırayla
    gerekenMax = taban(kalan / banknot)
    mevcut = nakitEnvanteri[banknot] // yoksa 0 say
    kullanilacak = min(gerekenMax, mevcut)
    eğer kullanilacak > 0 ise:
      plan[banknot] = kullanilacak
      kalan = kalan - (kullanilacak * banknot)

  eğer kalan == 0:
    geriDondur plan

  // Açgözlü algoritma başarısızsa, daha derin (backtracking / dinamik programlama) dene
  eğer AlternatifDagitimBul(nakitEnvanteri, miktar, plan):
    geriDondur bulunanPlan
  aksi halde:
    geriDondur BASARISIZ

fonksiyon AlternatifDagitimBul(nakitEnvanteri, miktar, mevcutPlan):
  // Sınırlandırılmış backtracking: sınırlı sayıda kombinasyonu dener
  // Uygun bir dağılım bulunursa, mevcutPlan doldurulur ve true döner
  ...

fonksiyon BankadanTahsilatIstegi(gonderenHesapId, miktar):
  // Banka API çağrısı: tahsilat (debit)
  cevap = BankaAPI.tahsilEt(gonderenHesapId, miktar)
  eğer cevap.durum == "TAMAM":
    geriDondur cevap.islemId
  aksi halde:
    geriDondur BASARISIZ

fonksiyon NakitDagit(dagitimPlani):
  // Donanımla iletişim: her banknot türü için dağıtım komutları gönder
  için banknot, adet in dagitimPlani:
    sonuc = NakitMekanizmasi.dagit(banknot, adet)
    eğer sonuc != TAMAM:
      // Hangi banknotun verilip verilmediğini raporla
      geriDondur false
  geriDondur true

fonksiyon BasarisizDagitimTelafisi(hesap, islemId, dagitimPlani):
  // Bankadan para tahsil edilmişse, geri ödeme (reversal) istenir
  geriOdeme = BankaAPI.geriAl(islemId)
  eğer geriOdeme.durum == "TAMAM":
    OlayKaydiEkle("GERI_ODEME_TAMAM", hesap.id, islemId)
  aksi halde:
    // Geri ödeme başarısızsa, müşteri hizmetleri bilgilendirilir
    SorunKaydiOlustur(hesap.id, islemId, dagitimPlani)

