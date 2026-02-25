# 🤖 QarzchiBot — To'liq Logika va Arxitektura Hujjati
> Bitta do'kon uchun professional qarz boshqaruv boti (Telegram)  
> Versiya: 2.0 | Til: O'zbek

---

## 📌 Mundarija
1. [Bot Maqsadi](#maqsad)
2. [Rollar va Huquqlar](#rollar)
3. [Ma'lumotlar Strukturasi](#struktura)
4. [Asosiy Flow — Kim bo'lsa shunday ko'radi](#flow)
5. [Mijozni Ro'yxatga Olish Jarayoni](#royxat)
6. [Admin Paneli](#admin)
7. [Mijoz Paneli](#mijoz)
8. [Xato Holatlari (Edge Cases)](#xato)
9. [Tasdiqlash Logikasi](#tasdiq)
10. [Eslatma Tizimi](#eslatma)
11. [Hisobot Tizimi](#hisobot)
12. [Xavfsizlik](#xavfsizlik)
13. [Ma'lumotlar Bazasi Sxemasi](#db)
14. [Fayl Tuzilmasi](#fayl)
15. [Texnik Stack](#stack)
16. [MVP Rejasi](#mvp)

---

## 🎯 Bot Maqsadi {#maqsad}

**Muammo:** Do'kon egalari qarzlarni daftarga yozadi → daftar yo'qoladi → kim qancha qarz olgani esdan chiqadi → pul yo'qoladi.

**Yechim:** Telegram bot orqali:
- Qarzlarni raqamli saqlash
- Mijozlar o'z qarzini o'zlari ko'rishi
- Avtomatik eslatmalar yuborish
- Hisobotlar olish

---

## 👤 Rollar va Huquqlar {#rollar}

| Imkoniyat | 👑 Admin | 👤 Mijoz |
|-----------|---------|---------|
| Qarz qo'shish | ✅ | ❌ |
| To'lov kiritish | ✅ | ❌ |
| Barcha mijozlarni ko'rish | ✅ | ❌ |
| Umumiy hisobot | ✅ | ❌ |
| Eslatma yuborish | ✅ | ❌ |
| Qarz tahrirlash | ✅ | ❌ |
| O'z qarzini ko'rish | ✅ | ✅ |
| Qarz tarixini ko'rish | ✅ | ✅ |
| Admin bilan bog'lanish | — | ✅ |
| Eslatma olish | — | ✅ |

> **Admin** — `config.py` dagi `ADMIN_ID` bilan aniqlanadi (Telegram ID).  
> **Mijoz** — Admin tomonidan qo'shilgan, `telegram_id` si bog'langan foydalanuvchi.

---

## 🗂️ Ma'lumotlar Strukturasi {#struktura}

### Mijoz (clients)
```json
{
  "id": 1,
  "ism": "Sardor Karimov",
  "telefon": "+998901234567",
  "telegram_id": 123456789,
  "holat": "faol",
  "yaratilgan": "2024-01-15T10:30:00",
  "izoh": "Mahalla boshi — ishonchli"
}
```

### Qarz (debts)
```json
{
  "id": 1,
  "mijoz_id": 1,
  "summa": 200000,
  "qolgan": 150000,
  "sabab": "Guruch 10kg + Yog 2L",
  "sana": "2024-01-15",
  "muddat": "2024-02-15",
  "holat": "ochiq",
  "eslatma_yuborildi": false
}
```
> `holat` qiymatlari: `ochiq` | `qisman_tolangan` | `yopiq` | `muddati_otgan`

### To'lov (payments)
```json
{
  "id": 1,
  "qarz_id": 1,
  "mijoz_id": 1,
  "summa": 50000,
  "sana": "2024-01-20T14:22:00",
  "izoh": "Naqd to'ladi"
}
```

### Eslatma Jurnali (notifications)
```json
{
  "id": 1,
  "mijoz_id": 1,
  "matn": "Qarzingiz: 150,000 so'm. Iltimos to'lang.",
  "yuborilgan": "2024-01-25T09:00:00",
  "usul": "telegram",
  "holat": "yetdi"
}
```

---

## 🔄 Asosiy Flow — Kim bo'lsa shunday ko'radi {#flow}

```
/start bosildi
      ↓
Telegram ID tekshiriladi
      ↓
┌─────────────────────────────────────────┐
│  ADMIN_ID ga teng?                      │
│  Ha ──→ 👑 Admin paneli ochiladi        │
│  Yo'q ↓                                 │
│  DB da telegram_id bor?                 │
│  Ha ──→ 👤 Mijoz paneli ochiladi        │
│  Yo'q ↓                                 │
│  Noma'lum foydalanuvchi xabari yuboriladi│
└─────────────────────────────────────────┘
```

**Noma'lum foydalanuvchiga xabar:**
```
⛔ Kechirasiz, siz ro'yxatda yo'qsiz.

Agar do'konimizdan qarz olgan bo'lsangiz,
do'kon egasiga murojaat qiling:
📞 +998901234567
```

---

## 🆕 Mijozni Ro'yxatga Olish Jarayoni {#royxat}

Bu eng muhim qism — bu bo'lmasa mijoz botga kira olmaydi.

### Admin yangi mijoz qo'shganda:

```
Admin: [👥 Mijozlar] → [➕ Yangi mijoz]
Bot:   Mijoz ismini kiriting:
Admin: Sardor Karimov
Bot:   Telefon raqami:
Admin: +998901234567
Bot:   Izoh (ixtiyoriy, /skip):
Admin: Mahalla boshi
Bot:   ✅ Mijoz qo'shildi!
       
       ─────────────────────────────
       Sardor Karimovga ushbu havolani yuboring:
       👉 https://t.me/QarzchiBot?start=REG_abc123
       
       U botga kirganda avtomatik ro'yxatdan o'tadi.
       ─────────────────────────────
       [📋 Havolani nusxalash]
```

### Mijoz havolaga bosib /start yuborganda:

```
Bot: Assalomu alaykum, Sardor! 👋

     Siz QarzchiBot ga ulandinggiz.
     Bu bot orqali do'kondagi qarzingizni
     istalgan vaqt ko'rishingiz mumkin.

     [💰 Mening qarzim]
```

> **Texnik:** Havola `start=REG_abc123` parametri bilan keladi. Bot shu tokenni DB da topib, mijozning `telegram_id` sini saqlaydi. Token bir martalik va 24 soat amal qiladi.

### Agar token eskirgan bo'lsa:
```
Bot: ⚠️ Bu havola eskirgan (24 soat o'tdi).
     Do'kon egasidan yangi havola so'rang.
     📞 +998901234567
```

---

## 👑 ADMIN PANELI {#admin}

### Asosiy Keyboard
```
┌─────────────────────┬─────────────────────┐
│  ➕ Qarz qo'shish    │  💰 To'lov qabul     │
├─────────────────────┼─────────────────────┤
│  📊 Hisobot          │  👥 Mijozlar          │
├─────────────────────┴─────────────────────┤
│  🔔 Eslatmalar       │  ⚙️ Sozlamalar        │
└───────────────────────────────────────────┘
```

---

### ➕ Qarz Qo'shish — To'liq Dialog

```
Admin: [➕ Qarz qo'shish]

Bot:   Mijoz ismini yoki telefon raqamini kiriting:
Admin: Sardor

Bot:   Quyidagi mijozlardan birini tanlang:
       [1. Sardor Karimov — 📞 +998901234567]
       [2. Sardor Toshmatov — 📞 +998933334455]
       [❌ Bekor qilish]

Admin: [Sardor Karimov]

Bot:   Qarz summasi (so'mda):
Admin: 150000

       ⚠️ Xato holati: agar harf kiritsa:
Bot:   ❌ Faqat raqam kiriting! Masalan: 150000
       
Admin: 150000 (to'g'ri kiritildi)

Bot:   Sababi nima? (tovar nomi yoki boshqa):
Admin: Guruch 10kg, Yog 2L

Bot:   To'lash muddati? (ixtiyoriy)
       [📅 Muddat belgilash]  [⏭️ O'tkazib yuborish]

Admin: [📅 Muddat belgilash]
Bot:   Sanani kiriting (KK.OO.YYYY):
Admin: 15.02.2024

Bot:   ─────────────────────────────
       ✅ Tasdiqlaysizmi?
       
       👤 Mijoz: Sardor Karimov
       💰 Summa: 150,000 so'm
       📦 Sabab: Guruch 10kg, Yog 2L
       📅 Muddat: 15.02.2024
       ─────────────────────────────
       [✅ Tasdiqlash]  [✏️ Tahrirlash]  [❌ Bekor qilish]

Admin: [✅ Tasdiqlash]
Bot:   ✅ Qarz muvaffaqiyatli qo'shildi!
       Sardor Karimovga bildirishnoma yuborildi.
```

**Mijozga avtomatik yuboriladi:**
```
Bot → Sardor:
📢 Yangi qarz qo'shildi!

💰 Summa: 150,000 so'm
📦 Sabab: Guruch 10kg, Yog 2L
📅 Muddat: 15.02.2024

Batafsil: [💰 Mening qarzim]
```

---

### 💰 To'lov Qabul Qilish — To'liq Dialog

```
Admin: [💰 To'lov qabul]
Bot:   Mijoz ismini kiriting:
Admin: Sardor

Bot:   👤 Sardor Karimov
       💰 Jami qarzi: 150,000 so'm
       
       Qancha to'ladi?
Admin: 200000

       ⚠️ Xato holati: to'lov qarzdan ko'p
Bot:   ❌ To'lov summasi (200,000) qarzdan (150,000) ko'p!
       To'g'ri summani kiriting:
Admin: 50000

Bot:   ─────────────────────────────
       ✅ Tasdiqlaysizmi?
       
       👤 Sardor Karimov
       💳 To'lov: 50,000 so'm
       📊 Qoladi: 100,000 so'm
       ─────────────────────────────
       [✅ Tasdiqlash]  [❌ Bekor qilish]

Admin: [✅ Tasdiqlash]
Bot:   ✅ To'lov kiritildi!
       Sardorga bildirishnoma yuborildi.
```

**Agar mijoz to'liq to'lasa:**
```
Bot: 🎉 Sardor Karimov qarzini to'liq to'ladi!
     Qarz yopildi va arxivga o'tkazildi.
```

---

### 👥 Mijozlar Boshqaruvi

```
Admin: [👥 Mijozlar]
Bot:   👥 Barcha mijozlar (7 kishi)
       ─────────────────────────────
       🔴 Muddati o'tgan (2):
       • Bahrom ──── 300,000 so'm ⚠️
       • Jasur  ──── 450,000 so'm ⚠️
       
       🟡 Faol qarzlar (3):
       • Sardor ──── 100,000 so'm
       • Malika ──── 200,000 so'm
       • Dilnoza ─── 200,000 so'm
       
       🟢 Qarzsiz (2):
       • Anvar (to'liq to'lagan)
       • Zulfiya (to'liq to'lagan)
       ─────────────────────────────
       [➕ Yangi mijoz]  [🔍 Qidirish]
```

**Bitta mijozga bosilganda:**
```
Bot:   👤 Sardor Karimov
       📞 +998901234567
       🔗 Telegram: @sardor (ulangan)
       ─────────────────────────────
       💰 Joriy qarz: 100,000 so'm
       📅 Muddat: 15.02.2024 (12 kun qoldi)
       ─────────────────────────────
       Qarz tarixi:
       📦 Guruch 10kg — 150,000 so'm (15.01)
       💳 To'landi — 50,000 so'm (20.01)
       ─────────────────────────────
       [🔔 Eslatma]  [💰 To'lov]  [✏️ Tahrirlash]  [🗑️ O'chirish]
```

---

### ✏️ Qarz Tahrirlash

```
Admin: [✏️ Tahrirlash] (Sarddor qarziga)
Bot:   Nimani tahrirlaysiz?
       [💰 Summani]  [📦 Sababni]  [📅 Muddatni]  [❌ Bekor]

Admin: [💰 Summani]
Bot:   Hozirgi summa: 150,000 so'm
       Yangi summa kiriting:
Admin: 180000
Bot:   ✅ Summa yangilandi: 180,000 so'm
```

---

### ⚙️ Sozlamalar

```
Admin: [⚙️ Sozlamalar]
Bot:   ⚙️ Bot Sozlamalari
       ─────────────────────────────
       🏪 Do'kon nomi: Nodir Do'koni
       📞 Telefon: +998901234567
       ⏰ Kunlik eslatma: 09:00 (yoqilgan ✅)
       🔔 Muddat eslatmasi: 3 kun oldin
       ─────────────────────────────
       [✏️ Tahrirlash]
```

---

## 👤 MIJOZ PANELI {#mijoz}

### Onboarding (birinchi kirish)
```
Bot: 👋 Assalomu alaykum, Sardor!

     Bu bot orqali siz:
     ✅ Qarzingizni ko'rishingiz
     ✅ To'lov tarixingizni ko'rishingiz
     ✅ Do'kon bilan bog'lanishingiz mumkin

     ━━━━━━━━━━━━━━━━━━━━━
     [💰 Mening qarzim]
```

### Asosiy Keyboard
```
┌──────────────────┬──────────────────┐
│  💰 Mening qarzim │  📋 Qarz tarixi   │
├──────────────────┴──────────────────┤
│       📞 Do'kon bilan bog'lanish      │
└─────────────────────────────────────┘
```

---

### 💰 Mening Qarzim

**Holat 1: Qarzi bor**
```
Bot:  💰 Sizning qarzingiz
      ━━━━━━━━━━━━━━━━━━━━━
      👤 Sardor Karimov
      
      📦 Guruch 10kg, Yog 2L
      💸 Qarz olindi: 150,000 so'm
      💳 To'langan:    50,000 so'm
      ─────────────────────────────
      💰 Qolgan qarz: 100,000 so'm
      📅 To'lash muddati: 15.02.2024
         (12 kun qoldi ⏳)
      ━━━━━━━━━━━━━━━━━━━━━
```

**Holat 2: Qarzi yo'q**
```
Bot:  ✅ Tabriklaymiz, Sardor!
      Hozirda sizda qarz yo'q. 🎉
```

**Holat 3: Muddati o'tgan**
```
Bot:  ⚠️ Sizning qarzingiz
      ━━━━━━━━━━━━━━━━━━━━━
      👤 Sardor Karimov
      💰 Qolgan qarz: 100,000 so'm
      📅 Muddat: 15.01.2024
         ❗ MUDDATI 10 KUN OLDIN O'TDI
      ━━━━━━━━━━━━━━━━━━━━━
      Iltimos, do'konga murojaat qiling.
```

---

### 📋 Qarz Tarixi

```
Bot:  📋 Qarz tarixingiz
      ━━━━━━━━━━━━━━━━━━━━━
      📅 15.01.2024
         ➕ Qarz: 150,000 so'm (Guruch 10kg)
      
      📅 20.01.2024
         💳 To'lov: 50,000 so'm
      ─────────────────────────────
      💰 Hozirgi qoldiq: 100,000 so'm
      ━━━━━━━━━━━━━━━━━━━━━
      
      (Agar tarixi ko'p bo'lsa)
      [⬅️ Oldingi]  1/3  [Keyingi ➡️]
```

---

### 📞 Do'kon bilan Bog'lanish

```
Bot:  📞 Do'kon Ma'lumotlari
      ━━━━━━━━━━━━━━━━━━━━━
      🏪 Nodir Do'koni
      📱 +998901234567
      ━━━━━━━━━━━━━━━━━━━━━
      [📲 Telegram'da yozish]  [📞 Qo'ng'iroq]
```

---

## ⚠️ Xato Holatlari (Edge Cases) {#xato}

| Holat | Nima bo'ladi | Bot javobi |
|-------|-------------|-----------|
| Summa o'rniga harf | Qayta so'raydi | ❌ Faqat raqam kiriting |
| Summa 0 yoki manfiy | Qayta so'raydi | ❌ Summa 0 dan katta bo'lishi kerak |
| To'lov > Qarz | Qayta so'raydi | ❌ To'lov (X) qarzdan (Y) ko'p! |
| Noma'lum mijoz ismi | Topilmadi xabari | ❌ "Sardor" topilmadi. Ismni tekshiring |
| Mijoz allaqachon to'lagan | Eslatadi | ⚠️ Bu mijozda faol qarz yo'q |
| Noma'lum foydalanuvchi /start | Kirish rad | ⛔ Siz ro'yxatda yo'qsiz |
| Token eskirgan | Yangi havola so'ra | ⚠️ Havola eskirgan (24 soat) |
| Bot token xatosi | Admin xabardor | 🔴 Texnik xato, admin bilan bog'laning |
| Bir xil ismli 2 mijoz | Tanlash taklif | Qaysi Sardor? [1] [2] |

---

## ✅ Tasdiqlash Logikasi {#tasdiq}

Har qanday **yozish** amali (qarz qo'shish, to'lov, o'chirish) uchun:

```
Ma'lumot kiritildi
       ↓
Bot xulosa ko'rsatadi
       ↓
[✅ Tasdiqlash] | [✏️ Tahrirlash] | [❌ Bekor qilish]
       ↓
Tasdiqlandi → DB ga yoziladi → Bildirishnoma yuboriladi
```

> **Sababı:** Admin tez-tez telefonda ishlaydi, tasodifiy xato kiritishi mumkin. Tasdiqlash qadami bu xatolarni oldini oladi.

---

## 🔔 Eslatma Tizimi {#eslatma}

### Eslatma Turlari

| Tur | Qachon | Kim yuboradi |
|-----|--------|-------------|
| Yangi qarz bildirishnomasi | Qarz qo'shilganda | Avtomatik |
| To'lov tasdig'i | To'lov kiritilganda | Avtomatik |
| Muddat eslatmasi | Muddat 3 kun oldin | Avtomatik (scheduler) |
| Muddati o'tgan eslatma | Muddat o'tgandan keyin har kun | Avtomatik |
| Qo'lda eslatma | Admin xohlagan vaqt | Admin |

### Qo'lda Eslatma — Dialog

```
Admin: [🔔 Eslatmalar]
Bot:   Kimga eslatma?
       [👤 Bitta mijozga]  [👥 Hammaga]  [⚠️ Muddati o'tganlarga]

Admin: [⚠️ Muddati o'tganlarga]
Bot:   Muddati o'tgan mijozlar (2 kishi):
       • Bahrom — 300,000 so'm (15 kun o'tdi)
       • Jasur  — 450,000 so'm (3 kun o'tdi)
       
       Eslatma matni (standart matni ishlatish uchun /default):
Admin: /default
Bot:   Standart matn:
       "Assalomu alaykum! Do'konimizdan olgan qarzingiz to'lash muddati o'tdi. Iltimos, imkon qadar tezroq to'lang. 🙏"
       
       [✅ Yuborish]  [✏️ O'zgartirish]

Admin: [✅ Yuborish]
Bot:   ✅ 2 ta mijozga eslatma yuborildi!
       • Bahrom ✅
       • Jasur  ✅
```

### Avtomatik Scheduler

```
Har kuni 09:00 da bot tekshiradi:
  → Muddati 3 kun qolganlar? → Eslatma yuboradi
  → Muddati bugun tugaganlar? → Eslatma yuboradi  
  → Muddati 1+ kun o'tganlar? → Har kun eslatma yuboradi
  → Adminga: "Bugungi holat: X ta muddati o'tgan"
```

---

## 📊 Hisobot Tizimi {#hisobot}

### Kunlik Hisobot

```
Bot:  📊 Hisobot — 15.01.2024
      ━━━━━━━━━━━━━━━━━━━━━━━━
      👥 Jami qarzdorlar: 5 kishi
      💰 Jami qarz: 1,250,000 so'm
      ⚠️ Muddati o'tgan: 2 kishi
      ━━━━━━━━━━━━━━━━━━━━━━━━
      
      ❗ Muddati o'tganlar:
      🔴 Bahrom ── 300,000 so'm (15 kun)
      🔴 Jasur  ── 450,000 so'm (3 kun)
      
      📋 Faol qarzlar:
      🟡 Sardor  ── 100,000 so'm (12 kun qoldi)
      🟡 Malika  ── 200,000 so'm (20 kun qoldi)
      🟡 Dilnoza ── 200,000 so'm (25 kun qoldi)
      ━━━━━━━━━━━━━━━━━━━━━━━━
      [🔔 Muddati o'tganlarga eslatma]
```

### Oylik Hisobot

```
Bot:  📈 Yanvar 2024 Oylik Hisobot
      ━━━━━━━━━━━━━━━━━━━━━━━━
      ➕ Berilgan qarz: 1,800,000 so'm
      💳 Qabul qilingan to'lov: 550,000 so'm
      💰 Qolgan umumiy qarz: 1,250,000 so'm
      ━━━━━━━━━━━━━━━━━━━━━━━━
      📊 Eng ko'p qarzdor: Jasur (450,000)
      ✅ To'liq to'laganlar: 2 kishi
```

---

## 🔐 Xavfsizlik {#xavfsizlik}

```python
# Middleware — har bir so'rovda ishlaydi
async def auth_middleware(handler, event, data):
    user_id = event.from_user.id
    
    if user_id == ADMIN_ID:
        data["role"] = "admin"
    elif db.get_mijoz_by_telegram_id(user_id):
        data["role"] = "mijoz"
        data["mijoz"] = db.get_mijoz_by_telegram_id(user_id)
    else:
        await event.answer("⛔ Siz ro'yxatda yo'qsiz!")
        return  # Handler ishlamaydi
    
    return await handler(event, data)
```

**Qo'shimcha xavfsizlik:**
- Mijoz faqat o'z `mijoz_id` si bilan so'rov qila oladi
- Har bir DB so'rovida `mijoz_id` filter qilinadi
- Admin buyruqlari alohida `@admin_only` decorator bilan
- Ro'yxatga olish tokenları bir martalik va 24 soatlik

---

## 🗄️ Ma'lumotlar Bazasi Sxemasi {#db}

```sql
-- Mijozlar
CREATE TABLE clients (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ism TEXT NOT NULL,
    telefon TEXT UNIQUE,
    telegram_id INTEGER UNIQUE,
    holat TEXT DEFAULT 'faol',
    izoh TEXT,
    reg_token TEXT,
    token_expires DATETIME,
    yaratilgan DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Qarzlar
CREATE TABLE debts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mijoz_id INTEGER REFERENCES clients(id),
    summa INTEGER NOT NULL,
    qolgan INTEGER NOT NULL,
    sabab TEXT,
    sana DATE DEFAULT CURRENT_DATE,
    muddat DATE,
    holat TEXT DEFAULT 'ochiq',
    yaratilgan DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- To'lovlar
CREATE TABLE payments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    qarz_id INTEGER REFERENCES debts(id),
    mijoz_id INTEGER REFERENCES clients(id),
    summa INTEGER NOT NULL,
    izoh TEXT,
    sana DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Eslatma jurnali
CREATE TABLE notifications (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mijoz_id INTEGER REFERENCES clients(id),
    matn TEXT,
    usul TEXT DEFAULT 'telegram',
    holat TEXT DEFAULT 'yuborildi',
    yuborilgan DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🗄️ Fayl Tuzilmasi {#fayl}

```
qarzchibot/
├── bot.py                    # Asosiy ishga tushirish
├── config.py                 # Token, Admin ID, sozlamalar
├── database.py               # DB ulanish va umumiy funksiyalar
│
├── handlers/
│   ├── __init__.py
│   ├── common.py             # /start, noma'lum foydalanuvchi
│   ├── admin/
│   │   ├── __init__.py
│   │   ├── qarz.py           # Qarz qo'shish, tahrirlash
│   │   ├── tolov.py          # To'lov qabul qilish
│   │   ├── mijozlar.py       # Mijozlar ro'yxati, qo'shish
│   │   ├── hisobot.py        # Kunlik, oylik hisobot
│   │   ├── eslatma.py        # Qo'lda eslatma yuborish
│   │   └── sozlamalar.py     # Bot sozlamalari
│   └── mijoz/
│       ├── __init__.py
│       ├── mening_qarzim.py  # Joriy qarz holati
│       ├── tarix.py          # Qarz tarixi
│       └── boglanish.py      # Do'kon bilan bog'lanish
│
├── keyboards/
│   ├── admin_kb.py           # Admin tugmalari
│   └── mijoz_kb.py           # Mijoz tugmalari
│
├── middlewares/
│   └── auth.py               # Autentifikatsiya middleware
│
├── scheduler/
│   └── reminder.py           # Avtomatik eslatmalar (APScheduler)
│
├── utils/
│   ├── formatters.py         # Pul formatlash, sana formatlash
│   ├── validators.py         # Summa, telefon tekshirish
│   └── token_gen.py          # Ro'yxat token generatsiyasi
│
└── data/
    └── db.sqlite3
```

---

## ⚙️ Texnik Stack {#stack}

| Komponent | Texnologiya | Sababi |
|-----------|------------|--------|
| Bot framework | `aiogram 3.x` | Eng zamonaviy async Python bot lib |
| Ma'lumotlar bazasi | `SQLite` | Oddiy, serverless, bitta fayl |
| Scheduler | `APScheduler` | Avtomatik eslatmalar uchun |
| Hosting | `Railway.app` yoki VPS | Arzon va ishonchli |
| Til | `Python 3.11+` | |

---

## 🚀 MVP Rejasi {#mvp}

### 1-Bosqich (Asosiy)
| # | Funksiya | Holat |
|---|----------|-------|
| 1 | Admin autentifikatsiya | ⬜ |
| 2 | Mijoz ro'yxatga olish (havola orqali) | ⬜ |
| 3 | Qarz qo'shish (tasdiqlash bilan) | ⬜ |
| 4 | To'lov qabul qilish | ⬜ |
| 5 | Mijoz o'z qarzini ko'radi | ⬜ |
| 6 | Kunlik hisobot | ⬜ |

### 2-Bosqich (Kengaytirilgan)
| # | Funksiya | Holat |
|---|----------|-------|
| 7 | Avtomatik eslatmalar (scheduler) | ⬜ |
| 8 | Qarz tarixi (sahifalash bilan) | ⬜ |
| 9 | Muddati o'tgan qarzlar filtri | ⬜ |
| 10 | Oylik hisobot | ⬜ |

### 3-Bosqich (Premium)
| # | Funksiya | Holat |
|---|----------|-------|
| 11 | Chek rasm yuborish | ⬜ |
| 12 | Qarz tahrirlash | ⬜ |
| 13 | Arxiv (yopilgan qarzlar) | ⬜ |
| 14 | Do'kon sozlamalari paneli | ⬜ |

---

*QarzchiBot v2.0 — Professional qarz boshqaruv tizimi*  
*Hujjat oxirgi yangilangan: 2024*
