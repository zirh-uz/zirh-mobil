# Zirh-mobil kutubxonasini ishlatish bo'yicha yo'riqnoma

---

[![Android](https://img.shields.io/badge/Android-blue?style=for-the-badge)](#android)
[![Flutter](https://img.shields.io/badge/Flutter-blue?style=for-the-badge)](#Flutter)

---

# Android

Zirh-mobil kutubxonasidan foydalanishni boshlash uchun uni o'z Android loyihangizga to'g'ri ulash lozim. Quyidagi bosqichlarni bajaring:

# `aar` fayl orqali kutubxonani ulash

`aar` faylni kutubxonaga qo'shish uchun loyihangizdagi `app` papkasining ichida yangi `libs` nomli papka yarating va unga `.aar` formatidagi
Zirh-mobil kutubxona faylini joylashtiring.

```
app/
 └── libs/
      └── zirhlib-release-v2.0.0.aar
```

#

## `settings.gradle` yoki `settings.gradle.kts` faylida `flatDir` sozlamasini yozing

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        flatDir{
            dirs("app/libs")
        }
    }
}
```

## `app` modulining `build.gradle` faylida kutubxonani ulash

#

```kotlin
dependencies {
    implementation(":zirhlib-release-v2.0.0.aar")
}
```

> **Eslatma:**
>
> - Kutubxonaning nomi `.aar` fayl nomi bilan to‘g‘ri kelishi kerak `(zirhlib-releasex.x.x.aar)`.

# Zirh kutubxonasini `jitpack` orqali loyihaga qo'shish

#

## `settings.gradle` yoki `settings.gradle.kts` faylida quydagicha yozing

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

## `app` modulining `build.gradle` faylida kutubxonani ulash

#

```kotlin
dependencies {
    implementation("com.github.Zirh-uz:mobil-lib:2.0.0")
}
```

#

`AndroidManifest.xml` faylliga internet uchun ruxsatlarni(permission) qo'shing.

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

Quydagi queriesni `AndroidManifest.xml` faylga qo'shing

```xml
 <queries>
       <package android:name="com.android.vending" />
       <package android:name="com.sec.android.app.samsungapps" />
</queries>
```

# Flutter

Zirh-mobile kutubxonasidan foydalanishni boshlash uchun uni o'z Flutter loyihangizga to'g'ri ulash lozim. Quyidagi bosqichlarni bajaring:

Flutter loyihamizni `android/app` papkani ichiga `libs` nomli kutubxona yaratib olamiz.

```
flutter_project/
├── android/
│   ├── app/
│   │   ├── libs/
│   │   │
│   │   ├── src/
│   │   └── build.gradle.kts
│   ├── build.gradle.kts
│   └── ...
├── lib/
├── pubspec.yaml
└── ...

```

Keyin esa libs papkani ichiga zirhlib-debug.aar kutubxonamizni joylashtirib olamiz

```
flutter_project/
├── android/
│   ├── app/
│   │   ├── libs/                            ← 📂 Bu yerga .aar fayl qo‘yiladi
│   │   │   └── zirhlib-release-v2.0.0.aar   ← 📦 JNI kutubxonani o‘z ichiga olgan .aar fayl
│   │   ├── src/
│   │   └── build.gradle.kts
│   └── ...
├── lib/
├── pubspec.yaml
└── ...
```

`settings.gradle.kts` faylida `flatDir` sozlamasini yozing

```
repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
        flatDir {
            dirs("app/libs")
        }
    }
```

app modulining build.gradle.kts faylida kutubxonani ulash

```
dependencies {
    implementation(files("libs/zirhlib-release-v2.0.0.aar"))
}
```

jitpack orqali app modulining build.gradle.kts faylida kutubxonani ulash

```
dependencies {
    implementation("com.github.Zirh-uz:mobil-lib:2.0.0")
}
```

jitpack orqali bo'lganda android/build.gradle.kts fayliga quyidagi o'zgarishni qilish kerak bo'ladi

```
allprojects {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

Eslatma:

Kutubxonaning nomi `.aar` fayl nomi bilan to'g'ri kelishi kerak `(zirh-mobil-lib-release.aar)`.

Yuqorida ko'rsatilgan quyidagi 2 faylni `assets/` ichiga joylashtiring:

```
app/
├── src/
│   └── main/
│       └── assets/
|           └── data.enc  ✅ Shifrlangan JSON
│           └── kalit.enc ✅ RSA bilan shifrlangan AES kalit
|
```

main.dart faylida native (C/C++) kutubxona bilan qanday ishlash ko‘rsatiladi. Bu yerda FFI (Foreign Function Interface) yordamida .so fayldan ma’lumotlar olinadi.

```dart
import 'dart:convert';
import 'dart:ffi' as ffi;
import 'package:ffi/ffi.dart';
import 'package:flutter/material.dart';
```

dart:ffi → native (C/C++) kod bilan ishlash uchun
ffi.dart → pointerlarni boshqarish (UTF8 conversion)
dart:convert → JSON parsing uchun

Native funksiya signaturalari

```dart
typedef NativeGetInfoFn = ffi.Pointer<Utf8> Function(ffi.Pointer<Utf8> key);
typedef DartGetInfoFn = ffi.Pointer<Utf8> Function(ffi.Pointer<Utf8> key);
```

Bu klass native kutubxona bilan ishlashni markazlashtiradi. (Ilovada faqat bitta instance ishlaydi)

```dart
static final ZirhService _instance = ZirhService._internal();
factory ZirhService() => _instance;
```

Native kutubxonani yuklash

```dart
final lib = ffi.DynamicLibrary.open('libmobil.so');
```

Funksiyani bog‘lash
Native funksiyani Flutterga ulaydi
Key orqali ma’lumot qaytaradi

```dart
_getInfo = lib.lookupFunction<NativeGetInfoFn, DartGetInfoFn>(
  'flutter_malumot_olish',
);

```

Ma’lumot olish funksiyalari

```dart
String get(String key)
```

## Minimal kod misolida ko'rishimiz mumkin

```dart
import 'dart:convert';
import 'dart:ffi' as ffi;
import 'package:ffi/ffi.dart';
import 'package:flutter/material.dart';

// Native signature
typedef NativeGetInfoFn = ffi.Pointer<Utf8> Function(ffi.Pointer<Utf8>);
typedef DartGetInfoFn = ffi.Pointer<Utf8> Function(ffi.Pointer<Utf8>);

class ZirhService {
  late final DartGetInfoFn _getInfo;

  void init() {
    final lib = ffi.DynamicLibrary.open('libmobil.so');
    _getInfo = lib.lookupFunction<NativeGetInfoFn, DartGetInfoFn>(
      'flutter_malumot_olish',
    );
  }

  List<String> getDomains() {
    final keyPtr = "domainlar".toNativeUtf8();
    final resPtr = _getInfo(keyPtr);
    malloc.free(keyPtr);

    if (resPtr.address == 0) return [];

    final raw = resPtr.toDartString();

    // JSON array bo‘lsa
    try {
      final decoded = jsonDecode(raw);
      if (decoded is List) {
        return decoded.map((e) => e.toString()).toList();
      }
    } catch (_) {}

    // fallback (a,b,c)
    return raw.split(',').map((e) => e.trim()).toList();
  }
}

void main() {
  final zirh = ZirhService();
  zirh.init();

  runApp(MaterialApp(
    home: Scaffold(
      appBar: AppBar(title: Text("Domains")),
      body: FutureBuilder(
        future: Future.value(zirh.getDomains()),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();

          final domains = snapshot.data!;
          return ListView(
            children: domains.map((d) => ListTile(title: Text(d))).toList(),
          );
        },
      ),
    ),
  ));
}
```

## Malumot Almashish

Dart tomonidan chaqiriladi va native kutubxona bilan to‘g‘ridan-to‘g‘ri bog‘lanadi.

```dart
typedef NativeRequestFn = ffi.Pointer<Utf8> Function(
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    ffi.Pointer<ffi.Uint8>,
    ffi.Int32,
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    );

typedef DartRequestFn = ffi.Pointer<Utf8> Function(
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    ffi.Pointer<ffi.Uint8>,
    int,
    ffi.Pointer<Utf8>,
    ffi.Pointer<Utf8>,
    );

```

Bizda data.json quyidagicha bo'lgan holatda

```dart
{
    "imzo": "f2eac1edc0332ffd6d47cc446f8cc636a5b207aeac49c397b8f1b497a36c6089",
    "playmarket": false,
    "emulyator": true,
    "vpn": true,

    "hashlar": [
        "sha256//k+swi1D7Mu27FDJ9DAfns27/YipZz5s7BezuYsaXM/s=",
        "sha256//ItYAkeNu4OWLwJwqsG+rlGN46LIFJkfrRcx9BFbuTtA=",
        "sha256//xvnBemDjgnzraqJYsDMz2CgXT2Zq3CFBfmyyYSdLdrU=",
        "sha256//5BWYNtPxvjsl+qhQLxo3jz3ZaK74xyHT/QdOhBB07i0="
    ],

    "domainlar": [
        "https://jsonplaceholder.typicode.com",
        "https://httpbin.org"
    ],

    "api": {
        "base_url": "https://jsonplaceholder.typicode.com",

        "endpoints": {
            "get_post": {
                "path": "/posts/1",
                "method": "GET"
            },
            "create_post": {
                "path": "/posts",
                "method": "POST"
            },
            "update_post": {
                "path": "/posts/1",
                "method": "PUT"
            },
            "delete_post": {
                "path": "/posts/1",
                "method": "DELETE"
            }
        }
    }
}
```

FFI orqali bog'lanish

```dart
 final lib = ffi.DynamicLibrary.open('libmobil.so');

_get = lib.lookupFunction<NativeGetInfoFn, DartGetInfoFn>(
  'flutter_malumot_olish',
);

_request = lib.lookupFunction<NativeRequestFn, DartRequestFn>(
  'flutter_malumot_almashish',
);
```

API chaqirish qismi

```dart
// GET
zirh.callApi("get_post");

// POST
zirh.callApi("create_post", body: {"title": "Test", "body": "Hello", "userId": 1});

// PUT
zirh.callApi("update_post", body: {"id": 1, "title": "Updated", "body": "Yangilandi"});

// DELETE
zirh.callApi("delete_post");
```

---

## 🔐 Ma'lumotlarni Shifrlash

### ✨ Nima uchun shifrlash zarur?

Ilovada ishlatiladigan ba'zi muhim ma'lumotlar — masalan, **backend server URL’lari**, **directory manzillar**, **tokenlar**, **hash qiymatlar**, **ilovaning imzo (signature) ma'lumotlari**, **ilova UI komponentalari**, **ilovada ishlatiladigan stringlar**, **ilovaga zarur mantiqiy qiymatlarni** va **ilovaning turli qurilmalarda ishlashi uchun konfiguratsiya parametrlarini** — maxfiy va xavfsizlik talablariga javob beruvchi shaklda saqlanishi kerak.

Agar bu ma’lumotlar shifrlanmagan holda apk ichida yoki fayl tizimida saqlansa, ular tahlil qilinib (reverse engineering), ilovaga hujum qilish, himoya vositalarini osonlik bilan olib tashlash yoki aylanib o'tishni, soxta so‘rov yuborish yoki serverdan noto‘g‘ri foydalanish uchun ishlatilishi mumkin.

Shuning uchun **shifrlash yordamida bu ma’lumotlarni himoyalash** va ularni faqat kerakli paytda, kerakli joyda yechib olish (deshifrovka qilish) orqali ilovaning xavfsizligini yaxshilashimiz mumkin bo'ladi.

#

### 🔒 Shifrlanadigan ma’lumotlar

- 🌐 Backend URL manzillari
- 📁 Server path / directory strukturalari
- 🔑 API tokenlar, maxfiy kalitlar
- 🧮 Hashlangan qiymatlar (masalan, SHA256)
- 🖋 Ilovaning imzo sertifikati (signature)
- 🎨 Ilova UI komponentalari
- ⚙️ Ilovaga zarur mantiqiy qiymatlar
- 🔤 Ilovada ishlatiladigan stringlar

#

### 📌 Shifrlash va foydalanish ketma-ketligi

Quyidagi bosqichlar orqali ilova uchun maxfiy ma’lumotlarni shifrlash va undan xavfsiz foydalanish mumkin.

### 1. 📄 JSON fayl tayyorlash

Birinchi bosqichda maxfiy ma'lumotlar `data.json` faylga quyidagi formatda yoziladi:

```json
{
	"imzo": "f2eac1edc0332ffd6d47cc446f8cc636a5b207aeac49c397b8f1b497a36c6089",
	"playmarket": false,
	"emulyator": true,
	"vpn": true,

	"hashlar": [
		"sha256//k+swi1D7Mu27FDJ9DAfns27/YipZz5s7BezuYsaXM/s=",
		"sha256//ItYAkeNu4OWLwJwqsG+rlGN46LIFJkfrRcx9BFbuTtA=",
		"sha256//xvnBemDjgnzraqJYsDMz2CgXT2Zq3CFBfmyyYSdLdrU=",
		"sha256//5BWYNtPxvjsl+qhQLxo3jz3ZaK74xyHT/QdOhBB07i0="
	],

	"domainlar": ["https://jsonplaceholder.typicode.com", "https://httpbin.org"],

	"api": {
		"base_url": "https://jsonplaceholder.typicode.com",
		"endpoints": {
			"get_post": { "path": "/posts/1", "method": "GET" },
			"create_post": { "path": "/posts", "method": "POST" },
			"update_post": { "path": "/posts/1", "method": "PUT" },
			"delete_post": { "path": "/posts/1", "method": "DELETE" }
		}
	},

	"ui": {
		"colors": {
			"background": "#0F172A",
			"card": "#1E293B",
			"primary": "#2563EB",
			"text_light": "#FFFFFFB3",
			"text_dark": "#FFFFFF"
		},
		"labels": {
			"dashboard_title": "Zirh Dashboard",
			"vpn_status": "VPN",
			"emulator_status": "Emulyator",
			"playmarket_status": "Play Market"
		},
		"buttons": {
			"refresh": "Yangilash"
		}
	}
}
```

Agar siz kutubxonani `malumotalmashish()` funksiyasidan foydalanmoqchi bo'lsangiz `data.json` faylni ichiga majburiy tarzda `hashlar` maydoni bo'lishi kerak. Ilova ushbu hashlar orqali server bilan xavfsiz aloqani yani SSL Pinningni amalga oshiradi. Yuqoridagi hashni farmati kabi o'z serveringizdan `sha256` farmatda olish uchun quydagi kamandadan foydalanishingiz mumkin bo'ladi.

```bash
echo | openssl s_client -connect kun.uz:443 -servername kun.uz 2>/dev/null \
| openssl x509 -pubkey -noout \
| openssl pkey -pubin -outform DER \
| openssl dgst -sha256 -binary \
| openssl enc -base64
```

> SSL Pinning bu ilova va server o'rtasidagi so'rovlarga uchinchi shaxs aralashishidan himoyalaydi.

Yuqoridagi data.jsondan ilovada quyidagicha UI komponentalarini chaqirib ishlatishimiz mumkin bo'ladi.
Bundan tashqari ichma-ich (nested) ma'lumotlarga murojaat qilishda (.) bilan murojaat qilinadi.

```dart
void _loadConfig() {
    // Ranglar, label va button textlarini o‘qish
    final uiColors = zirh.get("ui.colors");
    final uiLabels = zirh.get("ui.labels");
    final uiButtons = zirh.get("ui.buttons");
    final domainList = zirh.get("domainlar");

    setState(() {
      colors = uiColors.isNotEmpty ? jsonDecode(uiColors) : {};
      labels = uiLabels.isNotEmpty ? jsonDecode(uiLabels) : {};
      buttons = uiButtons.isNotEmpty ? jsonDecode(uiButtons) : {};
      domains = domainList.isNotEmpty
          ? (jsonDecode(domainList) as List).map((e) => e.toString()).toList()
          : [];
    });
  }
```

Bu fayl kutubxonani konfiguratsiya fayli bo'lib, agar ilovada `playmarket`, `emulyator` va `vpn` tekshiruvi joriy etish kerak bo'lsa ularni `true` qiymatga tenglab qo'yish kerak bo'ladi. Agarda bu tekshiruvlar kerak bo'lmasa `false` qiymatga tenglash kerak bo'ladi.

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

Natijada siz AES kalitni 64 ta belgidan iborat **hex formatdagi** qiymatini olasiz. Ushbu qiymatni siz pastdagi kodni ichidagagi `AES_KEY_HEX` shu o'zgaruvchini qiymatidagi `e5f3d69593946edbe60ad48663c15e2e503def3c6f7691b46fe81d9c67b15fd8` shu hashni o'rniga joylashtiring.

#

### 🔐 4. JSON faylni AES bilan shifrlash

`encrypt.sh` fayl oching va quydagi kodni fayl ichiga yozing va saqlang.

```python
#!/bin/bash

# Sozlamalar
INPUT_FILE="data.json"
OUTPUT_FILE="data.enc"
AES_KEY_HEX="e5f3d69593946edbe60ad48663c15e2e503def3c6f7691b46fe81d9c67b15fd8" # 64 ta belgi (256-bit)

# 1. Tasodifiy 16 baytli (32 ta hex belgisi) IV yaratish
IV_HEX=$(openssl rand -hex 16)
echo "Yaratilgan IV: $IV_HEX"

# 2. IV-ni HEX-dan Binary-ga o'tkazish (xxd-siz)
# Biz IV-ni shunchaki shifrlayotgandek qilib, lekin "null" shifr bilan binary-ga o'tkazamiz
echo -n "$IV_HEX" | openssl enc -aes-256-cbc -K "$AES_KEY_HEX" -iv "$IV_HEX" -p | grep "iv=" | cut -d= -f2 > /dev/null # Bu qator shunchaki tekshiruv uchun

# Eng oddiy yo'li: IV-ni binary faylga yozish uchun openssl rand-dan foydalanamiz
# Lekin bizga aynan o'sha $IV_HEX kerak. Shuning uchun printf-dan foydalanamiz:
printf "$(echo $IV_HEX | sed 's/\(..\)/\\x\1/g')" > iv.bin

# 3. Ma'lumotni shifrlash
openssl enc -aes-256-cbc -K "$AES_KEY_HEX" -iv "$IV_HEX" -in "$INPUT_FILE" -out "temp.bin" -nosalt

# 4. Birlashtirish: IV (16 byte) + Ciphertext
cat iv.bin temp.bin > "$OUTPUT_FILE"

# 5. Tozalash
rm iv.bin temp.bin

echo "Tayyor: $OUTPUT_FILE (IV boshiga biriktirildi)"

```

Faylni saqlab olganingizdan keyin faylni ishga tushuring natijada sizda shifrlangan `data.enc` fayli hosil bo'ladi.

#

### 🔄 5. AES kalitni **raw format**ga o‘tkazish

```bash
base64 -d aes.key > aes.raw
```

Bu qadamda `aes.key` ni raw ikkilamchi formatga o‘tkazasiz. Sababi, RSA bilan shifrlash uchun **real binar shakldagi** fayl kerak bo‘ladi.

#

### 🛡 6. RSA public kalit bilan AES kalitni shifrlash

```bash
openssl pkeyutl -encrypt -pubin -inkey public_key.pem -in aes.raw -out kalit.enc -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256
```

Bu yerda:

- `public_key.pem` – RSA ochiq kalit (yuqorida berilgan)
- `aes.raw` – AES kalitning binar ko‘rinishi
- `kalit.enc` – **shifrlangan AES kalit**, bu `data.enc` ni yechish uchun kerak bo‘ladi.

#

### 📦 7. Fayllarni `assets/` papkaga joylash

Quyidagi 2 faylni `assets/` ichiga joylashtiring:

```
app/
├── src/
│   └── main/
│       └── assets/
|           └── data.enc ✅ Shifrlangan JSON
│           └── kalit.enc ✅ RSA bilan shifrlangan AES kalit
|
```

> ⚠️ **Eslatma:** Fayl nomlari **majburiy**: `data.enc` va `kalit.enc` bo‘lishi kerak. Kutubxona faqat shu nomlar orqali fayllarni qidiradi.

### ❓ Nega `aes.key` ni `aes.raw` ga aylantirish kerak?

`aes.key` fayli `base64` kodlangan ko‘rinishda saqlanadi. RSA shifrlash esa **faqatgina xom (raw) baytli fayllar** bilan ishlaydi. Shuning uchun `base64 -d` orqali uni `aes.raw` formatga aylantiramiz.

---

## 🚦 Vpnni aniqlash

Ushbu funksiya ilova ishga tushgan qurilmada **VPN ulanishi mavjud yoki yo‘qligini** aniqlash uchun ishlatiladi.  
Agar qurilmada vpn holatini aniqlash uchun data.jsonda `vpn`:`true` qilish yetarli agar qurulmada vpn yoniq bo'lsa ilova o'z ish jarayonini yakunlaydi.

> 🔒 **VPN mavjudligi xavfsizlik talablariga zid bo‘lishi mumkin.**
> VPN orqali foydalanuvchi o'z IP manzilini yashirishi, trafikni ushlab tahlil qilishi (MITM) va xavfsizlikni chetlab o'tuvchi vositalardan foydalanishi mumkin.
> Bunday hollarda foydalanuvchiga ogohlantiruvchi xabar berish va ilovani ishlashini to‘xtatish yoki yopish tavsiya etiladi.

**Eslatma:** Funksiya ishlashi uchun internetga ruxsat kerak.

#

---

## 🧪 Emulatorni aniqlash

Ushbu funksiya ilova ishga tushgan qurilmaning **emulyator (soxta qurilma)** ekanligini aniqlash uchun mo‘ljallangan.  
Agar ilova real qurilmada ishlayotgan bo‘lsa, ilova ishlashni davom etadi aks holda ilova ish jarayonini yakunlaydi. Ushbu funksiyani ishlatish uchun `data.json` fayldagi `emulyator`:`true` qilish yetarli.

> ⚠️ **Emulyator (simulyator)da ishlayotgan ilova xavfsizlik nuqtai nazaridan ishonchsiz hisoblanadi.**  
> Emulyator yordamida ilovaning serverga yuborayotgan va qabul qilayotgan ma'lumotlari tahlil qilinib, maxfiy ma’lumotlar ko'rish mumkin.

---

## ⚠️ Root qurilmalarni aniqlash

Ilovani Play Marketga o'rnatishdan oldin quydagi sozlamalarni qilish kerak bo'ladi.
Console Play Marketga o'tib (https://play.google.com/console) quydagi ketma ketliklarni bajaring.

```
Dashboard
Statistics
Publishing overview
Test and release
Monitor and improve
│   └── Reach and devices
│          └── Overview
│          └── Device catalog
│  └── Ratings and reviews
│  └── Android vitals
│  └── Policy and programs
```

`Monitor and improve` bo'limini tanlang va uni ichidan `Reach and devices` ni tanlang va `Device catalog` ni tanlang buyerda ochilgan oynadan o'ng tomon yuqorida `Manage exclusion rules` ni bosing va Play Integrity qismidan Configure bo'limini tanlang keyin sizda `Store listing visibility` oynasi ochiladi bu yerda sizga 4 ta bo'lim:

- No integrity checks
- Basic integrity checks
- Device integrity checks
- Strong integrity checks

Bu bo'limlarda siz `Device integrity checks` yoki `Strong integrity checks` ni tanlab qo'yishingiz zarur bu root qilingan va haqiqiyligi yo'qolgan qurulmalarda play marketdan ilova chiqmasligini taminlaydi.

Ushbu funksiya ilova ishga tushgan qurilmaning **root qilinganligini** doimiy ravishda aniqlaydi.  
Agar qurilma root qilingan bo‘lsa, ilova rootlangan qurilmada ishga tushmaydi.
Bundan tashqari zararli scriptlarni qo'llanganda ham ilova ishga tushmaydi.

> 🔐 **Root qilingan qurilma xavfsizlik talablariga javob bermaydi.**  
> Bunday qurilmalar ilovalarning:
> Ilovaning ichki fayllari (token, ma’lumotlar bazasi) o'g'irlanishi mumkin.
> Ilova funksiyalari o'zgartirilishi yoki "buzilishi" mumkin (code injection).
> Trafik (token/parollar) kuzatilib, tahlil qilinadi.

## 🛡️ Ilovani play marketdan o'rnatilganligini tekshirish.

Ushbu funksiya ilova Play Market orqali o‘rnatilganligini aniqlash uchun ishlatiladi. Agar ilova boshqa manbadan (masalan .apk orqali qo‘lda) o‘rnatilgan bo‘lsa, bu xavfsizlikka tahdid solishi mumkin. Ushbu funksiya ishlashi uchun `data.json` fayldagi `playmarket`:`true` qilish yetarli.

---

## ✍️ Ilova imzosini tekshirish

Ilova imzosini tekshirish funksiyasi ilovaning ruxsatsiz o‘zgartirilgan APK emasligini aniqlash uchun mo‘ljallangan. U ilovaning imzosini tekshiradi va agar u original imzo bilan mos tushmasa, ilova ishlash jarayonini to'xtatadi. Bu funksiyadan foydalanish uchun `data.json` faylidagi `imzo` ga ilovani imzolash uchun imzo faylni `sha256` qiymatni joylashtiring.

---

## `malumotolish()` funksiyasi

`malumotolish()` funksiyasi orqali siz `data.enc` faylni ichiga saqlagan malumotlaringizni olishingiz mumkin bo'ladi. Bu funksiyadan foydalanish uchun siz quydagi usulda shifrlangan malumotlaringizni xavfsiz tarzda olishingiz mumkin bo'ladi.

```java
import uz.zirh.zirhlib.ZirhMilliy

class MainActivity : ComponentActivity() {

    private lateinit var lib: ZirhMilliy

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        lib = ZirhMilliy()

        val testM = lib.malumotolish("hashlar")

    }
}

```

---

## 📡 `malumotalmashish()` funksiyasi

`malumotalmashish()` funksiyasi — server bilan aloqani ta'minlovchi asosiy metod bo‘lib, u xavfsiz tarzda so‘rov yuborish va javob olish imkonini beradi. Bu funksiya orqali ilova server bilan xavfsiz HTTP aloqani taminlaydi. Funksiya quydagi parametrlarni kutub qoladi.

### ⚙️ Parametrlar

| Parametr    | Turi             | Tavsif                                               |
| ----------- | ---------------- | ---------------------------------------------------- |
| `url`       | `String`         | Url manzil masalan(https://example.com/api/v1/users. |
| `METHOD`    | `String`         | HTTP metod: `GET`, `POST`, `PUT`, `DELETE`.          |
| `body`      | `String?`        | Body ixtiyoriy.                                      |
| `headers`   | `String`         | Sarlavhalar (headers), ixtiyoriy.                    |
| `filePath`  | `String`         | Fayl joylashgan joyi                                 |
| `fileBytes` | `Array<String>?` | Fayl hajmi.                                          |
| `fileName`  | `String?`        | Fayl nomi.                                           |
| `fileField` | `String?`        | Fayl qiymati.                                        |

### 🔁 Natija

Funksiya `String` (odatda JSON) formatida javob qaytaradi. Uni `Gson` yordamida quyidagicha obyektga aylantirish mumkin:

```kotlin
val gson = Gson()
val parsed = gson.fromJson(jsonString, GetResponse::class.java)
```

Ushbu funksiyadan foydalanish uchun quydagi namunadan foydalansangiz bo'ladi.

### `GET` so‘rov

```kotlin
private fun sendGet() {
    Thread {
        try {

            runOnUiThread {
                resultText = "GET yuborilmoqda..."
            }

            val response = lib.malumotalmashish(
                "https://jsonplaceholder.typicode.com/posts/1", // url
                "GET",// method
                null, // body
                null, // headers
                null, // filePath
                null, // fileBytes
                null, // fileName
                null  // fileField
            )

            runOnUiThread {
                resultText = response
            }

        } catch (e: Exception) {
            Log.e("ApiTest", "Xatolik: ${e.message}", e)

            runOnUiThread {
                resultText = "Xatolik: ${e.message}"
            }
        }
    }.start()
}
```

### `POST` so‘rov

```kotlin
 private fun sendRequest(method: String) {
        Thread {
            try {
                runOnUiThread {
                    resultText = "Ma'lumot yuborilmoqda..."
                }

                val body = """{"title":"zirh test","body":"sample body","userId":1}"""
                val response = lib.malumotalmashish(
                    "https://jsonplaceholder.typicode.com/posts",
                    POST,
                    body,
                    """{"Content-Type":"application/json"}""",
                    null,
                    null,
                    null,
                    null
                )

                runOnUiThread {
                    resultText = "response:\n$response"
                }
            } catch (e: Exception) {
                Log.e("ApiTest", "Xatolik: ${e.message}", e)
                runOnUiThread {
                    resultText = "$method xatolik: ${e.message}"
                }
            }
        }.start()
    }
```

### `PUT` so‘rov

```kotlin
 private fun editRequest(method: String) {
        Thread {
            try {
                runOnUiThread {
                    resultText = "Ma'lumot tahrirlanmoqda..."
                }

                val body = """{"title":"zirh test","body":"sample body","userId":1}"""
                val response = lib.malumotalmashish(
                    "https://jsonplaceholder.typicode.com/posts/1",
                    PUT,
                    body,
                    """{"Content-Type":"application/json"}""",
                    null,
                    null,
                    null,
                    null
                )

                runOnUiThread {
                    resultText = "response:\n$response"
                }
            } catch (e: Exception) {
                Log.e("ApiTest", "Xatolik: ${e.message}", e)
                runOnUiThread {
                    resultText = "$method xatolik: ${e.message}"
                }
            }
        }.start()
    }
```

### `DELETE` so‘rov

```kotlin
 private fun deleteRequest(method: String) {
        Thread {
            try {
                runOnUiThread {
                    resultText = "Ma'lumot o'chirilmoqda..."
                }

                val response = lib.malumotalmashish(
                    "https://jsonplaceholder.typicode.com/posts/1",
                    DELETE,
                    null,
                    null,
                    null,
                    null,
                    null,
                    null
                )

                runOnUiThread {
                    resultText = "response:\n$response"
                }
            } catch (e: Exception) {
                Log.e("ApiTest", "Xatolik: ${e.message}", e)
                runOnUiThread {
                    resultText = "$method xatolik: ${e.message}"
                }
            }
        }.start()
    }

```
