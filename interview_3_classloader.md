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

---

# 🏛️ ClassLoader Turlari va Iyerarxiyasi

JVM ClassLoaderlarning uchta asosiy turi mavjud. Ular qatʼiy **delegatsiya (delegation)** prinsipiga asoslangan iyerarxiyada ishlaydi.

| # | ClassLoader Turi | Java Klassi | Yuklash Manbasi (Qayerdan Yuklaydi) |
| :--- | :--- | :--- | :--- |
| **1.** | **Bootstrap ClassLoader** | N/A (Native Code) | JVM ning asosiy kutubxonalari: `$JAVA_HOME/jre/lib/rt.jar`, shuningdek, boshqa core kutubxonalar. |
| **2.** | **Extension ClassLoader** | `jdk.internal.loader.ClassLoaders$PlatformClassLoader` | JDK ning kengaytma katalogidagi sinflar: `$JAVA_HOME/jre/lib/ext` (Java 8 va undan past) yoki Platform Class Path (Java 9+). |
| **3.** | **Application (System) ClassLoader** | `jdk.internal.loader.ClassLoaders$AppClassLoader` | Loyiha sinflari va uchinchi tomon kutubxonalari: `CLASSPATH` atrof-muhit oʻzgaruvchisida koʻrsatilgan barcha manbalar. |

---

## 1. 🚀 Bootstrap ClassLoader (Boshlangʻich Yuklovchi)

* **Tuzilishi:** Bu boshqa ClassLoaderlardan farqli oʻlaroq, Java sinfi emas. U **Native Code** (odatda C/C++) da amalga oshirilgan va JVM ni yuklashda ishga tushiriladi.
* **Vazifasi:** Eng asosiy va fundamental Java sinflarini yuklash uchun javobgar. Bular qatoriga `java.lang.Object`, `java.lang.String` kabi barcha asosiy API sinflarini oʻz ichiga olgan `rt.jar` (RunTime JAR) kabi kutubxonalar kiradi.
* **Delegatsiya:** Bu iyerarxiyaning eng tepasi. Uning **Parent ClassLoaderi mavjud emas** (uni soʻraganda `null` qaytaradi).

## 2. 🔌 Extension ClassLoader (Kengaytma Yuklovchi)

* **Tuzilishi:** Bootstrap ClassLoader tomonidan yuklangan Java klassi.
* **Vazifasi:** Standart Java kengaytmalari yoki qoʻshimcha tizim sinflarini yuklaydi.
    * **Java 8 va undan past:** `$JAVA_HOME/jre/lib/ext` katalogida joylashgan JAR fayllarini yuklaydi.
    * **Java 9+:** Modul tizimiga oʻtish tufayli u **Platform ClassLoader** deb nomlanadi va Java platformasiga tegishli modullarni yuklaydi.
* **Delegatsiya:** U **Bootstrap ClassLoader** ga delegatsiya qiladi.

## 3. 🌐 Application ClassLoader (Ilova Yuklovchi)

* **Tuzilishi:** Extension ClassLoader tomonidan yuklangan Java klassi. Bu, dastur sinflarini yuklash uchun ishlatiladigan **standart ClassLoader**.
* **Vazifasi:** Loyihaning oʻz sinflarini va **CLASSPATH** orqali bogʻlangan barcha uchinchi tomon (third-party) kutubxonalarini (dependencies) yuklash uchun javobgar.
* **Delegatsiya:** U **Extension ClassLoader** ga delegatsiya qiladi.

---

## 🔑 ClassLoaderlar va Delegatsiya Prinsipi

Sinflarni yuklash talabi (masalan, `new com.example.MyClass()`) har doim iyerarxiyaning pastki qismidan, yaʼni **Application ClassLoader** dan boshlanadi.

**Ishlash Qadami:**

1.  **Talab:** Application ClassLoaderga `MyClass` ni yuklash talab qilinadi.
2.  **Delegatsiya:** Application ClassLoader yuklashni oʻzining Parent (Extension ClassLoader) ga topshiradi.
3.  **Yuqoriga Chiqish:** Extension ClassLoader oʻzining Parent (Bootstrap ClassLoader) ga topshiradi.
4.  **Harakat:** Bootstrap ClassLoader yuklashga urinadi. Agar topa olsa, yuklaydi va natijani pastga (Extension ga) qaytaradi.
5.  **Pastga Tushish:** Agar Bootstrap topa olmasa, Extension ClassLoader yuklashga urinadi.
6.  **Yakunlash:** Agar Extension ham topa olmasa, Application ClassLoader yuklashga urinadi. Agar u topa olsa, sinf yuklanadi. Aks holda, `ClassNotFoundException` tashlanadi.

Bu **"Ota-ona ClassLoaderni afzal bilish" (Parent-First Delegation)** modeli deb ataladi. Bu, loyiha sinflari tasodifan Java core sinflarini (masalan, `java.lang.String`ni) oʻzgartirishining oldini oladi va **JVM xavfsizligi**ni taʼminlaydi.

---

## 💡 Maxsus (Custom) ClassLoaderlar

Senior darajada, ushbu uchta standart ClassLoaderdan tashqari, biz oʻz ClassLoaderlarimizni yaratishimiz mumkin:

* **Vazifasi:** Java sinfini `.class` faylidan emas, balki tarmoqdan, DB dan yoki shifrlangan manbalardan yuklash.
* **Qoʻllanilishi:**
    * **Plugin tizimlari:** Har bir plugin oʻzining kutubxonalari bilan alohida ClassLoader orqali yuklanadi, bu esa kutubxonalar versiyalari oʻrtasidagi ziddiyatlarni (version conflicts) oldini oladi.
    * **Web Serverlar (Tomcat, Jetty):** Har bir deploy qilingan Web Application (WAR) alohida ClassLoader orqali yuklanadi, bu esa ularning bir-biridan mustaqil ishlashini taʼminlaydi.

Maxsus ClassLoader yaratish uchun odatda **`java.lang.ClassLoader`** sinfidan vorislik olinadi va **`findClass()`** metodi bekor qilinadi (override qilinadi).
