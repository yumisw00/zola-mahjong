# Zola Mahjong - Technical Blueprint & Game Design Document (GDD)

## 1. METADATA PROYEK
- **Nama Game:** Zola Mahjong
- **Platform:** Android (Optimasi target: Xiaomi 12 5G)
- **Orientasi Layar:** Locked Portrait
- **Tech Stack:** Kotlin, Jetpack Compose, Room Database, WorkManager, Firebase (Auth, Firestore, Remote Config), Retrofit/Ktor.
- **Pola Arsitektur:** MVVM + Clean Architecture + Command Pattern (Khusus State Management Game).
- **Status Konektivitas:** Hybrid (Offline-First SSOT).

---

## 2. GAME DESIGN & CORE LOOP (GDD)
### Mekanik Utama (Match-Puzzle)
- **Papan Utama:** Berisi tumpukan ubin dengan tema yang berubah sesuai *stage*.
- **Tray (Nampan):** Berada di bawah papan, memiliki **4 slot (4 kolom, 1 baris)**.
- **Kondisi Menang:** Semua ubin di papan habis.
- **Kondisi Kalah:** Tray terisi penuh (4 ubin) tanpa ada kecocokan.
- **Animasi:** Ubin yang cocok menyatu ke tengah tray, lalu memicu efek ledakan partikel bintang/cahaya (menggunakan `Canvas` dan `Animatable`). Tersedia opsi *Haptic Feedback* dan *SoundPool* di pengaturan.

### Sistem Bantuan (Power-ups)
Terdapat 3 tombol di bawah tray:
1. **Acak Ulang (Shuffle):** Mengacak posisi ubin yang tersisa di papan.
2. **Bantuan (Hint):** Menyorot ubin yang bisa diambil (Didukung algoritma AI).
3. **Undo:** Mengembalikan langkah terakhir menggunakan *Command Pattern*.
*Aturan:* Bebas digunakan (*unlimited*) pada 2 level pertama di setiap stage. Setelahnya, dibatasi 3x penggunaan. Lebih dari itu wajib menonton iklan (*Rewarded Ads*).

### Flow Kegagalan (Game Over)
- Saat tray penuh, *popup* muncul menampilkan detail progres.
- **Opsi 1:** Kembali ke stage / Ulangi level.
- **Opsi 2:** "Lanjutkan" (Wajib menonton iklan). Selama memuat iklan, tampilkan *loading spinner* standar.

---

## 3. ARSITEKTUR SISTEM & STATE MANAGEMENT
### Command Pattern (Logika Undo)
- Setiap langkah pemain dienkapsulasi menjadi objek *Command* (misal: `MoveTileCommand`).
- Disimpan dalam struktur `Stack` (LIFO) di dalam ViewModel.
- Saat Undo ditekan, jalankan fungsi `.undo()` pada *Command* terakhir untuk merender ulang 2 ubin tanpa membebani memori Compose.

### Offline-First & Sinkronisasi
- **SSOT (Single Source of Truth):** Jetpack Compose HANYA membaca state dari Room Database.
- **WorkManager:** Mengatur antrean *task* saat perangkat *offline* (misal: "Pending_Review", "Pending_Friend_Delete"). Saat *online*, *task* ini dieksekusi ke Firestore.

---

## 4. SKEMA DATA (FIREBASE & ROOM)

### A. Firebase Remote Config (JSON)
Digunakan untuk *update stage* tanpa perlu *update* aplikasi di Play Store.
```json
{
  "stages": [
    {
      "stage_id": 1,
      "theme": "food_and_restaurant",
      "levels": [
        {"level_id": 101, "layout": [[0,0],[0,1]...], "difficulty": "easy"}
      ]
    }
  ]
}

```

### B. Firestore Collections

* `users`: `{ uid, generated_name, bind_status (google/fb/x), level_progress }`
* `friends`: Relasi pertemanan. Fitur tambah teman hanya bisa saat *online*. Jika teman < 10, tampilkan rekomendasi *user online*.
* `reviews`: `{ stage_id, user_id, rating (1-5), comment, timestamp }`
* *Pagination:* Kueri dibatasi 5 ulasan per halaman (Filter: Terbaru, Terlama, Bintang 1-5). Teks indikator: "Menampilkan 5 dari 20".


* `forum_threads`: `{ thread_id, user_id, level_id_attachment, content, zolai_response }`

---

## 5. FORUM & LOGIKA AI "ZOLAI"

* **Forum Sosial:** Ruang obrolan *real-time* antar pemain. Pemain dapat menyisipkan *level/stage* spesifik yang sedang sulit mereka lewati ke dalam forum.
* **Zolai (AI Assistant):**
* Agen AI yang memonitor forum dan merespons pertanyaan dengan parameter `@zolai`.
* **Fungsi:** Memberikan tips taktis mengenai level tertentu, menjelaskan mekanik game, atau menyemangati pemain.
* **Sistem Syarat Dosen:** Ini memenuhi poin "Penggunaan AI secara bijak" dengan mengintegrasikan LLM (seperti Gemini API / OpenAI API) di sisi *backend* (Firebase Cloud Functions).



---

## 6. ROADMAP SKALABILITAS & MONETISASI (FUTURE EXPANSION)

*Fitur di bawah ini disiapkan slot arsitekturnya, namun belum diaktifkan di versi rilis pertama (MVP).*

1. **In-App Purchases (IAP):**
* Pembelian paket *No-Ads* (Menghilangkan iklan pop-up dan *banner*, namun iklan *rewarded* tetap opsional untuk item).
* Bundel Item: Membeli koin untuk ditukar dengan tiket *Shuffle*, *Hint*, atau *Undo*.


2. **Kustomisasi Kosmetik:** Membeli tema ubin premium atau avatar profil.
3. **Papan Peringkat (Leaderboard) Regional:** Menampilkan pemain terbaik di area lokal (misal: Regional Ngawi/Jawa Timur).

```

```
