# hpAndro1337 ctf

Android uygulama güvenliği ile ilgili ctf tarzı birçok konuyu içeren challangeları bulunmaktadır.

**Github repo:** https://github.com/RavikumarRamesh/hpAndro1337

**Website:** http://ctf.hpandro.raviramesh.info/challenges

Uygulama şekildeki ekrana sahiptir. Listelenen konular ile ilgili belirli görevler bulunur ve istenen flag değerleri websitesi içerisinde ilgili yere girilmektedir.

![](../../images/Pasted%20image%2020220326200341.png)


## Android Security
Bu kategori içerisinde bir görev bulunmamakla beraber genel bilgilendirme yapılmaktadır.

## HTTP Traffic
Bu kategoride trafik analizi ile ilgili ctf soruları bulunmaktadır. 2 soru mevcuttur.

### HTTP Traffic

![](../../images/Pasted%20image%2020220326202459.png)

Bu görevde HTTP ağ trafiğini izleyip flag değerini bulmamız isteniyor. Basit bir burpsuite kullanımı bu problemi çözmeye yardımcı olacaktır. Burpsuite konfigurasyonlarının yapıldığı varsayılmıştır.

Android cihaz üzerinde uygulama başlatılıyor. Http traffic, kısmını açtığımızda görseldeki ekran ile karşılaşıyoruz.

![](../../images/Pasted%20image%2020220327120957.png)

Burpsuite aracı başlatılmaktadır. Proxy kısmına gelerek "Intercept on" ayarı açılmaktadır.

![](../../images/Pasted%20image%2020220327120704.png)

Eğer yapılan istekler gösterilmiyorsa "Options" seçeneğinden doğru arayüzün seçildiğini kontrol edebilirsiniz. Burada tüm arayüzler için 8080 portundan dinleme gerçekleşmektedir.

![](../../images/Pasted%20image%2020220327122041.png)

Gelen isteği "Forward" ile gönderdkten sonra "HTTP history" penceresinden yapılan istekler ve gelen yanıtlar listelenmektedir. Gelen yanıt içeriği incelendiğinde "Flag" isimli başlık bilgisi bulunmaktadır. Bu da istenen flag değeridir.

![](../../images/Pasted%20image%2020220327122553.png)

### HTTPS Traffic
Yine aynı şekilde https traffic sayfasını açıp istekleri takip ettiğimiz zaman gelen yanıt içerisinde flag değeri bulunabilecektir. Gelen isteğin okunabilmesi için burpsuite sertifikasının cihazda yüklü olması gerekmektedir.

![](../../images/Pasted%20image%2020220327124350.png)

## Root Detection
Cihaz root tespitinde kullanılan yöntemler [Other Topics](../AndroidSec.md#Other%20Topics) içerisinde detaylı olarak bahsedilmektedir. Root cihaz tespitini atlatmak için birçok araç ve yöntem bulunmaktadır. En basit yöntem frida ile metotları hooklamak. Bunun için yazılmış olan frida scriptleri mevcut. 
Root tespiti atlatmak için ben burada [9 Qu1cksc0pe All-in-One malware analysis tool ](../AndroidSec.md#9%20Qu1cksc0pe%20All-in-One%20malware%20analysis%20tool) isimli dosya analiz aracı içerisindeki runtime modülünü kullandım. Bu modül sayesinde apk dosyalarının dinamik olarak analizi yapılabilmektedir.

- Root Management task

![](../../images/Pasted%20image%2020220420041720.png)

- BusyBox binaires

![](../../images/Pasted%20image%2020220420041823.png)

- SU binary 

![](../../images/Pasted%20image%2020220420041916.png)

- Test-keys 

![](../../images/Pasted%20image%2020220420044919.png)

- Tehlikeli sistem özellikleri

![](../../images/Pasted%20image%2020220420050635.png)

bu şekilde bazı atlatma görevleri bulunmktadır.

>sudo python3 qu1icksc0pe.py --runtime

komutunu çalıştırdıktan sonra cihazları listeleyip bir tanesini seçmemizi istiyor.

![](../../images/Pasted%20image%2020220420043413.png)

Sonrasında hedef uygulamanın paket ismi seçiliyor.

![](../../images/Pasted%20image%2020220420043507.png)

Bu aşamadan sonra birçok frida scriptlerini listeleyerek kullanılacak olan seçiliyor. Burada cihaz frida sunucusunu çalıştırıyor olması gerekiyor aksi durumda hata mesajı alınabilir bu konuda.

![](../../images/Pasted%20image%2020220420044610.png)

Uygulamanın ilgili tasklarını tek tek açalım. Root management ve test-keys görevleri atlatılamadı.
Bypass edilen metotlar,
- busybox binaries

![](../../images/Pasted%20image%2020220420045111.png)

- SU binaries

![](../../images/Pasted%20image%2020220420050242.png)

- Dangerous props

![](../../images/Pasted%20image%2020220420045735.png)

[antiroot.js](https://gist.github.com/pich4ya/0b2a8592d3c8d5df9c34b8d185d2ea35) root tespitini bypasslamak için geliştirilmiş frida script. Frida aracını kullanarak script çalıştırılıyor ancak root management görevi için **com.genymotion.superuser** tespiti gösterilmekteydi. Script içerisine bu paketi de ekliyoruz.

![](../../images/Pasted%20image%2020220420055356.png)

>frida -l antiroot.js -U -f com.hpandro.androidsecurity --no-pause

komutu kullanılarak hooklama işlemini başlatıyoruz. Terminal üzerinde de takip edilirse paket değerini değiştirerek superuser paketinin varlığı bypass edilmiş oluyor.

![](../../images/Pasted%20image%2020220420055719.png)

cihaz üzerinde de check root butonuna tıklandığında flag ekranına yönlendirmektedir.

![](../../images/Pasted%20image%2020220420055853.png)

Test-keys görevi için cihazdaki build.tags değeri görseldeki gibidir. 

![](../../images/Pasted%20image%2020220420061748.png)

- [ ] #TODO testkeys bypass


## Insecure Data Storage
Android uygulama içerisinde birden çok depolama yöntemi bulunmaktadır. Shared Preferences, SQLite veritabanı, Firebase veritabanı ve harici/dahili kullanılan yöntemlerden birkaçıdır. 
İlk görev içerisinde SQLite veritabanına kaydedilen flag değerinin bulunması istenmektedir. Görev ekranında "Check sqlite flag" butonu bulunmaktadır.

![](../../images/Pasted%20image%2020220501040840.png)

Butona tıklandığında flag değerinin sqlite veritabanına eklendiği ile ilgili mesaj verilmektedir.

![](../../images/Pasted%20image%2020220501041003.png)

![](../../images/Pasted%20image%2020220501041051.png)

Terminal üzerinden `adb shell` komutu ile shell açtıktan sonra **/data/data/** dizininin altında yer alan uygulama paket klasörüne gidilerek databases içerisindeki veritabanı dosyalara erişilmektedir.

![](../../images/Pasted%20image%2020220501041612.png)

databases dizini içerisinde db uzantılı dosyalar ve uygulamanın kullanacağı geçici olan journal gibi dosyalar bulunabilir. İçerisini incelersek okunabilir olmayacaktır. Bunun için `sqlite3 AndroidSecurity.db` komutu ile veritabanını açılabilir.

![](../../images/Pasted%20image%2020220501042328.png)

Flag için bakılacak tablo Flags isimli tablodur. `SELECT * FROM Flags;` sorgusu çalıştırılarak tablo içerisindeki değerler listelenmektedir.

![](../../images/Pasted%20image%2020220501042524.png)

Burada birden fazla butona tıkladığım için flag değerini birden fazla kaydetmiştir. 

- [ ] #TODO EncSqlite

## Web Application
### HTML5 control
Görevde oturumda depolanmış(session storage) flag değerinin bulunması istenmektedir.  İlk başta hiç araştırma yapmadan burp proxy kullanarak task sayfasına girdim.

![](../../images/Pasted%20image%2020220507064817.png)


Yapılan istek GET  /web/task-html5.php şeklinde. Gelen yanıt değerine bakıldığında içeriğinde html sayfa kodlarını içeriyor. script tagları arasında eval() fonksiyonu içerisinde bi şeyler çalıştırılmış.

![](../../images/Pasted%20image%2020220507065420.png)

 Bu js kodu çalıştırıldığında flag değerini içeren bir hata mesajı verilmektedir.
 
![](../../images/Pasted%20image%2020220507064557.png)

### Bruteforce
Görevde bir login ekranı ile karşılaşılmaktadır. Kullanıcı adının admin olduğu söylenmiş ve bruteforce yöntemini kullanarak şifrenin bulunması gerekiyor.

![](../../images/Pasted%20image%2020220507065954.png)

Random bir parola değeri girildiğinde 200 OK yanıtı dönüp ekranda tekrar deneyin mesajı göstermektedir. Parola listesindekileri kullanarak burp intruder ile bruteforce yapılabilmektedir.

![](../../images/Pasted%20image%2020220507071822.png)


