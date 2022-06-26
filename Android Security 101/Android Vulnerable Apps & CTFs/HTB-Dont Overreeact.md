İndirilen zip dosyasını **hackhebox** şifresi ile arşivden çıkardıktan sonra, app-release.apk dosyasını

> adb devices 

komutu ile genymotion emülatörüne bağlantıyı tespit edebiliriz. Eğer bağlı cihaz bulunmuyorsa

> adb connect TELEFONIP:PORT

şeklinde bağlantı gerçekleştirilebilir. Sonraki işlemde uygulamamızı cihaza yüklemeden önce Manifest dosyasını kontrol etmekte fayda var. Uyumlu android sürümü gibi bilgileri elde ediyoruz.

jadx aracı ile apk dosyası açılıyor. Hedef sdk versiyonu 29, burada kullanılan emülatör Google Pixel 3 - 10.0 API 29.

![](../../images/Pasted%20image%2020220419050514.png)

Bir adet aktivite mevcut. Paketin ismi,  **com.awesomeproject.MainActivity**. 

>adb install app-release.apk 

komutu ile uygulama emülatöre yüklenmektedir.

![](../../images/Pasted%20image%2020220419050821.png)

Uygulama açıldığında aşağıdaki ekran görüntülenmektedir.

![](../../images/Pasted%20image%2020220419051002.png)

MainActivity içerisinde **getMainComponentName()** fonksiyonu var. 

![](../../images/Pasted%20image%2020220419051502.png)


- [ ] #TODO React reversing search 
