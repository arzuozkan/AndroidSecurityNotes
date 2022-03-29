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
	> - uygulama dex dosyası yani java bytekodlarını içerir. Çalışan java(kotlin) kodlar burada bulunur

> lib/
	> - uygulama için kullanılan yerel kütüphaneler burada bulunur.

> assets/
	> - uygulama için gerekli diğer dosyalar
	> - ek kütüphaneler
	> - dex dosyaları

**Dalvik vs Smali**
Uygulama geliştirilirken Java veya Kotlin kodları DEX(Dalvik executable) bytekoda compile edilir yani dex formata dönüştürülür.
Tersine mühendislik işlemlerinde ise DEX bytecode önce SMALI'ye yani Dalvik bytecodeun okunabilir haline getirilir, bu yüzden SMALI bytecode ile high level programlama dili arasındadır assembly gibi.

## Manifest Dosyası
Uygulama çalışmadan önce sistem hangi bileşenlerin var olduğunu AndroidManifest.xml dosyasından okumaktadır. Manifest içerisinde,
- Paket ismi , uygulama id, uygulama ikonu,
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

**Intents** 
Uygulama içerisindeki bileşenlerin birbirleriyle veya diğer bir uygulamadaki bileşenle iletşim kurmasını sağlayan yapı olarak tanımlanır. Aktiviteyi başlatmak, sistemi değişiklikler için bilgilendirmek,servisler ile iletişimi gerçekleştirmek, contentProvider ile verilere erişmek gibi çeşitli işlerde kullanılmaktadır. 
Uygulamanın bunu anlayabilmesi için türünü belirleyen yapı `intent-filter` olarak isimlendirilir ve manifest içerisinde kullanılır.Bu yapı sayesinde hangi aktivite veya servis  karşılık vereceğini anlar. Intent-filter içeriği \<action\> \<category\> ve \<data\> elementlerinden oluşur
  
**Uygulama Entrypointleri**
- *Launcher Activity(Activity Başlangıcı)* uygulamanın açılmasıyla beraber başlamaktadır
- *Services(Servisler)* Kullanıcı arayüzüne gerek duymaksızın arkaplanda çalışmaktadır. Intentler aracılığıyla uygulamanın entrypointi olabilir.
- *Broadcast Receiver(Alıcı)* kısaca dinleyiciler denilebilir.

### 1. Activities (Aktiviteler)
Uygulamanın giriş noktası olarak da söylenebilir. Kullanıcılar ile etkileşime girilen ekran olan Aktiviteler uygulama içerisinde birden çok oluşturulabilir. Android geliştirici çerçevesinden bakıldığında, manifest dosyası içerisinde her aktivitenin \<activity\> tagi içerisinde tanımlanması gerekmektedir ve tanımlanır. Kendine ait yaşam döngüleri bulunur. Activitiy manager(aktivite yöneticisi) aracı ile başlatılabilir exported olduğu zaman. 
- `android:exported` : değeri "true" ise aktivite başka uygulamalar tarafından başlatılabilmektedir yani cihazdaki herhangi uygulama kendi aktivite ismi ile erişebilir. Varsayılan değeri eğer intent-filter içermiyorsa "false"dur. 
- `android:icon` : uygulamanın ikonunu belirler. Uygulama ikona sahip değilse normal bir uygulama olmayabilir.
- 

### 2. Services (Servisler)
Android işletim sistemi içerisinde veri işleme,bildirimler gibi arkaplanda çalışan görevleri yerine getirmektedir. Servisler, aktivitelere benzer olarak ana thread içerisinde çalışırlar ve kendi threadini oluşturamaz ayrıca belirtilmediği sürece farklı process üzerinde çalışmazlar.

### 3. Broadcast Receivers (Alıcılar)
Sistem çapında bildirimleri yöneten yapıdır. Batarya zayıf olması,kulaklık girişi,mesaj iletildi gibi çeşitli olaylarda yayınlar(broadcast) gerçekleşir. Yayın almanın iki yolu vardır, manifest içerisinde tanımlama veya *registerReceiver* api çağrısını kullanarak dinamik olarak tanımlama. Genellikle arayüz güncelleme,servis başlatma ve kullanıcı bildirimleri oluşturmada kullanılır.

### 4. Content Provider (İçerik Sağlayıcı)
İçerik sağlayıcılar bir arayüz görevi görerek uygulamalar arası veri paylaşımı yapmanın bir yoludur. repository ile etkileşim içerisindedir. Uygulamanın verileri nereye depolayacağını, nasıl erişim sağlayacağını belirler. İçerik sağlayıcılar URI adresleme şeması araclığı ile **content://** olarak uygulanmaktadır.Veritabanı işlemlerine benzer olarak insert(), query(), update() ve delete() metotları bulunur.


---
# 3. Android analiz ortamı ve araçlar

Android kontrol listesi içerisinde, 
- Android temelleri, 
- Statik analiz, 
- Dinamik analiz, ve 
- diğer konular ,
şeklinde 4 başlık altında toplanmıştır

Konuya girmeden önce genel olarak mobil uygulama penetration test aşamalarından bahsedilebilir. Bu adımlar temelde

Reconnaissance(Keşif) ---> Statik Analiz ---> Dinamik Analiz  ---> Raporlama


**Keşif** aşamasında, uygulama hakkında bilgi toplanır, uygulama versiyonları ve yapılan güncellemeler incelenebilir, geliştiricileri araştırlır.

**Statik Analiz**, uygulama kodları statik analiz araçları kullanılarak incelenir. hatalı güvenlik konfigürasyonları, stringler kontrol edilebilir. API url bağlantıları, SQL veritabanı, değişkenler, yerel depolama alanları incelenebilir.

**Dinamik Analiz**, uygulamanın çalıştırıldığı ve manipüle edildiği aşamadır. Analiz süresince, yapılan ağ trafiği incelenir, uygulama dump alınır, çalışma zamanında oluşturulan dosyalar kontrol edilir, ssl pinning gibi işlemler gerçekleştirilebilir.

Analiz aşamasında bakılması gerekn noktalar,
- API çağrıları
- Uygulama Entrypointleri
- Obfuscate edilmiş metotları decrypt etmek

==Bazı yaygın kullanılan Android araçları
> **`apktool`:** APK dosyalarının  içeriğini çıkartarak okunabilir duruma getirebilir. `apktool d - o output_folder app.apk` komutu APK sıkıştırılmış dosyasını açmak için kullanılır.
>  **`dex2jar`:**  APK içerisindeki DEX dosyalarını analiz edilebilir JAR dosyalarına çevirir ve Java analiz araçlarından biridir. Kali Linux beraberinde gelir ve `d2j-dex2jar -o output.jar app.apk` komutu çalıştırılır.
>   **`Anbox`:** sanallaştırma veya emülatör ortamı oluştururak dinamik sürece yardımcı olur
>  **`Android Studio`:** Android geliştirme uygulaması olarak kullanılır. Analiz ve debug süreçlerinde fayda sağlar.

==_Tasarım_ --> _Kod_ --> _Derleme_ --> _Çıktı_
Uygulama geliştirme aşamaları --->
Tersine mühendislik işlemleri 	 <---


==APK dosyaları Play Store, GetAPK, GETJAR, F-Froid, Apkpure, Aptoide gibi platformlardan yüklenebilmektedir. Aynı zamanda adb aracı kullanarak da cihaza uygulama yüklenebilir.
`adb install /path/app.apk`
ilgili paketi kaldırmak için
`adb uninstall package.name`
yüklü paketlerin listesini 
`adb shell pm list packages`
komutu ile listelenmektedir.



==AndroidManifest.xml ve resource.arsc Decoder yardımı ile
classes.dex Disassembler/Decompilerile okunabilir duruma getirilebilir.
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

**Drozer**

https://github.com/FSecureLABS/drozer

Android cihaz içerisine native uygulama olarak yüklenip Dalvik sanal makine ile iletişim kurarak cihazdaki uygulamalar için zafiyet taraması gerçekleştirir. Sunucu ve ajan uygulaması bulunur. Sunucu konsol aracılığıyla ajan ile iletişimi gerçekleştirir. Python ile geliştirilmiş frameworktür.

> #########

## 1. adb
Android Debug Bridge, sunucu-istemci programıdır. İstemci komut satırında adb aracını kullanarak cihaz ile iletişim kurmaktadır.
Bazı adb komutları,
>adb devices : bağlı chiazları listeler

>adb connect HOST[:PORT]

> adb tcpip 5555 : telefonu 5555 portundan TCP bağlantısı gerçekleştirir.

> adb shell : bağlantısı kurulmuş cihazdan shell alınır

> adb root : root yetkilerinde çalışmak için


## 2. jadx
## 3. apktool
apktool aracı, APK dosyaların decompile ederek paketinden çıkarmaktadır. Bu sayede Manifest dosyası okunabilir duruma gelir. Normal olarak arşiv dosyasından çıkardığımız zaman manifestin okunur durumda olmadığını görebiliriz.

## 4 dex2jar
classes.dex dosyasını jar dosyasına çevirir. Bu sayede jd-gui gibi araçlar kullanrak java kodları okunabilir.

## 5. Frida 
Birçok işletim sistemleriyle uyumlu çalışabilir ve bu sistemlerin içerisinde bulunan uygulamaların içerdiği fonksiyonların çalışmasını değiştirebilmek gibi faklı amaçlar için kullanılan araç takımıdır. Sistem içerisindeki çalışan processleri analiz edebilir. Dinamik analiz sürecinde kullanılır. Uygulama içerinsine JavaScript kodu injecte etmektedir. Bunun dışında,
- Root tespitini atlatma
- SSL pinning bypass
- Fonksiyonların return değerini değiştirme
- gizli değerleri bulma
gibi işlevlere sahiptir.

> pip install frida
> pip install frida-tools

komutlarını çalıştırarak indirebiliriz. Kullancağımız android cihaza bağlı olarak uygun sürümü github reposundan indirebiliriz (frida-server-version-android.xz şeklinde ) 
`adb push` komutu kullanarak cihaza frida-server, /data/local/tmp konumuna yüklenmiş olur.

> adb push frida-server /data/local/tmp

`adb shell` komutu ile hedef cihazda komut çalıştırabiliyoruz. komutu çalıştırdıktan sonra frida-server dosyamıza `chmod +x` ile çalıştırma izni verilmektedir. Frida sunucusunu çalıştırmak için  `frida-server &` komutu çalıştırılır. Bağlantının kurul kurulmadığını kontrol etmek için terminal üzerinden `frida-ps -U` ile cihazda çalışan processler listelenebilir. `-U` parametresi USB cihaza bağlanmaktadır. `frida-ps --help`  komutu ile diğer paramtereler ve kullanım amaçları liselenmektedir. 

`frida-trace` aracı fonksiyonları ve kütüphaneleri izlemek için kullanılır.  `-i` parametresi bulmak istenen fonksiyon için pattern belirler. `-I` parametresi, belirtilen modül içerisindeki tüm fonksiyonları hooklamak için kullanılır.

> frida-trace -U -i open* -I \*sqlite\*  process


## 6.Dexcalibur
https://github.com/FrenchYeti/dexcalibur


## Otomatik Analiz yapan araçlar
**MobSF** : Docker üzerinde çalışır. Statik analizi otomatik gerçekleştirir. Güvenlik açığı oluşturabilecek yapıları ve kritik kod parçalarını, stringleri listelemektedir. Kritik izinleri, internet bağlantıları gibi bilgileri çıkartır. Oluşturduğu raporu, PDF formatında yazdırabilmektedir.





---

# 4. OWASP MOBILE TOP 10
OWASP Mobile Security Testing Guide içerisinde yer alan konular içerisinde;
- Temel güvenlik testi
- Android içerisinde veri depolama
- Kriptografik apiler
- yerel authentication
- network apileri
- platform apileri
- tersine mühendislik
- anti-reversing yöntemleri

gibi konu başlıkları yer almaktadır. Top 10 listesi de bu konularla bağlantılı riskleri içermektedir.

OWASP'ın en son 2016 yılında yayınladığı popüler 10 adet mobil uygulama riskleri:

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
Mobil cihaz içerisindeki dosya kalsörlerine erişim kolayca gerçekleşebilir. Cihazın bilgileri kaydettiği klasöre de erişim çeşitli yollardan gerçekleşebilir. Bu durumlarda uygulamanın statik analazini yapan ve hassas veriyi bulan birçok araç bulunmaktadır. (Örneğin [StaCoAn](https://github.com/vincentcox/StaCoAn), uygulamadaki API anahtarlarını, URL bağlantıları, hardcoded metinleri tespit edebilmektedir. Yaygın kullanılan araçlardan bir diğeri [apkleaks](https://github.com/dwisiswant0/apkleaks), APK dosyasını tarayarak içerisindeki bağlantıları tespit edebilir.) Bu güvenlik açığı, kimlik hırsızlığına, gizlilik ihlaline yol açabilir.Bu veriler nerede açığa çıkabilir,
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

>
>- [DIVA](https://github.com/payatu/diva-android)
>- [Insecure Shop](https://github.com/arzuozkan/MyAndroidSecurityNotes/blob/main/Android%20Security%20101/InsecureShop.md)
>- [OWASP MSTG Hacking Playground](https://github.com/OWASP/MSTG-Hacking-Playground)
>- [UnsafeBank](https://github.com/lucideus-repo/UnSAFE_Bank)
>- [InjuredAndroid](https://github.com/B3nac/InjuredAndroid)
>- [Android Application Security](https://github.com/RavikumarRamesh/hpAndro1337)
>- [AndroGoat](https://github.com/satishpatnayak/AndroGoat)
>- [OWASP MSTG crackmes](https://github.com/OWASP/owasp-mstg/tree/master/Crackmes/Android)
>- [Android App Hacking WorkShop](https://bughunters.google.com/learn/presentations/5783688075542528)
>- [hpAndro Vulnerable Application (Kotlin) CTF](https://github.com/RavikumarRamesh/hpAndro1337/tree/main/Android%20AppSec%20(Kotlin)/1.2)
>- [KGB Messenger CTF](https://github.com/tlamb96/kgb_messenger)
>- 
>


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
## Android Anti-Reversing Defenses
Android uygulamasının analizini ve tersine mühendislik işlemlerini engellemek amaçlı kullanılan yöntemlerdir. Güncel android zararlı yazılımlar içerisinde de bu yöntemlerin kullanımı yüksektir. cihaz içerisinde root tespiti, debugging tespiti, tersine mühendislik araçlarının tespiti, emülator kullanımının tespiti gibi birçok yöntem bulunmaktadır.

### 1. Testing Root Detection
Android uygulamanın analizi ve tersine müendislik işlemlerinde kullanılan araçlar root izninde çalıştığı için bunu önlemek adına root tespiti yapılabilir. Tek başına etkili yöntem olmamakla beraber birden fazla yerde root cihaz kontrolünün yapılması uygulamayı test etmeyi zorlaştırabilmektedir. [Rootbeer](https://github.com/scottyab/rootbeer) aracı cihaz üzerinde root kontrolü yapar. Uygulama cihaza yüklenip çalıştığında görseldeki gibi root kontrolü yapmaktadır.

![](../images/Pasted%20image%2020220327142501.png)

Root cihaz tespiti için kontrol noktaları,
- Test-keys: test anahtarları için build propertylerin (`android.os.BUILD.TAGS`) gözden geçirilmesi
- Su & Busybox binary: su ve busybox dosya dizinleri kontrollerinin yapılması (system/xbin/ ve /su/bin/). `checkForBinary("su")` ve `checkForBinary("busybox")` fonksiyonları ile kontrol yapılabilir.
- Superuser veya root yetkilerini yöneten magisk veya supersu gibi uygulamaların tespiti
- Android cihazın root durumunu gizleyen uygulamaların kontrolü. `com.devadvance.rootcloak` ,  `com.devadvance.rootcloakplus` , `com.koushikdutta.superuser` , `com.thirdparty.superuser` gibi root gizleme uygulamaların tespit edilmesi.

OWASP MSTG'nin hazırladığı Uncrackable 1 uygulama içerisinde root detection konusu ele alınmıştır. jadx aracı ile apk dosyası açıldığında MainActivity içerisinde `c` isimli sınıf içerisinde `a()` , `b()` , `c()`  fonksiyonlarının sağlanması durumunda root tespiti yapıldığı anlaşılmaktadır.

![](../images/Pasted%20image%2020220327232924.png)

İlgili sınıf içerisi incelendiğinde, root tespiti için yukarıda bahsedilen yöntemlerin birkaçının kullanıldığı görülmektedir.  `a()` metodunda, su isimli dosya dizinleri kontrol edilmektedir.

![](../images/Pasted%20image%2020220327233340.png)

 `b()` metodu içerisinde test-keylerin kontrolü yapılmaktadır.
 
![](../images/Pasted%20image%2020220327233521.png)

 `c()` metodunda ise root uygulaması ve dizinlerinin varlığınin tespiti yapılmaktadır.

![](../images/Pasted%20image%2020220327233658.png)

---

# Kaynak
- [AndroidDeveloperDoc](https://developer.android.com/guide/components/fundamentals.html)
- [Phone Forensics InfosecInstitute](https://resources.infosecinstitute.com/topic/practical-android-phone-forensics/)
- [maddiestone](https://github.com/maddiestone)
- [CS8038 Malware Analysis](https://class.malware.re/2020/04/18/android-intro-and-tools.html)
- [AndoridChecklist](https://book.hacktricks.xyz/mobile-apps-pentesting/android-checklist)
- [Get Started in Android Apps Pen-testing Part1](https://blog.securitybreached.org/2020/03/17/getting-started-in-android-apps-pentesting/)
- [DIVA Tutorials](https://resources.infosecinstitute.com/topic/cracking-damn-insecure-and-vulnerable-app-diva-part-5/)
- [Android Application Fundamentals](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/android-applications-basics#2-android-application-fundamentals)
- [# Android Application Pentesting - Mystikcon 2020](https://www.youtube.com/watch?v=NrxTBcjAL8A)
- [Android Anti-Reversing Defense](https://mobile-security.gitbook.io/mobile-security-testing-guide/android-testing-guide/0x05j-testing-resiliency-against-reverse-engineering)
- [Comparison of Different Android Root-Detection Bypass Tools](https://medium.com/secarmalabs/comparison-of-different-android-root-detection-bypass-tools-8fd477251640)
-  
