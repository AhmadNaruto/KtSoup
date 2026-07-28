# Panduan Penggunaan KtSoup Android Native ARM64 (`androidNativeArm64`)

Panduan ini menjelaskan cara mempublikasikan hasil build lokal, mengintegrasikannya ke proyek Kotlin Multiplatform (KMP) Anda, serta mekanisme kerja library native Lexbor pada target Android ARM64.

---

## 1. Publikasikan KtSoup ke Maven Lokal
Sebelum digunakan di proyek eksternal, Anda perlu mempublikasikan hasil modifikasi library KtSoup ini ke repositori Maven Lokal (`~/.m2/repository`) di mesin Anda.

Jalankan perintah berikut di direktori root KtSoup:
```bash
./gradlew publishToMavenLocal -Porg.gradle.java.installations.paths=/data/data/com.termux/files/usr/lib/jvm/java-17-openjdk
```
Perintah ini akan membangun dan mempublikasikan artifact `.klib` (Kotlin Library) dan `.jar` untuk seluruh target (termasuk `androidNativeArm64`) ke penyimpanan lokal Anda.

---

## 2. Integrasikan ke Proyek Kotlin Multiplatform (KMP) Anda

Pada proyek KMP Anda, ikuti langkah berikut untuk menggunakan target baru ini:

### A. Aktifkan Repositori Maven Local
Buka file `settings.gradle.kts` atau `build.gradle.kts` proyek Anda, lalu tambahkan `mavenLocal()` ke daftar repositori:
```kotlin
repositories {
    mavenLocal() // Wajib agar mendeteksi library yang dipublikasikan secara lokal
    mavenCentral()
}
```

### B. Tambahkan Target `androidNativeArm64` dan Dependensi
Buka file `build.gradle.kts` modul KMP Anda dan konfigurasi targetnya:
```kotlin
plugins {
    kotlin("multiplatform") version "2.2.10"
}

kotlin {
    // Target JVM Android biasa (opsional, jika Anda membangun aplikasi Android biasa)
    androidTarget()

    // Target Native Android ARM64 (menggunakan NDK & Lexbor)
    androidNativeArm64()

    sourceSets {
        commonMain.dependencies {
            // Pasang dependensi KtSoup Core dari lokal maven
            implementation("org.drewcarlson:ktsoup-core:1.0.0-SNAPSHOT")
            // implementasi modul pendukung jika dibutuhkan:
            implementation("org.drewcarlson:ktsoup-fs:1.0.0-SNAPSHOT")
            implementation("org.drewcarlson:ktsoup-ktor:1.0.0-SNAPSHOT")
        }
    }
}
```

> **Catatan:** Anda tidak perlu menyalin file `liblexbor_static.a` secara manual ke proyek KMP Anda. 
> Kotlin Multiplatform secara otomatis mengepak file static library (`liblexbor_static.a`) tersebut ke dalam file metadata `.klib` saat dipublikasikan, sehingga proses linking akan terjadi otomatis ketika Anda mengompilasi proyek KMP Anda untuk Android Native.

---

## 3. Contoh Penggunaan di Kode Program (`commonMain`)

Karena KtSoup adalah library multiplatform, Anda dapat langsung menggunakannya di source set `commonMain` (yang otomatis dibagikan ke platform `androidNativeArm64`):

```kotlin
import ktsoup.KtSoupParser

fun dapatkanJudulHalaman(htmlContent: String): String? {
    // Parsing dokumen HTML
    val document = KtSoupParser.parse(htmlContent)
    
    // Gunakan blok `.use` untuk menjamin dealokasi memori native (Lexbor)
    return document.use { doc ->
        val titleElement = doc.querySelector("title")
        titleElement?.textContent()
    }
}

fun main() {
    val html = """
        <html>
            <head>
                <title>Halaman Contoh</title>
            </head>
            <body>
                <div class="content">Hello dari KtSoup Native!</div>
            </body>
        </html>
    """.trimIndent()

    println(dapatkanJudulHalaman(html)) // Output: Halaman Contoh
}
```

---

## 4. Mekanisme Kerja di Belakang Layar
1. **Kotlin/Native Compiler** akan memproses source code Anda untuk target Android ARM64 menggunakan Android NDK.
2. Saat proses **Linking**, compiler akan mengambil binary `liblexbor_static.a` yang tertanam di dalam dependency `ktsoup-core.klib` dan menautkannya ke dalam output binary bersama aplikasi Android Native Anda.
3. Alokasi memori native Lexbor dilakukan menggunakan memori C standard (`malloc`), sehingga pastikan untuk selalu menggunakan ekstensi `.use { ... }` pada `KtSoupDocument` untuk menghindari kebocoran memori (memory leak) di runtime Android.
