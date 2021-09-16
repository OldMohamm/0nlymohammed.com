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
<br>```SSL Pinnig:``` <br>
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
- Modifying the return value

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

وبكذا بعد ماعرفنا خلونا نطبق :


