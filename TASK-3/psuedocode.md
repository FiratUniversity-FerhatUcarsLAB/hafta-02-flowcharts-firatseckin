BAŞLA RANDEVU_SISTEMI

    // =======================================================
    // 1. KULLANICI GİRİŞİ / KİMLİK DOĞRULAMA
    // =======================================================
    KIMLIK_NO = KULLANICIDAN_AL("TC Kimlik Numarası")
    SIFRE = KULLANICIDAN_AL("Şifre")

    KIMLIK_DURUMU = VERITABANI_KIMLIK_DOGRULA(KIMLIK_NO, SIFRE)

    EGER KIMLIK_DURUMU GECERLI DEGIL ISE
        MESAJ_GOSTER("Kimlik doğrulama başarısız. Lütfen bilgileri kontrol edin.")
        GERI_DON(0) // Sistemi sonlandır
    BITIR EGER

    // =======================================================
    // 2. POLİKLİNİK SEÇİMİ
    // =======================================================
    POLIKLINIK_LISTESI = VERITABANI_POLIKLINIK_GETIR()
    KULLANICI_POLIKLINIK_SECIMI = KULLANICIDAN_LISTEDEN_SEC(POLIKLINIK_LISTESI)

    // =======================================================
    // 3. DOKTOR LİSTESİ GÖRÜNTÜLEME
    // =======================================================
    DOKTOR_LISTESI = VERITABANI_DOKTOR_GETIR(KULLANICI_POLIKLINIK_SECIMI)
    KULLANICI_DOKTOR_SECIMI = KULLANICIDAN_LISTEDEN_SEC(DOKTOR_LISTESI)

    // =======================================================
    // 4. UYGUN SAATLERİ GÖSTERME (TARİH VE ZAMAN KONTROLÜ)
    // =======================================================
    SECILEN_TARIH = KULLANICIDAN_TARIH_AL("Randevu Tarihi")
    
    // Doktorun çalışma saatlerini ve mevcut randevularını getir
    DOLU_SAATLER = VERITABANI_DOLU_RANDEVU_GETIR(KULLANICI_DOKTOR_SECIMI, SECILEN_TARIH)
    TUM_SAATLER = DOKTOR_CALISMA_SAATLERI_GETIR(KULLANICI_DOKTOR_SECIMI)

    UYGUN_SAATLER = FARK_KUMESI_AL(TUM_SAATLER, DOLU_SAATLER)
    
    EGER UYGUN_SAATLER BOS ISE
        MESAJ_GOSTER("Bu tarihte uygun saat bulunmamaktadır. Lütfen başka bir tarih seçin.")
        GERI_DON(0) // Sistemi sonlandır veya başa döndür
    BITIR EGER

    KULLANICI_SAAT_SECIMI = KULLANICIDAN_LISTEDEN_SEC(UYGUN_SAATLER)

    // =======================================================
    // 5. RANDEVU ONAYLAMA VE KAYIT
    // =======================================================
    RAND_KIMLIK = YENI_RANDEVU_ID_OLUSTUR()
    ONAY_KODU = RASTGELE_KOD_URET(6)
    
    RANDEVU_KAYDI_BILGILERI = {
        "RandevuID": RAND_KIMLIK,
        "TC_Kimlik": KIMLIK_NO,
        "DoktorID": KULLANICI_DOKTOR_SECIMI,
        "TarihSaat": SECILEN_TARIH + KULLANICI_SAAT_SECIMI,
        "Durum": "ONAY BEKLİYOR"
    }

    // Randevuyu veritabanına kaydet
    VERITABANI_RANDEVU_KAYDET(RANDEVU_KAYDI_BILGILERI)
    
    // =======================================================
    // 6. SMS GÖNDERME
    // =======================================================
    HASTA_TELEFON = VERITABANI_TELEFON_GETIR(KIMLIK_NO)
    
    SMS_METNI = "Randevunuz oluşturulmuştur.\n" + \
                "Tarih: " + SECILEN_TARIH + " Saat: " + KULLANICI_SAAT_SECIMI + "\n" + \
                "Doktor: " + DOKTOR_ADI_GETIR(KULLANICI_DOKTOR_SECIMI) + "\n" + \
                "Onay Kodu: " + ONAY_KODU
                
    SMS_GONDER(HASTA_TELEFON, SMS_METNI)
    
    // =======================================================
    // 7. SİSTEM BİTİŞİ
    // =======================================================
    MESAJ_GOSTER("Randevunuz başarıyla oluşturuldu ve SMS gönderildi.")

BITIR RANDEVU_SISTEMI
