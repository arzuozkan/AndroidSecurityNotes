# KGB Messenger
Uygulamayı açmadan önce manifest dosyası içerisinden paket ismi, uyumlu android sürümü gibi bilgileri edinebiliriz.

**jadx** aracı ile apk dosyasını açıyorum. Hedef sdk versiyonu 25 ve paket ismi **com.tlamb96.spetsnazmessenger** .

![](../../../images/Pasted%20image%2020220322190827.png)

İlgili dosya dizinine gelindiğinde 3 adet aktivitenin bulunduğunu görebiliriz.

![](../../../images/Pasted%20image%2020220322191311.png)

MainActivity kodları incelendiğinde `onCreate` içerisinde sistem ortam değişkenleri kontrol edilmektedir. 
![](../../../images/Pasted%20image%2020220322191523.png)

Uygulama çalıştırıldığında da **Integrity Error** şeklinde görseldeki gibi bir hata mesaj gösterilmektedir.

![](../../../images/Pasted%20image%2020220322190207.png)

koşul ile kontrol edilen ifadenin koşulu sağlaması ile ekrana erişim sağlanmaktadır. Bunu atlatmanın birkaç yöntemi bulunmaktadır. Bunlardan biri de frida aracını kullanarak ilgili metodu ihtiyaca yönelik değiştirmek.

Frida sunucu-istemci şeklinde çalışmakatadır. Android cihaz içerisinde de frida sunucusunun çalıştığından emin olmak gerekir. Frida modülünü kullanarak python içerisinde metodu hooklamak için js kodları çalıştırılabilir.

![](Pasted%20image%2020220323121517.png)

`frida.get_usb_device()` usb cihaza bağlanmak için kullanılır. `attach()` parametre olarak verilen processe bağlanır.  Eğer uygulamayı kendimiz başlatmak istersek `spawn()` kullanabiliriz.

![](Pasted%20image%2020220323122836.png)


Değiştirilmesi gereken metotlar içerisinde,**System.getProperty**() ve **System.getenv("USER")** yani **R.string.USER**  değerini sağlamak gereklidir.

`getProperty` metodunu istenen değeri return edecek şekilde yeniden düzenliyoruz.

![](Pasted%20image%2020220323210522.png)

Bu şekilde çalıştırıldığında uygulama şekildeki hatayı verecektir. Bu hatayı atlatmak için [görseldeki](../../../images/Pasted%20image%2020220322191523.png) ilk **else if** koşulundaki  `R.string.USER` değerini almak gerekmektedir. 

![](Pasted%20image%2020220323210805.png)

`getString` metodu içerisinden koşulda eşitliği sağlayan string değeri return edilecektir.

![](Pasted%20image%2020220323212159.png)

python kodu çalıştırıldığında terminal çıktısında return edilen değeri görebiliriz.

![](Pasted%20image%2020220323210413.png)

Elde edilen değeri `getenv()` içerisinde rastgele return edilen değer yerine yazılıp tekrar çalıştırıldığında uygulama içerisinde giriş sayfasına ulaşılmaktadır.

![](../../../images/Pasted%20image%2020220323212934.png)

jadx ile LoginActivity sınıfı incelendiğinde giriş bilgilerinin **R.string.username** ve **R.string.password** şeklinde saklandığı anlaşılmaktadır. 

![](../../../images/Pasted%20image%2020220323213840.png)

Görseldeki dizin takip edilerek strings.xml dosyası bulunabilir. Uygulama içerisinde kullanılan metinleri içermektedir. 

![](../../../images/Pasted%20image%2020220323213725.png)
![](../../../images/Pasted%20image%2020220324093004.png)

Bulunan giriş değerlerini ekrana girdiğimiz zaman **Incorrect Password** 
Password olarak kaydedilen değer hash değeri 
olabilir ama 30 karakterli. Daha önce çözümü anlatılan blog yazılarında hash değerinin eksik olduğu görülmüştür. Çözüm olarak kullanıcı adını tarayıcı da aratarak şifre bulma yöntemi uygulanmıştır.

![](../../../images/Pasted%20image%2020220324111147.png)

Aramada çıkan sonuçlar içerisinde çıkan PDF dosyası içerisinde şifre yer almaktadır.

![](../../../images/Pasted%20image%2020220324111228.png)

Giriş yapıldığında mesajların olduğu bir ekran açılarak Toast mesajından flag değeri gösterilmektedir.

![](../../../images/Pasted%20image%2020220324112928.png)





# Kaynaklar
- [Android reverse engineering for beginners - Frida](https://braincoke.fr/blog/2021/03/android-reverse-engineering-for-beginners-frida/#about-frida) 
