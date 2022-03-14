# 0. İçindekiler

1. [Android Mimarisi](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#1-android-mimarisi)
2. [Android Uygulama ve APK](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#2-android-uygulama-temelleri-apk)
3. [Android Analiz Ortamı ve Araçlar](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#3-android-analiz-ortam%C4%B1-ve-ara%C3%A7lar)
4. [OWASP Mobile Top 10](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#4-owasp-mobile-top-10)
5. [Android Uygulama Güvenliği](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#5-android-uygulama-g%C3%BCvenli%C4%9Fi)
6. [Android Saldırı Yüzeyi](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#6--android-sald%C4%B1r%C4%B1-y%C3%BCzeyi)
7. [Android Malware Türleri](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#7-android-malware-t%C3%BCrleri)
8. [Diğer Konular](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#8-other-topics)
9. [Kaynak](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/AndroidSec.md#kaynak)

---
# 1. Android Mimarisi
- Android işletim sistemi içerisinde çok kullanıcılı linux sistemidir. Her bir uygulamanın linux kullanıcı idye sahip. 
- Sistem dosyalar için gerekli izinleri vermektedir.Böylece, sadece id ye sahip kullanıcılar erişebilmektedir.
- Her processin kendine ait sanal makinesi bulunmaktadır. Bu sayede, her uygulama birbirinden izole ortamlarda çalışır.

Android işletim sistemi katmanlarından bahsedecek olursaki bunlar,
\- *Sistem Uygulamaları*: içerisinde bulunan varsayılan uygulamaları içerir
\- *Native C/C++ Kütüphaneleri*: c ve c++ ile kodları ve gerekli kütüphaneleri içerir
\- *JAVA API Framework*: içerisinde uygulamaların çalışmasını sağlayacak bileşenler bulunur
\- *ART*: Android RunTime, Dalvik sanal makineyi içerir ve dex dosyalarını çalıştırır.
\- *HAL*: Hardware Abstraction Layer, uygulamanın direkt olarak donanımla iletişimini engelleyen arabirimdir.
\- *Linux kernel*: ses,klavye,bluetooth,kamera gibi sürücüleri içerir ve hafıza yönetimi yapılmaktadır

**Android Güvenliği**

Android güvenliği farklı güvenlik katmanlarında sağlanır, yani gizlilik(confidentiality), bütünlük(integrity) veya erişilebilirlik(availability) ilkelerinin tek bir önleme bağlı değildir.
Kabaca 4 başlıkta incelenir,
- System-wide security
- Software isolation
- Network Security
- Anti-exploitation


---
# 2. Android Uygulama temelleri, APK

Android APK içeriği kabaca,

> AndroidManifest.xml

> META-INF/ 
	> - sertifikalar

> classes.dex
	> - uygulama dex dosyası. Çalışan kodlar burada bulunur

> lib/
	> - uygulama için kullanılan yerel kütüphaneler burada bulunur.

> assets/
	> - uygulama için gerekli diğer dosyalar
	> - ek kütüphaneler
	> - dex dosyaları

**Dalvik vs Smali**
Uygulama geliştirilirken Java veya Kotlin kodları DEX(Dalvik executable) bytecode compile edilir yani dex formata dönüştürülür.
Tersine mühendislik işlemlerinde ise DEX bytecode önce SMALİ yani Dalvik bytecodeun okunabilir hali, bytecode ile high level programlama dili arasındadır assembly gibi.


**Manifest Dosyası**
Uygulama çalışmadan önce sistem hangi bileşenlerin var olduğunu AndroidManifest.xml dosyasından okumaktadır. Manifest içerisinde,
- Kullanıcı izinleri, internet erişimi,kamera erişimi gibi izinler tanımlanmaktadır.
- En düşük API seviyesi belirtilmektedir
- yazılım ve donanımsal özellikler tanımlanır, kamera veya bluetooth servisi gibi
- Google Maps gibi gerekli API kütüphaneleri tanımlanır.
- <*manifest*> elementi paket ismi ve uygulama idsini içerir
-  Uygulama bileşenleri 4'e ayrılır.
	- <*activity*> -> Activity
	- <*service*> -> Service
	- <*receiver*> -> Broadcast receivers
	- <*provider*> -> Content providers
elementleri altsınıflardır.

**Uygulama Entrypointleri**
*Launcher Activity(Activity Başlangıcı)* uygulamanın açılmasıyla beraber başlamaktadır
*Services(Servisler)* Kullanıcı arayüzüne gerek duymaksızın arkaplanda çalışmaktadır. Intentler aracılığıyla uygulamanın entrypointi olabilir.
*Broadcast Receiver(Alıcı)* kısaca dinleyiciler denilebilir.

**APK**
Apk(Android Application Pack) sıkıştırılmış bir klasör olarak açılabilir.

`unzip -d application.apk`

**AndroidManifest.xml**
İçerisinde uygulama ile ilgili bilgileri içerir. Bunlar, 
- paket ismi,
- idsi,
- ikonu, 
- kullanıcı izinleri,
- uygulama bileşenleri - Activity,Service,Receiver,Provider
gibi bilgilerdir.

1. Activity
	- entry point
	- uygulama içerisinde birden çok olabilir
2. Service
	- arka planda gerçekleşen aksiyonlar kullanıcının ne yaptığından bağımsız çalışır
3. Receiver
	- sistem çapında eventlere cevap verir
	- sistem eventleri başka bi uygulamaya - çalışmıyorsa bile - iletebilir

4. Provider
	- verilere erişebilen high-level api, diğer uygulama ver servisler ile etkileşim halindedir
	- genellikle sqlite veritabanı 

5. Intent
	- uygulama bileşenleri arasında gerçekleşen iletişim gerçekleşir

APK dosyaları Play Store, GetAPK, GETJAR, F-Froid, Apkpure, Aptoide gibi platformlardan yüklenebilmektedir. Aynı zamanda adb aracı kullanarak da cihaza uygulama yüklenebilir.

`adb install /path/app.apk`
ilgili paketi kaldırmak için

`adb uninstall package.name`
yüklü paketlerin listesini 

`adb shell pm list packages`
komutu ile listelenmektedir.


---
# 3. Android analiz ortamı ve araçlar

Android kontrol listesi içerisinde, 
- Android temelleri, 
- Statik analiz, 
- Dinamik analiz, ve 
- diğer konular ,
şeklinde 4 başlık altında toplanmıştır

Konuya girmeden önce genel olarak mobil uygulama penetration test aşamalarından bahsedilebilir. Bu adımlar temelde

Reconnaissance ---> Statik  ---> Dinamik  ---> Raporlama


**Keşif** aşamasında, uygulama hakkında bilgi toplanır, uygulama versiyonları ve yapılan güncellemeler incelenebilir, geliştiricileri araştırlır.

**Statik Analiz**, uygulama kodları statik analiz araçları kullanılarak incelenir. hatalı güvenlik konfigürasyonları, stringler kontrol edilebilir. API url bağlantıları, SQL veritabanı, değişkenler, yerel depolama alanları incelenebilir.

**Dinamik Analiz**, uygulamanın çalıştırıldığı ve manipüle edildiği aşamadır. Analiz süresince, yapılan ağ trafiği incelenir, uygulama dump alınır, çalışma zamanında oluşturulan dosyalar kontrol edilir, ssl pinning gibi işlemler gerçekleştirilebilir.

Analiz aşamasında bakılması gerekn noktalar,
- API çağrıları
- Uygulama Entrypointleri
- Obfuscate edilmiş metotları decrypt etmek

Bazı yaygın kullanılan Android araçları

> **`apktool`:** APK dosyalarının  içeriğini çıkartarak okunabilir duruma getirebilir. `apktool d - o output_folder app.apk` komutu APK sıkıştırılmış dosyasını açmak için kullanılır.

>  **`dex2jar`:**  APK içerisindeki DEX dosyalarını analiz edilebilir JAR dosyalarına çevirir ve Java analiz araçlarından biridir. Kali Linux beraberinde gelir ve `d2j-dex2jar -o output.jar app.apk` komutu çalıştırılır.

>   **`Anbox`:** sanallaştırma veya emülatör ortamı oluştururak dinamik sürece yardımcı olur

>  **`Android Studio`:** Android geliştirme uygulaması olarak kullanılır. Analiz ve debug süreçlerinde fayda sağlar.

_Tasarım_ --> _Kod_ --> _Derleme_ --> _Çıktı_

Uygulama geliştirme aşamaları --->
Tersine mühendislik işlemleri 	 <---

AndroidManifest.xml ve resource.arsc ==Decoder== yardımı ile
classes.dex ==Disassembler/Decompiler== ile okunabilir duruma getirilebilir.

Statik Analiz Araçları bazıları,
>  [axmlprinter](https://github.com/rednaga/axmlprinter) 
	>  AndroidManifest.xml **decode**
	
>[apktool](https://ibotpeaches.github.io/Apktool/) 
	>AndroidManifest ve resources **decode**
	>classes.dex **disassemble** SMALI

>[Jadx](https://github.com/skylot/jadx)
	>AndroidManifest ve resources **decode**
	>classes.dex **decode** Java code

>[Bytecode Viewer](https://bytecodeviewer.com/)
	>AndroidManifest ve resources **decode**
	>classes.dex **decompile** Java code

- [ ] #todo ==Tools kısmı düzenle==

Android uygulama pentesting için kullanılan Santoku ISO isimli  açık kaynak dağıtım bulunmaktadır.
Emülatör olarak Android studio ve Genymotion genellikle tercih edilmektedir. 

**adb**
Android Debug Bridge, sunucu-istemci programıdır. İstemci komut satırında adb aracını kullanarak cihaz ile iletişim kurmaktadır.
Bazı adb komutları,
>adb devices : bağlı chiazları listeler

> adb tcpip 5555 : telefonu 5555 portundan TCP bağlantısı gerçekleştirir.

> adb connect <\ telefonIP > : <\ port > Girilen adresdeki cihaz bağlantısını gerçekleştirir

> adb shell : bağlantısı kurulmuş cihazdan shell alınır

> adb root : root yetkilerinde çalışmak için


**Frida**
- [ ] [frida tutoriallarını](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/frida-tutorial) inceliyordum

- [ ] TODO:Android Analiz araçları düzenlenecek

**Drozer**

https://github.com/FSecureLABS/drozer

Android cihaz içerisine native uygulama olarak yüklenip Dalvik sanal makine ile iletişim kurarak cihazdaki uygulamalar için zafiyet taraması gerçekleştirir. Sunucu ve ajan uygulaması bulunur. Sunucu konsol aracılığıyla ajan ile iletişimi gerçekleştirir. Python ile geliştirilmiş frameworktür.


---

# 4. OWASP MOBILE TOP 10
OWASP'ın en son 2016 yılında yayınladığı popüler 10 adet mobil uygulama güvenlik zafiyetleri listesi:

1. Improper Platform Usage (Hatalı Platform Kullanımı)
2. Insecure Data Storage (Güvensiz Veri Depolama)
3. Insecure Communication (Güvensiz İletişim)
4. Insecure Authentication (Güvenli Olmayan Kimlik Doğrulama)
5. Insufficient Cryptography (Yetersiz Kriptografi)
6. Insecure Authorization (Güvensiz Yekilendirme)
7. Client Code Quality (İstemci Kod Kalitesi)
8. Code Tampering (Kod Kurcalama)
9. Reverse Engineering (Tersine Mühendislik)
10.Extraneous Functionality (Yabancı İşlevsellik)

## 1. Improper Plaform Usage

## 2. Insecure Data Storage (Güvensiz Veri Depolama)
Mobil cihaz içerisindeki dosya kalsörlerine erişim kolayca gerçekleşebilir. Cihazın bilgileri kaydettiği klasöre de erişim çeşitli yollardan gerçekleşebilir. Bu durumlarda uygulamanın statik analazini yapan ve hassas veriyi bulan birçok araç bulunmaktadır. (Örneğin [StaCoAn](https://github.com/vincentcox/StaCoAn), uygulamadaki API anahtarlarını, URL bağlantıları, hardcoded metinleri tespit edebilmektedir. Yaygın kullanılan araçlardan bir diğeri [apkleaks](https://github.com/dwisiswant0/apkleaks), APK dosyasını tarayarak içerisindeki bağlantıları, URI ve hassas bilgileri tespit edebilir.) Bu güvenlik açığı, kimlik hırsızlığına, gizlilik ihlaline yol açabilir.Bu veriler nerede açığa çıkabilir,
- SQL veritabanlarında,
- Log dosyalarında,
- Çerezlerde,
- SD Card,
- Bulut hizmetleri,
gibi birçok yerde güvensiz bir şekilde depolanabilir.
Korunma yöntemleri:


## 3. 
---


# 5. Android Uygulama Güvenliği

Android güvenli uygulama geliştirme ve güvenliği üzerine olası zafiyetleri konu almış ve güvenli olmayan zayıf kodlamaya değinilen bazı uygulamalar geliştirilmiştir

>- [DIVA](https://github.com/payatu/diva-android)
>- [Insecure Shop](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/InsecureShop.md)
>- [OWASP MSTG Hacking Playground](https://github.com/OWASP/MSTG-Hacking-Playground)
>- [UnsafeBank](https://github.com/lucideus-repo/UnSAFE_Bank)
>- [InjuredAndroid](https://github.com/B3nac/InjuredAndroid)
>- [Android Application Security](https://github.com/RavikumarRamesh/hpAndro1337)
>- [AndroGoat](https://github.com/satishpatnayak/AndroGoat)
>- [OWASP MSTG crackmes](https://github.com/OWASP/owasp-mstg/tree/master/Crackmes/Android)


uygulamaları örnek olarak verilebilir. Bu uygulamalarda yer alan başlıklar temel olarak
- Insecure Logging
- Hardcoding strings
- Insecure data storage
- Input validation problem
- Access Control
- Arbitrary Code Execution
- Insecure Broadcast receiver
- Insecure content provider
- Insecure webview property usage
- SSL Certificate validation problem
- Insufficient URL Validation
gibi konular yer almaktadır.


---

# 6.  Android Saldırı Yüzeyi
![Android Apps Pen-testing | securitybreached.org](https://i0.wp.com/blog.securitybreached.org/wp-content/uploads/2020/03/chart.png?resize=845%2C425&ssl=1)

- [ ] TODO:Olası saldırı türleri araştırılacak

---

# 7. Android Malware Türleri
- [ ] bilinen ve güncel zararlı yazılımların detaylı araştırılması yapılacak
---

# 8 Other Topics

## 1. Mobile Forensics

...

---

# Kaynak
- [AndroidDeveloperDoc](https://developer.android.com/guide/components/fundamentals.html)
- [Phone Forensics InfosecInstitute](https://resources.infosecinstitute.com/topic/practical-android-phone-forensics/)
- [maddiestone](https://github.com/maddiestone)
- [CS8038 Malware Analysis](https://class.malware.re/2020/04/18/android-intro-and-tools.html)
- [AndoridChecklist](https://book.hacktricks.xyz/mobile-apps-pentesting/android-checklist)
- [Get Started in Android Apps Pen-testing Part1](https://blog.securitybreached.org/2020/03/17/getting-started-in-android-apps-pentesting/)
- [DIVA Tutorials](https://resources.infosecinstitute.com/topic/cracking-damn-insecure-and-vulnerable-app-diva-part-5/)
- 
