# Beetlebug
[Beetlebug](https://github.com/hafiz-ng/Beetlebug), içerisinde android uygulama güvenliği ile ilgili konuları içeren zafiyetli bir android CTF uygulamasıdır. Uygulama açıldığında bir kullanıcı ismi girerek uygulamaya giriş yapılabilir. Toplamda 16 adet flag değeri bulunmaktadır.

![](../images/Pasted%20image%2020220626153010.png)

Güvenlik açıklarının bukunduğu başlıklar,
- Hardcoded secrets (Kodlanmış hassas bilgiler)
- Insecure Data Storage (Güvenli olmayan depolama)
- Sensitive Information Disclosure (Hassas bilgi ifşası)
- Vulnerable Android IPC Components (Zafiyetli processler arası iletişim bileşenleri, alıcılar,servisler ve içerik sağlayıcıları)
- Vulnerable Webviews (Zafiyetli webview)
- Insecure Deeplinks (Gğvenli olmayan deeplinkler)
- sqli, xss
- Authentication bypass
- Firebase misconfiguration


## 1. Hardcoded Secrets
### 1.1 Hardcoding sensitive data
Gizli dosyaya erişebilmek için PIN değeri girilmesi gerekmektedir. Verilen ipucunda, string kaynaklarına bakılması öneriliyor.

![](../images/Pasted%20image%2020220626153842.png)

jadx-gui ile apk uygulama dosyasını açıyoruz. Hangi aktivitenin ilk challange ait olduğunu ilk başta anlamayabiliriz.Burada res/value dizininin içerisindeki strings.xml dosyası incelendiğinde listelenen ilk değer dikkatimizi çekebilir.

![](../images/Pasted%20image%2020220626155832.png)

genel bir arama yaparsak,

![](../images/Pasted%20image%2020220626155551.png)

ilgili ekranın EmbeddedSecretStrings isimli aktivite olduğu anlaşılabilir. Aktivite içerisindeki kodlar incelendiğinde burada kullanıcıdan alınan girdi, PIN değeri ile eşit olup olmadığı koşulu bulunmaktadır. Buradaki karşılaştırılan string değeri (`R.string.V98bFQrpGkDJ`) istenen PIN değeridir.

![](../images/Pasted%20image%2020220626160223.png)

Bulduğumuz PIN değerini girdiğimiz zaman ilk flag değerini bulmuş oluyoruz.

![](../images/Pasted%20image%2020220626160453.png)

### 1.2 Hardcoding Secrets
Bu challange içerisinde hassas bilgilerin açık metin şekilde kodlanmasını içermektedir. Burada promosyon kodunu bulmamız gerekiyor.

![](../images/Pasted%20image%2020220626160806.png)

jadx-gui aracını kullanarak kaynak kod incelemesinde, EmbeddedSecretSourceCode isimli aktiviteyi inceliyoruz. Promosyon kodu direkt olarak, değer şeklinde tanımlanmış durumda.

![](../images/Pasted%20image%2020220626161118.png)

Bulunan kod girildiğinde flag değeri elde edilmiş oluyor.

## 2. Data Storage
### 2.1 Shared Preferences
Shared Preferences yapısı anahtar-değer(key-value) şeklinde küçük verileri depolamak için kullanılır.Verileri /data/data/paket_ismi/shared_prefs dizini altında bir xml dosyasına kaydeder. Burada kullanıcı adı ve parola girilmesi istenmektedir.

![](../images/Pasted%20image%2020220626170811.png)

Bilgiler girdikten sonra **Save** butonuna kaydedildiğinde ekranın alt kısmında flag değeri için submit butonu ortaya çıkmaktadır.

![](../images/Pasted%20image%2020220626171113.png)

Terminal üzerinden `adb shell` komutunu kullanarak cihaz üzerine bağlanabiliriz. */data/data* dizini altında ilgili uygulama paketine giderek *shared_prefs* klasörü içerisinde shared_pref_flag isimli xml dosya içeriği görüntülendiğinde girilen kullanıcı ve parola değerlerinin yanı sıra flag değeri de depolanmaktadır.

![](../images/Pasted%20image%2020220626171547.png)

### 2.2 External Storage

![](../images/Pasted%20image%2020220626172149.png)

Burada mail adresi ve parolayı kaydettiğimiz zaman ekranda bir toast mesajı belirmektedir. Verilerin saklandığı dosya yolu ve dosya ismi verilmiştir. Terminal üzerinden cihaza `adb shell` komutu ile erişim sağlanmaktadır. 

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/f40732f6-77d6-4de6-a81c-8c089b8066f5)

Diğer bir yolda ise InsecureStorageExternal aktivite kaynak kodları incelenebilir ve  girilen bilgilerin kaydedildiği bölüm bulunur. Flag değeri açık metin olarak verildiğini görebiliriz.

![](../images/Pasted%20image%2020220626174128.png)


### 2.3 SQLite Database

![](../images/Pasted%20image%2020220626174841.png)


Herhangi bir parola değeri girdiğimiz zaman flag submit butonu belirmektedir. Burada ilgili aktivite InsecureStorageSQLite. jadx-gui aracı ile kaynak kodu incelendiğinde girilen parola ve flag değerleri kaydedilmektedir. Flag değeri *sqlite_string* isimli string kaynağıdır.


![](../images/Pasted%20image%2020220626174753.png)


*resource* olarak arama yaparsak string.xml dosyasının içerisinde flag değerine erişmiş oluruz.

![](../images/Pasted%20image%2020220626181043.png)


Bir diğer yöntem parolanın saklandığı dosyayı incelemek olacaktır (girilen hassas bilgi şifrelenmeden açık metin olarak kaydedilmişse bu güvenli olmayan bir yöntemdir), kod içerisinde bulunan parola doğrulama fonksiyonunda belirtilen regex gösterimine uygun karmaşık parola değeri girildiğinde **Password Saved** mesajı ekranda gösterilmektedir.

![](../images/Pasted%20image%2020220626181627.png)

`adb shell` ile cihaza erişim sağlandığında */data/data* dizini altında ilgili uygulama paketine gidilir.

![](../images/Pasted%20image%2020220626181804.png)

burada listelenen *databases* dizini altında uygulamaya ait veritabanı bulunmaktadır. `sqlite3 user.db` komutunu kullanarak veritabanına erişilir. `.tables` içerisindeki tablolar listelenir ve `SELECT * FROM users` sql querysi ile tablodaki veriler listelenmektedir. Listelenen değer **girilen_parola | flag_degeri** şeklinde kaydedilmiştir. Yukarıda incelenen kaynak kod içerisinde de bu şekilde kaydedildiğini görmüştük.

![](../images/Pasted%20image%2020220626181958.png)


## 3.  Webviews
Webview, uygulama içerisinde kullanıcıya web içeriklerini iletmek için kullanılan objeler olarak kulllanılır.

### 3.1 Load Arbitrary Url
AndroidManifest dosyası içerisinde VulnerablaWebView aktivitesinin export edilmiş olduğu görülmektedir.

![](../images/Pasted%20image%2020220711124658.png)

Burada aktivite içerisindeki kodlar incelendiğinde, `loadWebView()` fonksiyonu içerisinde ilgili intent içerisinde gelen string ile url bağlantısını gerçekleştiriyor[1]. Basitçe bir intent yardımıyla uygulamada çalışacak bir url stringi gönderiliyor. 

![](../images/Pasted%20image%2020220711125000.png)

Verilen ipucunda, `adb` ile zararlı sayfayı uygulama içerisinde açılabileceği belirtiliyor. `adb shell` komutu ile cihazda shell açtıktan sonra `am start` ( am = activity manager) komutununu kullanarak export edilmiş aktivite dışarıdan veya uygulamadan bağımsız olarak başlatılabilmektedir. `-n` parametresine paket_ismi / aktivite_yolu olarak değerler verilmektedir. `-es` parametresi extra değerler (getStringExtra(), getExtra() vs. ) için kullanılmaktadır. reg_url verilen url değerine bağlantı gerçekleştirilir.
 
>am start -n app.beetlebug/.ctf.VulnerableWebView -es reg_url "malicious.site"

![](../images/Pasted%20image%2020220711152757.png)

Yukarıdaki komutu çalıştırdıktan sonra uygulama içerisinde bağlantı adresine ait sayfanın açıldığı gözlemlenmektedir. Örneğin google.com adresini girdiğimiz zaman, uygulama görüntüsü şekildeki gibidir.

![](../images/Pasted%20image%2020220711153802.png)

Burada istenen flag değeri kod içerisinde de görüldüğü gibi `file:///android_asset/pwn.html` sayfasının içerisinde bulunmaktadır. (Güncelleme aktif olarak verilen bağlantı adresinde görsel bulunmuyor)

![](../images/Pasted%20image%2020220711154001.png)

![](../images/Pasted%20image%2020220711154329.png)

Bu yöntem intent-filter yardımıyla export edilmiş bileşenler için çalışmayabilir. Aktivitenin direkt olarak exported edilmiş olması gerekir (androd:exported= true şeklinde).

### 3.2 Javascript Code Injection

Bu bölümde, javascript kodu çalıştırarak XSS zafiyetini kullanarak flag değerini elde edeceğiz. Basit bir şekilde `<script>alert("XSS");</script>` payload kullanılabilir.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/3964fa8f-1a64-4414-9545-9c8d1973faef)

Açılan pop-up mesajı kapattıktan sonra flag değerine erişebiliyoruz.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/c268640a-e43c-4d7e-a32e-502b1a9841b2)



## 4. Insecure Databases
### 4.1 SQL Injection
jadx ile SQLInjectionActivity kodları incelendiğinde aktivite içerisinde direkt olarak veritabanı oluşturulup, kullanıcıların manuel şekilde açık metin olarak eklendiği görülmektedir.

![](../images/Pasted%20image%2020220711163257.png)

Normalde diğer uygulamalar veya zararlı yazılımlarda okunur durumda olmayabilir. Kod obfuscated edilmiş olabilir veya verilen değerler encoded olabilir. Bu yüzden başka bir yoldan soruyu çözmeye çalışalım.

Burada, input olarak girilen değer, direkt olarak SQL sorgusu içerisine eklenmektedir. Bu durum sayesinde input olarak başka sorgular çalıştırılabilmektedir.

![](../images/Pasted%20image%2020220711164007.png)

`WHERE user = '"` kısmında eşitliğin sağlanması durumunda kullanıcı bilgileri çıktı olarak verilmektedir. Burada kendi payloadımızı da oluşturabiliriz veya internetten de bulup kullanabiliriz[2]. 

input değere `' or '1` payloadı girildiği zaman kullanıcılar listelenmektedir. beetle-bug kullanıcısı flag değerini içermektedir.

![](../images/Pasted%20image%2020220711165251.png)

### 4.2 Firebase Misconfiguration
Firebase veritabanının yanlış konfigure edilmesi kullanıcı bilgilerinin açığa çıkması ve yetkisiz erişimlere sebebiyet verebilir. Burada ilk kontrol edilecek kısım strings.xml dosyası olabilir. Çünkü burada yanlışlıkla bırakılmış bağlantı adresleri, api anahtarları ve token değerleri olabilir.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/5b055800-aea7-4e1b-ae98-cc1204eb11f6)

Bulduğumuz firebase proje adresini ipucunda da verilmiş olan /.json dizinine göz atalım.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/18a22b35-f22a-49f8-a2ab-e3eab69a4a29)

yapılan ayarları erişim bilgileri ve diğer hassas bilgilere erişim sağlanabildi. Eğer doğru bir şekilde yapılandırılsaydı içeriği görme iznine sahip olunmazdı ve "Permission Denied" hatası verebilir veya null olarak görülebilirdi. Firebase veritabanı enumeration ve keşif aşaması için çeşitli araçlar kullanılmaktadır [3].

## 5. Android Components
### 5.1 Unprotected Activity

![](../images/Pasted%20image%2020220711190628.png)

Login butonuna tıklandığında başka ekrana yönlendiren ve export edilmiş aktivite gerekli olduğu ipucu olarak verilmiş. Burada `adb` komutu ile export edilmiş aktivite başlatılabilmektedir. Farklı araçlar da kullanılabilir.

AndroidManifest dosyası içerisinde export edilmiş ilgili aktiviteyi araştıralım. `b33tlebugAdministrator` isimli bir aktivite var.

![](../images/Pasted%20image%2020220711191127.png)

`adb shell am start -n app.beetlebug/.ctf.b33tleAdministrator` komutunu çalıştırıyoruz 

![](../images/Pasted%20image%2020220711191359.png)

![](../images/Pasted%20image%2020220711191421.png)

Admin paneli ekranı içerisinde en alt kısımda flag değeri bulunmaktadır. b33tlebugAdministrator aktivitesi exported olduğu için uygulamanın akışından bağımsız çalıştırarak farklı bölümlere erişim sağlayabiliyoruz.

### 5.2 Vulnerable Service
Servis bileşeni arkaplanda çalışan bir arayüze sahip olmayan programlardır. Başka uygulamalar tarafından çağrılabilir, dışarıdan erişilebilir özelliklerine sahiptir. Yanlış kullanımı zafiyetlere yol açabilir. exported özelliği aktif halde olması uygulama dışından da erişilebileceğini belirtir.

`adb shell am startservice -n app.beetlebug/.handlers.VulnerableService ` komutu ile komut ekranından servisi başlatabiliriz.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/ac921cfb-fb5c-4db2-98eb-019b209a3588)

### 5.3 Vulnerable Content Provider
Content provider (içerik sağlayıcısı), uygulamalar arası veri paylaşımını sağlar, erişim kontrollerinin yapıldığı yer olarak belirtilebilir. Paketin ismini öğrendikten sonra `dumpsys` ile sağlayıcılar listelenebilir.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/f40f22d0-a6b1-4577-b24f-01b54b632b61)

Manifest dosyası içerisinden de görülebilir bu bileşenler.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/cbaf486a-8456-4223-b55b-a4329586c731)

Bu bileşenin içeriğinde URL bağlantısı verilmiş
![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/daf42399-d084-44be-96c1-87f119e5c8e7)

adb shell ile cihazın terminaline eriştikten sonra `content query --uri content://CONTENT_URL` şeklinde paylaşılan veriler görülebiliyor.  

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/6359509e-0876-4881-ab42-869f87028079)

Bu işlemi otomatik gerçekleştiren araçlar ile de yapabiliriz. Drozer bunlardan bir tanesi [4]

## 6. Information Disclosure (Bilgi İfşası)
### 6.1 Güvensiz Loglama
Çalışan uygulamnın kayıtlarını `adb logcat` komutu ile inceleyebiliriz. Bu kayıtlara erişim olduğundan içerisinde hassas bilgilerin bulunması bilgi ifşasına yol açmaktadır. Bu senaryoda kredi kart bilgilerinin girilerek ödeme yapılması istenmektedir.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/7b2663be-72e8-492c-ac39-a096b2a80139)

Bilgiler girilip, öde butonuna tıklandığında gösterilen kayıtlar içerisinde girilmiş kart bilgisi de yer almaktadır.

![image](https://github.com/arzuozkan/AndroidSecurityNotes/assets/48025290/ddd8defe-9ea8-4e7f-929f-7dffcdac5cde)

### 6.2 Clipboard Data (Pano Verileri)

Android 9 ve altı sürümlerde pano verilerine tam erişime izin veriliyordu. Android 10 ile birlikte pano verilerine erişim kısıtı geldi.

## Kaynaklar
1. https://medium.com/mobis3c/exploiting-android-webview-vulnerabilities-e2bcff780892
2. https://github.com/payloadbox/sql-injection-payload-list
3. https://cloud.hacktricks.xyz/pentesting-cloud/gcp-security/gcp-services/gcp-firebase-enum
4. https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/drozer-tutorial/exploiting-content-providers
