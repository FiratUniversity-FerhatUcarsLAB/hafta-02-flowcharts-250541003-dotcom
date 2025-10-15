MAIN_LOOP:
  while ATM is operational:
    display "Kartınızı takınız"
    card = waitForCardInsertion()
    if card == null:
      continue

    if not validateCard(card):
      ejectCard(card)
      display "Geçersiz kart"
      continue

    session = createSession(card)
    session.startTime = currentTime()

    // 1) PIN doğrulama
    pinRetries = 0
    authenticated = false
    while pinRetries < MAX_PIN_RETRIES and not authenticated:
      pin = promptUser("Lütfen PIN giriniz:")
      if verifyPIN(card, pin):
        authenticated = true
      else:
        pinRetries += 1
        display "Hatalı PIN. Kalan deneme: " + (MAX_PIN_RETRIES - pinRetries)
    if not authenticated:
      blockCardOrAccount(card)
      ejectCard(card)
      display "Kart bloke edildi. Banka ile iletişime geçiniz."
      logEvent("PIN_FAILED", card.id, session.id)
      continue

    // 2) Hesap seçimi
    accounts = fetchAccountsForCard(card)
    selectedAccount = promptAccountSelection(accounts)
    if selectedAccount == null:
      ejectCard(card)
      continue

    // 3) Menü: Para çekme
    action = promptMenu(["Para Çekme", "Bakiye Sorgulama", "Para Yatırma", "İptal"])
    if action == "İptal":
      ejectCard(card)
      continue

    if action == "Para Çekme":
      handleWithdrawalFlow(session, selectedAccount)

    ejectCard(card)
    display "İşlem tamamlandı. Kartınızı alınız."
