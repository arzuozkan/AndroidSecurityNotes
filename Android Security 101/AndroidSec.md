# 0. İçindekiler

1. [Android Mimarisi](#Android%20Mimarisi)
2. [Android Uygulama temelleri APK](AndroidSec.md#Android%20Uygulama%20temelleri%20APK)
3. [Android Uygulama Pentesting Süreci](#Android%20Uygulama%20Pentesting%20Süreci)
4. [OWASP MOBILE TOP 10](#OWASP%20MOBILE%20TOP%2010)
5. [Android Uygulama Güvenliği](#Android%20Uygulama%20Güvenliği)
6. [Android analiz ortamı ve araçlar](#Android%20analiz%20ortamı%20ve%20araçlar)
7. [Android Zararlı Yazılım Analizi](#Android%20Zararlı%20Yazılım%20Analizi)
8. [Android Zararlı Yazılım Türleri](#Android%20Zararlı%20Yazılım%20Türleri)
9. [Other Topics](#Other%20Topics)
10. [Kaynaklar](../Kaynaklar.md)

---
# Android Mimarisi
- Android işletim sistemi içerisinde çok kullanıcılı linux sistemidir. Her bir uygulamanın linux kullanıcı idye sahip. 
- Sistem dosyalar için gerekli izinleri vermektedir.Böylece, sadece id ye sahip kullanıcılar erişebilmektedir.
- Her processin kendine ait sanal makinesi bulunmaktadır. Bu sayede, her uygulama birbirinden izole ortamlarda çalışır.

Her uygulamanın kendi kullanıcı hesabı ve idsi bulunmaktadır.Bu sayede her uygulamanın izinleri de farklılaşmaktadır.Her uygulamanın kendi dizini bulunmaktadır. Uygulamanın işletim sistemine yapılan istekler **syscall** olarak isimlendirilir ve Java/Kotlin  -> libc -> syscalls  şeklinde gerçekleşir.
User-space, kullanıcı uygulama processlerin bulunduğu yerdir ve bu processlerin fiziksel sürücülere erişimi yoktur. Kernel-space, işletim sistemini içeren kritik bölge .  Farklı mimariler için kendi argumanları bulunur.
Kullanıcı profilleri,
- Primary User (birincil kullanıcı), cihazın ilk defa başlatılmasıyla oluşan kullanıcıdır. Cihaz fabrika ayarlarına döndürülmediği sürece her zaman çalışır.
- Secondary User - Cihaza eklenen ikincil kullanıcıdır.



Android işletim sistemi katmanlarından bahsedecek olursaki bunlar,
\- *Sistem Uygulamaları*: içerisinde bulunan varsayılan uygulamaları içerir
\- *Native C/C++ Kütüphaneleri*: c ve c++ ile kodları ve gerekli kütüphaneleri içerir
\- *JAVA API Framework*: içerisinde uygulamaların çalışmasını sağlayacak bileşenler bulunur
\- *ART*: Android RunTime, Dalvik sanal makineyi içerir ve dex dosyalarını çalıştırır.
\- *HAL*: Hardware Abstraction Layer, uygulamanın direkt olarak donanımla iletişimini engelleyen arabirimdir.
\- *Linux kernel*: ses,klavye,bluetooth,kamera gibi sürücüleri içerir ve hafıza yönetimi yapılmaktadır


**Android Uygulama İzinleri**

- RECEIVE_BOOT_COMPLETED: bu izin sayesinde uygulama, cihazın bootlama esnasında otomatik başlatılmasını sağlar. 
- READ/WRITE_EXTRERNAL_STORAGE : Cihazdaki fotoğraf ve indirlen dosyaların depolandığı yer /sdcard dizindir.Bu depolama alanına erişim sağlanır.
-  SYSTEM_ALERT_WINDOWS: başka uygulamaların  üzerine overlay oluşturabilir.

**Android Güvenliği**

Android güvenliği farklı güvenlik katmanlarında sağlanır, yani gizlilik(confidentiality), bütünlük(integrity) veya erişilebilirlik(availability) ilkelerinin tek bir önleme bağlı değildir.
Kabaca 4 başlıkta incelenir,
- System-wide security
- Software isolation
- Network Security
- Anti-exploitation


---
# Android Uygulama temelleri, APK

Detaya inmeden Android APK içeriği aşağıdaki şekilde gösterilebilir.

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
Uygulama içerisindeki bileşenlerin birbirleriyle veya diğer bir uygulamadaki bileşenle iletşim kurmasını sağlayan yapı olarak tanımlanır. Aktiviteyi başlatmak, sistemi değişiklikler için bilgilendirmek,servisler ile iletişimi gerçekleştirmek, contentProvider ile verilere erişmek gibi çeşitli işlerde kullanılmaktadır. İki türü bulunur, explicit  ve implicit intent. Explicit intent uygulama içerisindeki bileşenler arasındaki iletişimdir. Implicit intent ise, farklı bir uygulama ile ileişimi sağlamaktadır.
Uygulamanın bunu anlayabilmesi için türünü belirleyen yapı `intent-filter` olarak isimlendirilir ve manifest içerisinde kullanılır.Bu yapı sayesinde hangi aktivite veya servis  karşılık vereceğini anlar. Intent-filter içeriği  \<action\> \<category\> ve \<data\> elementlerinden oluşur.
  
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
İçerik sağlayıcılar bir arayüz görevi görerek uygulamalar arası veri paylaşımı yapmanın bir yoludur. repository ile etkileşim içerisindedir. Uygulamanın verileri nereye depolayacağını, nasıl erişim sağlayacağını belirler. İçerik sağlayıcılar URI adresleme şeması araclığı ile **content://** olarak uygulanmaktadır.Veritabanı işlemlerine benzer olarak insert(), query(), update() ve delete() metotları bulunur. Çoğunlukla SQLite tabanlıdır.


---
# Android Uygulama Pentesting Süreci
Android kontrol listesi içerisinde, 
- Android temelleri, 
- Statik analiz, 
- Dinamik analiz, ve 
- diğer konular ,
şeklinde 4 başlık altında toplanmıştır

Konuya girmeden önce genel olarak mobil uygulama penetration test aşamalarından bahsedilebilir. Bu adımlar temelde

Reconnaissance (Keşif aşaması ) ---> Statik Analiz ---> Dinamik Analiz  ---> Raporlama


**Keşif** aşamasında, uygulama hakkında bilgi toplanır, uygulama versiyonları ve yapılan güncellemeler incelenebilir, geliştiricileri araştırlır. 

**Statik Analiz**, uygulama kodları manuel veya statik analiz araçları kullanılarak incelenir. Hatalı güvenlik konfigürasyonları, hard-coded stringler kontrol edilebilir. API url bağlantıları, SQL veritabanı, değişkenler, yerel depolama alanları incelenebilir. Hard-coded stringler resources./strings.xml dosyası içerisinde bulunur. Aktiviteler içerisinde de bulunabilir. Ne işimize yarar? Giriş bilgilerini elde edebiliriz, bağlantı adresleri, api anahtarları gibi hassas bilgiler elde edilebilir.

**Dinamik Analiz**, uygulamanın çalıştırıldığı ve manipüle edildiği aşamadır. Analiz süresince, yapılan ağ trafiği incelenir, uygulama dump alınır, çalışma zamanında oluşturulan dosyalar kontrol edilir, ssl pinning atlatma gibi işlemler gerçekleştirilebilir.

Analiz aşamasında bakılması gereken noktalar,
- API çağrıları
- Uygulama Entrypointleri
- Obfuscate edilmiş metotları decrypt etmek




---

# OWASP MOBILE TOP 10
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


## 3. Insecure Communication

---


# Android Uygulama Güvenliği

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
>-  [BeetleBug](https://github.com/hafiz-ng/Beetlebug)
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

# Android analiz ortamı ve araçlar


## 1. adb
Android Debug Bridge, sunucu-istemci programıdır. İstemci komut satırında adb aracını kullanarak cihaz ile iletişim kurmaktadır.
Bazı adb komutları,
>adb devices : bağlı chiazları listeler

>adb connect HOST[:PORT]

> adb tcpip 5555 : telefonu 5555 portundan TCP bağlantısı gerçekleştirir.

> adb shell : bağlantısı kurulmuş cihazdan shell alınır

> adb root : root yetkilerinde çalışmak için


## 2. jadx
jadx aracı,apk dosya içeriğini decode ederek dex dosyalarını okunabilir hale dönüştürmektedir. apktool ve dex2jar araçları ile aynı işlevi gerçekleştirirler.

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


## 6. Objection
Runtime kullanılan mobil keşif aracıdır. Kullanım amacı, kullanıcının ana aksiyonlar çağırılabilmektedir.

> pip install objection

komutu ile kurulumu gerçekleşmektedir.  *--gadget* parametresine uygulama paket adı verilerek bağlantı sağlanmaktadır. Frida sunucusu bağlı olmalıdır. 

>  objection --gadget com.package.name explore

Komut çalıştırıldığında tab tuşuna basarak girilebilecek komutları ve açıklamaları listelenmektedir.

![](../images/Pasted%20image%2020220331225036.png)

Komutlar detaylı olarak incelenecektir. İlk olarak **android** komutuna göz atalım. Önemli olarak metod hooklama, intent listeleme gibi işlemleri yapabilir.  

> android hooking lis activities 

![](../images/Pasted%20image%2020220331225843.png)

komutu uygulama içerisindeki aktiviteleri listelemektedir. Diğer alıcılar veya servisler gibi bileşenler ise activites kısmını değiştirerek listelenebilmektedir.


## 7. Medusa
https://github.com/Ch0pin/medusa/blob/master/MEDUSA-Usage-workflows.pdf

## 8.Dexcalibur
https://github.com/FrenchYeti/dexcalibur

## 9. Qu1cksc0pe: All-in-One malware analysis tool 
https://github.com/CYB3RMX/Qu1cksc0pe



## Statik Analizde Kullanılan Araçlar
### Androwarn
[Androwarn](https://github.com/maaaaz/androwarn), Android uygulamalardaki zararlı işlemleri tespit eden python ile geliştirilmiş kod analiz aracıdır.Androguard aracına ihtiyaç duymaktadır. Yapılan analizi rapora çevirebilmektedir. Raporlar, txt, html ve json formatında oluşturulabilir.Özellikleri arasında telefon id,cihaz özellikleri çıkarımı, konum sızıntısı, telefon servislerinin kötüye kullanımı, isteğe bağlı kod çalıştırma ve uzaktan bağlantı kurulumu gibi buna benzer zararlı aktivitelerin analizleri yer almaktadır.

> pip install androwarn 

komutu ile veya github reposunu indirip 
> pip install -r requirements.txt

komutu çalıştırılarak yüklenebilir. Yükleme yapıldıktan sonra, `androwarn` komutu ile çalıştırılabilir. Gerekli paramatreler terminal çıktısında gösterilmektedir.

![](../images/Pasted%20image%2020220403180813.png)

`-i` parametresine analizi gerçekleşecek apk dosyası, `-v` detay seviyesi 1 temelden, 3 uzman yani çok daha detaylı verileri içermektedir. `-r` rapor formatını belirlemek için kullanılır. Temel kullanıma örnek olarak aşağıdaki komut çalıştırıldığında çıktı olarak  oluşturulan analiz raporun dizini verilmektedir.

![](../images/Pasted%20image%2020220403181859.png)

Oluşturulan detay seviyesi 3 olan rapor incelendiğinde  şekilde yer alan başlıklar bulunmaktadır.

![](../images/Pasted%20image%2020220403182137.png)

Kurulan şüpheli bağlantılar "Suspicious Connection Establishment" başlığında listelenmektedir.

![](../images/Pasted%20image%2020220403182449.png)

Fingerprint başlığında apk dosyasının hash bilgileri gösterilmektedir.

![](../images/Pasted%20image%2020220403182350.png)

Androidmanifest.xml başlığında manifest dosyası içerisinde sdk bilgisi, uygulama bileşenleri (aktivite,servis vs) ve uygulama izinleri yer almaktadır. 



## Otomatik Analiz yapan araçlar
**MobSF** : Docker üzerinde çalışır. Statik analizi otomatik gerçekleştirir. Güvenlik açığı oluşturabilecek yapıları ve kritik kod parçalarını, stringleri listelemektedir. Kritik izinleri, internet bağlantıları gibi bilgileri çıkartır. Oluşturduğu raporu, PDF formatında yazdırabilmektedir.


---

# Android Zararlı Yazılım Analizi
- [ ] #TODO 

---

# Android Zararlı Yazılım Türleri
- [ ] #TODO bilinen ve güncel zararlı yazılımların detaylı araştırılması yapılacak
---

# Other Topics
## Android Anti-Reversing Defenses
Android uygulamasının analizini ve tersine mühendislik işlemlerini engellemek amaçlı kullanılan yöntemlerdir. Güncel android zararlı yazılımlar içerisinde de bu yöntemlerin kullanımı yüksektir. Cihaz içerisinde root tespiti, debugging tespiti, tersine mühendislik araçlarının tespiti, emülator kullanımının tespiti gibi birçok yöntem bulunmaktadır. Bu gibi uygulamayı koruyan yöntemler şekilde gösterilmiştir. 

![](../images/Pasted%20image%2020220331104207.png)

Görsel Kaynak:(https://raw.githubusercontent.com/FrenchYeti/unrasp/main/Slides/Forging_golden_hammer_against_android_app_protections_INSO22_FINAL.pdf)

Uygulamanın kodu, içeriği, ortamı ve ağ trafiğini koruyan araçların atlatma teknikleri de bulunmaktadır.


### 1. Testing Root Detection
Android uygulamanın analizi ve tersine müendislik işlemlerinde kullanılan araçlar root izninde çalıştığı için bunu önlemek adına root tespiti yapılabilir. Tek başına etkili yöntem olmamakla beraber birden fazla yerde root cihaz kontrolünün yapılması uygulamayı test etmeyi zorlaştırabilmektedir. [Rootbeer](https://github.com/scottyab/rootbeer) aracı cihaz üzerinde root kontrolü yapar. Uygulama cihaza yüklenip çalıştığında görseldeki gibi root kontrolü yapmaktadır.

![](../images/Pasted%20image%2020220327142501.png)

Root cihaz tespiti için kontrol noktaları,
- Test-keys: test anahtarları için build propertylerin (`android.os.BUILD.TAGS`) gözden geçirilmesi
- Su & Busybox binary: su ve busybox dosya dizinleri kontrollerinin yapılması (system/xbin/ ve /su/bin/). `checkForBinary("su")` ve `checkForBinary("busybox")` fonksiyonları ile kontrol yapılabilir.
- Superuser veya root yetkilerini yöneten magisk veya supersu gibi uygulamaların tespiti
- Android cihazın root durumunu gizleyen uygulamaların kontrolü. `com.devadvance.rootcloak` ,  `com.devadvance.rootcloakplus` , `com.koushikdutta.superuser` , `com.thirdparty.superuser` gibi root gizleme uygulamaların tespit edilmesi.

OWASP MSTG'nin hazırladığı Uncrackable 1 uygulama içerisinde root detection konusu ele alınmıştır. jadx aracı ile apk dosyası açıldığında MainActivity içerisinde bazı fonksiyonlar ile root durumu kontrol edilmektedir.

![](../images/Pasted%20image%2020220327232924.png)

İlgili sınıf içerisi incelendiğinde, root tespiti için yukarıda bahsedilen yöntemlerin birkaçının kullanıldığı görülmektedir.  `a()` metodunda, su isimli dosya dizinleri kontrol edilmektedir.

![](../images/Pasted%20image%2020220327233340.png)

 `b()` metodu içerisinde test-keylerin kontrolü yapılmaktadır.
 
![](../images/Pasted%20image%2020220327233521.png)

 `c()` metodunda ise root uygulaması ve dizinlerinin varlığınin tespiti yapılmaktadır.

![](../images/Pasted%20image%2020220327233658.png)

---

# [Kaynaklar](../Kaynaklar.md)