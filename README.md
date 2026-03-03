# RENTIFY — Premium Vehicle Rental System
**Version 3.5.0 | QBCore Framework**

---

## ÖZELLİKLER

- **Çoklu Lokasyon** — İstediğin kadar kiralama noktası ekle, her birinin kendi NPC'si ve blip'i var
- **Dinamik Fiyatlandırma** — Fiyatlar sunucuda belirli aralıklarla otomatik güncellenir, tüm oyunculara bildirim gider
- **Seviye Sistemi** — Her kiralamada XP kazan, seviye atladıkça indirim kazan (max %20 indirim)
- **Hasar Sistemi** — İade anında motor ve kaporta sağlığı kontrol edilir, hasar varsa depozitoden kesinti yapılır
- **Depozito & Sigorta** — Konfigüre edilebilir depozito ve sigorta yüzdeleri
- **GPS Takip** — Kiralık araç haritada gerçek zamanlı izlenir, oyuncu kendi aracını, admin tüm araçları görebilir
- **Kiralama Fişi** — Kiralama ve iade işlemlerinde envantere fiziksel fiş düşer, kullanılabilir item
- **Anahtar Sistemi** — qb-vehiclekeys ile tam entegrasyon, iade/iptal/süre dolumunda otomatik silinir
- **Discord Loglama** — Tüm işlemler (kiralama, iade, iptal, süre dolumu) Discord'a loglanır
- **Admin Komutları** — Tüm aktif kiralamaları listele, istediğini iptal et, GPS paneli aç
- **Güvenlik** — Event spam koruması, araç model whitelist, cooldown sistemi, maksimum kiralama limiti
- **Temiz UI** — Kategori filtreleme, sıralama, araç istatistikleri, dinamik fiyat göstergesi, ödeme yöntemi seçimi
- **qb-target Desteği** — Opsiyonel, config ile aktif edilir
- **LegacyFuel Desteği** — Otomatik algılanır, yoksa sorunsuz çalışmaya devam eder

---

## GEREKSİNİMLER

| Kaynak | Durum |
|--------|-------|
| FiveM artifact 6683+ | Zorunlu |
| QBCore Framework (güncel) | Zorunlu |
| qb-inventory | Zorunlu |
| qb-vehiclekeys | Önerilen (olmadan da çalışır) |
| LegacyFuel | Opsiyonel |
| qb-target | Opsiyonel |

---

## KURULUM

### 1 — Klasörü kopyala

`advanced_rental` klasörünü `resources/` altına koy:
```
resources/
└── advanced_rental/
```

### 2 — server.cfg

`qb-core`'un **altına** ekle:
```
ensure qb-core
ensure advanced_rental
```

Admin komutlarını kullanacaksan aşağıdaki satırları da ekle:
```
add_ace group.admin command.rentallist   allow
add_ace group.admin command.rentalcancel allow
add_ace group.admin command.rentalgps    allow
```

### 3 — Item'ları tanımla

`qb-core/shared/items.lua` dosyasına ekle:

```lua
['vehicle_key'] = {
    name        = 'vehicle_key',
    label       = 'Araç Anahtarı',
    weight      = 100,
    type        = 'item',
    image       = 'vehicle_key.png',
    unique      = false,
    useable     = false,
    shouldClose = false,
    combinable  = nil,
    description = 'Bir araç anahtarı',
},
['rental_receipt'] = {
    name        = 'rental_receipt',
    label       = 'Kiralama Fişi',
    weight      = 10,
    type        = 'item',
    image       = 'rental_receipt.png',
    unique      = false,
    useable     = true,
    shouldClose = true,
    combinable  = nil,
    description = 'Kiralama fişi — kullanarak detayları görüntüle',
},
```

> **Not:** Item görselleri için `vehicle_key.png` ve `rental_receipt.png` dosyalarını qb-inventory'nin `html/images/` klasörüne koy.

### 4 — Araç görsellerini ekle

`html/vehicles/` klasörüne araç görsellerini koy:
- Önerilen format: `.webp`
- Önerilen boyut: **400×225 px** (16:9)
- Dosya adı = araç model adı. Örnek: `adder.webp`, `zentorno.png`
- Görsel bulunamazsa UI otomatik olarak emoji fallback gösterir, hata vermez

### 5 — Sunucuyu başlat

```
ensure advanced_rental
```

---

## YAPILANDIRMA

Tüm ayarlar `config.lua` dosyasındadır.

### Temel Ayarlar

```lua
Config.UseTarget       = false   -- true: qb-target kullanılır
Config.Currency        = 'bank'  -- 'bank' veya 'cash'
Config.ProximityDist   = 3.5     -- NPC yakınlık mesafesi (metre)
Config.AbandonDistance = 80.0    -- Araçtan uzaklaşma uyarı mesafesi
```

### Güvenlik

```lua
Config.Security = {
    ValidateVehicleModel = true,  -- Model whitelist kontrolü
    PreventEventSpam     = true,  -- Spam koruması
    CooldownSeconds      = 3,     -- İşlemler arası bekleme süresi
    MaxActiveRentals     = 1,     -- Kişi başı maksimum kiralama
}
```

### Hasar Sistemi

```lua
Config.DamageSystem = {
    Enabled              = true,
    EngineThreshold      = 700.0, -- Bu değerin altı = motor hasarlı (max 1000)
    BodyThreshold        = 700.0, -- Bu değerin altı = kaporta hasarlı (max 1000)
    DamagePenaltyPercent = 30,    -- Depozitadan kesilecek yüzde
}
```

### Dinamik Fiyatlandırma

```lua
Config.DynamicPricing = {
    Enabled        = true,
    UpdateInterval = 300000, -- Milisaniye (300000 = 5 dakika)
    MinMultiplier  = 0.85,
    MaxMultiplier  = 1.30,
}
```

### Discord Loglama

```lua
Config.Logging = {
    Enabled    = true,
    UseDiscord = true,
    Webhook    = 'https://discord.com/api/webhooks/ID/TOKEN',
}
```

### Yeni Lokasyon Ekleme

```lua
Config.Locations['ornek'] = {
    label      = 'Rentify Örnek Nokta',
    coords     = vector4(X, Y, Z, HEADING),
    spawnPoint = vector4(X, Y, Z, HEADING),
    npc        = { model = 's_m_m_autoshop_01', scenario = 'WORLD_HUMAN_CLIPBOARD' },
    blip       = { enabled = true, sprite = 225, color = 5, scale = 0.85 },
    allowedCategories = { 'sport', 'luxury' }, -- boş bırakılırsa tüm kategoriler görünür
}
```

### Yeni Araç Ekleme

```lua
{
    name   = 'Araç Adı',
    model  = 'model_adi',
    image  = 'model_adi',  -- html/vehicles/ içindeki dosya (uzantısız)
    price  = 500,          -- saatlik baz fiyat ($)
    rating = 5,            -- 1–5 yıldız
    emoji  = '🚗',
    stats  = { speed = 90, accel = 85, brake = 80, handling = 88 },
},
```

---

## OYUN İÇİ KULLANIM

### Araç Kiralama
1. Haritadaki Rentify blip'ine git
2. NPC'ye yaklaş → `E` tuşuna bas
3. Kategori seç → Araç seç → Süre seç
4. Ödeme yöntemini seç (Banka / Nakit) → Onayla
5. Araç spawn noktasında belirir, otomatik içine ışınlanırsın

### Araç İadesi
- NPC'ye git ve `G` tuşuna bas
- Veya `/rentalreturn` komutunu kullan (herhangi bir yerden)
- Hasarsız iade = depozito tamamen iade edilir
- Hasarlı iade = depozitoden kesinti yapılır, kalan iade edilir

### Süre Dolumu
- Süreye 5 dakika kala uyarı gelir
- Süre dolunca araç otomatik silinir, anahtar ve blip kaldırılır

---

## KOMUTLAR

### Oyuncu Komutları

| Komut | Açıklama |
|-------|----------|
| `/rentalreturn` | Aktif kiralık aracı iade et |
| `/rentalinfo` | Kiralama ID ve plakasını göster |
| `/rentalgps` | Aracın konumunu haritaya ekle |

### Admin Komutları

| Komut | Açıklama |
|-------|----------|
| `/rentallist` | Tüm aktif kiralamaları listele (konsol veya oyun içi) |
| `/rentalcancel <rentalId>` | Belirli bir kiralama ID'sini iptal et |
| `/rentalgps` | Tüm kiralık araçları haritada göster |

---

## UYUMLULUK

| Kaynak | Durum |
|--------|-------|
| qb-core | ✅ Tam uyumlu |
| qb-inventory | ✅ Tam uyumlu |
| qb-vehiclekeys | ✅ Tam uyumlu |
| ox_inventory | ⚠️ Item sistemi çalışmayabilir |
| LegacyFuel | ✅ Otomatik algılanır |
| qb-target | ✅ Config ile aktif edilir |
| ESX | ❌ Desteklenmez |

---

## KLASÖR YAPISI

```
advanced_rental/
├── fxmanifest.lua
├── config.lua
├── client/
│   └── main.lua
├── server/
│   └── main.lua
└── html/
    ├── index.html
    ├── style.css
    ├── script.js
    ├── images/
    │   └── rentify-logo.png
    └── vehicles/
        ├── arac_model.webp
        └── ...
```

---

## SSS

**Araç spawn olmuyor?**
Model adının doğru olduğunu kontrol et. `Config.Security.ValidateVehicleModel = false` yaparak test edebilirsin. F8 konsolunda hata var mı bak.

**Anahtar envantere düşmüyor?**
`qb-core/shared/items.lua`'da `vehicle_key` item'ının tanımlı olduğunu doğrula. `qb-vehiclekeys` resource'unun çalıştığından emin ol.

**Admin komutları çalışmıyor?**
`server.cfg`'ye `add_ace group.admin command.rentallist allow` satırlarının eklendiğinden emin ol (kurulum adım 2'ye bak).

**Dinamik fiyat güncellemiyor?**
`Config.DynamicPricing.Enabled = true` olduğunu ve `UpdateInterval`'ın milisaniye cinsinden yazıldığını kontrol et.

**ox_inventory kullanıyorum, item düşmüyor?**
ox_inventory için item ekleme sistemi farklı çalışır. `server/main.lua` içindeki `AddReceiptItem` ve `AddKeyItem` fonksiyonlarını ox_inventory API'sine göre düzenlemelisin.

---

*Rentify v3.5.0 — QBCore Premium Vehicle Rental System*
