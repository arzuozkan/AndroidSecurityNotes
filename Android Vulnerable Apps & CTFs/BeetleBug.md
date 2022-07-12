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

**Note: *storage/emulated/0/documents/* dizini içerisinde user.txt dosyası oluşmamış 
#TODO sebebini araştırcam

user.txt dosyasını bulamadım. InsecureStorageExternal aktivite kaynak kodları incelendiğinde girilen bilgilerin kaydedildiği bölüm bulunmaktadır. Flag değeri açık metin olarak verilmektedir.

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

Burada istenen flag değeri kod içerisinde de görüldüğü gibi `file:///android_asset/pwn.html` sayfasının içerisinde bulunmaktadır.

![](../images/Pasted%20image%2020220711154001.png)

![](../images/Pasted%20image%2020220711154329.png)

Bu yöntem intent-filter yardımıyla export edilmiş bileşenler için çalışmayabilir. Aktivitenin direkt olarak exported edilmiş olması gerekir (androd:exported= true şeklinde).

### 3.2 Javascript Code Injection
- [ ] TODO 

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
Firebase veritabanının yanlış konfigure edilmesi kullanıcı bilgilerine yetkisiz kullanıcının erişmesini sağlayabilir.

- [ ] TODO

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


### 5.3 Vulnerable Content Provider

## Kaynaklar
1. https://medium.com/mobis3c/exploiting-android-webview-vulnerabilities-e2bcff780892
2. https://github.com/payloadbox/sql-injection-payload-list
3. 