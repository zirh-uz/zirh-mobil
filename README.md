# Zirh-mobil kutubxonasini ishlatish bo'yicha yo'riqnoma
---
[![Android](https://img.shields.io/badge/Android-blue?style=for-the-badge)](#android)
[![Flutter](https://img.shields.io/badge/Flutter-blue?style=for-the-badge)](#Flutter)
[![iOS](https://img.shields.io/badge/iOS-blue?style=for-the-badge)](#iOS)

---
# Android

Zirh-mobil kutubxonasidan foydalanishni boshlash uchun uni o'z Android loyihangizga to'g'ri ulash lozim. Quyidagi bosqichlarni bajaring:


# `aar` fayl orqali kutubxonani ulash
`aar` faylni kutubxonaga qo'shish uchun loyihangizdagi `app` papkasining ichida yangi `libs` nomli papka yarating va unga `.aar` formatidagi
Zirh-mobil kutubxona faylini joylashtiring.
```
app/
 └── libs/
      └── zirhlib-releasex.x.x.aar
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
    implementation(":zirhlib-release-x.x.x.aar")
}
```
> **Eslatma:**
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
    implementation("com.github.Zirh-uz:mobil-lib:x.x.x")
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
│   │   │   └── zirh-mobil-lib-release.aar   ← 📦 JNI kutubxonani o‘z ichiga olgan .aar fayl
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
    implementation(files("libs/zirhlib-releasex.x.x.aar"))
}
```
jitpack orqali app modulining build.gradle.kts faylida kutubxonani ulash
```
dependencies {
    implementation("com.github.Zirh-uz:mobil-lib:x.x.x")
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
Zirh SDkni flutter loyihaning istalgan joyida chaqirib ishlatishimiz uchun bridge.dart faylini yaratib olamiz.
bridge.dart faylida native (C/C++) kutubxona bilan qanday ishlash ko‘rsatiladi. Bu yerda FFI (Foreign Function Interface) yordamida .so fayldan ma’lumotlar olinadi.
```
import 'dart:convert';
import 'dart:ffi' as ffi;
import 'dart:io';
import 'package:ffi/ffi.dart';
import 'package:ffi/ffi.dart' as ffi;

typedef NativeGetInfoFn = ffi.Pointer<ffi.Utf8> Function(ffi.Pointer<ffi.Utf8>);
typedef DartGetInfoFn = ffi.Pointer<ffi.Utf8> Function(ffi.Pointer<ffi.Utf8>);

typedef NativeExchangeFn = ffi.Pointer<ffi.Utf8> Function(
    ffi.Pointer<ffi.Utf8>, // url
    ffi.Pointer<ffi.Utf8>, // method
    ffi.Pointer<ffi.Utf8>, // body
    ffi.Pointer<ffi.Utf8>, // headers
    ffi.Pointer<ffi.Utf8>, // file_path
    ffi.Pointer<ffi.Uint8>, // file_bytes
    ffi.Int32,             // bytes_len
    ffi.Pointer<ffi.Utf8>, // file_name
    ffi.Pointer<ffi.Utf8>, // file_field
    );
typedef DartExchangeFn = ffi.Pointer<ffi.Utf8> Function(
    ffi.Pointer<ffi.Utf8>,
    ffi.Pointer<ffi.Utf8>,
    ffi.Pointer<ffi.Utf8>,
    ffi.Pointer<ffi.Utf8>,
    ffi.Pointer<ffi.Utf8>,
    ffi.Pointer<ffi.Uint8>,
    int,
    ffi.Pointer<ffi.Utf8>,
    ffi.Pointer<ffi.Utf8>,
    );

typedef NativeFreeFn = ffi.Void Function(ffi.Pointer<ffi.Utf8>);
typedef DartFreeFn = void Function(ffi.Pointer<ffi.Utf8>);

class ZirhSDK {
  static final ZirhSDK shared = ZirhSDK._internal();
  factory ZirhSDK() => shared;
  ZirhSDK._internal();

  DartGetInfoFn? _getInfo;
  DartExchangeFn? _exchange;
  DartFreeFn? _freeMemory;
  bool _ready = false;
  void init() {
    try {
      final lib = Platform.isAndroid
          ? ffi.DynamicLibrary.open('libmobil.so')
          : ffi.DynamicLibrary.process();

      _getInfo = lib.lookupFunction<NativeGetInfoFn, DartGetInfoFn>('flutter_malumot_olish');
      _exchange = lib.lookupFunction<NativeExchangeFn, DartExchangeFn>('flutter_malumot_almashish');
      _freeMemory = lib.lookupFunction<NativeFreeFn, DartFreeFn>('flutter_xotirani_tozalash');

      _ready = true;
      print("Zirh SDK tayyor");
    } catch (e) {
      print("SDK init xato: $e");
    }
  }
  String malumotOlish(String path) {
    if (!_ready || _getInfo == null) return "";

    final pathPtr = path.toNativeUtf8();
    ffi.Pointer<ffi.Utf8> resPtr = ffi.nullptr;

    try {
      resPtr = _getInfo!(pathPtr);
      if (resPtr.address == 0) return "";
      return resPtr.toDartString();
    } finally {
      if (resPtr.address != 0) _freeMemory?.call(resPtr);
      malloc.free(pathPtr);
    }
  }

  String malumotAlmashish({
    required String url,
    String method = "POST",
    String? body,
    String? headers,
    String? filePath,
    List<int>? fileBytes,
    String? fileName,
    String? fileField,
  }) {
    if (!_ready || _exchange == null) return "";

    final urlPtr = url.toNativeUtf8();
    final methodPtr = method.toNativeUtf8();
    final bodyPtr = (body ?? "").toNativeUtf8();
    final headersPtr = (headers ?? "").toNativeUtf8();
    final filePathPtr = (filePath ?? "").toNativeUtf8();
    final fileNamePtr = (fileName ?? "").toNativeUtf8();
    final fileFieldPtr = (fileField ?? "").toNativeUtf8();

    ffi.Pointer<ffi.Uint8> bytesPtr = ffi.nullptr;
    int bytesLen = fileBytes?.length ?? 0;
    if (fileBytes != null && fileBytes.isNotEmpty) {
      bytesPtr = malloc.allocate<ffi.Uint8>(bytesLen);
      bytesPtr.asTypedList(bytesLen).setAll(0, fileBytes);
    }

    ffi.Pointer<ffi.Utf8> resPtr = ffi.nullptr;

    try {
      resPtr = _exchange!(
        urlPtr,
        methodPtr,
        bodyPtr,
        headersPtr,
        filePathPtr,
        bytesPtr,
        bytesLen,
        fileNamePtr,
        fileFieldPtr,
      );

      if (resPtr.address == 0) return "";
      return resPtr.toDartString();
    } finally {
      if (resPtr.address != 0) _freeMemory?.call(resPtr);

      malloc.free(urlPtr);
      malloc.free(methodPtr);
      malloc.free(bodyPtr);
      malloc.free(headersPtr);
      malloc.free(filePathPtr);
      malloc.free(fileNamePtr);
      malloc.free(fileFieldPtr);
      if (bytesPtr.address != 0) malloc.free(bytesPtr);
    }
  }
}
```
## Minimal kod misolida ko'rishimiz mumkin
```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'bridge.dart'; // Sizning ZirhSDK wrapper faylingiz

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  ZirhSDK.shared.init(); // SDKni ishga tushurish
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Dashboard(),
    );
  }
}

class Dashboard extends StatefulWidget {
  const Dashboard({super.key});

  @override
  State<Dashboard> createState() => _DashboardState();
}

class _DashboardState extends State<Dashboard> {
  final zirh = ZirhSDK.shared;

  Map<String, dynamic> ui = {};
  List<String> domains = [];
  Map<String, dynamic> api = {};
  String responseText = "";
  bool loading = false;

  @override
  void initState() {
    super.initState();
    _load();
  }

  // SDK dan JSON konfiguratsiyani o‘qish
  void _load() {
    final colors = _parseJson(zirh.malumotOlish("ui.colors"));
    final labels = _parseJson(zirh.malumotOlish("ui.labels"));
    final buttons = _parseJson(zirh.malumotOlish("ui.buttons"));

    setState(() {
      ui = {
        "colors": colors,
        "labels": labels,
        "buttons": buttons,
      };

      final domainData = _parseJson(zirh.malumotOlish("domainlar"));
      domains = (domainData is List) ? domainData.map((e) => e.toString()).toList() : [];

      api = _parseJson(zirh.malumotOlish("api")) ?? {};
      responseText = "";
    });
  }

  dynamic _parseJson(String raw) {
    try {
      return jsonDecode(raw);
    } catch (_) {
      return null;
    }
  }

  Color _c(String? hex, Color fallback) {
    if (hex == null || !hex.startsWith("#")) return fallback;
    return Color(int.parse(hex.substring(1), radix: 16) + 0xFF000000);
  }

  // API chaqiruv
  void _callApi(String method, String path) async {
    final baseUrl = api['base_url'] ?? "";

    setState(() {
      loading = true;
      responseText = "";
    });

    try {
      final res = zirh.malumotAlmashish(
        url: baseUrl + path,
        method: method,
        body: (method == "POST" || method == "PUT") ? jsonEncode({"title": "test", "body": "demo", "userId": 1}) : null,
      );

      setState(() {
        responseText = res;
      });
    } catch (e) {
      setState(() {
        responseText = "Xatolik: $e";
      });
    } finally {
      setState(() {
        loading = false;
      });
    }
  }

  String _formatJson(String raw) {
    try {
      final obj = jsonDecode(raw);
      return const JsonEncoder.withIndent('  ').convert(obj);
    } catch (_) {
      return raw;
    }
  }

  @override
  Widget build(BuildContext context) {
    final primary = _c(ui['colors']?['primary'], Colors.blue);
    final bg = _c(ui['colors']?['background'], Colors.black);
    final card = _c(ui['colors']?['card'], Colors.white10);
    final title = ui['labels']?['dashboard_title'] ?? "Dashboard";
    final endpoints = api['endpoints'] ?? {};

    return Scaffold(
      backgroundColor: bg,
      appBar: AppBar(
        backgroundColor: primary,
        title: Text(title),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Domainlar
            Align(
              alignment: Alignment.centerLeft,
              child: Text(
                "Domainlar",
                style: TextStyle(
                  color: primary,
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            const SizedBox(height: 10),
            SizedBox(
              height: 120,
              child: domains.isEmpty
                  ? const Center(child: Text("Domain yo‘q"))
                  : ListView.builder(
                scrollDirection: Axis.horizontal,
                itemCount: domains.length,
                itemBuilder: (_, i) {
                  return Container(
                    width: 250,
                    margin: const EdgeInsets.only(right: 10),
                    decoration: BoxDecoration(
                      color: card,
                      borderRadius: BorderRadius.circular(12),
                    ),
                    child: ListTile(
                      leading: Icon(Icons.link, color: primary),
                      title: Text(domains[i]),
                    ),
                  );
                },
              ),
            ),
            const SizedBox(height: 20),

            // Endpointlar
            Align(
              alignment: Alignment.centerLeft,
              child: Text(
                "API Endpointlar",
                style: TextStyle(
                  color: primary,
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            const SizedBox(height: 10),
            Expanded(
              child: ListView(
                children: endpoints.keys.map<Widget>((key) {
                  final item = endpoints[key];
                  return Card(
                    color: card,
                    child: ListTile(
                      leading: Icon(Icons.api, color: primary),
                      title: Text(key),
                      subtitle: Text("${item['method']}  ${item['path']}"),
                      onTap: () {
                        _callApi(item['method'], item['path']);
                      },
                    ),
                  );
                }).toList(),
              ),
            ),

            const SizedBox(height: 10),

            // Response preview
            Align(
              alignment: Alignment.centerLeft,
              child: Text(
                "Natija",
                style: TextStyle(
                  color: primary,
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            const SizedBox(height: 10),
            Container(
              width: double.infinity,
              height: 150,
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: card,
                borderRadius: BorderRadius.circular(12),
              ),
              child: loading
                  ? const Center(child: CircularProgressIndicator())
                  : SingleChildScrollView(
                child: Text(
                  responseText.isEmpty ? "Natija yo‘q" : _formatJson(responseText),
                  style: const TextStyle(fontFamily: 'monospace'),
                ),
              ),
            ),

            const SizedBox(height: 10),
            SizedBox(
              width: double.infinity,
              height: 50,
              child: ElevatedButton(
                style: ElevatedButton.styleFrom(
                  backgroundColor: primary,
                ),
                onPressed: _load,
                child: Text(ui['buttons']?['refresh'] ?? "Yangilash"),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```
Bizda data.json quyidagicha bo'lgan holatda
```dart
{
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
## FFI orqali bog‘lanish 
```
import 'dart:ffi' as ffi;
import 'bridge.dart'; // Zirh uchun yaratgan bridge.dart

void main() {
  final zirh = ZirhService();
  zirh.init();
}
```
API bilan ishlash uchun quyidagi kod namunasidan foydalansak bo'ladi.
```
// GET request
String getResp = zirh.exchange(
  url: "https://api.example.com/posts",
  method: "GET",
);
print("GET javob: $getResp");

// POST request
String postResp = zirh.exchange(
  url: "https://api.example.com/posts",
  method: "POST",
  body: {"title": "Test", "body": "Hello", "userId": 1},
);
print("POST javob: $postResp");

// PUT request
String putResp = zirh.exchange(
  url: "https://api.example.com/posts/1",
  method: "PUT",
  body: {"title": "Updated", "body": "Yangilandi"},
);
print("PUT javob: $putResp");

// DELETE request
String delResp = zirh.exchange(
  url: "https://api.example.com/posts/1",
  method: "DELETE",
);
print("DELETE javob: $delResp");
```

---
### 🔐 Ma'lumotlarni Shifrlash
[![Shifrlash](https://img.shields.io/badge/Ma'lumotlarni%20Shifrlash-blue?style=for-the-badge)](#-ma-lumotlarni-shifrlash)

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
    "imzo": "c4fbec9f103e46f13864c208ea93a26a48cd2092428c5dae3b7529703b74f7c9",
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
            "get_post": {"path": "/posts/1", "method": "GET"},
            "create_post": {"path": "/posts", "method": "POST"},
            "update_post": {"path": "/posts/1", "method": "PUT"},
            "delete_post": {"path": "/posts/1", "method": "DELETE"}
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
>SSL Pinning bu ilova va server o'rtasidagi so'rovlarga uchinchi shaxs aralashishidan himoyalaydi.


Yuqoridagi data.jsondan ilovada quyidagicha UI komponentalarini chaqirib ishlatishimiz mumkin bo'ladi.

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
>    Ilovaning ichki fayllari (token, ma’lumotlar bazasi) o'g'irlanishi mumkin.
>    Ilova funksiyalari o'zgartirilishi yoki "buzilishi" mumkin (code injection).
>    Trafik (token/parollar) kuzatilib, tahlil qilinadi.



## 🛡️ Ilovani play marketdan o'rnatilganligini tekshirish.

Ushbu funksiya ilova Play Market orqali o‘rnatilganligini aniqlash uchun ishlatiladi. Agar ilova boshqa manbadan (masalan .apk orqali qo‘lda) o‘rnatilgan bo‘lsa, bu xavfsizlikka tahdid solishi mumkin. Ushbu funksiya ishlashi uchun `data.json` fayldagi `playmarket`:`true` qilish yetarli.

---
## ✍️ Ilova imzosini tekshirish

Ilova imzosini tekshirish — ilovaning haqiqiy ishlab chiquvchi tomonidan chiqarilganini va ilova o‘zgartirilmaganini aniqlash uchun ishlatiladi. Android tizimida har bir APK fayl maxsus raqamli sertifikat bilan imzolanadi. Agar kimdir ilovani decompile qilib, kodini o‘zgartirib yoki zararli kod qo‘shib qayta yig‘sa, u holda yangi APK boshqa kalit bilan imzolanadi va asl imzo bilan mos kelmaydi. Ilova ichida imzo tekshiruvi bo‘lsa, u bunday o‘zgartirilgan APK ni aniqlab, ishlashni to‘xtatishi yoki ba’zi funksiyalarni bloklashi mumkin. Qisqacha aytganda, ilova imzosini tekshirish ilovaning haqiqiyligini tasdiqlash, modifikatsiya qilingan yoki zararli o‘zgartirilgan APK fayllarni aniqlash va foydalanuvchi hamda server xavfsizligini ta’minlash uchun kerak.

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

| Parametr      | Turi             | Tavsif |
|---------------|------------------|--------|
| `url` | `String`            | Url manzil masalan(https://example.com/api/v1/users. |
| `METHOD`   | `String`            | HTTP metod: `GET`, `POST`, `PUT`, `DELETE`. |
| `body`          | `String?`        | Body ixtiyoriy. |
| `headers`      | `String`         | Sarlavhalar (headers), ixtiyoriy. |
| `filePath`      | `String`         | Fayl joylashgan joyi |
| `fileBytes`     | `Array<String>?` | Fayl hajmi. |
| `fileName`        | `String?`        | Fayl nomi. |
| `fileField`        | `String?`        | Fayl qiymati. |

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

---
# iOS

Zirh-mobil kutubxonasidan foydalanishni boshlash uchun uni o'z Xcode loyihangizga to'g'ri ulash lozim. Quyidagi bosqichlarni bajaring:

```
loyihanomi/
├── 📦 ZirhIosSDK
├── 📂 loyihanomi
├── 📂 loyihanomiTests
├── 📂 loyihanomiUITests
└── 📂 Products
```

### 📦 SDK Integration

ZirhIosSDK.xcframework muvaffaqiyatli integratsiya qilinganini tekshirish uchun quyidagi bosqichlarni bajaring:

1. Xcode-da loyiha sozlamalariga kiring (Project Navigator -> testzirh1).

2. General tabiga o'ting.

3. Frameworks, Libraries, and Embedded Content bo'limiga tushing.

4. Ro'yxatda ZirhIosSDK.xcframework borligiga ishonch hosil qiling.

### ⚙️ Framework sozlash

| Target | Framework | Embed Setting |
| :--- | :--- | :--- |
| **loyihanomi** | 📦 `ZirhIosSDK.xcframework` | **Embed & Sign** |

### ⚙️ Bridging Header sozlash

Swift loyihangizda C++ yoki Objective-C kodlaridan foydalanish uchun Bridging Header yaratishingiz va sozlashingiz kerak:

1. Fayl strukturasi (Project Skeleton):
Loyiha ildizida (root) quyidagi fayl mavjudligini tekshiring:
```
loyihanomi/
├── ...
├── 📄 Zirh-Bridging-Header.h  <-- Ushbu faylni yarating
└── 📂 loyihanomi/
```
2. Header tarkibi:
Zirh-Bridging-Header.h fayli ichiga quyidagi kodni kiriting:

```dart
#ifndef Zirh_Bridging_Header_h
#define Zirh_Bridging_Header_h

#include "zirh_ios.h"

#endif /* Zirh_Bridging_Header_h */
```
3. Xcode sozlamalari (Build Settings):
Header faylini loyihaga tanitish uchun quyidagi sxema bo'yicha yo'lni ko'rsating:
```
Xcode Project 
└── 🛠️ Build Settings
    └── 🔍 Search: "Objective-C Bridging Header"
        └── 🟢 Value: loyihanomi/Zirh-Bridging-Header.h
```
| Setting Key | Value |
| :--- | :--- |
| **Objective-C Bridging Header** | `loyihanomi/Zirh-Bridging-Header.h` |

### 🛡️ Swift Wrapper yaratish

Native funksiyalarni Swift muhitida oson ishlatish uchun Zirh.swift (wrapper) faylini yarating. Bu klass SDK funksiyalari va Swift kodingiz o'rtasida ko'prik vazifasini o'taydi.

1. Loyiha arxitekturasi va wrapper faylining joylashuvi quyidagicha:
```
loyihanomi/
├── 📦 ZirhIosSDK.xcframework
├── 📄 Zirh-Bridging-Header.h
├── 📂 loyihanomi/
│   ├── 🖼️ Assets
│   ├── 📱 ContentView.swift
│   ├── 🚀 loyihanomiApp.swift
│   └── 🛡️ ZirhSDK.swift          <-- Swift Wrapper File
├── 📂 loyihanomiTests/
│   └── 📄 loyihanomiTests.swift
├── 📂 loyihanomiUITests/
│   ├── 📄 loyihanomiUITests.swift
│   └── 📄 loyihanomiUITestsLaunchTests.swift
├── 📁 Products/
├── 🔑 kalit.enc
└── 🔒 data.enc
```

2. Wrapper kodi:
```
import Foundation
class ZirhSDK {
    static let shared = ZirhSDK()
    private init() {}
    func boshlash(keyPath: String, dataPath: String) {
        zirh_ios_boshlash(keyPath, dataPath)
    }
    func malumotOlish(path: String) -> String? {
        guard let ptr = ios_malumot_olish(path) else {
            return nil
        }
        let result = String(cString: ptr)
        ios_xotirani_tozalash(ptr)
        return result
    }
    func malumotAlmashish(
        url: String,
        method: String = "POST",
        body: String? = nil,
        headers: String? = nil,
        filePath: String? = nil,
        fileBytes: [UInt8]? = nil,
        fileName: String? = nil,
        fileField: String? = nil
    ) -> String? {
        let bytesCount = Int32(fileBytes?.count ?? 0)
        let resultPtr = ios_malumot_almashish(
            url,
            method,
            body,
            headers,
            filePath,
            fileBytes,
            bytesCount,
            fileName,
            fileField
        )
        guard let ptr = resultPtr else {
            return nil
        }
        let response = String(cString: ptr)
        ios_xotirani_tozalash(ptr)
        return response
    }
}
```
### 🚀 SDKni ishga tushirish

SDK to'g'ri ishlashi uchun u ilova yuklanayotgan vaqtda (init) ishga tushirilishi kerak. Buning uchun testzirh1App.swift faylida quyidagi sozlamani bajaring:

1. Ilova ishga tushish nuqtasi:

@main strukturasi ichida init() funksiyasidan foydalanib, .enc fayllari orqali SDK-ni konfiguratsiya qilamiz.

```
import SwiftUI

@main
struct testzirh1App: App {
    
    init() {
        // Resurs fayllari yo'lini aniqlash
        if let keyPath = Bundle.main.path(forResource: "kalit", ofType: "enc"),
           let dataPath = Bundle.main.path(forResource: "data", ofType: "enc") {
            
            // SDK-ni resurslar bilan boshlash
            ZirhSDK.shared.boshlash(keyPath: keyPath, dataPath: dataPath)
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```
### ⚠️ Muhim eslatma:

kalit.enc va data.enc fayllari Xcode loyihasiga qo'shilganiga va Copy Bundle Resources (Build Phases) bo'limida mavjudligiga ishonch hosil qiling. Aks holda Bundle.main.path qiymati nil qaytaradi.

kalit.enc va data.enc yuqorida berilgan holatda yaratiladi. [Ma'lumotlarni Shifrlash](#maʼlumotlarni-shifrlash) qismida batafsil keltirilgan.

### Namunaviy kod

data.jsonni quyidagicha yaratib oldik.
```
{
  "jailbreak":true,
  "app_config": {
    "app_name": "Zirh Security Monitor",
    "theme_color": "#1A73E8",
    "status_colors": {
      "safe": "#28A745",
      "warning": "#FFC107",
      "error": "#DC3545"
    }
  },
  "ui_theme": {
    "primary_color": "#1A73E8",
    "success_color": "#28A745",
    "danger_color": "#DC3545",
    "background_style": "dark_gradient",
    "labels": {
      "title": "Zirh Security Shield",
      "check_button": "Xavfsizlikni Tekshirish",
      "status_secure": "Tizim xavfsiz",
      "status_compromised": "Xavf aniqlandi!"
    }
  },
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
### ⚠️ Muhim eslatma:

jailbreakni true qilish orqali biz ilovani jailbreaklangan qurilmalarda ishlamasligini taminlashimiz mumkin.

ContentView.swift fayliga quyidagi namunaviy kod misolida ishlatib ko'rishimiz mumkin:
```
import SwiftUI

struct ContentView: View {
    
    // --- SDK dan olinadigan holatlar ---
    @State private var isSecure: Bool = true
    @State private var resultString: String = "Javob kutilmoqda..."
    @State private var isLoading: Bool = false
    
    @State private var primaryColor: Color!
    @State private var successColor: Color!
    @State private var dangerColor: Color!
    @State private var labels: [String: String] = [:]
    
    @State private var baseURL: String!
    
    var body: some View {
        ZStack {
            LinearGradient(gradient: Gradient(colors: [Color.black.opacity(0.9), Color.blue.opacity(0.2)]),
                           startPoint: .top, endPoint: .bottom)
                .ignoresSafeArea()
            
            VStack(spacing: 20) {
                if primaryColor != nil && baseURL != nil {
                    headerView
                    statusCard
                    Divider().background(Color.white.opacity(0.2))
                    Text("Tarmoq Operatsiyalari (CRUD)")
                        .font(.system(size: 14, weight: .semibold, design: .monospaced))
                        .foregroundColor(.gray)
                        .frame(maxWidth: .infinity, alignment: .leading)
                        .padding(.horizontal)
                    actionsGrid
                    consoleView
                    Spacer()
                } else {
                    Text("SDK dan ma’lumot olinmoqda...")
                        .foregroundColor(.white)
                        .padding()
                }
            }
            .padding(.top, 20)
        }
        .onAppear {
            // --- SDK dan ranglar ---
            guard
                let primaryHex = ZirhSDK.shared.malumotOlish(path: "ui_theme.primary_color"),
                let successHex = ZirhSDK.shared.malumotOlish(path: "ui_theme.success_color"),
                let dangerHex = ZirhSDK.shared.malumotOlish(path: "ui_theme.danger_color"),
                let titleLabel = ZirhSDK.shared.malumotOlish(path: "ui_theme.labels.title"),
                let checkButton = ZirhSDK.shared.malumotOlish(path: "ui_theme.labels.check_button"),
                let statusSecure = ZirhSDK.shared.malumotOlish(path: "ui_theme.labels.status_secure"),
                let statusCompromised = ZirhSDK.shared.malumotOlish(path: "ui_theme.labels.status_compromised"),
                let apiBase = ZirhSDK.shared.malumotOlish(path: "api.base_url")
            else {
                fatalError("SDK dan barcha kerakli ma’lumotlar olinmadi!")
            }
            
            primaryColor = Color(hex: primaryHex)
            successColor = Color(hex: successHex)
            dangerColor  = Color(hex: dangerHex)
            
            labels = [
                "title": titleLabel,
                "check_button": checkButton,
                "status_secure": statusSecure,
                "status_compromised": statusCompromised
            ]
            
            baseURL = apiBase
        }
    }
    
    private var headerView: some View {
        HStack {
            Image(systemName: "shield.rhombus.fill")
                .font(.system(size: 30))
                .foregroundColor(primaryColor)
            
            Text(labels["title"]!)
                .font(.system(size: 22, weight: .bold, design: .rounded))
                .foregroundColor(.white)
            
            Spacer()
            
            Button(action: {}) {
                Image(systemName: "arrow.clockwise.circle")
                    .foregroundColor(.gray)
            }
        }
        .padding(.horizontal)
    }
    
    private var statusCard: some View {
        HStack(spacing: 15) {
            ZStack {
                Circle()
                    .stroke(isSecure ? successColor.opacity(0.3) : dangerColor.opacity(0.3), lineWidth: 4)
                    .frame(width: 50, height: 50)
                
                Image(systemName: isSecure ? "checkmark.shield.fill" : "exclamationmark.shield.fill")
                    .foregroundColor(isSecure ? successColor : dangerColor)
                    .font(.system(size: 24))
            }
            
            VStack(alignment: .leading) {
                Text(isSecure ? labels["status_secure"]! : labels["status_compromised"]!)
                    .font(.headline)
                    .foregroundColor(.white)
                Text("SSL Pinning faol | Hash tekshiruvi OK")
                    .font(.caption)
                    .foregroundColor(.gray)
            }
            
            Spacer()
        }
        .padding()
        .background(Color.white.opacity(0.05))
        .cornerRadius(15)
        .padding(.horizontal)
    }
    
    private var actionsGrid: some View {
        LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 15) {
            ActionButton(title: "GET Post", icon: "doc.text.magnifyingglass", color: primaryColor) {
                runSDKAction(key: "get_post")
            }
            
            ActionButton(title: "POST Create", icon: "plus.square.fill", color: successColor) {
                runSDKAction(key: "create_post")
            }
            
            ActionButton(title: "PUT Update", icon: "pencil.circle.fill", color: .orange) {
                runSDKAction(key: "update_post")
            }
            
            ActionButton(title: "DELETE Post", icon: "trash.fill", color: dangerColor) {
                runSDKAction(key: "delete_post")
            }
        }
        .padding(.horizontal)
    }
    
    private var consoleView: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("TERMINAL OUTPUT")
                .font(.system(size: 10, weight: .bold))
                .foregroundColor(.blue)
            
            ScrollView {
                if isLoading {
                    ProgressView()
                        .progressViewStyle(CircularProgressViewStyle(tint: .white))
                        .padding()
                } else {
                    Text(resultString)
                        .font(.system(size: 12, design: .monospaced))
                        .foregroundColor(.green.opacity(0.9))
                        .frame(maxWidth: .infinity, alignment: .leading)
                        .padding(10)
                }
            }
            .frame(maxWidth: .infinity, maxHeight: 250)
            .background(Color.black.opacity(0.6))
            .cornerRadius(10)
            .overlay(RoundedRectangle(cornerRadius: 10).stroke(Color.white.opacity(0.1)))
        }
        .padding()
    }
    
    func runSDKAction(key: String) {
        isLoading = true
        
        let path: String
        let method: String
        
        switch key {
        case "get_post": path = "/posts/1"; method = "GET"
        case "create_post": path = "/posts"; method = "POST"
        case "update_post": path = "/posts/1"; method = "PUT"
        case "delete_post": path = "/posts/1"; method = "DELETE"
        default: fatalError("Noto'g'ri endpoint kalit!")
        }
        
        DispatchQueue.global(qos: .userInitiated).async {
            let body = (method == "POST" || method == "PUT") ? "{\"title\": \"Zirh Test\", \"body\": \"Swift\", \"userId\": 1}" : nil
            
            guard let response = ZirhSDK.shared.malumotAlmashish(url: baseURL + path, method: method, body: body) else {
                DispatchQueue.main.async {
                    self.resultString = "Xato: SDK dan javob olinmadi"
                    self.isLoading = false
                }
                return
            }
            DispatchQueue.main.async {
                self.resultString = response
                self.isLoading = false
            }
        }
    }
}

// MARK: - ActionButton Helper
struct ActionButton: View {
    let title: String
    let icon: String
    let color: Color
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            VStack(spacing: 8) {
                Image(systemName: icon)
                    .font(.title2)
                Text(title)
                    .font(.system(size: 12, weight: .bold))
            }
            .frame(maxWidth: .infinity)
            .padding(.vertical, 16)
            .background(color.opacity(0.15))
            .foregroundColor(color)
            .cornerRadius(12)
            .overlay(RoundedRectangle(cornerRadius: 12).stroke(color.opacity(0.4), lineWidth: 1))
        }
    }
}

// MARK: - Hex Color Extension
extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let r, g, b: UInt64
        switch hex.count {
        case 6: (r, g, b) = (int >> 16, int >> 8 & 0xFF, int & 0xFF)
        default: fatalError("Hex rang noto‘g‘ri formatda: \(hex)")
        }
        self.init(.sRGB,
                  red: Double(r) / 255,
                  green: Double(g) / 255,
                  blue: Double(b) / 255,
                  opacity: 1)
    }
}
```
Kodimizni qisqacha tavsifi

data.encdan ma'lumotlarni olish uchun o'zgaruvchilar yaratib oldik.
```
@State private var primaryColor: Color!
@State private var labels: [String: String] = [:]
@State private var baseURL: String!
@State private var successColor: Color!
@State private var dangerColor: Color!
```
Ma'lumot olish uchun quyidagicha foydalanamiz.
```
primaryColor = Color(hex: ZirhSDK.shared.malumotOlish(path: "ui_theme.primary_color")!)
labels["title"] = ZirhSDK.shared.malumotOlish(path: "ui_theme.labels.title")
baseURL = ZirhSDK.shared.malumotOlish(path: "api.base_url")
```
Ma'lumot olish uchun esa quyidagicha koddan foydalanamiz.
(GET, POST, PUT, DELETE ...) so‘rovlarini amalga oshirish mumkin.
```
func runSDKAction(key: String) {
    let path: String
    let method: String
    
    switch key {
    case "get_post": path = "/posts/1"; method = "GET"
    case "create_post": path = "/posts"; method = "POST"
    case "update_post": path = "/posts/1"; method = "PUT"
    case "delete_post": path = "/posts/1"; method = "DELETE"
    default: fatalError("Noto'g'ri endpoint kalit!")
    }
    
    let body = (method == "POST" || method == "PUT") ? "{\"title\": \"Zirh Test\", \"body\": \"Swift\", \"userId\": 1}" : nil
    
    let response = ZirhSDK.shared.malumotAlmashish(
        url: baseURL + path,
        method: method,
        body: body
    )
}
```
UI bilan integratsiyada quyidagicha namunaviy kod kabi foydalanishimiz mumkin.
```
Text(labels["title"]!)
    .foregroundColor(.white)

Circle()
    .stroke(isSecure ? successColor.opacity(0.3) : dangerColor.opacity(0.3), lineWidth: 4)
```
Hex ranglarni ishlatish
data.jsonda odatda ranglarni hex string ("#2563EB") shaklida bergandik. Uni Color ga o‘tkazish uchun extension yaratdik:
```
extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: .alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let r = Double((int >> 16) & 0xFF) / 255
        let g = Double((int >> 8) & 0xFF) / 255
        let b = Double(int & 0xFF) / 255
        self.init(.sRGB, red: r, green: g, blue: b, opacity: 1)
    }
}
```
Shu bilan, SDK dan olingan hex ranglarni SwiftUI komponentlarida bevosita ishlatish mumkin.

---

---
### Flutter iOS
Flutter orqali iOS ilovalarini yaratishda quyidagicha ZirhIosSDK.xcframeworkdan foydalanishimiz mumkin.

# Flutterda loyihani tayyorlab olish
Flutter loyihanggiz joylashgan joydan terminal orqali podsni o'rnatib olishinggiz kerak bo'adi.
```
flutter clean
flutter pub get
cd ios
pod install
cd ..
```
# Xcode sozlash

Terminal orqali yoki Xcode orqali yaratgan flutter loyihanggizni ios qismiga kirasiz.
Terminal orqali esa quyidagicha bo'ladi:
```
open ios/Runner.xcworkspace
```
ZirhIosSDK.xcframeworkni ulash yuqorida berilgan iOS qo'llanmasidagi bilan bir xil bo'ladi.
1. ZirhIosSDK.xcframework ni Xcode ga qo‘shing.
2. Target → General → Frameworks ```Embed & Sign``` tanlangan bo‘lishi kerak

# Bridging Header sozlash
Runner papkasiga yangi fayl yaratamiz.
```Zirh-Bridging-Header.h```
Yaratgan yangi faylimiz quyidagicha kod kiritamiz.

```
#ifndef Zirh_Bridging_Header_h
#define Zirh_Bridging_Header_h
#import "GeneratedPluginRegistrant.h"

#include "zirh_ios.h"

#endif
```
Keyin esa Xcode → Build Settings qismidan 
Objective-C Bridging Header = Runner/Zirh-Bridging-Header.h qiymatni berib qoyishimiz kerak bo'ladi.

# Ma'lumotlarni tayyorlash
kalit.enc va data.enc yuqorida berilgan holatda yaratiladi. [Ma'lumotlarni Shifrlash](#maʼlumotlarni-shifrlash) qismida batafsil keltirilgan.

### Namunaviy kod

data.jsonni quyidagicha yaratib oldik.
```
{
  "jailbreak":true,
  "app_config": {
    "app_name": "Zirh Security Monitor",
    "theme_color": "#1A73E8",
    "status_colors": {
      "safe": "#28A745",
      "warning": "#FFC107",
      "error": "#DC3545"
    }
  },
  "ui_theme": {
    "primary_color": "#1A73E8",
    "success_color": "#28A745",
    "danger_color": "#DC3545",
    "background_style": "dark_gradient",
    "labels": {
      "title": "Zirh Security Shield",
      "check_button": "Xavfsizlikni Tekshirish",
      "status_secure": "Tizim xavfsiz",
      "status_compromised": "Xavf aniqlandi!"
    }
  },
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
### ⚠️ Muhim eslatma:

jailbreakni true qilish orqali biz ilovani jailbreaklangan qurilmalarda ishlamasligini taminlashimiz mumkin.
# Swift Wrapper (ZirhSDK.swift) yaratish
ZirhSDK.swift fayl yaratamiz Runner papkamizga va quyidagi kodni kiritamiz.
```
//
//  ZirhSDK.swift
//  Runner
//
//  Created by MACBOOK PRO on 03/04/26.
//
import Foundation

class ZirhSDK {
    static let shared = ZirhSDK()
    
    private init() {}


    func boshlash(keyPath: String, dataPath: String) {
     
        zirh_ios_boshlash(keyPath, dataPath)
    }


    func malumotOlish(path: String) -> String? {
        
        guard let ptr = ios_malumot_olish(path) else {
            return nil
        }
        
        let result = String(cString: ptr)
        

        ios_xotirani_tozalash(ptr)
        
        return result
    }


    func malumotAlmashish(
        url: String,
        method: String = "POST",
        body: String? = nil,
        headers: String? = nil,
        filePath: String? = nil,
        fileBytes: [UInt8]? = nil,
        fileName: String? = nil,
        fileField: String? = nil
    ) -> String? {
        

        let bytesCount = Int32(fileBytes?.count ?? 0)
        

        let resultPtr = ios_malumot_almashish(
            url,
            method,
            body,
            headers,
            filePath,
            fileBytes,
            bytesCount,
            fileName,
            fileField
        )
        
        guard let ptr = resultPtr else {
            return nil
        }
        
        let response = String(cString: ptr)
        
        ios_xotirani_tozalash(ptr)
        
        return response
    }
}

```
# SDK init qilish

AppDeleget da sdkni ishga tushirish qismini yozib olamiz.
```
if let keyPath = Bundle.main.path(forResource: "kalit", ofType: "enc"),
           let dataPath = Bundle.main.path(forResource: "data", ofType: "enc") {

            ZirhSDK.shared.boshlash(keyPath: keyPath, dataPath: dataPath)
        }
```
# Flutter ↔ iOS MethodChannel
Quyidagi namuna kabi yozishimiz mumkin iOS va Flutter uchun iOS qismida methodchannel
```
let channel = FlutterMethodChannel(name: "zirh_sdk", binaryMessenger: controller.binaryMessenger)

channel.setMethodCallHandler { call, result in
    switch call.method {
    case "malumotOlish":
        let args = call.arguments as! [String: Any]
        let path = args["path"] as! String
        result(ZirhSDK.shared.malumotOlish(path: path))
    default:
        result(FlutterMethodNotImplemented)
    }
}
```

Quyida AppDelegete to'liq qilingan namunaviy kodi berilgan:

```
import UIKit
import Flutter

@main
@objc class AppDelegate: FlutterAppDelegate {
    
    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        
        if let keyPath = Bundle.main.path(forResource: "kalit", ofType: "enc"),
           let dataPath = Bundle.main.path(forResource: "data", ofType: "enc") {

            ZirhSDK.shared.boshlash(keyPath: keyPath, dataPath: dataPath)
        }
        
        let controller: FlutterViewController = window?.rootViewController as! FlutterViewController
        let channel = FlutterMethodChannel(name: "zirh_sdk", binaryMessenger: controller.binaryMessenger)
        
        // --- Malumot olish ---
        channel.setMethodCallHandler { call, result in
            switch call.method {
            case "malumotOlish":
                guard let args = call.arguments as? [String: Any],
                      let path = args["path"] as? String else {
                    result(FlutterError(code: "INVALID_ARGS", message: "Path missing", details: nil))
                    return
                }
                let jsonString = ZirhSDK.shared.malumotOlish(path: path)
                result(jsonString)
            
            case "malumotAlmashish":
                guard let args = call.arguments as? [String: Any],
                      let url = args["url"] as? String,
                      let method = args["method"] as? String else {
                    result(FlutterError(code: "INVALID_ARGS", message: "URL/method missing", details: nil))
                    return
                }
                
                let body = args["body"] as? String
                let headers = args["headers"] as? String
                let filePath = args["filePath"] as? String
                let fileBytes = args["fileBytes"] as? FlutterStandardTypedData
                let fileName = args["fileName"] as? String
                let fileField = args["fileField"] as? String
                
                let bytes: [UInt8]? = fileBytes?.data.toBytes()
                
                let response = ZirhSDK.shared.malumotAlmashish(
                    url: url,
                    method: method,
                    body: body,
                    headers: headers,
                    filePath: filePath,
                    fileBytes: bytes,
                    fileName: fileName,
                    fileField: fileField
                )
                result(response)
                
            default:
                result(FlutterMethodNotImplemented)
            }
        }
        
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
}

extension Data {
    func toBytes() -> [UInt8] {
        return [UInt8](self)
    }
}
```
# Flutter tomonda ham methodchannel yaratish
Flutterda ham methodchannel yaratib olamiz. bridge.dart faylini yaratib uning ichiga quyidagicha qilib methodchannel uchun kod kiritamiz.

```
class ZirhService {
  static const _platform = MethodChannel('zirh_sdk');

  Future<String?> malumotOlish(String path) async {
    return await _platform.invokeMethod('malumotOlish', {
      "path": path,
    });
  }
}
```

Namunaviy kod quyidagicha keltirilgan:
```
import 'package:flutter/services.dart';

class ZirhService {
  static const _platform = MethodChannel('zirh_sdk');

  // 📥 data.enc dan ma'lumot olish
  Future<String?> malumotOlish(String path) async {
    try {
      final res = await _platform.invokeMethod<String>(
        'malumotOlish',
        {"path": path},
      );
      return res;
    } catch (e) {
      print("Xato: $e");
      return null;
    }
  }

  // 🌐 API bilan ishlash
  Future<String?> malumotAlmashish({
    required String url,
    required String method,
    String? body,
    String? headers,
    String? filePath,
    List<int>? fileBytes,
    String? fileName,
    String? fileField,
  }) async {
    try {
      final res = await _platform.invokeMethod<String>(
        'malumotAlmashish',
        {
          "url": url,
          "method": method,
          "body": body,
          "headers": headers,
          "filePath": filePath,
          "fileBytes": fileBytes,
          "fileName": fileName,
          "fileField": fileField,
        },
      );
      return res;
    } catch (e) {
      print("Xato: $e");
      return null;
    }
  }
}
```
# Ma'lumot olish va almashish
data.encdan sdk orqali malumotOlish va malumotAlmashishni quyidagicha qilib bajaraishimiz mumkin.
```
final zirh = ZirhService();

String? title = await zirh.malumotOlish("ui_theme.labels.title");
```
```
await zirh.malumotAlmashish(
  url: "https://jsonplaceholder.typicode.com/posts",
  method: "POST",
  body: jsonEncode({
    "title": "test",
    "body": "demo",
    "userId": 1
  }),
  headers: jsonEncode({
    "Content-Type": "application/json"
  }),
);
```

Namunaviy kod 
```
import 'dart:convert';
import 'package:flutter/material.dart';
import 'bridge.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: Home());
  }
}

class Home extends StatefulWidget {
  const Home({super.key});

  @override
  State<Home> createState() => _HomeState();
}

class _HomeState extends State<Home> {
  final zirh = ZirhService();

  String title = "Loading...";
  String response = "";

  @override
  void initState() {
    super.initState();
    loadData();
  }

  // 📥 data.enc dan olish
  void loadData() async {
    final res = await zirh.malumotOlish("ui_theme.labels.title");

    setState(() {
      title = res ?? "Topilmadi";
    });
  }

  // 🌐 API chaqirish
  void callApi() async {
    final baseUrl = await zirh.malumotOlish("api.base_url");

    final res = await zirh.malumotAlmashish(
      url: "$baseUrl/posts/1",
      method: "GET",
    );
    print(res);

    setState(() {
      response = res ?? "Xatolik";
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(title)),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            ElevatedButton(
              onPressed: callApi,
              child: const Text("API chaqirish"),
            ),
            const SizedBox(height: 20),
            Expanded(
              child: SingleChildScrollView(
                child: Text(
                  response.isEmpty
                      ? "Natija yo‘q"
                      : const JsonEncoder.withIndent('  ')
                          .convert(jsonDecode(response)),
                ),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

# Muhim eslatma
Agar hashlar noto‘g‘ri bo‘lsa:
request ishlamaydi
500 yoki empty response keladi.
Hash qiymatlar doim tog'ri kiritilishiga etibor bering.
"hashlar": []

---

