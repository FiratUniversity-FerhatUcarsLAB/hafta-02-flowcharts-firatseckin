BAŞLA E_TICARET_ISLEMLERI

    // =======================================================
    // 1. OTURUM YÖNETİMİ
    // =======================================================
    KULLANICI_OTURUMU = OTURUM_VERISI_KONTROL_ET()
    
    EGER KULLANICI_OTURUMU MEVCUT ISE
        KULLANICI = KULLANICI_OTURUMU
        SEPET = KULLANICI_KAYITLI_SEPETI_GETIR(KULLANICI)
    ISE
        KULLANICI = "MISAFIR"
        SEPET = YENI_SEPET_OLUSTUR()
    BITIR EGER

    // =======================================================
    // 2. ÜRÜN EKLEME ve STOK KONTROLÜ
    // Örnek: Kullanıcı bir ürün eklemek istiyor (URUN_X, 5 Adet)
    // =======================================================
    TALEP_ADET = 5
    URUN_ID = "URUN_X"
    
    STOK_DURUMU = VERITABANI_STOK_KONTROL(URUN_ID)

    // *** KONTROL NOKTASI 1: Yeterli Stok Kontrolü ***
    EGER STOK_DURUMU >= TALEP_ADET ISE
        SEPET = SEPET_URUN_EKLE(SEPET, URUN_ID, TALEP_ADET)
    ISE
        MEVCUT_ADET = STOK_DURUMU
        SEPET = SEPET_URUN_EKLE(SEPET, URUN_ID, MEVCUT_ADET) // Maksimum adedi ekle
        MESAJ_GOSTER("Hata: Stok yetersiz, " + MEVCUT_ADET + " adet eklendi.")
    BITIR EGER

    // Sepet Toplamını Güncelle
    SEPET.TOPLAM_TUTAR = SEPET_TUTAR_HESAPLA(SEPET)

    // =======================================================
    // 3. SEPET GÜNCELLEME ve İNDİRİM UYGULAMA
    // =======================================================
    INDİRİM_KODU = KULLANICIDAN_GELEN_KOD

    // *** KONTROL NOKTASI 2: İndirim Kodu Kontrolü ***
    EGER INDİRİM_KODU MEVCUT ISE
        KOD_DURUMU = INDIRIM_KODU_DOGRULA(INDİRİM_KODU, SEPET.TOPLAM_TUTAR, KULLANICI)
        
        EGER KOD_DURUMU GECERLI ISE
            SEPET.INDIRIM_TUTARI = INDIRIM_ORANI_HESAPLA(INDİRİM_KODU, SEPET.TOPLAM_TUTAR)
        ISE
            SEPET.INDIRIM_TUTARI = 0
            MESAJ_GOSTER("Hata: İndirim kodu geçersiz.")
        BITIR EGER
    BITIR EGER

    SEPET.ARA_TOPLAM = SEPET.TOPLAM_TUTAR - SEPET.INDIRIM_TUTARI

    // =======================================================
    // 4. ADRES ve KARGO HESAPLAMA
    // =======================================================
    TESLIMAT_ADRESI = KULLANICIDAN_GELEN_ADRES

    // *** KONTROL NOKTASI 3: Adres/Bölge Kontrolü ***
    EGER ADRES_GECERLI_MI(TESLIMAT_ADRESI) ISE
        KARGO_UCRETI = EN_UYGUN_KARGO_UCRETI_BUL(SEPET.AGIRLIK, TESLIMAT_ADRESI)
        
        // *** KONTROL NOKTASI 4: Ücretsiz Kargo Kontrolü ***
        EGER SEPET.ARA_TOPLAM >= UCRETSIZ_KARGO_LIMITI ISE
            SEPET.KARGO_UCRETI = 0
        ISE
            SEPET.KARGO_UCRETI = KARGO_UCRETI
        BITIR EGER
    ISE
        SEPET.KARGO_UCRETI = HATA_KARGO_UCRETI
        MESAJ_GOSTER("Hata: Lütfen geçerli bir adres girin.")
    BITIR EGER

    SEPET.ODENECEK_TUTAR = SEPET.ARA_TOPLAM + SEPET.KARGO_UCRETI
    
    // =======================================================
    // 5. ÖDEME İŞLEMİ
    // =======================================================
    ODEME_BILGILERI = KULLANICIDAN_GELEN_KART_BILGISI
    SIPARIS_DURUMU = "BASARISIZ"
    
    // *** KONTROL NOKTASI 5: Son Stok Kontrolü (Ödeme Öncesi) ***
    DONGU SEPET'teki Her Urun Icin
        EGER VERITABANI_STOK_KONTROL(URUN) < URUN.ADET ISE
            MESAJ_GOSTER("Hata: Stok yetersiz. Ödeme engellendi.")
            GERI_DON(0) // Akışı durdur
        BITIR EGER
    BITIR DONGU
    
    // *** KONTROL NOKTASI 6: Ödeme Sağlayıcı İletişimi ***
    EGER ODEME_BILGILERI.YONTEM == "KREDI_KARTI" ISE
        ISLEM_SONUCU = SANAL_POS_ODEME_ISTEGI(SEPET.ODENECEK_TUTAR, ODEME_BILGILERI.KART)
    DEGILSE EGER ODEME_BILGILERI.YONTEM == "HAVALE" ISE
        ISLEM_SONUCU = "BEKLEYEN_ODEME"
    DEGILSE
        ISLEM_SONUCU = "HATA"
    BITIR EGER

    // *** KONTROL NOKTASI 7: Ödeme Başarı Kontrolü ***
    EGER ISLEM_SONUCU == "BASARILI" ISE
        SIPARIS_DURUMU = "ONAYLANDI"
        SIPARIS_ID = VERITABANI_SIPARIS_KAYDET(SEPET, KULLANICI)
        STOK_MIKTARLARINI_GUNCELLE(SEPET)
        KULLANICI_MAIL_GONDER(SIPARIS_ID)
        MESAJ_GOSTER("Siparişiniz başarıyla tamamlandı.")
    DEGILSE EGER ISLEM_SONUCU == "BEKLEYEN_ODEME" ISE
        SIPARIS_DURUMU = "ODEME_BEKLIYOR"
        // ... Bilgilendirme
    ISE
        SIPARIS_DURUMU = "BASARISIZ"
        MESAJ_GOSTER("Hata: Ödeme işlemi başarısız oldu.")
    BITIR EGER
    
BITIR E_TICARET_ISLEMLERI
