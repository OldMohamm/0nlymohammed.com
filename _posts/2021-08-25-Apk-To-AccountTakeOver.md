---
title: The journey from (.apk) to Account TakeOver
author: Mesh3l_911
date: 2021-08-25 18:32:00 -0500
categories: [Blogging, Tutorial]
tags: [Apk, TakeOver]
---


<p class="aligncenter">
    <img src="/pics/LOGO.png" alt="centered image" />
</p>

<center><b> ^_^ السلام عليكم ورحمة الله وبركاته, يالله حيهم </b></center><br> 

> <html><body><b><p style="color:#A52A2A;font-size:25px">Agenda:</p></b></body></html>

<ul><li><b>SSL Pinning Bypass</b></li></ul>
<ul><li><b>Network Traffic Analysis</b></li></ul>
<ul><li><b>Basic Dynamic Analysis(Genymotion, adb, Frida, Objection)</b></li></ul>
<ul><li><b>Basic Static Analysis( Jadx-GUI )</b></li></ul>
<ul><li><b>Weak Encryption Algorithm</b></li></ul>
<ul><li><b>Password Generator(Crunch + Cusom Java script)</b></li></ul>
<ul><li><b>Account TakeOver</b></li></ul><br>

> <html><body><b><p style="color:#A52A2A;font-size:25px">SSL Pinning Bypass:</p></b></body></html>

<br>يب قبل نتكلم عن طرق التخطي خلونا ناخذ فكرة بسيطة وسطحية عن إيش هو من الأساس ؟
<br>```SSL Pinning:``` <br>
 بطرق مختلفة مثل(Pinned == Hard Coded) بداخل التطبيق تكون trustful certificates بكل بساطة خلونا نقول فيه  
- Certificate
- Public Key
- Hash

Conection Establishment وفي خلال عملية الـ
 <br>
 الكونيكشن بيتعطل MisMatch وإذا كان فيه أي Certificate  يقوم الكلاينت سايد بالتأكد من الـ  

طيب بعد ماأخذنا نبذة سطحية عن الموضوع (الموضوع أكبر من كذا بكثير جدا) خلونا الحين نذكر بعض طرق التخطي
 تصير لازم بطريقة أو بأخرى نوقف العملية هذي علشان نصير قادرين نكون طرف ثالث في منتصف الكونيكشن ونقدر نعترض الترافيك Check زي ماقلنا فوق فيه عملية 
 
:التالي Process فيه بعض الطرق تعتمد على الـ 
- Identifying the CLASS
- Identifying the method
- Finding the return value
- Hooking the method and modifying the return value

من هذي الطرق 
- SSLUnpinning - Xposed Module ( تطبيقات جاهزة وهذا سيناريو اليوم )
- Objection 
- Custom Frida Scripts

الطريقتين الثانية والثالثة بإذن الله بشرحها مستقبلا , وهي إن شاء الله سهلة في حال الوصول للدالة المعنيًة 

<br>طيب ندخل في سيناريو اليوم , زي ماذكرت استخدمنا الطريقة الأولى بس قبل مانستخدمها على عماها خلونا نفهم وش اللي يصير خلف الكواليس 
 UnSSLPinning هنا كلام من الريبو الرسمي الخاص ببروجكت
 
 ``` 
If you need to intercept the traffic from an app which uses certificate pinning, with a tool like Burp Proxy, the SSLUnpinning will help you with this hard work! The SSLUnpinning through Xposed Framework, makes several hooks in SSL classes to bypass the certificate verifications for one specific app, then you can intercept all your traffic
```
طيب معليكم من فلسفتهم الزايدة , بكل إختصار لو نشوف السورس كود الخاص بالبروجكت
<br>
 ويطبق عليه البروسيس المذكور فوق Common Libraries  هو يعتمد بشكل كامل على بعض الـ 

okhttp3 ومنها اللي كانت موجودة بسيناريو اليوم
<br>
 decompiled apk source code طيب كيف عرفنا التطبيق يستخدمها ؟ عن طريق الـ  
<br>
<br>
:<b>وبكذا بعد ماعرفنا خلونا نطبق</b>
<br>
<br>
 SSL Pinning هنا أول ماإكتشفت إن فيه 
<br>

![](../../posts_pics/-1.png)
<br>
<br>
 راح نفهم في قسم الستاتيك اناليسس كيف طلعناها okhttp3 library وهنا نشوف الـ 
<br>
![](../../posts_pics/0.png)
<br>
وعشان نفهم أكثر كيف قدر البروجكت يتخطاها , زي ماقلت لكم بتطبيق البروسيس المذكور فوق
<br>
 بالسطرين 10 , 13 Check وهنا نشوف عملية الـ
<br>
```java
public void check(String hostname, List<Certificate> peerCertificates) 
throws SSLPeerUnverifiedException {
    List<Pin> pins = findMatchingPins(hostname);
    if (pins.isEmpty()) return;
 …
      for (int p = 0, pinsSize = pins.size(); p < pinsSize; p++) {
        Pin pin = pins.get(p);
        if (pin.hashAlgorithm.equals("sha256/")) {
          if (sha256 == null) sha256 = sha256(x509Certificate);
          if (pin.hash.equals(sha256)) return; // Success!
        } else if (pin.hashAlgorithm.equals("sha1/")) {
          if (sha1 == null) sha1 = sha1(x509Certificate);
          if (pin.hash.equals(sha1)) return; // Success!
        } else {
          throw new AssertionError();
        }
      }
    }
…
}
```
<br>
<br>
SSLUnpinning - Xposed Module وهنا الـ 
<br>
![](../../posts_pics/1.png)
<br>
<br>
طريقة إستعماله سهلة جدا 
<br>
![](../../posts_pics/2_1.png)
<br>
<br>
وأخيرا 
<br>
![](../../posts_pics/3.png)
<br>
<br>

> <html><body><b><p style="color:#A52A2A;font-size:25px">Network Traffic Analysis:</p></b></body></html>

 :توصلت لثلاث أشياء API endpoints بعد فحص بعض الـ 
<br>
-  Rate Limit مافيه أي 
-  بمجرد عمل ريسيت للباسورد راح يتم عمل باسوورد جديد مكون من 4 أرقام فقط ( بإعتبار الشخص مفروض يدخل وبعدين يغيره )
-   Encryption صاير لها Request Body النقطة الأخيرة وهي اللي جابت كل الشغل اللي تحت كانت قيم المتغيرات فالـ  

فهنا كل اللي علينا نسوي ريسيت باسوورد على أي حساب وبعد مايصير الباسوورد مجرد 4 أرقام نبدأ بتخمينها بحكم إن مافيه ريت ليمت
<br>
Encrypted  لكن المشكلة قيم البارامترز كانت 
<br>
 مثل 0000 أو 9999 وأبدأ أخمن لأن الباك ايند سيرفر ماراح يتعرف عليها أبدأ  Clear Text Passwords بما معناه ماأقدر أحط 
 <br>
 بلشة صح ؟ اتفق 
 <br>
 <br>
  Encryption لاحظوا هنا قيمة الباسوورد , على فكرة أنا أدخلتها 0000 بس زي ماذكرت فيه عملية 
  <br>
  ![](../../posts_pics/3.png)
<br>
<br>
 Login endpoint وهنا للتأكيد مافيه أي ريت ليمت على الـ 
 <br>
   ![](../../posts_pics/4.png)
<br>
<br>

> <html><body><b><p style="color:#A52A2A;font-size:25px">Network Traffic Analysis:</p></b></body></html>