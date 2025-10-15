BAŞLA DERS_KAYIT_SISTEMI

    // =======================================================
    // 1. ÖĞRENCİ GİRİŞİ ve HAZIRLIK
    // =======================================================
    OGRENCI_NO = KULLANICIDAN_AL("Öğrenci Numarası")
    SIFRE = KULLANICIDAN_AL("Şifre")

    KULLANICI = VERITABANI_KIMLIK_DOGRULA(OGRENCI_NO, SIFRE)

    EGER KULLANICI GECERLI DEGIL ISE
        MESAJ_GOSTER("Hata: Geçersiz Öğrenci No veya Şifre.")
        GERI_DON(0) // Sistemi sonlandır
    BITIR EGER

    // Öğrenci bilgilerini çek
    OGRENCI_GPA = KULLANICI.GPA
    KAYITLI_DERSLER = KULLANICI.KAYITLI_DERSLER // Daha önce kesinleşmiş dersler
    
    // Geçici kayıt sepetini ve krediyi başlat
    SECILEN_DERSLER_SEPETI = BOS_LISTE
    TOPLAM_KREDI = 0
    KREDI_LIMITI = 35

    // =======================================================
    // 2. DERS EKLEME/ÇIKARMA DÖNGÜSÜ
    // =======================================================
    DONGU Kullanıcı Kayıt İşlemine Devam Ettiği Sürece
        DERS_LISTESI = VERITABANI_TUM_DERSLERI_GETIR()
        DERS_LISTESI_GOSTER(DERS_LISTESI)

        ISLEM_TURU = KULLANICIDAN_SEC("EKLE, ÇIKAR veya ONAYLA")

        EGER ISLEM_TURU == "ONAYLA" ISE
            DONGUYU_BITIR
        BITIR EGER

        DERS_SECIMI = KULLANICIDAN_AL("İşlem Yapılacak Ders Kodu")

        // =======================================================
        // 3. DERS EKLEME İŞLEMİ VE ÇOKLU KONTROLLER
        // =======================================================
        EGER ISLEM_TURU == "EKLE" ISE
            DERS = VERITABANI_DERS_BILGISI_GETIR(DERS_SECIMI)
            
            // --- Kontrol Mekanizmaları ---

            // A. KONTROL: Kontenjan Kontrolü
            EGER DERS.KONTENJAN > DERS.KAYITLI_OGRENCI_SAYISI ISE
                
                // B. KONTROL: Ön Koşul Kontrolü
                EGER DERS_ON_KOSUL_GEREKLI_MI(DERS.ON_KOSUL_KODU, KAYITLI_DERSLER) ISE
                    
                    // C. KONTROL: Kredi Limiti Kontrolü
                    EGER (TOPLAM_KREDI + DERS.KREDI) <= KREDI_LIMITI ISE
                        
                        // D. KONTROL: Zaman Çakışması Kontrolü
                        EGER ZAMAN_CAKISMASI_VAR_MI(DERS.ZAMAN, SECILEN_DERSLER_SEPETI) ISE
                            
                            // Tüm kontroller başarılı: Ders Sepete Eklenebilir
                            SECILEN_DERSLER_SEPETI.EKLE(DERS)
                            TOPLAM_KREDI = TOPLAM_KREDI + DERS.KREDI
                            MESAJ_GOSTER(DERS_SECIMI + " başarıyla sepete eklendi.")
                            
                        ISE
                            MESAJ_GOSTER("Hata: " + DERS_SECIMI + " seçilen başka bir ders ile zaman çakışıyor.")
                        BITIR EGER
                    ISE
                        MESAJ_GOSTER("Hata: Kredi limitini (" + KREDI_LIMITI + ") aşıyorsunuz.")
                    BITIR EGER
                ISE
                    MESAJ_GOSTER("Hata: " + DERS_SECIMI + " dersinin ön koşulunu sağlamıyorsunuz.")
                BITIR EGER
            ISE
                MESAJ_GOSTER("Hata: " + DERS_SECIMI + " dersinin kontenjanı dolmuştur.")
            BITIR EGER
        
        // =======================================================
        // 4. DERS ÇIKARMA İŞLEMİ
        // =======================================================
        DEGILSE EGER ISLEM_TURU == "ÇIKAR" ISE
            EGER DERS_SECIMI SECILEN_DERSLER_SEPETI ICINDE ISE
                DERS = SEPETTEN_DERS_BUL(DERS_SECIMI)
                SECILEN_DERSLER_SEPETI.ÇIKAR(DERS)
                TOPLAM_KREDI = TOPLAM_KREDI - DERS.KREDI
                MESAJ_GOSTER(DERS_SECIMI + " başarıyla sepetten çıkarıldı.")
            ISE
                MESAJ_GOSTER("Hata: Sepetinizde böyle bir ders bulunmamaktadır.")
            BITIR EGER
        BITIR EGER
    BITIR DONGU

    // =======================================================
    // 5. KAYIT ÖZETİ, DANIŞMAN ONAYI VE KESİNLEŞTİRME
    // =======================================================
    KAYIT_OZETI_GOSTER(SECILEN_DERSLER_SEPETI, TOPLAM_KREDI)
    
    DANISMAN_ONAYI_GEREKLI = YANLIS
    
    // E. KONTROL: Danışman Onayı Koşulu
    EGER OGRENCI_GPA < 2.5 ISE
        DANISMAN_ONAYI_GEREKLI = DOGRU
        MESAJ_GOSTER("GPA 2.5 altında olduğu için danışman onayı bekleniyor.")
        
        DONGU DANISMAN_ONAYI_ALININCAYA_KADAR
            BEKLE(60) // Onay için bekle
            ONAY_DURUMU = VERITABANI_DANISMAN_ONAY_KONTROL(OGRENCI_NO)
            EGER ONAY_DURUMU == "ONAYLANDI" ISE
                DONGUYU_BITIR
            DEGILSE EGER ONAY_DURUMU == "REDDEDILDI" ISE
                MESAJ_GOSTER("Kayıt Danışman Tarafından Reddedildi. Lütfen tekrar düzenleyin.")
                GERI_DON(0) // Veya 2. Adıma geri döndür
            BITIR EGER
        BITIR DONGU
    BITIR EGER

    // Kayıt Kesinleştirme
    VERITABANI_DERS_KAYIT_KESINLESTIR(OGRENCI_NO, SECILEN_DERSLER_SEPETI)
    MESAJ_GOSTER("Ders kaydınız başarıyla tamamlanmıştır.")

BITIR DERS_KAYIT_SISTEMI
