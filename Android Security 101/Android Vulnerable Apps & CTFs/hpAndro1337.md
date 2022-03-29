
Android uygulama güvenliği ile ilgili ctf tarzı birçok konuyu içeren challangeları bulunmaktadır.

**Github repo:** https://github.com/RavikumarRamesh/hpAndro1337

**Website:** http://ctf.hpandro.raviramesh.info/challenges

Uygulama şekildeki ekrana sahiptir. Listelenen konular ile ilgili belirli görevler bulunur ve istenen flag değerleri websitesi içerisinde ilgili yere girilmektedir.

![](../../images/Pasted%20image%2020220326200341.png)


# Android Security
Bu kategori içerisinde bir görev bulunmamakla beraber genel bilgilendirme yapılmaktadır.

# HTTP Traffic
Bu kategoride trafik analizi ile ilgili ctf soruları bulunmaktadır. 2 soru mevcuttur.

## HTTP Traffic
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

## HTTPS Traffic
Yine aynı şekilde https traffic sayfasını açıp istekleri takip ettiğimiz zaman gelen yanıt içerisinde flag değeri bulunabilecektir. Gelen isteğin okunabilmesi için burpsuite sertifikasının cihazda yüklü olması gerekmektedir.

![](../../images/Pasted%20image%2020220327124350.png)

# Root Detection
