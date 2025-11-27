Albatta! Senior Java Software Engineer darajasida **JVM Memory Management** (Xotira Boshqaruvi) ni chuqur tahlil qilib beraman. Bu nafaqat xotira maydonlarini bilishni, balki **Garbage Collector (GC)** ning ishlash mexanizmlari, optimallashtirish usullari va xotira sizib chiqishini (memory leaks) aniqlashni ham oʻz ichiga oladi.

Quyida $Markdown$ formatidagi toʻliq tushuntirish berilgan.

---

# 🧠 JVM Memory Management (Xotira Boshqaruvi)

JVM xotira boshqaruvi ikkita asosiy qismga boʻlinadi: **Runtime Data Area** (xotira maydonlari) va **Garbage Collection (GC)** mexanizmi.

## I. 💾 Runtime Data Area (Xotira Maydonlari)

JVM xotira maydonlari ikki turga boʻlinadi: **Thread-Shared (Umumiy)** va **Thread-Private (Har bir Thread uchun alohida)**.

### A. 🌐 Thread-Shared (Umumiy) Xotira Maydonlari

Bir vaqtda ishlaydigan barcha threadlar bu xotira maydonlariga kirish huquqiga ega.

#### 1. Heap Area (Heap Maydoni)

* **Vazifasi:** JVM da yaratilgan barcha **obyektlar** va **massivlar** joylashadigan joy. Bu Java xotirasining eng katta qismi hisoblanadi.
* **GC Nazorati:** Heap butunlay **Garbage Collector (GC)** nazorati ostida boʻladi. GC bu maydondagi foydalanilmayotgan obyektlarni avtomatik ravishda tozalaydi.
* **Tuzilishi:** GC samaradorligini oshirish uchun Heap odatda uch avlodga boʻlinadi (**Generational Heap Layout**):
    * **Young Generation (Yosh Avlod):** Yangi yaratilgan obyektlar joylashadi. U `Eden Space` va ikkita `Survivor Space (S0 va S1)` ga boʻlinadi.
    * **Old Generation (Keksa Avlod):** Young Generation dagi bir nechta kichik GC sikllaridan (Minor GC) omon qolgan uzoq yashovchi obyektlar koʻchiriladi.
    * **Permanent Generation (PermGen) / Metaspace:**
        * **PermGen (Java 7 gacha):** Class meta-maʼlumotlari (sinflar, metodlar) va statik maʼlumotlarni saqlagan. Hajmi cheklangan va `OutOfMemoryError: PermGen space` xatosiga sabab boʻlgan.
        * **Metaspace (Java 8+):** PermGen oʻrnini egalladi. U JVM Heapining emas, balki **Native Memory** (operatsion tizim xotirasi) dan foydalanadi. Sinf meta-maʼlumotlarini saqlaydi. Hajmini `-XX:MaxMetaspaceSize` bilan boshqarish mumkin.

#### 2. Method Area (Metod Maydoni)

* **Vazifasi:** Yuklangan sinflar haqidagi maʼlumotlarni (meta-maʼlumotlar) saqlaydi.
* **Saqlanadigan Maʼlumotlar:**
    * Sinfning toʻliq kvalifikatsiyalangan nomi.
    * Metodlar (metod nomi, parametrlar, qaytish turi, bayt-kod).
    * Statik oʻzgaruvchilar.
    * Constant Pool (konstanta doimiy hovuzi).

### B. 🔒 Thread-Private (Har bir Thread uchun alohida) Xotira Maydonlari

Bu maydonlar faqat uni yaratgan thread tomonidan kirish uchun ruxsat etiladi.

#### 1. JVM Stacks (JVM Steklari)

* **Vazifasi:** Har bir metod chaqiruvi uchun **Stack Frame (stek doirasi)** saqlaydi.
* **Stack Frame:** Har bir metod chaqirilganda yaratiladi va metod qaytganda yoʻq qilinadi. U quyidagilarni oʻz ichiga oladi:
    * **Local Variables Array (Mahalliy Oʻzgaruvchilar Massivi):** Metoddagi mahalliy oʻzgaruvchilar (primitiv turlar va obyektlarga havolalar) joylashadi.
    * **Operand Stack (Operand Steki):** Metod bajarilayotganda hisoblashlar uchun va amallarni bajarish uchun vaqtinchalik saqlash joyi.
    * **Frame Data:** Metod normal tugagan yoki istisno (exception) tashlagan taqdirda, ijro etishni qayta boshlash uchun maʼlumotlar.

#### 2. PC Registers (Dastur Hisoblagich Registrlari)

* **Vazifasi:** Har bir threadda ijro etilayotgan **navbatdagi bayt-kod koʻrsatmasining manzilini** saqlaydi.

#### 3. Native Method Stacks (Mahalliy Metod Steklari)

* **Vazifasi:** **Java Native Interface (JNI)** orqali chaqirilgan Java boʻlmagan (masalan, C/C++) metodlar uchun stack xotirasini saqlaydi.

---

## II. 🗑️ Garbage Collection (GC) Mexanizmi

Java xotira boshqaruvining yuragi boʻlib, foydalanilmayotgan obyektlarni avtomatik ravishda tozalaydi.

### A. GC Asosiy Prinsipi: Generational Hypothesis (Avlod Gʻoyasi)

Bu gʻoyaga koʻra, obyektlarning aksariyati tezda, qolganlari esa uzoq vaqt yashaydi. Shu sababli, Heap avlodlarga boʻlinadi:

1.  **Minor GC:** Young Generation (Eden va Survivor Spaces) da amalga oshiriladi. Juda tez-tez, lekin tez bajariladi.
2.  **Major GC / Full GC:** Old Generation da amalga oshiriladi. Kamdan-kam, lekin juda sekin va **Stop-The-World (STW)** pauzalariga sabab boʻlishi mumkin.

### B. Obyektning Erishib Boʻlmasligini Aniqlash (Reachability)

GC obyektning foydalanilayotganligini aniqlash uchun **Mark-and-Sweep** kabi algoritmlardan foydalanadi.

* **GC Roots:** Heap tashqarisidan keladigan havolalar (masalan, Stackdagi mahalliy oʻzgaruvchilar, statik maydonlar, JNI havolalari) "ildiz" hisoblanadi.
* **Reachability:** GC Roots dan havolalar orqali erishish mumkin boʻlmagan obyektlar **"oʻlik"** (dead) hisoblanadi va tozalanishi kerak.

### C. Senior Darajadagi GC Turlari

| GC Turi | Xususiyat | Qachon Tanlanadi |
| :--- | :--- | :--- |
| **Parallel GC** | Yuqori oʻtkazuvchanlik (High Throughput) uchun optimallashtirilgan. | Agar ilova yuqori yuklamada ishlashi va GC pauzalari nisbatan uzoq boʻlishi mumkin boʻlsa (masalan, batch processing). |
| **CMS (Concurrent Mark Sweep)** | Konkurent ishlashga harakat qiladi. Young GC vaqtida STW pauzasi bor, lekin Old GC ning koʻp qismi ilova bilan birga ishlaydi. | Oʻrtacha-yuqori oʻtkazuvchanlik va qisqaroq STW talab qilinganda. |
| **G1 (Garbage-First)** | Katta Heaplar (multi-GB) uchun moʻljallangan. Heapni kichik **Region** larga boʻladi. **Prognoz qilinadigan** (predictable) pauzalarga erishishga harakat qiladi. | Keng koʻlamli, umumiy maqsadli ilovalar uchun standart tanlov. |
| **ZGC / Shenandoah** | **Juda past kechikish (Ultra Low Latency)** talab qilingan holatlar uchun. STW pauzalarini deyarli yoʻq qiladi (odatda < 10ms), hatto juda katta (TB) Heaplarda ham. | Real-time trading, yuqori tezlikdagi maʼlumotlarni qayta ishlash kabi past kechikish ustuvor boʻlgan tizimlar. |

---

## III. 🚨 Memory Leaks (Xotira Sizib Chiqishi)

Senior darajada xotira sizib chiqishini tushunish va bartaraf etish muhim.

* **Sababi:** Java da xotira sizib chiqishi — bu GC tozalashi kerak boʻlgan, lekin dasturdagi baʼzi **yaroqsiz havolalar (stale references)** tufayli tozalay olmaydigan obyektlar toʻplanishi.
* **Eng keng tarqalgan holatlar:**
    1.  **Uzoq yashovchi Collectionlar:** `static` yoki global Collection (HashMap, List) larga obyekt qoʻshilganda, lekin kerak boʻlmaganida oʻchirilmaganda.
    2.  **Session/Cache:** Foydalanuvchi sessiyalari yoki kesh yozuvlari vaqtida eskirganda, lekin oʻchirilmaganda.
    3.  **ThreadLocals:** `ThreadLocal` oʻzgaruvchilari GC tozalay olmaydigan havolalarni qoldirishi mumkin.
* **Diagnostika:** Profiler vositalari (JVisualVM, JProfiler) yordamida **Heap Dump** (Heap xotirasining lahzali nusxasi) tahlil qilinadi, **Dominator Tree** yordamida eng koʻp joy egallayotgan obyektlar va ularga boʻlgan havolalar topiladi.
