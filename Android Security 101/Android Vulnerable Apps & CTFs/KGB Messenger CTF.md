# KGB Messenger
Uygulamayı açmadan önce manifest dosyası içerisinden paket ismi, uyumlu android sürümü gibi bilgileri edinebiliriz.

**jadx** aracı ile apk dosyasını açıyorum. Hedef sdk versiyonu 25 ve paket ismi **com.tlamb96.spetsnazmessenger** .

![](../../images/Pasted%20image%2020220322190827.png)

İlgili dosya dizinine gelindiğinde 3 adet aktivitenin bulunduğunu görebiliriz.

![](../../images/Pasted%20image%2020220322191311.png)

MainActivity kodları incelendiğinde `onCreate` içerisinde sistem ortam değişkenleri kontrol edilmektedir. 
![](../../images/Pasted%20image%2020220322191523.png)

Uygulama çalıştırıldığında da **Integrity Error** şeklinde görseldeki gibi bir hata mesaj gösterilmektedir.

![](../../images/Pasted%20image%2020220322190207.png)

koşul ile kontrol edilen ifadenin koşulu sağlaması ile ekrana erişim sağlanmaktadır. Bunu atlatmanın birkaç yöntemi bulunmaktadır. Bunlardan biri de frida aracını kullanarak kullanılan metodu ihtiyaca yönelik değiştirmek.


# Kaynaklar
- [Android reverse engineering for beginners - Frida](https://braincoke.fr/blog/2021/03/android-reverse-engineering-for-beginners-frida/#about-frida) 
