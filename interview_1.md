# ☕ Senior Java Software Engineer (Savol-Javob Qoʻllanmasi)

Ushbu hujjat Senior darajadagi Java dasturchilar uchun moʻljallangan chuqur texnik intervyu savollari va yuqori yuklamali tizim dizayni (Project Case) yechimlarini oʻz ichiga oladi.

## I. 📚 Java Core va Til Xususiyatlari

| Savol (Q) | Javob (A) |
| :--- | :--- |
| **Q1: Garbage Collection (GC) Tanlovi** | **A:** Katta (multi-GB) heaplar va bashoratli past kechikish (predictable low latency) uchun **G1 (Garbage-First)** standart tanlovdir. Real vaqtda juda past pauzalar (odatda < 10ms) talab qilinganda, **ZGC** yoki **Shenandoah** afzal koʻriladi. |
| **Q2: CompletableFuture** | **A:** Asinxron hisoblashlarni yaratish, birlashtirish va kompozitsiyalash uchun ishlatiladi. I/O-intensive operatsiyalarni (masalan, tashqi API chaqiruvlari) blokirovka qilmasdan parallel bajarish uchun ideal. |
| **Q3: Java Memory Model (JMM) va `volatile`** | **A:** JMM bir vaqtda ishlaydigan threadlarning bir-birining xotira yozuvlarini koʻrishini boshqaradi. `volatile` oʻzgaruvchining qiymati har doim asosiy xotiradan oʻqilishini/yozilishini taʼminlab, barcha threadlar uchun **koʻrinish (visibility)**ni kafolatlaydi (lekin atomarlikni emas). |
| **Q4: Type Erasure** | **A:** Kompilyatsiya vaqtida Generics tiplari haqidagi maʼlumotlarning oʻchirilishi. Natijada, run-time da `new T()` yoki `instanceof List<String>` kabi operatsiyalar bajarilmaydi. |

## II. 🧩 Dizayn Namunalari (Design Patterns) va SOLID

| Savol (Q) | Javob (A) |
| :--- | :--- |
| **Q5: Dependency Inversion Principle (DIP)** | **A:** Yuqori darajadagi modullar (masalan, `UserService`) past darajadagi modullarga (`MySQLUserRepository`) toʻgʻridan-toʻgʻri bogʻliq boʻlmasligi kerak, balki ikkalasi ham abstraktsiyalarga (`UserRepository` interfeysiga) bogʻliq boʻlishi kerak. |
| **Q6: Circuit Breaker** | **A:** Bir xizmatning ishdan chiqishi butun tizimning ishdan chiqishiga (Cascading Failures) yoʻl qoʻymaslik uchun xizmat chaqiruvlari orasida qoʻllaniladi. Agar xato darajasi oshsa, tashqi chaqiruvni vaqtincha toʻxtatadi (Open State). |
| **Q7: Abstract Factory vs. Factory Method** | **A:** **Factory Method** bir oiladan bitta obyekt yaratadi. **Abstract Factory** esa bir-biriga bogʻliq boʻlgan obyektlarning **butun oilasini** yaratish uchun ishlatiladi (Masalan, turli operatsion tizimlar uchun UI komponentlari oilasi). |

## III. 🛠️ Freymvorklar va Ekosistema (Spring)

| Savol (Q) | Javob (A) |
| :--- | :--- |
| **Q8: Spring AOP Stsenariysi** | **A:** Logging, tranzaksiya va xavfsizlikdan tashqari, biznes logikasiga tegmasdan maxsus **audit trail** (maʼlumotlarning oldingi qiymatlarini avtomatik yozib olish) yoki **data masking** mexanizmlarini amalga oshirishda foydalanish. |
| **Q9: Tranzaksiya Izolyatsiya Darajalari** | **A:** Asosiy daraja **`READ_COMMITTED`** hisoblanadi. U `Dirty Reads`ning oldini oladi. Eng qatʼiy daraja **`SERIALIZABLE`** boʻlib, u barcha oʻqish anomaliyalarini (Dirty, Non-Repeatable, Phantom) bartaraf etadi, lekin ishlash tezligini pasaytiradi. |
| **Q10: Performans Tuning Usullari** | **A:** 1. **Monitoring** (Actuator, Prometheus/Grafana). 2. **Profiling** (JProfiler/JVisualVM yordamida CPU/Heapdagi "hotspot"larni aniqlash). 3. **Caching** (Redis). 4. **GC Tuning** (G1 parametrlarini optimallashtirish). |

## IV. 🌐 Loyiha Keysi: E-Tijorat Buyurtma Jarayoni Dizayni

### 🎯 Stsenariy: Yuqori Yuklamali Buyurtma Boshqaruvi (5000+ orders/sec)

| Savol (Q) | Dizayn Qarori va Asoslash (A) |
| :--- | :--- |
| **Q14: Arxitektura Tanlovi** | **A: Mikroservislar + Asinxron Xabarlar (Kafka).** <br/> **Asos:** Yuqori yuklamani gorizontal taqsimlash, xatolarga chidamlilik va mustaqil kengaytiriluvchanlikni taʼminlash. Buyurtma qabul qilish xizmati faqat validatsiya qilib, darhol xabarni navbatga yuboradi (OrderCreated Event). |
| **Q15: Buyurtma Bajarilish Garantiyasi** | **A: Kafka + Saga Pattern (Xoreografiya).** <br/> **Saga:** Buyurtma jarayonidagi (Order -> Inventory -> Payment) har bir mahalliy tranzaksiyani boshqaradi. Agar biron bir qadam muvaffaqiyatsiz tugasa, u kompensatsion tranzaksiyalar (masalan, inventarizatsiyani tiklash) orqali tizimni oldingi holatiga qaytaradi. |
| **Q16: Maʼlumotlar Bazasi Strategiyasi** | **A: Polyglot Persistency.** <br/> - **Tranzaksiyalar:** Relational DB (PostgreSQL) - `ACID` mustahkamligi uchun. <br/> - **Kuzatuv/Tez Oʻqish:** NoSQL (masalan, MongoDB) - Buyurtma holatini tezkor oʻqish (Read Scale) uchun. **Event Sourcing**ni joriy etish orqali buyurtmaning barcha holat oʻzgarishlari saqlanadi. |
| **Q17: Real vaqtda Holat Uzatish** | **A: Server-Sent Events (SSE).** <br/> Buyurtma holatidagi yangilanishlarni mijozlarga bir tomonlama, past kechikish bilan yetkazish uchun Spring WebFlux yordamida amalga oshiriladi. |
| **Q18: Circuit Breaker Ni Qoʻllash** | **A:** **Order Service** va **Payment/Inventory Service** orasida. <br/> **Sabab:** Past oqimli xizmatlardan biri ishdan chiqqan taqdirda, Order Service ularni chaqirishdan toʻxtaydi, oʻzining resurslarini himoya qiladi va buyurtmalarni yoʻqotmasdan, ularni keyinroq qayta ishlash uchun xabar navbatiga qayta yoʻnaltiradi. |
