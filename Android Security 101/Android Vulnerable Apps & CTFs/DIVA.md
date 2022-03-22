# [DIVA App](https://github.com/payatu/diva-android)
İçerisinde toplamda 13 adet güvenli veya yanlış kodlama üzerine örnek bulunmkatadır. Geliştiricilerin yaptığı veya yapabileceği yanlış kodlama uygulama içerisinde zafiyet oluşturabilmektedir.

# **Insecure Data Storage
## 1. Part 1

İlk bölümde bizden bir kullanıcı adı ve şifre girip onun nereye nasıl kaydedildiğini bulmamız isteniyor. Kullanıcı adı ve şifre kısmına, *third_party_username,third_party_password* girdim.
![diva1.png](../../images/diva1.png)

Bilgileri kaydettikten sonra `jadx` aracını kullanarak apk dosyasını okunabilir olarak açabiliyoruz. Kodu incelememizin amacı bilgilerin nasıl kaydedildiğini anlayabilmek.
Uygulamamızı açtıktan sonra MainActivity kısmına gelerek ilgili sınıfı bulmamız gerekiyor
![diva2.png](../../images/diva2.png)

InsecureDataStorage1Activity üzerine gelerek çift tıklıyoruz ve ilgili sınıf açılıyor. içerisinde  `saveCredentials()` isimli fonksiyon bulunmakta. Burada yapılan işlem kullanıcıdan alınan bilgilerin açık metin olarak SharedPreferences yapısını kullanarak kadyedilmesidir.
![diva3.png](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/images/diva3.png?raw=true)
Kod incelememiz burada tamamlanıyor ve `adb shell` komutu ile cihaz içerisinde shell açabiliyoruz ve uygulamanın oluşturduğu dosyaları inceleyebiliyoruz. uygulama paket dizinine gidiyoruz. 
![diva4.png](../../images/diva6.png)
shared_prefs klasörü içerisinde girilen bilgileri içeren xml dosyası bulunmaktadır. Dosyanın içeriğine bakıldığında açık metin olarak kullanıcı bilgilerine kolay bir şekilde erişilebilmektedir. 
![diva5.png](../../images/diva5.png)

## Part 2
Birinci kısıma benzer şekilde kullanıcı adı ve şifre isteniyor. *username_part2, password_part2* olarak girdim.  `saveCredentials()` fonksiyonunda kullanıcı bilgileri sql veritabanı içerisine açık metin olarak kaydedilmektedir.
![diva7.png](../../images/diva7.png)
`adb shell` komutu ile cihaza erişim sağlanır ve `/data/data` dizini içerisinde database klasörüne gidilmektedir. Çünkü bilgiler veritabanına kaydediliyor. Oluşturulan veritabanı ismi ids2.
![diva8.png](../../images/diva8.png)
shell üzerinden dosyayı açtığımızda tam okunabilir olmuyor ama girilen bilgiler görüntülenebilmektedir.
![diva9.png](../../images/diva9.png)
Dosyayı `adb pull` komutu ile  yerel cihaza yükleyip uygun program ile açabiliriz.
![diva10.png](../../images/diva10.png)

## Part 3

## Part 4
