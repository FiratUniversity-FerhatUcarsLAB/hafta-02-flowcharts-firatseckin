BAŞLA ATM_PARA_CEKME_SISTEMI
    
    // =======================================================
    // 1. KART VE PIN DOĞRULAMA DÖNGÜSÜ
    // =======================================================
    KART_TAKILDI = OKU("Lütfen kartınızı takınız.")
    PIN_HAKKI = 3
    GIRIS_BASARILI = YANLIS

    DONGU PIN_HAKKI > 0 VE GIRIS_BASARILI == YANLIS
        YAZ("Lütfen PIN şifrenizi giriniz.")
        GIRILEN_PIN = OKU()
        
        EGER GIRILEN_PIN == KARTIN_GERCEK_PINI ISE
            GIRIS_BASARILI = DOGRU
            DONGUYU_BITIR
        ISE
            PIN_HAKKI = PIN_HAKKI - 1
            YAZ("Hatalı PIN. Kalan hakkınız: " + PIN_HAKKI)
        BITIR EGER
    BITIR DONGU

    // *** KONTROL NOKTASI 1: PIN Blokesi ***
    EGER GIRIS_BASARILI == YANLIS ISE
        YAZ("Kartınız 3 hatalı giriş nedeniyle bloke edilmiştir.")
        KART_BLOKE_ET()
        GIT END_SISTEM
    BITIR EGER

    // =======================================================
    // 2. ANA İŞLEM DÖNGÜSÜ
    // =======================================================
    DEVAM_ET = DOGRU
    GUNLUK_CEKILEN_TUTAR = 0
    GUNLUK_LIMIT = KART_LIMITI_GETIR() // Örn: 1000 TL

    DONGU DEVAM_ET == DOGRU
        BAKIYE = BAKIYE_SORGULA()
        YAZ("Mevcut Bakiyeniz: " + BAKIYE)
        
        YAZ("Lütfen çekmek istediğiniz tutarı giriniz.")
        CEKILMEK_ISTENEN_TUTAR = OKU()

        // *** KONTROL NOKTASI 2: Tutar 20 TL Katı Kontrolü ***
        EGER MOD(CEKILMEK_ISTENEN_TUTAR, 20) != 0 ISE
            YAZ("Hata: Çekmek istediğiniz tutar 20 TL'nin katları olmalıdır.")
            DEVAM_ET_KONTROL_SONU
        BITIR EGER

        // *** KONTROL NOKTASI 3: Yetersiz Bakiye Kontrolü ***
        EGER CEKILMEK_ISTENEN_TUTAR > BAKIYE ISE
            YAZ("Hata: Yetersiz bakiye.")
            DEVAM_ET_KONTROL_SONU
        BITIR EGER

        // *** KONTROL NOKTASI 4: Günlük Limit Kontrolü ***
        EGER (GUNLUK_CEKILEN_TUTAR + CEKILMEK_ISTENEN_TUTAR) > GUNLUK_LIMIT ISE
            YAZ("Hata: Günlük limitinizi aşıyorsunuz. Kalan limit: " + (GUNLUK_LIMIT - GUNLUK_CEKILEN_TUTAR))
            DEVAM_ET_KONTROL_SONU
        BITIR EGER

        // =======================================================
        // 3. PARA ÇEKME VE KAYIT
        // =======================================================
        PARA_CEKME_ISLEMI_TAMAMLA(CEKILMEK_ISTENEN_TUTAR)
        GUNLUK_CEKILEN_TUTAR = GUNLUK_CEKILEN_TUTAR + CEKILMEK_ISTENEN_TUTAR
        YAZ("Lütfen paranızı alınız: " + CEKILMEK_ISTENEN_TUTAR)
        FİS_CIKAR()

        // =======================================================
        // 4. BAŞKA İŞLEM SORUSU
        // =======================================================
        DEVAM_ET_KONTROL_SONU: // Kontrol sonrası devam noktası
        YAZ("Başka bir işlem yapmak ister misiniz? (E/H)")
        YANIT = OKU()

        EGER YANIT == "H" ISE
            DEVAM_ET = YANLIS
        BITIR EGER
    BITIR DONGU

    YAZ("Bizi tercih ettiğiniz için teşekkür ederiz. Kartınızı alınız.")

END_SISTEM:
    KART_CİKAR()
    YAZ("İyi günler dileriz.")

BITIR ATM_PARA_CEKME_SISTEMI
