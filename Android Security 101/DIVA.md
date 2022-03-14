# [DIVA App](https://github.com/payatu/diva-android)
İçerisinde toplamda 13 adet güvenli veya yanlış kodlama üzerine örnek bulunmkatadır. Geliştiricilerin yaptığı veya yapabileceği yanlış kodlama uygulama içerisinde zafiyet oluşturabilmektedir.

# **Insecure Data Storage
## 1. Part 1

İlk bölümde bizden bir kullanıcı adı ve şifre girip onun nereye nasıl kaydedildiğini bulmamız isteniyor.
![diva1.png](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/images/diva1.png?raw=true)

Bilgileri kaydettikten sonra `jadx` aracını kullanarak apk dosyasını okunabilir olarak açabiliyoruz. Kodu incelememizin amacı bilgilerin nasıl kaydedildiğini anlayabilmek.
Uygulamamızı açtıktan sonra MainActivity kısmına gelerek ilgili sınıfı bulmamız gerekiyor
![diva2.png](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/images/diva2.png?raw=true)

InsecureDataStorage1Activity üzerine gelerek çift tıklıyoruz ve ilgili sınıf açılıyor. içerisinde  `saveCredentials()` isimli fonksiyon bulunmakta. Burada yapılan işlem kullanıcıdan alınan bilgilerin açık metin olarak SharedPreferences yapısını kullanarak kadyedilmesidir.
![diva3.png](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/images/diva3.png?raw=true)
Kod incelememiz burada tamamlanıyor ve `adb shell` komutu ile cihaz içerisinde shell açabiliyoruz ve uygulamanın oluşturduğu dosyaları inceleyebiliyoruz. uygulama paket dizinine gidiyoruz. 
![diva4.png](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/images/diva6.png?raw=true)
shared_prefs klasörü içerisinde girilen bilgileri içeren xml dosyası bulunmaktadır. Dosyanın içeriğine bakıldığında açık metin olarak kullanıcı bilgilerine kolay bir şekilde erişilebilmektedir. 
![diva5.png](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/images/diva5.png?raw=true)

## Part 2


## Part 3

## Part 4
