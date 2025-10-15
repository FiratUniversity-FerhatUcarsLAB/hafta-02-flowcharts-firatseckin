BAŞLA

  // Ana Menü
  1. Görüntüle: "Hastane Bilgi Sistemine Hoş Geldiniz"
  2. Görüntüle: "1. Randevu Alma"
  3. Görüntüle: "2. Tahlil Sonuçları Görüntüleme"
  4. Görüntüle: "3. Çıkış"
  5. Kullanıcıdan bir seçenek girmesini iste (secim)

  // Kullanıcı Seçimine Göre Yönlendirme
  EĞER secim == 1 İSE
    GİT Modul_Randevu_Alma
  DEĞİLSE EĞER secim == 2 İSE
    GİT Modul_Tahlil_Sonuclari
  DEĞİLSE EĞER secim == 3 İSE
    GİT Cikis
  DEĞİLSE
    Görüntüle: "Geçersiz seçim, lütfen tekrar deneyin."
    GİT BAŞLA
  SON EĞER

  // Modül 1: Randevu Alma
  Modul_Randevu_Alma:
    1. Görüntüle: "Randevu Alma Modülü"
    2. Kullanıcıdan T.C. Kimlik Numarası ve şifre iste
    3. Kimlik Doğrulama:
       EĞER kimlik bilgileri doğru İSE
         4. Poliklinik listesini göster
         5. Kullanıcıdan poliklinik seçmesini iste
         6. Seçilen polikliniğe ait doktor listesini göster
         7. Kullanıcıdan doktor seçmesini iste
         8. Seçilen doktora ait uygun randevu saatlerini göster
         9. Kullanıcıdan bir saat seçmesini iste
         10. Randevu bilgilerini (poliklinik, doktor, tarih, saat) onaya sun
         11. Kullanıcıdan onayı al (Evet/Hayır)
         12. EĞER kullanıcı onaylarsa İSE
              13. Randevuyu kaydet
              14. SMS ile randevu bilgilerini gönder
              15. Görüntüle: "Randevunuz başarıyla oluşturulmuştur."
            DEĞİLSE
              16. Görüntüle: "Randevu işlemi iptal edildi."
            SON EĞER
       DEĞİLSE
         17. Görüntüle: "Kimlik doğrulama başarısız."
       SON EĞER
    18. GİT BAŞLA // İşlem bitince ana menüye dön

  // Modül 2: Tahlil Sonuçları
  Modul_Tahlil_Sonuclari:
    1. Görüntüle: "Tahlil Sonuçları Görüntüleme Modülü"
    2. Kullanıcıdan T.C. Kimlik Numarası ve barkod numarası iste
    3. Kimlik Doğrulama:
       EĞER kimlik bilgileri doğru İSE
         4. Tahlilin varlığını kontrol et
         5. EĞER tahlil mevcut İSE
              6. Tahlil sonucunun hazır olup olmadığını kontrol et
              7. EĞER sonuç hazır İSE
                   8. Görüntüle: "Tahlil sonuçlarınız aşağıdadır."
                   9. Sonuçları ekranda göster
                   10. "PDF olarak indir" seçeneği sun
                   11. EĞER kullanıcı indirmek isterse İSE
                        12. Sonuçları PDF olarak indir
                      SON EĞER
                 DEĞİLSE
                   13. Görüntüle: "Tahlil sonuçlarınız henüz hazır değil. Lütfen daha sonra tekrar deneyin."
                 SON EĞER
            DEĞİLSE
              14. Görüntüle: "Bu bilgilere ait bir tahlil bulunamadı."
            SON EĞER
       DEĞİLSE
         15. Görüntüle: "Kimlik doğrulama başarısız."
       SON EĞER
    16. GİT BAŞLA // İşlem bitince ana menüye dön

  // Çıkış
  Cikis:
    1. Görüntüle: "Sistemden çıkılıyor..."

SON
