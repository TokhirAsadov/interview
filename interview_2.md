# ⚙️ Java Virtual Machine (JVM) Tahlili

## 🌟 Senior darajasida JVM ning Ishlash Prinsipi

JVM (Java Virtual Machine) Java bayt-kodini (bytecode) tushunadigan va ijro etadigan virtual mashinadir. U Java tilining asosiy xususiyati boʻlmish **"Bir marta yoz, istalgan joyda ishga tushir" (Write Once, Run Anywhere - WORA)** prinsipini taʼminlaydi.

JVM ning ishlash jarayoni 3 ta asosiy bosqichga boʻlinadi:

1.  **Class Loading (Sinflarni Yuklash)**
2.  **Memory Management (Xotira Boshqaruvi)**
3.  **Execution Engine (Ijro Etish Mexanizmi)**

---

## 🏗️ JVM ning Ichki Tuzilishi (Architecture)

JVM uchta asosiy qismdan iborat:

### 1. ClassLoader Subsystem (Sinflarni Yuklash Tizimi)

U Java sinfini (odatda `.class` fayli) diskdan olib, uni JVM ning Xotira Maydoniga (Memory Area) joylashtirish uchun javobgardir.

* **Yuklash (Loading):** Uchta delegatsiya iyerarxiyasiga asoslangan Loaderlar mavjud:
    * `Bootstrap ClassLoader`: Java core API sinflarini ($java.lang.\*$) yuklaydi.
    * `Extension ClassLoader`: JDK ning `ext` katalogidagi sinflarni yuklaydi.
    * `Application ClassLoader`: Loyiha sinflarini (CLASSPATH da joylashgan) yuklaydi.
* **Bogʻlash (Linking):**
    * `Verify`: Yuklangan sinfning bayt-kodi xavfsiz ekanligini tekshiradi.
    * `Prepare`: Sinfning statik oʻzgaruvchilariga xotira ajratadi va ularga standart qiymatlarni tayinlaydi (masalan, `0`, `null`).
    * `Resolve`: Ramziy havolalarni (Symbolic References) toʻgʻridan-toʻgʻri havolalarga (Direct References) aylantiradi.
* **Boshlash (Initialization):** Statik bloklar va statik oʻzgaruvchilarga dasturchi tomonidan berilgan haqiqiy qiymatlar beriladi.

### 2. Runtime Data Area (Xotira Maydonlari)

Bu - JVM sinflar, obyektlar, metodlar va threadlar uchun maʼlumotlarni saqlaydigan xotira qismlari. U besh asosiy qismdan iborat:

| Xotira Maydoni | Tushuntirish | Threadga Aloqadorlik |
| :--- | :--- | :--- |
| **Method Area** | Har bir sinf haqidagi maʼlumotni (metodlar, statik oʻzgaruvchilar, konstantalar) saqlaydi. | Umumiy (Shared) |
| **Heap Area** | Barcha obyektlar va massivlar uchun xotira ajratiladi. Garbage Collector bu maydonni boshqaradi. | Umumiy (Shared) |
| **JVM Stacks** | Har bir thread uchun alohida. Metod chaqiruvlari uchun **Stack Frame**larni saqlaydi. | Har bir Thread uchun alohida |
| **PC Registers** | Har bir thread uchun alohida. Ijro etilayotgan navbatdagi bayt-kod koʻrsatmasining manzilini saqlaydi. | Har bir Thread uchun alohida |
| **Native Method Stacks** | Java boʻlmagan (C/C++ kabi) mahalliy (native) kod metodlari uchun xotira ajratadi. | Har bir Thread uchun alohida |

### 3. Execution Engine (Ijro Etish Mexanizmi)

Bayt-kodni bajarish uchun javobgar qism. U uchta asosiy komponentga ega:

* **Interpreter (Tarjimon):** Bayt-kodni koʻrsatma-koʻrsatma oʻqiydi va bajaradi. Tez ishga tushiradi, lekin doimiy ishda sekin.
* **JIT Compiler (Just-In-Time Compiler):** Interpreterning zaifligini bartaraf etadi. Tez-tez takrorlanadigan kod qismlarini (`hotspot`larni) aniqlaydi va ularni juda tez ishlaydigan **mashina kodiga (native code)** kompilyatsiya qiladi. Bu **Adaptive Optimization (Moslashuvchan Optimallashtirish)** deb ataladi.
* **Garbage Collector (GC):** Heap Maydonidagi foydalanilmayotgan obyektlarni aniqlash va oʻchirish orqali xotirani avtomatik boshqaradi. GC qoidalari (masalan, Generational Hypothesis) va turlari (G1, CMS, ZGC) Senior darajasida alohida mavzu.

---

## 🏃 Java Kodining Ishlash (Run) Jarayoni

Java kodi quyidagi bosqichlardan oʻtadi:

### Bosqich 1: Kompilyatsiya (Compilation)

1.  Dasturchi Java manba kodini (`.java` fayli) yozadi.
2.  **Java Kompilyatori (`javac`)** manba kodini platformadan mustaqil boʻlgan **bayt-kodga** (`.class` fayli) aylantiradi.

### Bosqich 2: Sinfni Yuklash (Class Loading)

1.  Dastur ishga tushirilganda, **ClassLoader Subsystem** kerakli `.class` fayllarini topadi.
2.  Sinflar **Yuklanadi**, **Bogʻlanadi** (tekshirish, tayyorlash, ramziy havolalarni hal qilish) va **Boshlanadi**.
3.  Sinf maʼlumotlari **Method Area** ga joylashtiriladi.

### Bosqich 3: Ijro Etish (Execution)

1.  Har bir yangi thread uchun **JVM Stack** va **PC Register** yaratiladi.
2.  **Execution Engine** asosiy metodni (`main`) topadi va bayt-kodni ijro etishni boshlaydi.
3.  Dastlab **Interpreter** bayt-kodni koʻrsatma-koʻrsatma bajaradi.
4.  Agar **JIT Compiler** biror kod qismi tez-tez ishlatilayotganini aniqlasa, uni **mashina kodiga** kompilyatsiya qiladi va kelgusi chaqiriqlar uchun tezkor Native Code ni ishlatadi.
5.  Metod chaqiruvlari va mahalliy oʻzgaruvchilar **JVM Stack** dagi **Stack Frame** larda boshqariladi.
6.  `new` kalit soʻzi bilan yaratilgan obyektlar **Heap Area** ga joylashtiriladi.

### Bosqich 4: Xotira Boshqaruvi

1.  Obyektlarga havolalar mavjud boʻlmay qolganda, ular **Garbage Collector** uchun nomzod boʻladi.
2.  GC doimiy ravishda Heap ni monitoring qiladi va avtomatik ravishda foydalanilmayotgan obyektlarni oʻchirib, xotirani boʻshatadi.

---

## 🔑 Senior Nuqtai Nazardan Muhim Jihatlar

* **Tuning:** Senior muhandislar JVM ning ishlashini optimallashtirish uchun `-Xms`, `-Xmx` (Heap hajmi), `-XX:+UseG1GC` (GC tanlovi) kabi **JVM argumentlarini** toʻgʻri sozlashlari kerak.
* **Profilash (Profiling):** Ishlash muammolarini (masalan, sekinlik, xotira sizib chiqishi) bartaraf etish uchun JVisualVM, JProfiler kabi vositalar yordamida **Heap Dump** va **Thread Dump** larni tahlil qilish.
* **Metaspace (Java 8+):** Oldingi versiyalardagi **PermGen** oʻrnini egallagan. Bu joy ClassLoader, sinf meta-maʼlumotlarini saqlaydi va **Native Memory** dan foydalanadi. Uning hajmini `-XX:MaxMetaspaceSize` bilan boshqarish mumkin.
