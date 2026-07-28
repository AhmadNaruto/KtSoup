# Panduan Produksi JAR & Integrasi Manual di Proyek Android

Panduan ini menjelaskan cara membuat file `.jar` dari modul `ktsoup-core` dan cara memasang serta mengintegrasikannya secara manual ke proyek aplikasi Android Studio Anda.

---

## Bagian 1: Cara Memproduksi File JAR

Untuk menghasilkan file `.jar` untuk target JVM (Java/Android), ikuti langkah-langkah berikut:

1. Buka terminal di direktori root proyek `KtSoup`.
2. Jalankan perintah kompilasi khusus target JVM berikut:
   ```bash
   ./gradlew :ktsoup-core:jvmJar -Porg.gradle.java.installations.paths=/data/data/com.termux/files/usr/lib/jvm/java-17-openjdk
   ```
3. Setelah proses kompilasi selesai (`BUILD SUCCESSFUL`), file `.jar` akan tersimpan di lokasi berikut:
   `ktsoup-core/build/libs/ktsoup-core-jvm-1.0.0-SNAPSHOT.jar`

---

## Bagian 2: Cara Integrasi Manual ke Proyek Android Studio

Karena kita menggunakan file `.jar` hasil kompilasi lokal secara mandiri, silakan lakukan integrasi dengan salah satu cara berikut:

### Langkah 1: Salin File JAR ke Proyek Android
1. Buka proyek aplikasi Android Anda di Android Studio.
2. Ubah struktur tampilan direktori ke mode **Project**.
3. Salin file `ktsoup-core-jvm-1.0.0-SNAPSHOT.jar` yang telah diproduksi sebelumnya.
4. Tempel (Paste) file tersebut ke direktori `app/libs/` di dalam modul aplikasi Android Anda.

### Langkah 2: Tambahkan Dependensi Transitif (Penting)
Modul `ktsoup-core-jvm` pada platform JVM/Android bekerja sebagai wrapper di atas library **JSoup** (library HTML Parser Java). Oleh karena itu, Anda **wajib** menyertakan library JSoup agar aplikasi tidak mengalami error `NoClassDefFoundError` di runtime.

Buka file `build.gradle` atau `build.gradle.kts` tingkat modul aplikasi (`app/build.gradle`), lalu tambahkan konfigurasi berikut:

#### Opsi A: Menggunakan Gradle Dependensi untuk JSoup (Disarankan)
Opsi ini sangat praktis karena Gradle akan mengunduh JSoup secara otomatis:
```kotlin
dependencies {
    // Membaca semua file .jar di dalam direktori libs secara otomatis
    implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
    
    // Dependensi transitif wajib yang dibutuhkan oleh ktsoup-core-jvm
    implementation("org.jsoup:jsoup:1.18.1")
}
```

#### Opsi B: Mengunduh JSoup Secara Manual (Fully Offline)
Jika proyek Anda sepenuhnya offline:
1. Unduh file `jsoup-1.18.1.jar` dari repositori Maven Central.
2. Salin dan tempel file `jsoup-1.18.1.jar` tersebut ke direktori `app/libs/` bersama dengan file `ktsoup-core-jvm-1.0.0-SNAPSHOT.jar`.
3. Di file `app/build.gradle`, cukup pastikan folder `libs` dibaca:
   ```kotlin
   dependencies {
       implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
   }
   ```

### Langkah 3: Sinkronisasi Proyek
Klik tombol **Sync Project with Gradle Files** di bagian pojok kanan atas Android Studio untuk memuat library baru Anda.

---

## Bagian 3: Contoh Penggunaan di Aplikasi Android

Setelah sinkronisasi sukses, Anda dapat memanggil API KtSoup langsung di dalam file Activity atau Fragment Anda (baik menggunakan Kotlin maupun Java):

### Contoh Menggunakan Kotlin (`MainActivity.kt`)
```kotlin
package com.example.myapp

import android.os.Bundle
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import ktsoup.KtSoupParser

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val htmlContent = """
            <html>
                <body>
                    <h1 id="header">Halo dari KtSoup JAR!</h1>
                </body>
            </html>
        """.trimIndent()

        // Parsing HTML menggunakan KtSoup
        val doc = KtSoupParser.parse(htmlContent)
        val teksHeader = doc.use { document ->
            document.querySelector("#header")?.textContent()
        }

        // Tampilkan teks ke TextView
        findViewById<TextView>(R.id.textView).text = teksHeader
    }
}
```
