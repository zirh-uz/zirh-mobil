# Zirh kutubxonasini ishlatish bo‘yicha qo‘llanma
Zirh kutubxonasidan foydalanishni boshlash uchun uni o‘z Android loyihangizga to‘g‘ri ulash lozim. Quyidagi bosqichlarni bajaring:
#
`settings.gradle.kts` faylini oching, `dependencyResolutionManagement` bo'limidagi `repositories` qatoriga jitpack orqali manzilini qo‘shing: 
```kotlin
  dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://jitpack.io")
            credentials.username = providers.gradleProperty("authToken").get()
        }
    }
}
```
#
`jitpack.io` repozitoriyasiga ulanish uchun token kerak bo‘ladi. Buning uchun quyidagi qatorni `.gradle/gradle.properties` fayliga qo‘shing:
```kotlin
authToken=jp_uhvnfp5qnsucafao6b0hplsmvs
```
#
Endi esa, `app` modulining `build.gradle.kts` faylida `dependencies` bo‘limiga quyidagicha yozing:
```kotlin
  dependencies {
    implementation("com.github.XojiakbarJamoldinov:zirh-mobil-lib:v1.0.0")
  }
```
#
Zirh kutubxonasini loyihangizga ulab bo'lganingizdan so'ng, kod takrorlanishining oldini olish va strukturalashtirish maqsadida barcha `Activity`lar uchun umumiy ota klass yaratish tavsiya etiladi. Odatda bu klass `BaseActivity.kt` (yoki `.java`) deb nomlanadi.Bu klass kutubxonani boshlang'ich sozlash (initializatsiya) uchun xizmat qiladi.
Quyidagi kabi `BaseActivity.kt` faylini yarating:
```kotlin
import uz.zirh.zirhlib.ZirhMilliy

open class BaseActivity : ComponentActivity() {
    private lateinit var lib: ZirhMilliy
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ZirhMilliy.faylManzili(assets)
        lib = ZirhMilliy() 
    }
}
```
#
`AndroidManifest.xml` faylliga internet uchun ruxsat(permission) qo'shing.
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
---
## 🔐 Ma'lumotlarni Shifrlash

### ✨ Nima uchun shifrlash zarur?

Ilovada ishlatiladigan ba'zi muhim ma'lumotlar — masalan, **backend server URL’lari**, **directory manzillar**, **tokenlar**, **hash qiymatlar**, va **ilovaning imzo (signature) ma'lumotlari** — maxfiy va xavfsizlik talablariga javob beruvchi shaklda saqlanishi kerak.

Agar bu ma’lumotlar shifrlanmagan holda apk ichida yoki fayl tizimida saqlansa, ular tahlil qilinib (reverse engineering), ilovaga hujum qilish, soxta so‘rov yuborish yoki serverdan noto‘g‘ri foydalanish uchun ishlatilishi mumkin.

Shuning uchun **shifrlash yordamida bu ma’lumotlarni himoyalash** va ularni faqat kerakli paytda, kerakli joyda yechib olish (deshifrovka qilish) lozim bo‘ladi.

#

### 🔒 Shifrlanadigan ma’lumotlar
- 🌐 Backend URL manzillari  
- 📁 Server path / directory strukturalari  
- 🔑 API tokenlar, maxfiy kalitlar  
- 🧮 Hashlangan qiymatlar (masalan, SHA256)  
- 🖋 Ilovaning imzo sertifikati (signature)  

#

### 📌 Shifrlash va foydalanish ketma-ketligi
Quyidagi bosqichlar orqali ilova uchun maxfiy ma’lumotlarni shifrlash va undan xavfsiz foydalanish mumkin.
### 1. 📄 JSON fayl tayyorlash

Birinchi bosqichda maxfiy ma'lumotlar `data.json` faylga quyidagi formatda yoziladi:

```json
{
    "domainlar": ["https://jsonplaceholder.typicode.com", "https://httpbin.org"],
    "havolalar": ["posts", "post"],
    "hashlar": ["sha256///UzJAZYxLBnEpBwXAcmd4WHi7f8aYgfMExGnoyp5B04=", "sha256//IFG+z/oQKXfpUYOHgWHy5axgkT9B01XSxwb2AHDyN34="],
    "tokenlar": ["abc123", "def456", "ghi789"],
    "imzo": "B83BC82F4E631114B79E5E01DB8590CAF76D3384B2CFCFEE27F31B0C886262B1"
}
```
### 🔑 2. AES-256 kalitni yaratish

```bash
openssl rand -base64 32 > aes.key
```

Bu buyruq orqali **256-bit AES kalit** yaratiladi va `base64` formatda `aes.key` faylga yoziladi.

#

### 🔎 3. AES kalitni hex (hash ko‘rinish) ga o‘tkazish

```bash
base64 -d aes.key | xxd -p -c 256
```

Natijada siz AES kalitni 64 ta belgidan iborat **hex formatdagi** qiymatini olasiz. Ushbu qiymat quyida `-K` parametriga qo‘shiladi.

#

### 🔐 4. JSON faylni AES-256-CBC bilan shifrlash

```bash
openssl enc -aes-256-cbc -in data.json -out data.enc -K <AES_KALIT_HEX_QIYMAT> -iv 00000000000000000000000000000000 -nosalt
```

> **Eslatma:**
> - `-K` – yuqorida olingan hex AES kalit
> - `-iv` – **o‘zgarmas** (fixed) IV qiymat: `00000000000000000000000000000000`
> - `-nosalt` – deterministik natija olish uchun

#

### 🔄 5. AES kalitni **raw format**ga o‘tkazish

```bash
base64 -d aes.key > aes.raw
```

Bu qadamda `aes.key` ni raw ikkilamchi formatga o‘tkazasiz. Sababi, RSA bilan shifrlash uchun **real binar shakldagi** fayl kerak bo‘ladi.

#

### 🛡 6. RSA public kalit bilan AES kalitni shifrlash

```bash
openssl pkeyutl -encrypt -pubin -inkey public.pem -in aes.raw -out kalit.enc
```

Bu yerda:
- `public.pem` – RSA ochiq kalit
- `aes.raw` – AES kalitning binar ko‘rinishi
- `kalit.enc` – **shifrlangan AES kalit**, bu `data.enc` ni yechish uchun kerak bo‘ladi.
#

### 📦 7. Fayllarni `assets/` papkaga joylash

Quyidagi 2 faylni `assets/` ichiga joylashtiring:

```
app/src/main/assets/
├── data.enc      ✅ Shifrlangan JSON
└── kalit.enc     ✅ RSA bilan shifrlangan AES kalit
```

> ⚠️ **Eslatma:** Fayl nomlari **majburiy**: `data.enc` va `kalit.enc` bo‘lishi kerak. Kutubxona faqat shu nomlar orqali fayllarni qidiradi.


### ❓ Nega `aes.key` ni `aes.raw` ga aylantirish kerak?

`aes.key` fayli `base64` kodlangan ko‘rinishda saqlanadi. RSA shifrlash esa **faqatgina xom (raw) baytli fayllar** bilan ishlaydi. Shuning uchun `base64 -d` orqali uni `aes.raw` formatga aylantiramiz.

---


## 📁 `faylManzili()` Funksiyasi

Ushbu funksiya kutubxonaga **`assets` papkasidagi faylga kirish imkonini beradi**.  
Funksiyani chaqirish orqali kutubxona ichida joylashgan faylga to‘g‘ridan-to‘g‘ri kirish va uni o‘qish imkoniyati paydo bo‘ladi.

> 📌 **Eslatma:** Faylga kirish uchun Android ilovasidagi `assets` obyektini funksiya ichiga uzatishingiz kerak.

#

### 💻 Foydalanish namunasi

Quyidagi kod `BaseActivity` klassida `onCreate()` ichida `faylManzili()` funksiyasini qanday chaqirishni ko‘rsatadi:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    // assets papkasiga kirish uchun kutubxonaga kontekst beriladi
    ZirhMilliy.faylManzili(assets)
}
```

#

### 📂 Qayerdagi faylni o‘qiydi?

`assets/` papkasida joylashgan fayllarni o‘qish uchun mo‘ljallangan.  
Masalan:

```
app/
├── src/
│   └── main/
│       └── assets/
│           └── config.json
```

Yuqoridagi holatda `faylManzili()` yordamida `config.json` fayl o‘qilishi mumkin.

---
## 🚦 `vpnniAniqlash()` Funktsiyasi

Ushbu funksiya ilova ishga tushgan qurilmada **VPN ulanishi mavjud yoki yo‘qligini** aniqlash uchun ishlatiladi.  
Agar qurilmada faol VPN ulanishi mavjud bo‘lsa, funksiya `true` qiymat qaytaradi, aks holda `false`.

> 🔒 **VPN mavjudligi xavfsizlik talablariga zid bo‘lishi mumkin.**  
> Foydalanuvchiga ogohlantiruvchi xabar berish va ilovani ishlashini to‘xtatish yoki yopish tavsiya etiladi.

**Eslatma:** Funksiya ishlashi uchun internetga ruxsat kerak.
#

### 💻 Foydalanish namunasi

Quyidagi kod `BaseActivity` klassida `onResume()` ichida `vpnniAniqlash()` funksiyasini qanday chaqirishni ko‘rsatadi:

```kotlin
  override fun onResume() {
        super.onResume()
        val isVpn = lib.vpnniAniqlash()
        if (isVpn) {
            Toast.makeText(this, "Sizning qurulmangizda vpn aniqlandi", Toast.LENGTH_LONG).show()
            finishAffinity()
            System.exit(0)
        }
    }
```
---

## 🧪 `emulyatorniAniqlash()` Funktsiyasi

Ushbu funksiya ilova ishga tushgan qurilmaning **emulyator (soxta qurilma)** ekanligini aniqlash uchun mo‘ljallangan.  
Agar ilova real qurilmada ishlayotgan bo‘lsa, funksiya `false` qaytaradi, aks holda emulyator aniqlansa `true` qiymat qaytaradi.

> ⚠️ **Emulyatorda ishlayotgan ilova xavfsizlik jihatidan ishonchsiz hisoblanadi.**  
> Bu holatda ilova buzilishi, teskari tahlil (reverse engineering) qilinishi yoki yolg‘on ma’lumotlar bilan test qilinishi mumkin.

> ✅ **Eslatma:**  
> Agar emulyator aniqlansa, foydalanuvchiga ogohlantiruvchi xabar chiqaring va ilovani yopish uchun quyidagi funksiyalardan foydalaning:
>
> ```kotlin
> finishAffinity() 
> System.exit(0)  
> ```
#

### 💻 Foydalanish namunasi

Quyidagi kod `BaseActivity` klassida `onResume()` ichida `emulyatorniAniqlash()` funksiyasini qanday chaqirishni ko‘rsatadi:

```kotlin
 override fun onResume() {
        super.onResume()
        val isEmulyator = lib.emulyatorniAniqlash(this)
        if (isEmulyator) {
            Toast.makeText(this, "Ilova emulyatorda ishga tushdi", Toast.LENGTH_LONG).show()
            finishAffinity()
            System.exit(0)
        }
    }
```
---
## ⚠️ `rootniAniqlash()` Funktsiyasi

Ushbu funksiya ilova ishga tushgan qurilmaning **root qilinganligini** aniqlash uchun ishlatiladi.  
Agar qurilma root qilingan bo‘lsa, funksiya `true` qiymat qaytaradi, aks holda `false`.

> 🔐 **Root qilingan qurilma xavfsizlik talablariga javob bermaydi.**  
> Bunday qurilmalarda foydalanuvchi yoki zararli dastur tizimga chuqur kirib, ilova ma’lumotlarini o‘zgartirishi yoki o‘g‘irlashi mumkin.

> ✅ **Eslatma:**  
> Agar qurilma root qilingan bo‘lsa (`true` qaytsa), foydalanuvchiga ogohlantiruvchi xabar chiqarish va ilovani darhol yopish kerak:
>
> ```kotlin
> finishAffinity() 
> System.exit(0)  
> ```
#

### 💻 Foydalanish namunasi

Quyidagi kod `BaseActivity` klassida `onResume()` ichida `rootniAniqlash()` funksiyasini qanday chaqirishni ko‘rsatadi:

```kotlin
 override fun onResume() {
        super.onResume()
        val isRoot = lib.rootniAniqlash()
        if (isRoot) {
            Toast.makeText(this, "Ilova root qurulmada ishga tushdi", Toast.LENGTH_LONG).show()
            finishAffinity()
            System.exit(0)
        }
    }
```
---
## 🛡️ `playMarketniAniqlash()` Funktsiyasi

Ushbu funksiya ilova Play Market orqali o‘rnatilganligini aniqlash uchun ishlatiladi. Agar ilova boshqa manbadan (masalan .apk orqali qo‘lda) o‘rnatilgan bo‘lsa, bu xavfsizlikka tahdid solishi mumkin.

> ✅ **Eslatma:**  
> Agar ilova Play Marketdan yuklanmagan bo‘lsa (`false` qaytsa), foydalanuvchiga ogohlantiruvchi xabar chiqarish va ilovani darhol yopish kerak:
>
> ```kotlin
> finishAffinity() 
> System.exit(0)  
> ```
#

### 💻 Foydalanish namunasi

Quyidagi kod `BaseActivity` klassida `onResume()` ichida `playMarketniAniqlash()` funksiyasini qanday chaqirishni ko‘rsatadi:

```kotlin
 override fun onResume() {
    super.onResume()
    val isPlayMarket = lib.playMarketniAniqlash(this)
    if (!isPlayMarket) {
        Toast.makeText(this, "Iltimos ilovani Play Marketdan yuklab oling", Toast.LENGTH_LONG).show()
        finishAffinity()
        System.exit(0)
    }
}
```
---
## ✍️ `imzoAniqlash()` Funktsiyasi

imzoAniqlash() funksiyasi ilovaning ruxsatsiz yoki o‘zgartirilgan APK emasligini aniqlash uchun mo‘ljallangan. U ilovaning imzosini tekshiradi va agar u original imzo bilan mos tushmasa, false qiymat qaytaradi.

> ✅ **Eslatma:**  
> Agar ilova Imzosi xato bo‘lsa (`false` qaytsa), foydalanuvchiga ogohlantiruvchi xabar chiqarish va ilovani darhol yopish kerak:
>
> ```kotlin
> finishAffinity() 
> System.exit(0)  
> ```
#

### 💻 Foydalanish namunasi

Quyidagi kod `BaseActivity` klassida `onResume()` ichida `imzoAniqlash()` funksiyasini qanday chaqirishni ko‘rsatadi:

```kotlin
 override fun onResume() {
        super.onResume()
        val isSignature = lib.imzoAniqlash(this)
        if (!isSignature) {
            Toast.makeText(this, "Ilova imzosi xato!", Toast.LENGTH_LONG).show()
            finishAffinity()
            System.exit(0)
        }
    }
```
---
## 📡 `malumotOlish()` Funksiyasi

`malumotOlish()` funksiyasi — server bilan aloqani ta'minlovchi asosiy metod bo‘lib, u xavfsiz tarzda so‘rov yuborish va javob olish imkonini beradi. Bu funksiya AES yordamida shifrlangan `data.enc` faylidagi domen, path va boshqa parametrlar asosida HTTP so‘rov yuboradi.

## 🛡️ SSL Pinning

> **SSL Pinning** vositasi orqali ilova faqat **ishonchli sertifikat** bilan ishlovchi serverga ulanadi. Bu orqali man-in-the-middle (MITM) hujumlarining oldi olinadi.

**Server sertifikatidan hash olish:**

```bash
echo | openssl s_client -servername your_server_address -connect your_server_address:443 | openssl x509 -pubkey -noout | openssl pkey -pubin -outform DER | openssl dgst -sha256 -binary | openssl enc -base64
```
> ✅ **Eslatma:**  
> Olingan hash qiymat json faylidagi hashlar maydonida bo'lishi kerak aks holda serverga so'rov yuborilmaydi.
>
### ⚙️ Parametrlar

| Parametr      | Turi             | Tavsif |
|---------------|------------------|--------|
| `domainIndex` | `Int`            | Domenlar ro‘yxatidan indeks (0 — birinchi domen). |
| `pathIndex`   | `Int`            | Havolalar (`endpoint`) ro‘yxatidan indeks. |
| `id`          | `String?`        | Qo‘shimcha identifikator (masalan: `12` → `/posts/12`). |
| `method`      | `String`         | HTTP metod: `GET`, `POST`, `PUT`, `DELETE`. |
| `headers`     | `Array<String>?` | Sarlavhalar (headers), ixtiyoriy. |
| `body`        | `String?`        | JSON formatdagi so‘rov ma’lumotlari, ixtiyoriy. |

### 🔁 Natija
Funksiya `String` (odatda JSON) formatida javob qaytaradi. Uni `Gson` yordamida quyidagicha obyektga aylantirish mumkin:

```kotlin
val gson = Gson()
val parsed = gson.fromJson(jsonString, GetResponse::class.java)
```
### `GET` so‘rov
```kotlin
   Thread{
        val res: String = lib.malumotOlish(0, 0, null, "GET", null, null)
  //            MALUMOT STRING FARMATDA KELADI MALUMOTNI JSON PARSE QILISH KERAK
        runOnUiThread {
            Toast.makeText(this, "GET: ${res}", Toast.LENGTH_SHORT).show()
        }
  //            JSON PARSE QILISH MALUMOTNI
        val gson = Gson()
        val parsed = gson.fromJson(res, GetResponse::class.java)
    }.start()
```
### `POST` so‘rov
```kotlin
      val name = "Sarlavha"
      val desc = "Bu postning tanasi"

      val postData = HashMap<String, String>()
      postData["title"] = name
      postData["body"] = desc
      postData["userId"] = "1"
      val jsonBody: String = gson.toJson(postData)
      val headers = arrayOf("Content-Type: application/json" /*,"Authorization: Bearer YOUR_TOKEN"*/)
      Thread {
          try {
              val res = lib.malumotOlish(0, 0, null, "POST", headers, jsonBody)
//                JSON PARSEDAN OLDIN STRING FARMATDA RESPONSE QAYTADI MALUMOTNI JSON PARSE QILISH KERAK
              runOnUiThread {
                  Toast.makeText(this, "POST: ${res}", Toast.LENGTH_SHORT).show()
              }
//                JSON PARSE QILISH
              val gson = Gson()
              val parsed = gson.fromJson(res, DataResponse::class.java)

          } catch (e: Exception) {
              Log.e("POST ERROR", "Xatolik: ${e.message}")
          }
      }.start()
```

### `PUT` so‘rov
```kotlin
      val updatedData: MutableMap<String, String> = HashMap()
      updatedData["title"] = "Yangi sarlova"
      updatedData["body"] = "Put so'rovini test qilish"
      updatedData["userId"] = java.lang.String.valueOf(1)

      val putBody = gson.toJson(updatedData)
      val putHeaders = arrayOf("Content-Type: application/json")
      Thread {
          try {
              // IDISI 7 GA TENG MALUMOTNI TAHRIRLASH
              val res = lib.malumotOlish(0, 0, "7", "PUT", putHeaders, putBody)
              runOnUiThread {
                  Toast.makeText(this, "PUT: ${res}", Toast.LENGTH_SHORT).show()
              }

              val parsed = gson.fromJson(res, DataResponse::class.java)

          } catch (e: java.lang.Exception) {
              Log.e("PUT REQUEST ERROR", "Xatolik yuz berdi:", e)
          }
      }.start()
```

### `DELETE` so‘rov
```kotlin
   Thread {
        try {
            // IDISI 2 GA TENG MALUMOTNI O;CHIRISH
            val res: String = lib.malumotOlish(0, 0, "2", "DELETE", null, null)
            runOnUiThread {
                Toast.makeText(this, "DELETE: ${res}", Toast.LENGTH_SHORT).show()
            }

            val parsed = gson.fromJson(res, DataResponse::class.java)

        } catch (e: java.lang.Exception) {
            Log.e("DELETE ERROR", "Xatolik yuz berdi:", e)
        }
    }.start()
```
