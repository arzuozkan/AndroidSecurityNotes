# Beetlebug
[Beetlebug](https://github.com/hafiz-ng/Beetlebug), içerisinde android uygulama güvenliği ile ilgili konuları içeren zafiyetli bir android CTF uygulamasıdır. Uygulama açıldığında bir kullanıcı ismi girerek uygulamaya giriş yapılabilir. Toplamda 16 adet flag değeri bulunmaktadır.

![](../../images/Pasted%20image%2020220626153010.png)

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

![](../../images/Pasted%20image%2020220626153842.png)

jadx-gui ile apk uygulama dosyasını açıyoruz. Hangi aktivitenin ilk challange ait olduğunu ilk başta anlamayabiliriz.Burada res/value dizininin içerisindeki strings.xml dosyası incelendiğinde listelenen ilk değer dikkatimizi çekebilir.

![](../../images/Pasted%20image%2020220626155832.png)

genel bir arama yaparsak,

![](../../images/Pasted%20image%2020220626155551.png)

ilgili ekranın EmbeddedSecretStrings isimli aktivite olduğu anlaşılabilir. Aktivite içerisindeki kodlar incelendiğinde burada kullanıcıdan alınan girdi, PIN değeri ile eşit olup olmadığı koşulu bulunmaktadır. Buradaki karşılaştırılan string değeri (`R.string.V98bFQrpGkDJ`) istenen PIN değeridir.

![](../../images/Pasted%20image%2020220626160223.png)

Bulduğumuz PIN değerini girdiğimiz zaman ilk flag değerini bulmuş oluyoruz.

![](../../images/Pasted%20image%2020220626160453.png)

### 1.2 Hardcoding Secrets
Bu challange içerisinde hassas bilgilerin açık metin şekilde kodlanmasını içermektedir. Burada promosyon kodunu bulmamız gerekiyor.

![](../../images/Pasted%20image%2020220626160806.png)

jadx-gui aracını kullanarak kaynak kod incelemesinde, EmbeddedSecretSourceCode isimli aktiviteyi inceliyoruz. Promosyon kodu direkt olarak, değer şeklinde tanımlanmış durumda.

![](../../images/Pasted%20image%2020220626161118.png)

Bulunan kod girildiğinde flag değeri elde edilmiş oluyor.

## 2. Data Storage
### 2.1 Shared Preferences
Shared Preferences yapısı anahtar-değer(key-value) şeklinde küçük verileri depolamak için kullanılır.Verileri /data/data/paket_ismi/shared_prefs dizini altında bir xml dosyasına kaydeder. Burada kullanıcı adı ve parola girilmesi istenmektedir.

![](../../images/Pasted%20image%2020220626170811.png)

Bilgiler girdikten sonra **Save** butonuna kaydedildiğinde ekranın alt kısmında flag değeri için submit butonu ortaya çıkmaktadır.

![](../../images/Pasted%20image%2020220626171113.png)

Terminal üzerinden `adb shell` komutunu kullanarak cihaz üzerine bağlanabiliriz. */data/data* dizini altında ilgili uygulama paketine giderek *shared_prefs* klasörü içerisinde shared_pref_flag isimli xml dosya içeriği görüntülendiğinde girilen kullanıcı ve parola değerlerinin yanı sıra flag değeri de depolanmaktadır.

![](../../images/Pasted%20image%2020220626171547.png)

### 2.2 External Storage

![](../../images/Pasted%20image%2020220626172149.png)

Burada mail adresi ve parolayı kaydettiğimiz zaman ekranda bir toast mesajı belirmektedir. Verilerin saklandığı dosya yolu ve dosya ismi verilmiştir. Terminal üzerinden cihaza `adb shell` komutu ile erişim sağlanmaktadır. 

**Note: *storage/emulated/0/documents/* dizini içerisinde user.txt dosyası oluşmamış 
#TODO sebebini araştırcam

user.txt dosyasını bulamadım. InsecureStorageExternal aktivite kaynak kodları incelendiğinde girilen bilgilerin kaydedildiği bölüm bulunmaktadır. Flag değeri açık metin olarak verilmektedir.

![](../../images/Pasted%20image%2020220626174128.png)


### 2.3 SQLite Database

![](../../images/Pasted%20image%2020220626174841.png)


Herhangi bir parola değeri girdiğimiz zaman flag submit butonu belirmektedir. Burada ilgili aktivite InsecureStorageSQLite. jadx-gui aracı ile kaynak kodu incelendiğinde girilen parola ve flag değerleri kaydedilmektedir. Flag değeri *sqlite_string* isimli string kaynağıdır.


![](../../images/Pasted%20image%2020220626174753.png)


*resource* olarak arama yaparsak string.xml dosyasının içerisinde flag değerine erişmiş oluruz.

![](../../images/Pasted%20image%2020220626181043.png)


Bir diğer yöntem parolanın saklandığı dosyayı incelemek olacaktır (girilen hassas bilgi şifrelenmeden açık metin olarak kaydedilmişse bu güvenli olmayan bir yöntemdir), kod içerisinde bulunan parola doğrulama fonksiyonunda belirtilen regex gösterimine uygun karmaşık parola değeri girildiğinde **Password Saved** mesajı ekranda gösterilmektedir.

![](../../images/Pasted%20image%2020220626181627.png)

`adb shell` ile cihaza erişim sağlandığında */data/data* dizini altında ilgili uygulama paketine gidilir.

![](../../images/Pasted%20image%2020220626181804.png)

burada listelenen *databases* dizini altında uygulamaya ait veritabanı bulunmaktadır. `sqlite3 user.db` komutunu kullanarak veritabanına erişilir. `.tables` içerisindeki tablolar listelenir ve `SELECT * FROM users` sql querysi ile tablodaki veriler listelenmektedir. Listelenen değer **girilen_parola | flag_degeri** şeklinde kaydedilmiştir. Yukarıda incelenen kaynak kod içerisinde de bu şekilde kaydedildiğini görmüştük.

![](../../images/Pasted%20image%2020220626181958.png)


## 3.  Webviews
### 3.1 Load Arbitrary Url
### 3.2 Javascript Code Injection


