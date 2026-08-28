# Implementation Plan: Productivity Dashboard

## Overview

Implementasi single-file web application (`index.html`) dengan embedded CSS dan JavaScript. Pendekatan incremental: mulai dari fondasi HTML structure dan utility layer, lalu bangun tiap modul secara berurutan, dan akhiri dengan wiring semua modul melalui `App.init()`. Setiap task menghasilkan kode yang langsung terintegrasi — tidak ada kode yang menggantung.

## Tasks

- [ ] 1. Buat struktur dasar HTML dan CSS layout
  - Buat file `index.html` dengan doctype, meta charset, meta viewport, dan title "Productivity Dashboard"
  - Tambahkan `<style>` block dengan CSS Custom Properties (design tokens: warna, spacing, font)
  - Implementasi CSS Grid untuk dashboard layout 2x2 (greeting, timer, todo, links)
  - Tambahkan responsive breakpoints via media queries untuk lebar 320px hingga 2560px (Requirement 6.4)
  - Tambahkan elemen HTML untuk keempat komponen: `#greeting-component`, `#timer-component`, `#todo-component`, `#links-component`
  - Pastikan tidak ada scroll horizontal pada semua breakpoint
  - _Requirements: 6.1, 6.4_

- [x] 2. Implementasi StorageUtil dan konstanta storage keys
  - [x] 2.1 Implementasi StorageUtil dan STORAGE_KEYS
    - Tambahkan `<script>` block di akhir `<body>`
    - Definisikan konstanta `STORAGE_KEYS = { TASKS: 'productivity_tasks', LINKS: 'productivity_links' }`
    - Implementasi `StorageUtil.get(key, defaultValue)`: baca dari `localStorage`, parse JSON, return `defaultValue` jika key tidak ada, JSON parse gagal, atau `localStorage` tidak tersedia — semua dalam try-catch
    - Implementasi `StorageUtil.set(key, value)`: stringify JSON dan tulis ke `localStorage`, return `true` jika berhasil, `false` jika gagal
    - _Requirements: 3.12, 3.13, 4.8, 5.5_

- [x] 3. Implementasi TimeUtil
  - [x] 3.1 Implementasi fungsi-fungsi TimeUtil
    - Implementasi `TimeUtil.formatTime(date)`: return string `HH:MM` dengan zero-padding, return `'--:--'` jika `date` bukan objek Date valid
    - Implementasi `TimeUtil.formatDate(date)`: return string format Bahasa Indonesia "Rabu, 27 Agustus 2025" menggunakan `Intl.DateTimeFormat` atau mapping manual nama hari/bulan, return `'--'` jika tidak valid
    - Implementasi `TimeUtil.getGreeting(hour)`: return `'Selamat Pagi'` (5–11), `'Selamat Siang'` (12–14), `'Selamat Sore'` (15–17), `'Selamat Malam'` (18–23 dan 0–4)
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.8_

- [x] 4. Implementasi GreetingModule
  - [x] 4.1 Implementasi GreetingModule dengan setInterval
    - Implementasi IIFE `GreetingModule` dengan fungsi `init()`, `tick()`, dan `render(time, date, greeting)`
    - `init()`: ambil referensi DOM `#greeting-text`, `#clock-display`, `#date-display`, panggil `tick()` sekali, lalu mulai `setInterval(tick, 1000)`
    - `tick()`: buat `new Date()`, tangani Invalid Date dengan try-catch, panggil `TimeUtil.formatTime()`, `TimeUtil.formatDate()`, `TimeUtil.getGreeting()`, lalu panggil `render()`
    - `render(time, date, greeting)`: update `textContent` ketiga elemen DOM
    - Tambahkan HTML structure di `#greeting-component`: elemen untuk `#greeting-text`, `#clock-display`, `#date-display`
    - Tambahkan CSS styling untuk komponen greeting (font size jam besar, hierarki visual)
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7, 1.8_

- [x] 5. Implementasi TimerModule
  - [x] 5.1 Implementasi state machine TimerModule
    - Implementasi IIFE `TimerModule` dengan state internal: `status` (`'idle'|'running'|'finished'`), `remainingSeconds` (default 1500), `intervalId` (null)
    - Implementasi `start()`: catat `startTime = Date.now()` dan `startRemaining`, mulai `setInterval`, set status `'running'`, panggil `render()`
    - Implementasi `tick()`: hitung `elapsed = Math.floor((Date.now() - startTime) / 1000)`, set `remainingSeconds = Math.max(0, startRemaining - elapsed)`, jika `remainingSeconds === 0` panggil `onFinish()`, selalu panggil `render()`
    - Implementasi `stop()`: `clearInterval`, set status `'idle'`, panggil `render()`
    - Implementasi `reset()`: `clearInterval`, set `remainingSeconds = 1500`, set status `'idle'`, panggil `render()`
    - Implementasi `onFinish()`: `clearInterval`, set status `'finished'`, tampilkan teks "Waktu Habis!" dengan warna berbeda, setelah 3 detik sembunyikan teks (gunakan `setTimeout`)
    - Implementasi `render()`: format `remainingSeconds` ke `MM:SS`, update display, set `disabled` state tombol Start/Stop/Reset sesuai tabel state machine di design
    - Tambahkan HTML di `#timer-component`: display waktu `#timer-display`, tombol `#btn-start`, `#btn-stop`, `#btn-reset`, elemen `#timer-message` untuk "Waktu Habis!"
    - Tambahkan CSS: warna timer normal vs finished, styling tombol dengan state disabled
    - `init()`: attach event listener ke tombol, panggil `render()` untuk tampilkan state awal
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8, 2.9_

- [x] 6. Implementasi TodoModule
  - [x] 6.1 Implementasi CRUD TodoModule dengan inline edit
    - Implementasi IIFE `TodoModule` dengan state: `tasks = []`, `editingId = null`
    - `init()`: load tasks via `StorageUtil.get(STORAGE_KEYS.TASKS, [])`, panggil `render()`
    - `addTask(text)`: validasi `text.trim()` tidak kosong dan `≤ 200` karakter, buat objek Task dengan `id = String(Date.now())`, `text`, `done: false`, `createdAt: Date.now()`, push ke array, panggil `save()` dan `render()`; jika tidak valid, tidak lakukan apa-apa
    - `editTask(id)`: set `editingId = id`, panggil `render()` untuk masuk mode edit
    - `saveEdit(id, newText)`: validasi `newText.trim()` tidak kosong dan `≤ 200` karakter, update `tasks[i].text`, set `editingId = null`, panggil `save()` dan `render()`; jika tidak valid, panggil `cancelEdit(id)`
    - `cancelEdit(id)`: set `editingId = null`, panggil `render()`
    - `toggleTask(id)`: toggle `tasks[i].done`, panggil `save()` dan `render()`
    - `deleteTask(id)`: filter tasks, panggil `save()` dan `render()`
    - `save()`: `StorageUtil.set(STORAGE_KEYS.TASKS, tasks)`
    - `render()`: re-render seluruh list ke DOM `#todo-list`; tiap task dalam mode normal tampilkan teks (dengan strikethrough dan opacity 50% jika `done`), checkbox toggle, tombol edit, tombol delete; tiap task dalam mode edit (`id === editingId`) tampilkan input text berisi teks saat ini, tombol simpan, tombol batal
    - Tambahkan HTML di `#todo-component`: input `#todo-input` (maxlength 200), tombol `#todo-add`, container `#todo-list`
    - Tambahkan event listener: tombol add (klik), input (keydown Enter), event delegation untuk tombol edit/simpan/batal/delete dan checkbox di dalam `#todo-list`, keydown Escape pada input edit untuk `cancelEdit`
    - Tambahkan CSS: strikethrough + opacity 50% untuk task selesai, styling mode edit
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 3.10, 3.11, 3.12, 3.13_

- [~] 7. Checkpoint — Verifikasi komponen greeting, timer, dan todo
  - Buka `index.html` via `file://` di browser, pastikan semua tombol timer berfungsi, todo list bisa ditambah/diedit/dihapus, greeting menampilkan waktu real-time. Tanya user jika ada pertanyaan sebelum lanjut.

- [x] 8. Implementasi LinksModule
  - [x] 8.1 Implementasi CRUD LinksModule dengan validasi dan normalisasi URL
    - Implementasi IIFE `LinksModule` dengan state: `links = []`
    - `init()`: load links via `StorageUtil.get(STORAGE_KEYS.LINKS, [])`, panggil `render()`
    - `normalizeUrl(url)`: jika `url.trim()` tidak dimulai dengan `http://` atau `https://` (case-insensitive check), tambahkan `'https://'` di depan; return URL yang sudah dinormalisasi
    - `addLink(name, url)`: validasi — jika `name.trim()` kosong tampilkan error `"Nama link tidak boleh kosong."`, jika `url.trim()` kosong tampilkan error `"URL tidak boleh kosong."`, jika `name.trim().length > 100` tampilkan error `"Nama link maksimal 100 karakter."`, jika `links.length >= 20` tampilkan error `"Batas maksimal 20 link telah tercapai."`, jika semua valid: normalisasi URL, buat objek Link dengan `id`, `name`, `url`, `createdAt`, push ke array, panggil `save()` dan `render()`, kosongkan form input
    - `deleteLink(id)`: filter links, panggil `save()` dan `render()`
    - `openLink(url)`: `window.open(url, '_blank', 'noopener,noreferrer')`
    - `showError(fieldId, msg)`: tampilkan pesan error di elemen DOM yang sesuai
    - `save()`: `StorageUtil.set(STORAGE_KEYS.LINKS, links)`
    - `render()`: re-render seluruh list ke DOM `#links-list`; tiap link tampilkan nama sebagai tombol/link yang memanggil `openLink`, dan tombol delete
    - Tambahkan HTML di `#links-component`: input `#link-name` (maxlength 100), input `#link-url`, tombol `#link-add`, elemen error `#link-name-error` dan `#link-url-error`, container `#links-list`
    - Tambahkan event listener: tombol add, event delegation untuk tombol delete dan klik link di `#links-list`
    - Tambahkan CSS: styling grid/flex untuk link cards, styling tombol link dan delete
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8, 4.9_

- [x] 9. Implementasi App.init() dan bootstrap
  - [x] 9.1 Wire semua modul melalui App.init()
    - Definisikan objek `App` dengan method `init()`
    - `App.init()`: panggil `GreetingModule.init()`, `TimerModule.init()`, `TodoModule.init()`, `LinksModule.init()` secara berurutan
    - Tambahkan event listener `DOMContentLoaded` yang memanggil `App.init()`
    - Pastikan urutan script: `StorageUtil` → `STORAGE_KEYS` → `TimeUtil` → `GreetingModule` → `TimerModule` → `TodoModule` → `LinksModule` → `App` → `App.init()` call
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 6.1, 6.2, 6.5_

- [x] 10. Polish UI dan verifikasi kompatibilitas
  - [x] 10.1 Finalisasi CSS dan verifikasi lintas browser
    - Pastikan semua state tombol (disabled, hover, active, focus) memiliki visual yang jelas dan accessible
    - Tambahkan CSS transitions ringan (max 150ms) untuk feedback interaksi agar perubahan visual terlihat dalam < 100ms (Requirement 6.3)
    - Verifikasi layout tidak overflow secara horizontal pada lebar 320px (Requirement 6.4)
    - Pastikan semua elemen interaktif memiliki `:focus` outline yang visible
    - Tambahkan `<meta name="color-scheme" content="light dark">` jika ingin support dark mode otomatis, atau tetapkan skema warna tetap
    - Review kode JS — pastikan tidak ada `console.error` atau error message teknis yang muncul ke pengguna (Requirement 5.5, 3.13, 6.1)
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [x] 11. Checkpoint akhir — Verifikasi full application
  - Buka `index.html` via `file://`, verifikasi: (1) semua data Task dan Link persist setelah refresh, (2) timer berfungsi akurat, (3) greeting update setiap detik, (4) tidak ada JavaScript error di console, (5) layout responsif di berbagai ukuran viewport. Tanya user jika ada pertanyaan.

## Notes

- Semua kode berada dalam satu file `index.html` — embedded `<style>` dan `<script>` tanpa file eksternal
- Urutan definisi dalam `<script>` penting: utility objects harus didefinisikan sebelum modul yang menggunakannya
- Gunakan IIFE untuk tiap modul agar scope terisolasi dan tidak mencemari global namespace
- Timer menggunakan strategi `Date.now()` bukan pure decrement counter untuk ketepatan saat tab tidak aktif
- StorageUtil menangani semua error localStorage secara silent — komponen tidak perlu try-catch sendiri
- Quick Links membuka URL dengan `noopener,noreferrer` untuk keamanan
- Setiap task dapat dieksekusi secara berurutan; task 9 harus dikerjakan terakhir karena mengandalkan semua modul sebelumnya

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["2.1"] },
    { "id": 1, "tasks": ["3.1"] },
    { "id": 2, "tasks": ["4.1"] },
    { "id": 3, "tasks": ["5.1"] },
    { "id": 4, "tasks": ["6.1"] },
    { "id": 5, "tasks": ["8.1"] },
    { "id": 6, "tasks": ["9.1"] },
    { "id": 7, "tasks": ["10.1"] }
  ]
}
```
