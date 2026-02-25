# 🤖 QarzchiBot — To'liq Logika va Arxitektura Hujjati (v2.1)
> Bitta do'kon uchun professional qarz boshqaruv tizimi  
> Django Admin Panel + Telegram Bot  
> Versiya: 2.1 | Til: O'zbek

---

## 📌 Mundarija
1. [Bot Maqsadi](#maqsad)
2. [Arxitektura Yondashuvi](#arxitektura)
3. [Rollar va Huquqlar](#rollar)
4. [Ma'lumotlar Strukturasi](#struktura)
5. [Django Admin Panel](#django-admin)
6. [Telegram Bot — Mijozlar Uchun](#telegram-bot)
7. [Mijozni Ro'yxatga Olish Jarayoni](#royxat)
8. [Xato Holatlari (Edge Cases)](#xato)
9. [Eslatma Tizimi](#eslatma)
10. [Hisobot Tizimi](#hisobot)
11. [Xavfsizlik](#xavfsizlik)
12. [Ma'lumotlar Bazasi Sxemasi](#db)
13. [Fayl Tuzilmasi](#fayl)
14. [Texnik Stack](#stack)
15. [MVP Rejasi](#mvp)

---

## 🎯 Bot Maqsadi {#maqsad}

**Muammo:** Do'kon egalari qarzlarni daftarga yozadi → daftar yo'qoladi → kim qancha qarz olgani esdan chiqadi → pul yo'qoladi.

**Yechim:** Django Admin + Telegram bot orqali:
- Django admin panel orqali qarzlarni boshqarish (veb brauzer)
- Telegram bot orqali mijozlar o'z qarzini ko'rishi
- Avtomatik eslatmalar yuborish
- Hisobotlar olish

---

## 🏗️ Arxitektura Yondashuvi {#arxitektura}

### Nega Django Admin + Telegram Bot?

**Django Admin Panel:**
- ✅ Tayyor interfeys — tez ishga tushirish
- ✅ CRUD operatsiyalari avtomatik
- ✅ Qidirish, filter, pagination bor
- ✅ Tarixiy o'zgarishlar avtomatik saqlanadi
- ✅ Xavfsizlik mexanizmlari o'rnatilgan
- ✅ Kompyuterdan to'liq nazorat

**Telegram Bot (Mijozlar uchun):**
- ✅ Mijozlar telefonda tezkor ko'radi
- ✅ Push bildirishnomalar avtomatik
- ✅ 24/7 qulaylik
- ✅ O'rnatish kerak emas

### Arxitektura Diagrammasi

```
┌────────────────────────────────────────────────────┐
│              DJANGO BACKEND                         │
│  ┌──────────────┐        ┌──────────────┐          │
│  │   Models     │◄──────►│   Admin      │          │
│  │   (DB)       │        │   Panel      │          │
│  └──────────────┘        └──────────────┘          │
│         ▲                                           │
│         │                                           │
│         ▼                                           │
│  ┌──────────────┐                                   │
│  │   API/       │                                   │
│  │   Signals    │                                   │
│  └──────────────┘                                   │
│         │                                           │
└─────────┼───────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────┐
│           TELEGRAM BOT                              │
│  ┌──────────────┐        ┌──────────────┐          │
│  │   Handlers   │◄──────►│  Keyboards   │          │
│  │              │        │              │          │
│  └──────────────┘        └──────────────┘          │
│         │                                           │
│         ▼                                           │
│  ┌──────────────┐                                   │
│  │   Mijozlar   │                                   │
│  └──────────────┘                                   │
└─────────────────────────────────────────────────────┘
```

**Qanday ishlaydi:**
1. **Admin** Django admin panelda qarz qo'shadi
2. **Django signals** ishga tushadi
3. **Telegram bot** mijozga bildirishnoma yuboradi
4. **Mijoz** botda o'z qarzini ko'radi

---

## 👤 Rollar va Huquqlar {#rollar}

| Imkoniyat | 👑 Admin (Django) | 👤 Mijoz (Telegram) |
|-----------|-------------------|---------------------|
| Qarz qo'shish | ✅ Django Admin | ❌ |
| To'lov kiritish | ✅ Django Admin | ❌ |
| Mijoz qo'shish | ✅ Django Admin | ❌ |
| Barcha mijozlarni ko'rish | ✅ Django Admin | ❌ |
| Umumiy hisobot | ✅ Django Admin | ❌ |
| Eslatma yuborish | ✅ Django Admin | ❌ |
| Qarz tahrirlash/o'chirish | ✅ Django Admin | ❌ |
| O'z qarzini ko'rish | ✅ | ✅ Telegram Bot |
| Qarz tarixini ko'rish | ✅ | ✅ Telegram Bot |
| Do'kon bilan bog'lanish | — | ✅ Telegram Bot |
| Bildirishnoma olish | — | ✅ Telegram Bot |

---

## 🗂️ Ma'lumotlar Strukturasi {#struktura}

### Django Models

```python
# models.py

from django.db import models
from django.contrib.auth.models import User
from django.utils import timezone


class Mijoz(models.Model):
    """Mijozlar jadvali"""
    HOLAT_CHOICES = [
        ('faol', 'Faol'),
        ('nofaol', 'Nofaol'),
        ('bloklangan', 'Bloklangan'),
    ]
    
    ism = models.CharField('Ism-sharif', max_length=200)
    telefon = models.CharField('Telefon', max_length=20, unique=True)
    telegram_id = models.BigIntegerField('Telegram ID', null=True, blank=True, unique=True)
    telegram_username = models.CharField('Telegram @username', max_length=100, blank=True)
    holat = models.CharField('Holat', max_length=20, choices=HOLAT_CHOICES, default='faol')
    izoh = models.TextField('Izoh', blank=True)
    
    # Ro'yxatga olish uchun
    reg_token = models.CharField('Ro\'yxat Token', max_length=50, blank=True, unique=True)
    token_expires = models.DateTimeField('Token Muddati', null=True, blank=True)
    
    yaratilgan = models.DateTimeField('Yaratilgan', auto_now_add=True)
    yangilangan = models.DateTimeField('Yangilangan', auto_now=True)
    
    class Meta:
        verbose_name = 'Mijoz'
        verbose_name_plural = 'Mijozlar'
        ordering = ['-yaratilgan']
    
    def __str__(self):
        return f"{self.ism} ({self.telefon})"
    
    @property
    def jami_qarz(self):
        """Jami qarzi"""
        return sum(q.qolgan for q in self.qarzlar.filter(holat__in=['ochiq', 'qisman']))
    
    @property
    def telegram_havola(self):
        """Telegram bot havolasini qaytaradi"""
        if self.reg_token:
            return f"https://t.me/YourBotUsername?start=REG_{self.reg_token}"
        return None


class Qarz(models.Model):
    """Qarzlar jadvali"""
    HOLAT_CHOICES = [
        ('ochiq', 'Ochiq'),
        ('qisman', 'Qisman to\'langan'),
        ('yopiq', 'Yopiq'),
        ('muddati_otgan', 'Muddati o\'tgan'),
    ]
    
    mijoz = models.ForeignKey(Mijoz, on_delete=models.CASCADE, related_name='qarzlar', verbose_name='Mijoz')
    summa = models.DecimalField('Summa', max_digits=12, decimal_places=2)
    qolgan = models.DecimalField('Qolgan summa', max_digits=12, decimal_places=2)
    sabab = models.TextField('Sabab/Mahsulot', blank=True)
    
    sana = models.DateField('Qarz olgan sana', default=timezone.now)
    muddat = models.DateField('To\'lash muddati', null=True, blank=True)
    
    holat = models.CharField('Holat', max_length=20, choices=HOLAT_CHOICES, default='ochiq')
    eslatma_yuborildi = models.BooleanField('Eslatma yuborildi', default=False)
    
    yaratilgan = models.DateTimeField('Yaratilgan', auto_now_add=True)
    yangilangan = models.DateTimeField('Yangilangan', auto_now=True)
    
    class Meta:
        verbose_name = 'Qarz'
        verbose_name_plural = 'Qarzlar'
        ordering = ['-yaratilgan']
    
    def __str__(self):
        return f"{self.mijoz.ism} - {self.summa:,.0f} so'm"
    
    def save(self, *args, **kwargs):
        # Qarzni birinchi marta yaratganda qolgan = summa
        if not self.pk:
            self.qolgan = self.summa
        
        # Holatni avtomatik yangilash
        if self.qolgan <= 0:
            self.holat = 'yopiq'
        elif self.qolgan < self.summa:
            self.holat = 'qisman'
        elif self.muddat and self.muddat < timezone.now().date():
            self.holat = 'muddati_otgan'
        else:
            self.holat = 'ochiq'
        
        super().save(*args, **kwargs)


class Tolov(models.Model):
    """To'lovlar jadvali"""
    qarz = models.ForeignKey(Qarz, on_delete=models.CASCADE, related_name='tolovlar', verbose_name='Qarz')
    mijoz = models.ForeignKey(Mijoz, on_delete=models.CASCADE, related_name='tolovlar', verbose_name='Mijoz')
    summa = models.DecimalField('To\'lov summasi', max_digits=12, decimal_places=2)
    izoh = models.TextField('Izoh', blank=True)
    
    sana = models.DateTimeField('To\'lov sanasi', default=timezone.now)
    yaratilgan = models.DateTimeField('Yaratilgan', auto_now_add=True)
    
    class Meta:
        verbose_name = 'To\'lov'
        verbose_name_plural = 'To\'lovlar'
        ordering = ['-sana']
    
    def __str__(self):
        return f"{self.mijoz.ism} - {self.summa:,.0f} so'm ({self.sana.strftime('%d.%m.%Y')})"
    
    def save(self, *args, **kwargs):
        # To'lov qo'shilganda qarzni yangilash
        super().save(*args, **kwargs)
        
        self.qarz.qolgan -= self.summa
        self.qarz.save()


class Bildirishnoma(models.Model):
    """Yuborilgan bildirishnomalar jurnali"""
    TURI_CHOICES = [
        ('yangi_qarz', 'Yangi qarz'),
        ('tolov_tasdig', 'To\'lov tasdig\'i'),
        ('muddat_eslatma', 'Muddat eslatmasi'),
        ('muddati_otgan', 'Muddati o\'tgan'),
        ('qolda', 'Qo\'lda yuborilgan'),
    ]
    
    HOLAT_CHOICES = [
        ('yuborildi', 'Yuborildi'),
        ('yetdi', 'Yetdi'),
        ('xato', 'Xato'),
    ]
    
    mijoz = models.ForeignKey(Mijoz, on_delete=models.CASCADE, related_name='bildirishnomalar', verbose_name='Mijoz')
    turi = models.CharField('Turi', max_length=20, choices=TURI_CHOICES)
    matn = models.TextField('Matn')
    holat = models.CharField('Holat', max_length=20, choices=HOLAT_CHOICES, default='yuborildi')
    xato_sabab = models.TextField('Xato sababi', blank=True)
    
    yuborilgan = models.DateTimeField('Yuborilgan vaqt', auto_now_add=True)
    
    class Meta:
        verbose_name = 'Bildirishnoma'
        verbose_name_plural = 'Bildirishnomalar'
        ordering = ['-yuborilgan']
    
    def __str__(self):
        return f"{self.mijoz.ism} - {self.get_turi_display()} ({self.yuborilgan.strftime('%d.%m.%Y %H:%M')})"


class Sozlama(models.Model):
    """Tizim sozlamalari"""
    kalit = models.CharField('Kalit', max_length=100, unique=True)
    qiymat = models.TextField('Qiymat')
    tavsif = models.CharField('Tavsif', max_length=255, blank=True)
    
    class Meta:
        verbose_name = 'Sozlama'
        verbose_name_plural = 'Sozlamalar'
    
    def __str__(self):
        return f"{self.kalit}: {self.qiymat}"
```

---

## 🎛️ Django Admin Panel {#django-admin}

### Admin Konfiguratsiyasi

```python
# admin.py

from django.contrib import admin
from django.utils.html import format_html
from django.urls import reverse
from django.db.models import Sum, Count
from .models import Mijoz, Qarz, Tolov, Bildirishnoma, Sozlama
from .utils import generate_token, send_telegram_notification


@admin.register(Mijoz)
class MijozAdmin(admin.ModelAdmin):
    list_display = ['ism', 'telefon', 'telegram_status', 'jami_qarz_display', 'holat', 'yaratilgan']
    list_filter = ['holat', 'yaratilgan']
    search_fields = ['ism', 'telefon', 'telegram_username']
    readonly_fields = ['telegram_havola_display', 'reg_token', 'token_expires', 'yaratilgan', 'yangilangan']
    
    fieldsets = (
        ('Asosiy Ma\'lumotlar', {
            'fields': ('ism', 'telefon', 'izoh', 'holat')
        }),
        ('Telegram Ma\'lumotlari', {
            'fields': ('telegram_id', 'telegram_username', 'telegram_havola_display', 'reg_token', 'token_expires'),
            'classes': ('collapse',)
        }),
        ('Tizim Ma\'lumotlari', {
            'fields': ('yaratilgan', 'yangilangan'),
            'classes': ('collapse',)
        }),
    )
    
    actions = ['generate_registration_links', 'send_reminders']
    
    def telegram_status(self, obj):
        if obj.telegram_id:
            return format_html('<span style="color: green;">✅ Ulangan</span>')
        elif obj.reg_token:
            return format_html('<span style="color: orange;">⏳ Kutilmoqda</span>')
        return format_html('<span style="color: red;">❌ Ulanmagan</span>')
    telegram_status.short_description = 'Telegram'
    
    def jami_qarz_display(self, obj):
        jami = obj.jami_qarz
        if jami > 0:
            return format_html('<span style="color: red; font-weight: bold;">{:,.0f} so\'m</span>', jami)
        return format_html('<span style="color: green;">0 so\'m</span>')
    jami_qarz_display.short_description = 'Jami Qarz'
    
    def telegram_havola_display(self, obj):
        if obj.telegram_havola:
            return format_html(
                '<a href="{}" target="_blank">📲 Havolani nusxalash</a><br><code>{}</code>',
                obj.telegram_havola, obj.telegram_havola
            )
        return '-'
    telegram_havola_display.short_description = 'Telegram Havola'
    
    def generate_registration_links(self, request, queryset):
        """Tanlangan mijozlar uchun ro'yxat havolalarini yaratish"""
        count = 0
        for mijoz in queryset:
            if not mijoz.telegram_id:
                mijoz.reg_token = generate_token()
                mijoz.token_expires = timezone.now() + timezone.timedelta(hours=24)
                mijoz.save()
                count += 1
        self.message_user(request, f'{count} ta mijoz uchun ro\'yxat havolalari yaratildi.')
    generate_registration_links.short_description = 'Ro\'yxat havolalarini yaratish'
    
    def send_reminders(self, request, queryset):
        """Tanlangan mijozlarga eslatma yuborish"""
        count = 0
        for mijoz in queryset:
            if mijoz.telegram_id and mijoz.jami_qarz > 0:
                text = f"📢 Eslatma!\n\nSizning jami qarzingiz: {mijoz.jami_qarz:,.0f} so'm\n\nIltimos, imkon qadar tezroq to'lang. 🙏"
                if send_telegram_notification(mijoz.telegram_id, text):
                    count += 1
        self.message_user(request, f'{count} ta mijozga eslatma yuborildi.')
    send_reminders.short_description = 'Eslatma yuborish'


@admin.register(Qarz)
class QarzAdmin(admin.ModelAdmin):
    list_display = ['mijoz', 'summa_display', 'qolgan_display', 'holat_badge', 'muddat', 'sana']
    list_filter = ['holat', 'sana', 'muddat']
    search_fields = ['mijoz__ism', 'mijoz__telefon', 'sabab']
    readonly_fields = ['holat', 'yaratilgan', 'yangilangan']
    date_hierarchy = 'sana'
    
    fieldsets = (
        ('Asosiy Ma\'lumotlar', {
            'fields': ('mijoz', 'summa', 'qolgan', 'sabab')
        }),
        ('Sanalar', {
            'fields': ('sana', 'muddat')
        }),
        ('Holat', {
            'fields': ('holat', 'eslatma_yuborildi'),
            'classes': ('collapse',)
        }),
        ('Tizim Ma\'lumotlari', {
            'fields': ('yaratilgan', 'yangilangan'),
            'classes': ('collapse',)
        }),
    )
    
    def summa_display(self, obj):
        return f"{obj.summa:,.0f} so'm"
    summa_display.short_description = 'Summa'
    
    def qolgan_display(self, obj):
        if obj.qolgan > 0:
            return format_html('<span style="color: red; font-weight: bold;">{:,.0f} so\'m</span>', obj.qolgan)
        return format_html('<span style="color: green;">0 so\'m (to\'liq)</span>')
    qolgan_display.short_description = 'Qolgan'
    
    def holat_badge(self, obj):
        colors = {
            'ochiq': 'orange',
            'qisman': 'blue',
            'yopiq': 'green',
            'muddati_otgan': 'red'
        }
        return format_html(
            '<span style="background: {}; color: white; padding: 3px 10px; border-radius: 3px;">{}</span>',
            colors.get(obj.holat, 'gray'),
            obj.get_holat_display()
        )
    holat_badge.short_description = 'Holat'
    
    def save_model(self, request, obj, form, change):
        """Qarz saqlanganida mijozga bildirishnoma yuborish"""
        is_new = not obj.pk
        super().save_model(request, obj, form, change)
        
        if is_new and obj.mijoz.telegram_id:
            # Yangi qarz bildirishnomasi
            text = f"""
📢 Yangi qarz qo'shildi!

💰 Summa: {obj.summa:,.0f} so'm
📦 Sabab: {obj.sabab or 'Ko\'rsatilmagan'}
"""
            if obj.muddat:
                text += f"📅 Muddat: {obj.muddat.strftime('%d.%m.%Y')}\n"
            
            text += "\nBatafsil: /qarzim"
            
            send_telegram_notification(obj.mijoz.telegram_id, text)
            
            # Bildirishnoma jurnalga yozish
            Bildirishnoma.objects.create(
                mijoz=obj.mijoz,
                turi='yangi_qarz',
                matn=text,
                holat='yuborildi'
            )


@admin.register(Tolov)
class TolovAdmin(admin.ModelAdmin):
    list_display = ['mijoz', 'qarz_info', 'summa_display', 'sana', 'yaratilgan']
    list_filter = ['sana', 'yaratilgan']
    search_fields = ['mijoz__ism', 'izoh']
    readonly_fields = ['yaratilgan']
    date_hierarchy = 'sana'
    
    fieldsets = (
        ('Asosiy Ma\'lumotlar', {
            'fields': ('qarz', 'mijoz', 'summa', 'izoh')
        }),
        ('Sana', {
            'fields': ('sana',)
        }),
        ('Tizim Ma\'lumotlari', {
            'fields': ('yaratilgan',),
            'classes': ('collapse',)
        }),
    )
    
    def qarz_info(self, obj):
        return f"{obj.qarz.summa:,.0f} so'm"
    qarz_info.short_description = 'Qarz'
    
    def summa_display(self, obj):
        return format_html('<span style="color: green; font-weight: bold;">{:,.0f} so\'m</span>', obj.summa)
    summa_display.short_description = 'To\'lov'
    
    def save_model(self, request, obj, form, change):
        """To'lov saqlanganida mijozga bildirishnoma yuborish"""
        is_new = not obj.pk
        super().save_model(request, obj, form, change)
        
        if is_new and obj.mijoz.telegram_id:
            # To'lov tasdig'i bildirishnomasi
            text = f"""
✅ To'lovingiz qabul qilindi!

💳 To'lov: {obj.summa:,.0f} so'm
📊 Qolgan qarz: {obj.qarz.qolgan:,.0f} so'm
📅 Sana: {obj.sana.strftime('%d.%m.%Y %H:%M')}
"""
            if obj.qarz.qolgan <= 0:
                text += "\n🎉 Qarzingiz to'liq to'landi! Rahmat!"
            
            send_telegram_notification(obj.mijoz.telegram_id, text)
            
            # Bildirishnoma jurnalga yozish
            Bildirishnoma.objects.create(
                mijoz=obj.mijoz,
                turi='tolov_tasdig',
                matn=text,
                holat='yuborildi'
            )


@admin.register(Bildirishnoma)
class BildirishnomaAdmin(admin.ModelAdmin):
    list_display = ['mijoz', 'turi_badge', 'holat_badge', 'yuborilgan']
    list_filter = ['turi', 'holat', 'yuborilgan']
    search_fields = ['mijoz__ism', 'matn']
    readonly_fields = ['yuborilgan']
    date_hierarchy = 'yuborilgan'
    
    def turi_badge(self, obj):
        return format_html(
            '<span style="background: #2196F3; color: white; padding: 3px 8px; border-radius: 3px;">{}</span>',
            obj.get_turi_display()
        )
    turi_badge.short_description = 'Turi'
    
    def holat_badge(self, obj):
        colors = {'yuborildi': 'orange', 'yetdi': 'green', 'xato': 'red'}
        return format_html(
            '<span style="background: {}; color: white; padding: 3px 8px; border-radius: 3px;">{}</span>',
            colors.get(obj.holat, 'gray'),
            obj.get_holat_display()
        )
    holat_badge.short_description = 'Holat'


@admin.register(Sozlama)
class SozlamaAdmin(admin.ModelAdmin):
    list_display = ['kalit', 'qiymat', 'tavsif']
    search_fields = ['kalit', 'qiymat', 'tavsif']
```

### Admin Panelda Ko'rinadigan Shart

Django admin panelda quyidagi ma'lumotlar ko'rinadi:

1. **Dashboard (Asosiy sahifa)**
   - Jami mijozlar soni
   - Jami qarz summasi
   - Bugungi to'lovlar
   - Muddati o'tgan qarzlar soni

2. **Mijozlar ro'yxati**
   - Ism, telefon
   - Telegram ulangan/ulanmagan
   - Jami qarzi
   - Qarz qo'shish, to'lov qabul tugmalari

3. **Qarzlar ro'yxati**
   - Mijoz nomi
   - Summa, qolgan summa
   - Holat (badge)
   - Muddat
   - Filter: holat, sana, muddat

4. **To'lovlar ro'yxati**
   - Mijoz
   - Qarz ma'lumoti
   - To'lov summasi
   - Sana

5. **Bildirishnomalar jurnali**
   - Kimga yuborilgan
   - Turi (yangi qarz, to'lov, eslatma)
   - Holat
   - Vaqt

---

## 📱 Telegram Bot — Mijozlar Uchun {#telegram-bot}

### Bot Imkoniyatlari

Telegram bot **faqat mijozlar uchun** - admin Django panelda ishlaydi.

```
┌─────────────────────────────────────┐
│         💰 MENING QARZIM            │
│                                     │
│  📊 Jami qarz: 150,000 so'm        │
│  🟡 Faol qarzlar: 1 ta             │
│  ⏰ Muddati: 15.02.2024            │
│                                     │
│  [📜 Batafsil tarix]               │
│  [📞 Do'kon bilan bog'lanish]      │
└─────────────────────────────────────┘
```

### Telegram Bot Kodi (aiogram 3.x)

```python
# bot/main.py

from aiogram import Bot, Dispatcher, types, F
from aiogram.filters import Command
from aiogram.types import ReplyKeyboardMarkup, KeyboardButton
from aiogram.fsm.storage.memory import MemoryStorage
import asyncio
import logging
from django.core.wsgi import get_wsgi_application
import os

# Django setup
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
application = get_wsgi_application()

from shop.models import Mijoz, Qarz, Tolov

logging.basicConfig(level=logging.INFO)

# Bot token
BOT_TOKEN = os.getenv('BOT_TOKEN', 'YOUR_BOT_TOKEN')

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher(storage=MemoryStorage())


# Keyboards
def main_keyboard():
    kb = ReplyKeyboardMarkup(
        keyboard=[
            [KeyboardButton(text="💰 Mening qarzim")],
            [KeyboardButton(text="📜 Qarz tarixi")],
            [KeyboardButton(text="📞 Do'kon bilan bog'lanish")]
        ],
        resize_keyboard=True
    )
    return kb


# Handlers

@dp.message(Command("start"))
async def cmd_start(message: types.Message):
    telegram_id = message.from_user.id
    
    # Ro'yxatga olish tokeni bormi?
    if len(message.text.split()) > 1:
        token = message.text.split()[1]
        if token.startswith("REG_"):
            actual_token = token[4:]
            
            try:
                mijoz = Mijoz.objects.get(reg_token=actual_token)
                
                # Token muddati tekshirish
                if mijoz.token_expires and mijoz.token_expires < timezone.now():
                    await message.answer(
                        "⚠️ Bu havola eskirgan (24 soat o'tdi).\n"
                        "Do'kon egasidan yangi havola so'rang.\n"
                        f"📞 {os.getenv('SHOP_PHONE', '+998901234567')}"
                    )
                    return
                
                # Mijozni ro'yxatdan o'tkazish
                mijoz.telegram_id = telegram_id
                mijoz.telegram_username = message.from_user.username or ''
                mijoz.reg_token = ''
                mijoz.token_expires = None
                mijoz.save()
                
                await message.answer(
                    f"Assalomu alaykum, {mijoz.ism}! 👋\n\n"
                    "Siz QarzchiBot ga ulandingiz.\n"
                    "Bu bot orqali do'kondagi qarzingizni\n"
                    "istalgan vaqt ko'rishingiz mumkin.\n\n"
                    "Pastdagi tugmalardan foydalaning:",
                    reply_markup=main_keyboard()
                )
                return
                
            except Mijoz.DoesNotExist:
                await message.answer(
                    "⚠️ Havola yaroqsiz.\n"
                    "Do'kon egasiga murojaat qiling."
                )
                return
    
    # Oddiy /start
    try:
        mijoz = Mijoz.objects.get(telegram_id=telegram_id)
        await message.answer(
            f"Assalomu alaykum, {mijoz.ism}! 👋\n\n"
            "Xush kelibsiz!",
            reply_markup=main_keyboard()
        )
    except Mijoz.DoesNotExist:
        await message.answer(
            "⛔ Kechirasiz, siz ro'yxatda yo'qsiz.\n\n"
            "Agar do'konimizdan qarz olgan bo'lsangiz,\n"
            "do'kon egasiga murojaat qiling:\n"
            f"📞 {os.getenv('SHOP_PHONE', '+998901234567')}"
        )


@dp.message(F.text == "💰 Mening qarzim")
async def show_debts(message: types.Message):
    telegram_id = message.from_user.id
    
    try:
        mijoz = Mijoz.objects.get(telegram_id=telegram_id)
    except Mijoz.DoesNotExist:
        await message.answer("⛔ Siz ro'yxatda yo'qsiz.")
        return
    
    # Faol qarzlar
    qarzlar = mijoz.qarzlar.filter(holat__in=['ochiq', 'qisman', 'muddati_otgan'])
    
    if not qarzlar.exists():
        await message.answer(
            "✅ Sizda hozirda faol qarz yo'q.\n"
            "Rahmat! 😊"
        )
        return
    
    # Qarzlar haqida ma'lumot
    jami_qarz = sum(q.qolgan for q in qarzlar)
    
    text = f"💰 MENING QARZIM\n"
    text += "━━━━━━━━━━━━━━━━━━━━━\n\n"
    text += f"📊 Jami qarz: {jami_qarz:,.0f} so'm\n"
    text += f"🔢 Faol qarzlar: {qarzlar.count()} ta\n\n"
    
    for qarz in qarzlar:
        holat_emoji = {
            'ochiq': '🟡',
            'qisman': '🔵',
            'muddati_otgan': '🔴'
        }
        
        text += f"{holat_emoji.get(qarz.holat, '⚪')} {qarz.sabab or 'Qarz'}\n"
        text += f"   💰 Qolgan: {qarz.qolgan:,.0f} so'm\n"
        
        if qarz.muddat:
            muddat_text = qarz.muddat.strftime('%d.%m.%Y')
            if qarz.holat == 'muddati_otgan':
                text += f"   ⚠️ Muddat o'tgan: {muddat_text}\n"
            else:
                text += f"   📅 Muddat: {muddat_text}\n"
        
        text += "\n"
    
    text += "━━━━━━━━━━━━━━━━━━━━━\n"
    text += "To'lov qilish uchun do'kon bilan bog'laning."
    
    await message.answer(text)


@dp.message(F.text == "📜 Qarz tarixi")
async def show_history(message: types.Message):
    telegram_id = message.from_user.id
    
    try:
        mijoz = Mijoz.objects.get(telegram_id=telegram_id)
    except Mijoz.DoesNotExist:
        await message.answer("⛔ Siz ro'yxatda yo'qsiz.")
        return
    
    # Barcha qarzlar va to'lovlar
    qarzlar = mijoz.qarzlar.all()[:10]  # Oxirgi 10 ta
    
    if not qarzlar.exists():
        await message.answer("📜 Qarz tarixi bo'sh.")
        return
    
    text = "📜 QARZ TARIXI\n"
    text += "━━━━━━━━━━━━━━━━━━━━━\n\n"
    
    for qarz in qarzlar:
        holat_text = {
            'ochiq': '🟡 Ochiq',
            'qisman': '🔵 Qisman',
            'yopiq': '🟢 Yopiq',
            'muddati_otgan': '🔴 Muddati o\'tgan'
        }
        
        text += f"📅 {qarz.sana.strftime('%d.%m.%Y')}\n"
        text += f"   💰 Summa: {qarz.summa:,.0f} so'm\n"
        text += f"   📦 {qarz.sabab or 'Qarz'}\n"
        text += f"   {holat_text.get(qarz.holat, 'N/A')}\n"
        
        # To'lovlar
        tolovlar = qarz.tolovlar.all()
        if tolovlar.exists():
            text += "   💳 To'lovlar:\n"
            for tolov in tolovlar:
                text += f"      • {tolov.sana.strftime('%d.%m.%Y')} — {tolov.summa:,.0f} so'm\n"
        
        if qarz.qolgan > 0:
            text += f"   📊 Qolgan: {qarz.qolgan:,.0f} so'm\n"
        
        text += "\n"
    
    await message.answer(text)


@dp.message(F.text == "📞 Do'kon bilan bog'lanish")
async def contact_shop(message: types.Message):
    shop_name = os.getenv('SHOP_NAME', 'Do\'kon')
    shop_phone = os.getenv('SHOP_PHONE', '+998901234567')
    
    text = f"📞 {shop_name} Ma'lumotlari\n"
    text += "━━━━━━━━━━━━━━━━━━━━━\n"
    text += f"📱 {shop_phone}\n"
    
    await message.answer(text)


# Bot ishga tushirish
async def main():
    await dp.start_polling(bot)


if __name__ == '__main__':
    asyncio.run(main())
```

---

## 🆕 Mijozni Ro'yxatga Olish Jarayoni {#royxat}

### Django Admin Panelda

1. **Admin Django panelga kiradi**
2. **Mijozlar → Yangi mijoz qo'shish**
3. **Ma'lumotlarni to'ldiradi:**
   - Ism: Sardor Karimov
   - Telefon: +998901234567
   - Izoh: Mahalla boshi (ixtiyoriy)

4. **Saqlaydi → "Generate registration links" action ni tanlaydi**
5. **Sahifa yangilanadi, "Telegram Havola" maydonida havola paydo bo'ladi:**
   ```
   https://t.me/YourBot?start=REG_abc123xyz
   ```

6. **Admin bu havolani mijozga yuboradi** (SMS, WhatsApp, yoki shaxsan)

### Mijoz Tomoni

1. **Mijoz havolaga bosadi**
2. **Telegram bot ochiladi, /start avtomatik yuboriladi**
3. **Bot mijozni taniydi va ro'yxatdan o'tkazadi:**
   ```
   Assalomu alaykum, Sardor! 👋

   Siz QarzchiBot ga ulandingiz.
   Bu bot orqali do'kondagi qarzingizni
   istalgan vaqt ko'rishingiz mumkin.

   [💰 Mening qarzim]
   [📜 Qarz tarixi]
   [📞 Do'kon bilan bog'lanish]
   ```

4. **Mijoz endi botdan foydalana oladi**

---

## ⚠️ Xato Holatlari (Edge Cases) {#xato}

| Holat | Nima bo'ladi | Javob |
|-------|-------------|-------|
| Token eskirgan (24 soat) | Bot rad etadi | ⚠️ Havola eskirgan, yangi havola so'rang |
| Token yaroqsiz | Bot rad etadi | ⚠️ Havola yaroqsiz |
| Noma'lum foydalanuvchi /start | Kirish rad | ⛔ Siz ro'yxatda yo'qsiz |
| Mijozda qarz yo'q | Tabriklaydi | ✅ Sizda faol qarz yo'q! |
| Admin to'lovni noto'g'ri kiritdi | Django validatsiya | To'lov qarzdan oshmasligi kerak |

---

## 🔔 Eslatma Tizimi {#eslatma}

### Avtomatik Eslatmalar

Django signallar orqali:

```python
# signals.py

from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Qarz, Tolov
from .utils import send_telegram_notification


@receiver(post_save, sender=Qarz)
def qarz_yaratilganda(sender, instance, created, **kwargs):
    """Yangi qarz qo'shilganda mijozga bildirishnoma yuborish"""
    if created and instance.mijoz.telegram_id:
        text = f"""
📢 Yangi qarz qo'shildi!

💰 Summa: {instance.summa:,.0f} so'm
📦 Sabab: {instance.sabab or 'Ko\'rsatilmagan'}
"""
        if instance.muddat:
            text += f"📅 Muddat: {instance.muddat.strftime('%d.%m.%Y')}\n"
        
        text += "\nBatafsil: /qarzim"
        
        send_telegram_notification(instance.mijoz.telegram_id, text)


@receiver(post_save, sender=Tolov)
def tolov_qabul_qilindi(sender, instance, created, **kwargs):
    """To'lov qo'shilganda mijozga bildirishnoma yuborish"""
    if created and instance.mijoz.telegram_id:
        text = f"""
✅ To'lovingiz qabul qilindi!

💳 To'lov: {instance.summa:,.0f} so'm
📊 Qolgan qarz: {instance.qarz.qolgan:,.0f} so'm
📅 Sana: {instance.sana.strftime('%d.%m.%Y %H:%M')}
"""
        if instance.qarz.qolgan <= 0:
            text += "\n🎉 Qarzingiz to'liq to'landi! Rahmat!"
        
        send_telegram_notification(instance.mijoz.telegram_id, text)
```

### Muddati O'tgan Qarzlar (Celery/APScheduler)

```python
# tasks.py (Celery yoki management command)

from django.utils import timezone
from .models import Qarz
from .utils import send_telegram_notification


def muddati_otgan_eslatma():
    """Har kuni 09:00 da ishga tushadi"""
    bugun = timezone.now().date()
    
    # Muddati o'tgan qarzlar
    muddati_otgan = Qarz.objects.filter(
        muddat__lt=bugun,
        holat__in=['ochiq', 'qisman', 'muddati_otgan'],
        eslatma_yuborildi=False
    )
    
    for qarz in muddati_otgan:
        if qarz.mijoz.telegram_id:
            text = f"""
⚠️ Diqqat!

Qarzingizning to'lash muddati o'tgan.

💰 Qarz: {qarz.qolgan:,.0f} so'm
📅 Muddat edi: {qarz.muddat.strftime('%d.%m.%Y')}

Iltimos, imkon qadar tezroq to'lang. 🙏
"""
            send_telegram_notification(qarz.mijoz.telegram_id, text)
            qarz.eslatma_yuborildi = True
            qarz.save()
```

---

## 📊 Hisobot Tizimi {#hisobot}

### Django Admin Custom View

```python
# admin.py (qo'shimcha)

from django.contrib import admin
from django.urls import path
from django.shortcuts import render
from django.db.models import Sum, Count
from django.utils import timezone


class CustomAdminSite(admin.AdminSite):
    def get_urls(self):
        urls = super().get_urls()
        custom_urls = [
            path('hisobot/', self.admin_view(self.hisobot_view), name='hisobot'),
        ]
        return custom_urls + urls
    
    def hisobot_view(self, request):
        """Kunlik/Oylik hisobot"""
        bugun = timezone.now().date()
        oy_boshi = bugun.replace(day=1)
        
        # Jami ma'lumotlar
        jami_mijozlar = Mijoz.objects.filter(holat='faol').count()
        jami_qarz = Qarz.objects.filter(holat__in=['ochiq', 'qisman']).aggregate(Sum('qolgan'))['qolgan__sum'] or 0
        muddati_otgan = Qarz.objects.filter(muddat__lt=bugun, holat__in=['ochiq', 'qisman']).count()
        
        # Oylik statistika
        oy_yangi_qarz = Qarz.objects.filter(sana__gte=oy_boshi).aggregate(Sum('summa'))['summa__sum'] or 0
        oy_tolov = Tolov.objects.filter(sana__gte=oy_boshi).aggregate(Sum('summa'))['summa__sum'] or 0
        
        context = {
            'bugun': bugun,
            'jami_mijozlar': jami_mijozlar,
            'jami_qarz': jami_qarz,
            'muddati_otgan': muddati_otgan,
            'oy_yangi_qarz': oy_yangi_qarz,
            'oy_tolov': oy_tolov,
        }
        
        return render(request, 'admin/hisobot.html', context)


# settings.py da
from django.contrib.admin import site
admin.site = CustomAdminSite()
```

---

## 🔐 Xavfsizlik {#xavfsizlik}

### Django Xavfsizlik

```python
# settings.py

# CSRF Protection
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_SECURE = True

# XSS Protection
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True

# HTTPS
SECURE_SSL_REDIRECT = True

# Admin URL o'zgartirish
ADMIN_URL = os.getenv('ADMIN_URL', 'admin/')  # admin/ o'rniga secret-admin-panel-xyz/
```

### Telegram Bot Xavfsizlik

- Har bir so'rovda `telegram_id` tekshiriladi
- Mijoz faqat o'z qarzini ko'ra oladi
- Admin funksiyalari bot orqali mavjud emas

---

## 🗄️ Ma'lumotlar Bazasi Sxemasi {#db}

Django ORM avtomatik yaratadi, lekin qo'lda SQL kerak bo'lsa:

```sql
-- Mijozlar
CREATE TABLE mijoz (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ism VARCHAR(200) NOT NULL,
    telefon VARCHAR(20) UNIQUE NOT NULL,
    telegram_id BIGINT UNIQUE,
    telegram_username VARCHAR(100),
    holat VARCHAR(20) DEFAULT 'faol',
    izoh TEXT,
    reg_token VARCHAR(50) UNIQUE,
    token_expires DATETIME,
    yaratilgan DATETIME DEFAULT CURRENT_TIMESTAMP,
    yangilangan DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Qarzlar
CREATE TABLE qarz (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mijoz_id INTEGER REFERENCES mijoz(id),
    summa DECIMAL(12,2) NOT NULL,
    qolgan DECIMAL(12,2) NOT NULL,
    sabab TEXT,
    sana DATE DEFAULT CURRENT_DATE,
    muddat DATE,
    holat VARCHAR(20) DEFAULT 'ochiq',
    eslatma_yuborildi BOOLEAN DEFAULT FALSE,
    yaratilgan DATETIME DEFAULT CURRENT_TIMESTAMP,
    yangilangan DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- To'lovlar
CREATE TABLE tolov (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    qarz_id INTEGER REFERENCES qarz(id),
    mijoz_id INTEGER REFERENCES mijoz(id),
    summa DECIMAL(12,2) NOT NULL,
    izoh TEXT,
    sana DATETIME DEFAULT CURRENT_TIMESTAMP,
    yaratilgan DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Bildirishnomalar
CREATE TABLE bildirishnoma (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mijoz_id INTEGER REFERENCES mijoz(id),
    turi VARCHAR(20),
    matn TEXT,
    holat VARCHAR(20) DEFAULT 'yuborildi',
    xato_sabab TEXT,
    yuborilgan DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Sozlamalar
CREATE TABLE sozlama (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    kalit VARCHAR(100) UNIQUE,
    qiymat TEXT,
    tavsif VARCHAR(255)
);
```

---

## 🗄️ Fayl Tuzilmasi {#fayl}

```
qarzchibot/
├── manage.py
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── shop/                      # Asosiy Django app
│   ├── __init__.py
│   ├── models.py              # Mijoz, Qarz, Tolov, Bildirishnoma
│   ├── admin.py               # Admin panel konfiguratsiyasi
│   ├── signals.py             # Post-save signals
│   ├── utils.py               # Yordamchi funksiyalar
│   ├── views.py               # API view'lar (agar kerak bo'lsa)
│   └── migrations/
│
├── bot/                       # Telegram bot
│   ├── __init__.py
│   ├── main.py                # Bot asosiy fayl
│   ├── handlers.py            # Bot handlerlar
│   ├── keyboards.py           # Telegram klaviaturalar
│   └── utils.py               # Bot uchun yordamchi funksiyalar
│
├── templates/
│   └── admin/
│       └── hisobot.html       # Custom hisobot sahifasi
│
├── static/
│   └── admin/
│       └── custom.css         # Custom CSS
│
├── requirements.txt
├── .env
└── README.md
```

---

## ⚙️ Texnik Stack {#stack}

| Komponent | Texnologiya | Sababi |
|-----------|------------|--------|
| Backend | `Django 5.0+` | Admin panel tayyor, ORM kuchli |
| Bot framework | `aiogram 3.x` | Async, zamonaviy, tez |
| Ma'lumotlar bazasi | `PostgreSQL` (prod) / `SQLite` (dev) | Ishonchli va professional |
| Task Queue | `Celery + Redis` | Eslatmalar uchun |
| Hosting | `Railway.app`, `Heroku`, yoki VPS | Arzon va ishonchli |
| Til | `Python 3.11+` | |

### requirements.txt

```txt
Django==5.0
psycopg2-binary==2.9.9
aiogram==3.3.0
python-dotenv==1.0.0
celery==5.3.4
redis==5.0.1
Pillow==10.1.0
```

---

## 🚀 MVP Rejasi {#mvp}

### 1-Bosqich (Django Admin + Asosiy Bot) — 1 hafta

| # | Funksiya | Holat |
|---|----------|-------|
| 1 | Django models yaratish | ⬜ |
| 2 | Admin panel konfiguratsiya | ⬜ |
| 3 | Mijoz ro'yxatdan o'tkazish (havola orqali) | ⬜ |
| 4 | Bot: /start, qarz ko'rish | ⬜ |
| 5 | Signals: yangi qarz bildirishnomasi | ⬜ |
| 6 | Signals: to'lov bildirishnomasi | ⬜ |

### 2-Bosqich (Eslatmalar) — 3 kun

| # | Funksiya | Holat |
|---|----------|-------|
| 7 | Celery setup | ⬜ |
| 8 | Muddati o'tgan eslatmalar (avtomatik) | ⬜ |
| 9 | Qo'lda eslatma yuborish (admin action) | ⬜ |

### 3-Bosqich (Hisobotlar) — 2 kun

| # | Funksiya | Holat |
|---|----------|-------|
| 10 | Admin dashboard statistika | ⬜ |
| 11 | Custom hisobot sahifasi | ⬜ |
| 12 | Export Excel/PDF | ⬜ |

### 4-Bosqich (Polish) — 2 kun

| # | Funksiya | Holat |
|---|----------|-------|
| 13 | Admin panel customization (CSS) | ⬜ |
| 14 | Error handling va logging | ⬜ |
| 15 | Deploy va testing | ⬜ |

---

## 📝 Qo'shimcha Eslatmalar

### Afzalliklari

1. **Tezkor ishga tushirish** — Django admin tayyor, faqat konfiguratsiya kerak
2. **Ikki interfeys** — Admin web'da, mijozlar Telegram'da
3. **Xavfsizlik** — Django'ning o'rnatilgan xavfsizlik mexanizmlari
4. **Kengaytirish oson** — Keyinchalik API, mobil ilova qo'shish mumkin
5. **Professional** — To'liq tarix, logging, hisobotlar

### Kamchiliklari

1. **Ikki qism** — Django va Bot alohida ishga tushirish kerak
2. **Hosting** — VPS yoki Railway kerak (shared hosting yetmaydi)
3. **Admin mobilda noqulay** — Veb interfeys telefondan qiyin

### Kelajakda Qo'shish Mumkin

- 📱 **Telegram Admin Panel** — Telegram orqali ham admin boshqaradi
- 📊 **Grafik hisobotlar** — Chart.js bilan
- 💳 **To'lov integatsiyasi** — Click, Payme
- 📸 **Chek yuklash** — To'lov cheklari rasm
- 🔔 **SMS eslatmalar** — Telegram'da bo'lmaganlarga
- 🌐 **REST API** — Mobil ilova uchun
- 👥 **Ko'p do'kon** — Bir tizimda bir nechta do'kon

---

*QarzchiBot v2.1 — Django Admin + Telegram Bot*  
*Professional qarz boshqaruv tizimi*  
*Hujjat yangilangan: 2026*
