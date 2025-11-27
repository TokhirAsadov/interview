# 📚 ClassLoader Subsystem (Sinflarni Yuklash Tizimi) Tahlili

**ClassLoader Subsystem** JVM ning ajralmas qismi boʻlib, u sinflarning **Run-Time Data Area** (Xotira Maydonlari) ga dinamik tarzda yuklanishi, bogʻlanishi va boshlanishini boshqaradi.

Uning asosiy vazifasi — `$Fully Qualified Class Name` (Toʻliq Kvalifikatsiyalangan Sinf Nomi) orqali fayl tizimidan, tarmoqdan yoki boshqa manbalardan `.class` fayllarini topish va ularni JVM tushunadigan **`java.lang.Class`** obyektlariga aylantirishdir.

## 1. ClassLoader ning Uch Bosqichi

Sinflarni yuklash jarayoni mantiqiy ravishda uch asosiy bosqichga boʻlinadi:

### A. 🔄 Yuklash (Loading)

Bu bosqichda ClassLoader tarmoq yoki fayl tizimi kabi manbalardan bayt-kodni oladi va uni JVM ga oʻqiydi.

1.  **Topish:** ClassLoader soʻralgan sinfga mos keladigan `.class` faylini **delegatsiya modeli** yordamida izlaydi.
2.  **Oʻqish:** `.class` faylidagi bayt-kodni oʻqiydi.
3.  **Yaratish:** Oʻqilgan bayt-koddan **`java.lang.Class`** turidagi obyektni yaratadi va uni **Method Area** (metamaʼlumotlar uchun xotira maydoni) ga joylashtiradi.

### B. 🔗 Bogʻlash (Linking)

Bu bosqich sinfning JVM da toʻliq ishlashi uchun zarur boʻlgan asosiy tekshiruv va tayyorgarliklarni oʻz ichiga oladi.

1.  **Tekshirish (Verify):**
    * Yuklangan sinfning bayt-kodi toʻgʻri formatlanganligini va sintaktik xatolarga ega emasligini tekshiradi.
    * Bu **JVM xavfsizligi** ning muhim qismi boʻlib, buzuq yoki zararli kodni ishga tushirishning oldini oladi.
2.  **Tayyorlash (Prepare):**
    * Sinfning **statik oʻzgaruvchilari** uchun **Method Area** da xotira ajratadi.
    * Bu oʻzgaruvchilarga standart boshlangʻich qiymatlarni beradi (masalan, `int` uchun `0`, obyekt havolalari uchun `null`). *Dasturchi bergan qiymat hali tayinlanmaydi.*
3.  **Hal Qilish (Resolve):**
    * Sinfdagi **ramziy havolalarni** (Symbolic References) toʻgʻridan-toʻgʻri **haqiqiy xotira manzillariga** (Direct References) aylantiradi.
    * Masalan, `System.out.println()` chaqirilganida, "System.out.println" nomli ramziy havola JVM dagi metodning haqiqiy manziliga bogʻlanadi.

### C. 🚀 Boshlash (Initialization)

Bu sinfni ishga tushirish jarayonining oxirgi bosqichidir.

1.  **Qiymat Berish:** Tayyorgarlik bosqichida standart qiymat berilgan **statik oʻzgaruvchilarga** dasturchi belgilagan haqiqiy qiymatlar beriladi.
2.  **Statik Bloklar:** Sinfdagi barcha **statik bloklar** ijro etiladi.
3.  Sinfning boshlanishi, u birinchi marta faol ishlatilishidan oldin (masalan, uning obyektini yaratish yoki statik metodini chaqirish) sodir boʻladi.

---

## 2. 🏛️ ClassLoader Iyerarxiyasi va Delegatsiya Modeli

JVM uchta asosiy ClassLoader ni ishlatadi, ular qatʼiy **delegatsiya iyerarxiyasi** ga amal qiladi:

| ClassLoader Turi | Vazifasi va Yuklash Manbasi | Parent Loader |
| :--- | :--- | :--- |
| **1. Bootstrap ClassLoader** | **JVM ni yuklaydi.** Java Runtime Environment (JRE) ning asosiy kutubxonalarini (`rt.jar`, `core` sinflar) yuklaydi. | N/A (Null) |
| **2. Extension ClassLoader** | JDK ning **`ext`** (extension) katalogidagi sinflarni yuklaydi. | Bootstrap |
| **3. Application (System) ClassLoader** | Loyihaning oʻz sinflarini (`CLASSPATH` da belgilangan) yuklaydi. | Extension |

### Delegatsiya Prinsipi

Sinfni yuklash talabi (masalan, `new MyClass()`) paydo boʻlganda, har doim **Application ClassLoader** dan boshlanadi.

1.  **Application ClassLoader** sinfni yuklashni oʻzining **Parent** (Extension ClassLoader) ga topshiradi.
2.  **Extension ClassLoader** oʻz navbatida yuklashni oʻzining **Parent** (Bootstrap ClassLoader) ga topshiradi.
3.  **Bootstrap ClassLoader** yuklashga urinadi. Agar u topa olsa, sinfni yuklaydi.
4.  Agar **Bootstrap** topa olmasa, u yuklashni **Extension ClassLoader** ga qaytaradi.
5.  Agar **Extension** ham topa olmasa, u yuklashni **Application ClassLoader** ga qaytaradi.
6.  Faqat shu bosqichda **Application ClassLoader** oʻzining manbalaridan (CLASSPATH) sinfni yuklashga urinadi.
7.  Agar barcha urinishlar muvaffaqiyatsiz tugasa, **`ClassNotFoundException`** tashlanadi.

Bu model asosiy Java API sinflarining (masalan, `java.lang.String`) loyiha sinflari tomonidan notoʻgʻri oʻzgartirilishining oldini oladi va **JVM mustahkamligini** taʼminlaydi.

## 3. 🛡️ ClassLoader ning Seniorlar Uchun Axamiyati

* **Custom ClassLoaders:** Murakkab tizimlarda (masalan, Plugin arxitekturalari, Application Serverlar) foydalanuvchiga **maxsus ClassLoader** yaratish zarur boʻlishi mumkin. Bu, turli versiyadagi kutubxonalarni bir xil JVM da ajratib turish (Isolation) uchun muhim.
* **Hot Deployment:** Maxsus ClassLoaderlar yordamida ilovani qayta ishga tushirmasdan yangi sinf versiyalarini dinamik ravishda yuklash mumkin.
* **JNDI/JDBC Drajverlarida:** Koʻpincha turli ClassLoaderlar orasidagi koʻrinish (Visibility) muammolari yuzaga keladi. Bu muammolarni hal qilish uchun **Thread Context ClassLoader (TCCL)** tushunchasi ishlatiladi, u yuklashni qandaydir **Current Thread** ga bogʻliq ClassLoader ga topshiradi.
