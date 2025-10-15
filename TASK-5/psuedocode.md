BAŞLA AKILLI_EV_GÜVENLİK_SISTEMI
    
    // =======================================================
    // 1. SİSTEMİN SÜREKLİ ÇALIŞMASI VE DURUM KONTROLÜ
    // =======================================================
    SISTEM_DURUMU = VERITABANI_SISTEM_DURUMU_GETIR() // "AKTIF" veya "PASIF"
    EV_SAHIBI_KONUM = VERITABANI_KONUM_GETIR() // "EVDE" veya "DISARIDA"
    ALARM_CALIYOR_MU = YANLIS

    DONGU DOGRU // Ana döngü: Sistem sürekli çalışır
        
        // ** KONTROL 1: SİSTEM AKTİF Mİ? **
        EGER SISTEM_DURUMU == "AKTIF" ISE
            
            // =======================================================
            // 2. SENSÖR OKUMA DÖNGÜSÜ
            // =======================================================
            TEHLIKE_ALGILANDI = YANLIS
            ALARM_SEVIYESI = 0 // 0:Yok, 1:Düşük, 2:Orta, 3:Yüksek

            // Tüm sensörleri kontrol et
            DONGU TUM_SENSORLER ICIN
                SENSOR_VERISI = SENSOR_OKU(SENSOR)
                
                // ** KONTROL 2: HAREKET SENSÖRÜ **
                EGER SENSOR.TIPI == "HAREKET" VE SENSOR_VERISI == "ALGILANDI" ISE
                    TEHLIKE_ALGILANDI = DOGRU
                    ALARM_SEVIYESI = MAKSIMUM(ALARM_SEVIYESI, 1) // En az Düşük seviye
                BITIR EGER

                // ** KONTROL 3: KAPI/PENCERE SENSÖRÜ **
                EGER SENSOR.TIPI == "GIRIS" VE SENSOR_VERISI == "ACIK" ISE
                    TEHLIKE_ALGILANDI = DOGRU
                    ALARM_SEVIYESI = MAKSIMUM(ALARM_SEVIYESI, 2) // En az Orta seviye
                BITIR EGER
            BITIR DONGU // Tüm sensörlerin kontrolü bitti

            // =======================================================
            // 3. TEHDİT VE YANLIŞ ALARM KONTROLÜ
            // =======================================================
            EGER TEHLIKE_ALGILANDI ISE
                
                // ** KONTROL 4: YANLIŞ ALARM KONTROLÜ (Ev Sahibi Evde Mi?) **
                EGER EV_SAHIBI_KONUM == "EVDE" ISE
                    // Düşük seviye alarmı ev sahibi evdeyken genellikle göz ardı et
                    EGER ALARM_SEVIYESI <= 1 ISE
                        MESAJ_GOSTER("Evdeyken Düşük Seviye Hareket Algılandı. Alarm devreye alınmadı.")
                        DEVAM_ET // Ana döngünün başına dön
                    BITIR EGER
                BITIR EGER
                
                // =======================================================
                // 4. ALARM AKSİYONLARI VE BİLDİRİM
                // =======================================================
                
                EGER ALARM_SEVIYESI >= 2 ISE
                    // Orta veya Yüksek Seviye (Kapı açılması gibi)
                    ALARM_CALIYOR_MU = DOGRU
                    ALARM_SIRENI_CALISMAYI_BASLAT()
                BITIR EGER
                
                // ** KONTROL 5: KAMERA AKTİVASYONU **
                EGER ALARM_SEVIYESI >= 1 ISE
                    KAMERA_VIDEO_KAYDINI_BASLAT()
                BITIR EGER

                // Bildirim Gönderme
                BILDİRİM_GONDER(KULLANICI_TELEFON, SMS)
                BILDİRİM_GONDER(KULLANICI_EMAIL, EMAIL)
                MOBIL_UYGULAMA_PUSH_BILDIRIMI_GONDER()
                
                // =======================================================
                // 5. BEKLEME ve SIFIRLAMA DÖNGÜSÜ
                // =======================================================
                DONGU ALARM_CALIYOR_MU == DOGRU
                    BEKLE(10) // 10 saniye bekle
                    
                    // Alarm Sıfırlama komutu geldi mi kontrol et
                    KULLANICI_KOMUTU = KULLANICIDAN_KOMUT_AL()
                    
                    // ** KONTROL 6: ALARM SIFIRLAMA VEYA DEVAM ETTİRME **
                    EGER KULLANICI_KOMUTU == "SIFIRLA" ISE
                        ALARM_CALIYOR_MU = YANLIS
                        ALARM_SIRENI_CALISMAYI_DURDUR()
                        KAMERA_KAYDINI_KAPAT()
                        MESAJ_GOSTER("Alarm Sıfırlandı.")
                        DONGUYU_BITIR // İç döngüden çık
                    DEGILSE EGER KULLANICI_KOMUTU == "POLIS_CAGIR" ISE
                        POLIS_BIRIMI_BILGILENDIR(EV_ADRESI, ALARM_SEVIYESI)
                        DONGUYU_BITIR // İç döngüden çık
                    BITIR EGER
                BITIR DONGU // Alarm Sıfırlama Döngüsü Bitti

            BITIR EGER // Tehlike Algılandı Kontrolü Bitti

        DEGILSE // SISTEM_DURUMU == "PASIF"
            // Sistem pasif durumdayken sadece bekle
            BEKLE(5)
        BITIR EGER
        
        // Temel sürekli döngüde bekleme
        BEKLE(1) // Yeni sensör okuması için kısa bir bekleme
        
    BITIR DONGU // Ana sonsuz döngü bitti
    
BITIR AKILLI_EV_GÜVENLİK_SISTEMI
