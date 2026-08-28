# Design Document: Productivity Dashboard

## Overview

Productivity Dashboard adalah sebuah single-page web application yang dirancang sebagai halaman tab baru yang produktif. Aplikasi ini dibangun menggunakan HTML, CSS, dan Vanilla JavaScript murni tanpa framework atau library eksternal, sehingga dapat berjalan langsung melalui protokol `file://` di semua modern browser.

Tujuan utama desain ini adalah memberikan antarmuka yang bersih, responsif, dan cepat, di mana semua state dikelola di sisi klien menggunakan browser Local Storage — tanpa memerlukan server, API, atau koneksi internet.

**Keputusan desain utama:**
- **Single HTML file** (`index.html`) dengan embedded `<style>` dan `<script>` — menyederhanakan distribusi dan deployment via `file://`.
- **Module pattern** melalui IIFE (Immediately Invoked Function Expression) per komponen untuk mengisolasi scope dan mencegah global namespace pollution tanpa membutuhkan bundler.
- **Event-driven architecture** yang ringan: setiap komponen mendengarkan event DOM dan memanggil storage layer untuk persistensi.
- **Graceful degradation**: setiap komponen menangani kegagalan Local Storage dan clock secara mandiri tanpa error yang terlihat oleh pengguna.

---

## Architecture

Aplikasi terdiri dari satu layer presentasi (DOM manipulation) dan satu layer data (Local Storage abstraction). Tidak ada layer server, tidak ada state management library.

```
┌──────────────────────────────────────────────────────────┐
│                      index.html                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │                  <style> (CSS)                      │ │
│  │  - CSS Custom Properties (design tokens)            │ │
│  │  - Layout: CSS Grid (dashboard grid)                │ │
│  │  - Responsive breakpoints via media queries         │ │
│  └─────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────┐ │
│  │                  <body> (HTML)                      │ │
│  │  #greeting-component                                │ │
│  │  #timer-component                                   │ │
│  │  #todo-component                                    │ │
│  │  #links-component                                   │ │
│  └─────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────┐ │
│  │                  <script> (JS)                      │ │
│  │                                                     │ │
│  │  ┌─────────────┐   ┌──────────────────────────┐    │ │
│  │  │ StorageUtil │   │       TimeUtil            │    │ │
│  │  │ (CRUD wrap) │   │  (clock, greeting logic)  │    │ │
│  │  └──────┬──────┘   └──────────────────────────┘    │ │
│  │         │                                           │ │
│  │  ┌──────▼──────────────────────────────────────┐   │ │
│  │  │              Components (IIFE)               │   │ │
│  │  │  GreetingModule  │  TimerModule              │   │ │
│  │  │  TodoModule      │  LinksModule              │   │ │
│  │  └─────────────────────────────────────────────┘   │ │
│  │                                                     │ │
│  │  App.init() — bootstrap semua modul                 │ │
│  └─────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
                            │
                ┌───────────▼──────────────┐
                │    Browser Local Storage  │
                │  "productivity_tasks"     │
                │  "productivity_links"     │
                └──────────────────────────┘
```

### Alur Inisialisasi

```
DOMContentLoaded
    │
    ├── App.init()
    │       ├── GreetingModule.init()   → mulai setInterval clock (1 detik)
    │       ├── TimerModule.init()      → render UI timer default 25:00
    │       ├── TodoModule.init()       → load dari Local Storage, render list
    │       └── LinksModule.init()      → load dari Local Storage, render list
    │
    └── Selesai (halaman fully interactive)
```

---

## Components and Interfaces

### 1. StorageUtil

Utilitas wrapper untuk `window.localStorage` yang mengabstraksi error handling sehingga komponen tidak perlu menangani exception secara langsung.

```javascript
const StorageUtil = {
  // Membaca nilai dari Local Storage, mengembalikan defaultValue jika:
  // - key tidak ada
  // - JSON.parse gagal
  // - localStorage tidak tersedia
  get(key, defaultValue) { ... },

  // Menyimpan nilai ke Local Storage sebagai JSON string.
  // Mengembalikan true jika berhasil, false jika gagal (misal: storage penuh).
  set(key, value) { ... }
};
```

**Konstanta storage keys:**
```javascript
const STORAGE_KEYS = {
  TASKS: 'productivity_tasks',
  LINKS: 'productivity_links'
};
```

---

### 2. TimeUtil

Utilitas murni (pure functions) untuk logika waktu dan sapaan.

```javascript
const TimeUtil = {
  // Mengembalikan string waktu HH:MM dari objek Date.
  // Mengembalikan '--:--' jika Date tidak valid.
  formatTime(date) { ... },

  // Mengembalikan string tanggal dalam format Bahasa Indonesia:
  // "Rabu, 27 Agustus 2025"
  // Mengembalikan '--' jika Date tidak valid.
  formatDate(date) { ... },

  // Mengembalikan string sapaan berdasarkan jam (0-23).
  // Mengembalikan salah satu dari:
  // "Selamat Pagi"   (5–11)
  // "Selamat Siang"  (12–14)
  // "Selamat Sore"   (15–17)
  // "Selamat Malam"  (18–4)
  getGreeting(hour) { ... }
};
```

---

### 3. GreetingModule

Mengelola tampilan waktu, tanggal, dan sapaan. Menggunakan `setInterval` untuk memperbarui tampilan setiap detik.

```javascript
const GreetingModule = (() => {
  // DOM refs: #greeting-text, #clock-display, #date-display
  // State: intervalId (untuk cleanup jika diperlukan)

  function init() { ... }      // Attach DOM refs, mulai interval
  function tick() { ... }      // Dipanggil tiap detik, update DOM
  function render(time, date, greeting) { ... }  // Update textContent

  return { init };
})();
```

**Behavior tick():**
1. Buat `new Date()` — jika hasilnya `Invalid Date`, tampilkan fallback.
2. Panggil `TimeUtil.formatTime()`, `TimeUtil.formatDate()`, `TimeUtil.getGreeting()`.
3. Panggil `render()` untuk update DOM.

---

### 4. TimerModule

Mengelola Focus Timer dengan state machine sederhana.

```javascript
const TimerModule = (() => {
  // State
  // status: 'idle' | 'running' | 'finished'
  // remainingSeconds: number (default 1500)
  // intervalId: number | null

  function init() { ... }       // Render initial state
  function start() { ... }      // Mulai countdown, update status → 'running'
  function stop() { ... }       // Pause countdown, status → 'idle'
  function reset() { ... }      // Reset ke 1500, status → 'idle'
  function tick() { ... }       // Decrement remainingSeconds, check 0
  function onFinish() { ... }   // Status → 'finished', show "Waktu Habis!"
  function render() { ... }     // Update display dan button states

  return { init };
})();
```

**Timer State Machine:**

```
        ┌──────────────┐
        │     idle     │◄────────────────────────────┐
        └──────┬───────┘                             │
               │ [Start]                         [Reset]
               ▼                                     │
        ┌──────────────┐                     ┌───────┴──────┐
        │   running    │──── [Stop] ────────►│     idle     │
        └──────┬───────┘                     └──────────────┘
               │
         [reaches 00:00]
               │
               ▼
        ┌──────────────┐
        │   finished   │──── [Reset] ──────► idle
        └──────────────┘
```

**Button states per status:**

| Status   | Start   | Stop    | Reset   |
|----------|---------|---------|---------|
| idle     | enabled | disabled| enabled |
| running  | disabled| enabled | enabled |
| finished | enabled | disabled| enabled |

---

### 5. TodoModule

Mengelola CRUD untuk task list dengan inline editing.

```javascript
const TodoModule = (() => {
  // State: tasks = Task[]
  // editingId: string | null (ID task yang sedang diedit)

  function init() { ... }              // Load dari storage, render
  function addTask(text) { ... }       // Validasi, buat Task baru, simpan, render
  function editTask(id) { ... }        // Masuk mode edit untuk task[id]
  function saveEdit(id, newText) { ... } // Validasi, update, simpan, render
  function cancelEdit(id) { ... }      // Kembali ke tampilan normal
  function toggleTask(id) { ... }      // Toggle status.done, simpan, render
  function deleteTask(id) { ... }      // Hapus dari array, simpan, render
  function save() { ... }              // Tulis ke Local Storage
  function render() { ... }            // Re-render seluruh list ke DOM

  return { init };
})();
```

**Validasi teks task:**
- Bukan string kosong setelah `.trim()`
- Panjang setelah `.trim()` ≤ 200 karakter

---

### 6. LinksModule

Mengelola CRUD untuk quick links.

```javascript
const LinksModule = (() => {
  // State: links = Link[]

  function init() { ... }              // Load dari storage, render
  function addLink(name, url) { ... }  // Validasi, normalisasi URL, simpan, render
  function deleteLink(id) { ... }      // Hapus dari array, simpan, render
  function openLink(url) { ... }       // window.open(url, '_blank')
  function normalizeUrl(url) { ... }   // Tambahkan https:// jika tidak ada protokol
  function save() { ... }              // Tulis ke Local Storage
  function render() { ... }            // Re-render seluruh list ke DOM
  function showError(fieldId, msg) { ... }  // Tampilkan pesan error di UI

  return { init };
})();
```

**Validasi link:**
- `name`: wajib tidak kosong, max 100 karakter setelah `.trim()`
- `url`: wajib tidak kosong setelah `.trim()`
- `links.length < 20` sebelum penambahan

**Normalisasi URL:**
- Jika URL tidak dimulai dengan `http://` atau `https://` → tambahkan `https://` di depan.

---

## Data Models

### Task

```javascript
/**
 * @typedef {Object} Task
 * @property {string} id        - UUID atau timestamp string, unik per task
 * @property {string} text      - Teks deskripsi task, 1-200 karakter
 * @property {boolean} done     - Status selesai
 * @property {number} createdAt - Unix timestamp (ms) saat task dibuat
 */
```

**Contoh:**
```json
{
  "id": "1724745600000",
  "text": "Baca dokumentasi CSS Grid",
  "done": false,
  "createdAt": 1724745600000
}
```

**Local Storage key:** `"productivity_tasks"`  
**Format penyimpanan:** JSON array dari Task objects.

---

### Link

```javascript
/**
 * @typedef {Object} Link
 * @property {string} id        - UUID atau timestamp string, unik per link
 * @property {string} name      - Nama tampilan link, 1-100 karakter
 * @property {string} url       - URL lengkap dengan protokol (http:// atau https://)
 * @property {number} createdAt - Unix timestamp (ms) saat link dibuat
 */
```

**Contoh:**
```json
{
  "id": "1724745700000",
  "name": "GitHub",
  "url": "https://github.com",
  "createdAt": 1724745700000
}
```

**Local Storage key:** `"productivity_links"`  
**Format penyimpanan:** JSON array dari Link objects.

---

### TimerState (in-memory only, tidak dipersist)

```javascript
/**
 * @typedef {'idle' | 'running' | 'finished'} TimerStatus
 *
 * @typedef {Object} TimerState
 * @property {TimerStatus} status           - Status timer saat ini
 * @property {number} remainingSeconds      - Detik tersisa (default: 1500)
 * @property {number|null} intervalId       - ID setInterval aktif, atau null
 */
```

> **Keputusan desain:** Timer state tidak dipersist ke Local Storage secara sengaja. Behavior yang diharapkan adalah timer selalu kembali ke 25:00 saat halaman di-refresh — sesuai dengan paradigma "fresh start" Pomodoro. Persisting timer state akan menambah kompleksitas edge case (misalnya: berapa lama halaman tertutup?).

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

---

### Property 1: Greeting sesuai dengan rentang jam

*For any* jam (0–23), fungsi `getGreeting(hour)` SHALL mengembalikan tepat satu sapaan yang sesuai dengan rentang jamnya, dan tidak ada jam yang tidak menghasilkan sapaan.

**Validates: Requirements 1.3, 1.4, 1.5, 1.6**

---

### Property 2: Format waktu selalu HH:MM

*For any* objek `Date` yang valid, `formatTime(date)` SHALL mengembalikan string dalam format `HH:MM` di mana jam adalah dua digit (00–23) dan menit adalah dua digit (00–59).

**Validates: Requirements 1.1**

---

### Property 3: Normalisasi URL adalah idempoten

*For any* string URL, memanggil `normalizeUrl` dua kali berturut-turut SHALL menghasilkan hasil yang sama dengan memanggil sekali — karena URL yang sudah memiliki protokol tidak akan mendapat awalan ganda.

**Validates: Requirements 4.5**

---

### Property 4: Task yang ditambahkan dapat dipulihkan dari storage

*For any* teks task yang valid (non-kosong, tidak hanya spasi, max 200 karakter), setelah `addTask(text)` dipanggil dan `save()` dilakukan, memanggil `StorageUtil.get(STORAGE_KEYS.TASKS, [])` SHALL menghasilkan array yang mengandung task dengan teks tersebut.

**Validates: Requirements 3.2, 3.11, 5.2**

---

### Property 5: Validasi task menolak semua teks tidak valid

*For any* string yang seluruhnya terdiri dari karakter whitespace, atau string dengan panjang lebih dari 200 karakter, `addTask(text)` SHALL menolak penambahan dan jumlah task dalam daftar SHALL tetap tidak berubah.

**Validates: Requirements 3.3**

---

### Property 6: Toggle task dua kali mengembalikan state semula

*For any* task dengan status awal `done`, memanggil `toggleTask(id)` dua kali SHALL menghasilkan task dengan status `done` yang sama persis dengan status awalnya (round-trip property).

**Validates: Requirements 3.8, 3.9**

---

### Property 7: Edit task mempertahankan ID dan createdAt

*For any* task yang ada, memanggil `saveEdit(id, newText)` dengan teks valid SHALL memperbarui `text` task tersebut, namun `id` dan `createdAt` SHALL tetap tidak berubah.

**Validates: Requirements 3.5**

---

### Property 8: Link yang ditambahkan selalu memiliki protokol valid

*For any* input URL (dengan atau tanpa protokol), setelah `addLink(name, url)` berhasil, URL yang tersimpan SHALL selalu dimulai dengan `"http://"` atau `"https://"`.

**Validates: Requirements 4.2, 4.5**

---

### Property 9: Batas 20 link selalu ditegakkan

*For any* daftar link yang sudah berisi tepat 20 item, memanggil `addLink(name, url)` SHALL menolak penambahan dan jumlah link SHALL tetap 20.

**Validates: Requirements 4.9**

---

### Property 10: Data storage round-trip mempertahankan struktur

*For any* array Task atau Link yang valid, melakukan `StorageUtil.set(key, data)` kemudian `StorageUtil.get(key, [])` SHALL menghasilkan array yang ekuivalen secara struktural dengan array asli (deep equality).

**Validates: Requirements 5.1, 5.2**

---

## Error Handling

### Strategi Umum

Semua error ditangani secara diam-diam (silent) di sisi teknis dan digantikan dengan perilaku fallback yang ramah pengguna. Tidak ada stack trace, nama fungsi, atau kode error yang ditampilkan ke pengguna.

### Per Komponen

| Komponen | Skenario Error | Perilaku Fallback |
|---|---|---|
| **StorageUtil** | `localStorage` tidak tersedia (private mode, storage penuh) | `get()` → return `defaultValue`; `set()` → return `false`, data tidak disimpan |
| **StorageUtil** | Data JSON corrupt/tidak valid | `get()` → catch `JSON.parse` error, return `defaultValue` |
| **GreetingModule** | `new Date()` menghasilkan Invalid Date | Tampilkan `"--:--"` untuk waktu dan `"--"` untuk tanggal |
| **TimerModule** | `setInterval` terlambat (tab tidak aktif) | Kompensasi dengan membandingkan `Date.now()` — bukan hanya decrement counter |
| **TodoModule** | Input kosong/spasi/terlalu panjang | Tolak penambahan/edit, tidak tampilkan pesan error (form tetap kosong atau teks kembali ke nilai lama) |
| **LinksModule** | Nama kosong | Tampilkan pesan: `"Nama link tidak boleh kosong."` |
| **LinksModule** | URL kosong | Tampilkan pesan: `"URL tidak boleh kosong."` |
| **LinksModule** | Nama melebihi 100 karakter | Tampilkan pesan: `"Nama link maksimal 100 karakter."` |
| **LinksModule** | Sudah 20 link | Tampilkan pesan: `"Batas maksimal 20 link telah tercapai."` |

### Catatan tentang TimerModule dan Tab Visibility

Browser dapat men-throttle `setInterval` ketika tab tidak aktif (hingga 1000ms+). Untuk ketepatan, timer SEBAIKNYA menggunakan strategi berbasis `Date.now()`:

```javascript
// Saat start():
const startTime = Date.now();
const startRemaining = remainingSeconds;

// Saat setiap tick:
const elapsed = Math.floor((Date.now() - startTime) / 1000);
remainingSeconds = Math.max(0, startRemaining - elapsed);
```

Ini memastikan tampilan tetap akurat meskipun interval terlambat.

---

## Testing Strategy

### Pendekatan Dual Testing

Strategi pengujian menggunakan dua pendekatan yang saling melengkapi:

1. **Unit tests (example-based)**: Memverifikasi skenario spesifik, edge case, dan kondisi error.
2. **Property-based tests (PBT)**: Memverifikasi properti universal yang harus berlaku untuk semua input valid.

### Library yang Digunakan

- **Test runner**: [Vitest](https://vitest.dev/) — kompatibel dengan Node.js, mendukung ES modules, zero-config.
- **Property-based testing**: [fast-check](https://fast-check.dev/) — library PBT untuk JavaScript/TypeScript.
- **Target minimum iterasi PBT**: 100 iterasi per properti.

### Catatan Arsitektur untuk Testability

Agar unit dan property test dapat dijalankan di lingkungan Node.js (tanpa DOM), semua fungsi **pure logic** (TimeUtil, StorageUtil.get/set, validasi, normalisasi URL) HARUS diekspor sebagai fungsi mandiri yang tidak bergantung pada `window` atau `document`. Komponen UI (GreetingModule, TimerModule, dst.) dapat diuji dengan mock DOM sederhana atau jsdom.

### Struktur File Test

```
tests/
├── unit/
│   ├── timeUtil.test.js       (greeting, formatTime, formatDate)
│   ├── storageUtil.test.js    (get/set dengan mock localStorage)
│   ├── timerModule.test.js    (state machine transitions)
│   ├── todoModule.test.js     (CRUD, validasi)
│   └── linksModule.test.js    (CRUD, validasi, normalisasi URL)
└── property/
    ├── timeUtil.prop.test.js  (Properties 1, 2)
    ├── linksModule.prop.test.js (Properties 3, 8, 9)
    ├── todoModule.prop.test.js  (Properties 4, 5, 6, 7)
    └── storageUtil.prop.test.js (Property 10)
```

### Contoh Property Test (fast-check)

```javascript
// Property 1: Greeting sesuai rentang jam
// Feature: productivity-dashboard, Property 1: Greeting sesuai dengan rentang jam
import * as fc from 'fast-check';
import { getGreeting } from '../src/timeUtil.js';

test('Property 1: setiap jam menghasilkan tepat satu sapaan yang valid', () => {
  const validGreetings = [
    'Selamat Pagi', 'Selamat Siang', 'Selamat Sore', 'Selamat Malam'
  ];
  fc.assert(
    fc.property(fc.integer({ min: 0, max: 23 }), (hour) => {
      const result = getGreeting(hour);
      return validGreetings.includes(result);
    }),
    { numRuns: 100 }
  );
});
```

```javascript
// Property 3: Normalisasi URL idempoten
// Feature: productivity-dashboard, Property 3: Normalisasi URL adalah idempoten
import * as fc from 'fast-check';
import { normalizeUrl } from '../src/linksModule.js';

test('Property 3: normalizeUrl idempoten', () => {
  fc.assert(
    fc.property(fc.webUrl(), (url) => {
      return normalizeUrl(url) === normalizeUrl(normalizeUrl(url));
    }),
    { numRuns: 100 }
  );
});
```

```javascript
// Property 10: Storage round-trip
// Feature: productivity-dashboard, Property 10: Data storage round-trip mempertahankan struktur
import * as fc from 'fast-check';
import { StorageUtil } from '../src/storageUtil.js';

test('Property 10: storage round-trip mempertahankan data', () => {
  const mockStorage = {};
  const mockLocalStorage = {
    getItem: (k) => mockStorage[k] ?? null,
    setItem: (k, v) => { mockStorage[k] = v; }
  };
  fc.assert(
    fc.property(
      fc.array(fc.record({
        id: fc.string({ minLength: 1 }),
        text: fc.string({ minLength: 1, maxLength: 200 }),
        done: fc.boolean(),
        createdAt: fc.integer({ min: 0 })
      })),
      (tasks) => {
        StorageUtil.set('test_key', tasks, mockLocalStorage);
        const result = StorageUtil.get('test_key', [], mockLocalStorage);
        return JSON.stringify(result) === JSON.stringify(tasks);
      }
    ),
    { numRuns: 100 }
  );
});
```

### Cakupan Unit Test (Example-Based)

| Area | Test Cases |
|---|---|
| `getGreeting` | Batas tepat: 05:00, 12:00, 15:00, 18:00, 04:59 |
| `formatTime` | Midnight (00:00), noon (12:00), invalid date |
| `formatDate` | Tanggal normal, tanggal awal tahun, invalid date |
| Timer state machine | idle→running→idle (Stop), idle→running→finished, finished→idle (Reset) |
| TodoModule | Tambah valid, tolak kosong, tolak spasi, tolak >200 char, edit valid, edit invalid, toggle, delete |
| LinksModule | Tambah valid, tolak nama kosong, tolak URL kosong, tolak nama >100, auto-prefix https, batas 20 |
| StorageUtil | Corrupt JSON, missing key, localStorage unavailable |

### Integrasi dan Smoke Test

- **Smoke test manual**: Buka `index.html` via `file://`, verifikasi semua komponen tampil dan interaktif dalam < 2 detik.
- **Cross-browser**: Test manual di Chrome, Firefox, Edge, Safari latest.
- **Responsive**: Test di viewport 320px, 768px, 1280px, 2560px menggunakan DevTools.
