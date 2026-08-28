# Requirements Document

## Introduction

Productivity Dashboard adalah sebuah web application berbasis HTML, CSS, dan Vanilla JavaScript yang berfungsi sebagai halaman tab baru yang produktif. Aplikasi ini menggabungkan empat fitur utama: Greeting dengan tampilan waktu dan tanggal, Focus Timer (Pomodoro-style 25 menit), To-Do List untuk manajemen tugas, dan Quick Links untuk akses cepat ke website favorit. Semua data disimpan menggunakan browser Local Storage sehingga tidak memerlukan backend server. Aplikasi dirancang dengan antarmuka yang bersih dan minimalis agar mudah digunakan tanpa konfigurasi yang rumit.

## Glossary

- **Dashboard**: Halaman utama aplikasi yang menampilkan semua komponen secara bersamaan.
- **Greeting_Component**: Komponen yang menampilkan sapaan berdasarkan waktu hari, serta tanggal dan waktu saat ini.
- **Focus_Timer**: Komponen timer Pomodoro 25 menit dengan kontrol Start, Stop, dan Reset.
- **TodoList**: Komponen manajemen tugas yang memungkinkan pengguna menambah, mengedit, menandai, dan menghapus task.
- **Task**: Satu item pekerjaan dalam TodoList yang memiliki teks dan status selesai/belum selesai.
- **QuickLinks**: Komponen yang menampilkan daftar tautan menuju website favorit yang dapat dikustomisasi.
- **Link**: Satu entri dalam QuickLinks yang terdiri dari nama tampilan dan URL.
- **Local_Storage**: Browser API yang digunakan untuk menyimpan data secara persisten di sisi klien.
- **Pomodoro**: Teknik manajemen waktu yang menggunakan interval kerja 25 menit.
- **Modern_Browser**: Browser versi terkini dari Chrome, Firefox, Edge, dan Safari.

---

## Requirements

### Requirement 1: Tampilan Waktu dan Sapaan (Greeting)

**User Story:** Sebagai pengguna, saya ingin melihat waktu, tanggal, dan sapaan yang relevan saat membuka dashboard, sehingga saya dapat langsung mengetahui konteks waktu dan merasa disambut dengan hangat.

#### Acceptance Criteria

1. THE Greeting_Component SHALL menampilkan waktu saat ini dalam format HH:MM menggunakan zona waktu lokal perangkat, diperbarui setiap detik.
2. THE Greeting_Component SHALL menampilkan tanggal saat ini dalam format hari, tanggal bulan tahun (contoh: Rabu, 27 Agustus 2025) menggunakan zona waktu lokal perangkat.
3. IF waktu lokal perangkat berada antara pukul 05:00 dan 11:59, THEN THE Greeting_Component SHALL menampilkan sapaan "Selamat Pagi".
4. IF waktu lokal perangkat berada antara pukul 12:00 dan 14:59, THEN THE Greeting_Component SHALL menampilkan sapaan "Selamat Siang".
5. IF waktu lokal perangkat berada antara pukul 15:00 dan 17:59, THEN THE Greeting_Component SHALL menampilkan sapaan "Selamat Sore".
6. IF waktu lokal perangkat berada antara pukul 18:00 dan 04:59, THEN THE Greeting_Component SHALL menampilkan sapaan "Selamat Malam".
7. WHEN tanggal sistem melewati tengah malam, THE Greeting_Component SHALL memperbarui tampilan tanggal secara otomatis tanpa perlu reload halaman.
8. IF sistem clock tidak tersedia atau gagal dibaca, THEN THE Greeting_Component SHALL menampilkan "--:--" untuk waktu dan "--" untuk tanggal tanpa menampilkan pesan error teknis kepada pengguna.

---

### Requirement 2: Focus Timer

**User Story:** Sebagai pengguna, saya ingin menggunakan timer 25 menit untuk sesi kerja fokus, sehingga saya dapat mengelola waktu dengan teknik Pomodoro.

#### Acceptance Criteria

1. THE Focus_Timer SHALL menampilkan durasi default 25:00 (dua puluh lima menit nol detik) ketika pertama kali dimuat.
2. WHEN pengguna menekan tombol Start, THE Focus_Timer SHALL mulai menghitung mundur dari waktu yang ditampilkan saat ini, satu detik setiap detiknya.
3. WHILE Focus_Timer sedang berjalan, THE Focus_Timer SHALL memperbarui tampilan waktu setiap satu detik.
4. WHEN pengguna menekan tombol Stop, THE Focus_Timer SHALL menghentikan countdown dan mempertahankan waktu tersisa yang ditampilkan.
5. WHEN pengguna menekan tombol Reset, THE Focus_Timer SHALL menghentikan countdown dan mengembalikan tampilan ke 25:00.
6. WHEN Focus_Timer mencapai 00:00, THE Focus_Timer SHALL menghentikan countdown secara otomatis dan mengubah tampilan timer menjadi warna berbeda (contoh: merah atau hijau) serta menampilkan teks "Waktu Habis!" selama minimal 3 detik.
7. WHILE Focus_Timer sedang berjalan, THE Focus_Timer SHALL menonaktifkan tombol Start agar tidak dapat ditekan kembali.
8. WHILE Focus_Timer dalam keadaan berhenti, belum pernah dimulai, atau sudah di-reset, THE Focus_Timer SHALL menonaktifkan tombol Stop agar tidak dapat ditekan.
9. WHEN Focus_Timer mencapai 00:00, THE Focus_Timer SHALL mengaktifkan kembali tombol Start sehingga pengguna dapat memulai sesi baru setelah melakukan Reset.

---

### Requirement 3: Manajemen To-Do List

**User Story:** Sebagai pengguna, saya ingin mengelola daftar tugas saya langsung dari dashboard, sehingga saya dapat melacak pekerjaan yang harus diselesaikan tanpa berpindah aplikasi.

#### Acceptance Criteria

1. WHEN halaman dimuat, THE TodoList SHALL menampilkan semua Task yang tersimpan di Local_Storage dalam urutan yang sama dengan urutan terakhir disimpan.
2. WHEN pengguna memasukkan teks maksimal 200 karakter dan menekan tombol tambah atau menekan tombol Enter, THE TodoList SHALL menambahkan Task baru dengan teks tersebut dan status belum selesai ke dalam daftar.
3. IF pengguna mencoba menambahkan Task dengan teks kosong, teks yang hanya berisi spasi, atau teks yang melebihi 200 karakter, THEN THE TodoList SHALL menolak penambahan dan tidak membuat Task baru.
4. WHEN pengguna menekan tombol edit pada sebuah Task, THE TodoList SHALL menampilkan teks Task tersebut dalam mode input yang dapat diubah.
5. WHEN pengguna menyimpan hasil edit dengan menekan tombol simpan atau menekan Enter, dan teks baru valid (tidak kosong, tidak hanya spasi, tidak melebihi 200 karakter), THE TodoList SHALL memperbarui teks Task dengan teks baru tersebut.
6. IF pengguna menyimpan hasil edit dengan teks kosong, teks yang hanya berisi spasi, atau teks yang melebihi 200 karakter, THEN THE TodoList SHALL membatalkan perubahan dan mengembalikan teks Task ke nilai sebelumnya.
7. WHEN pengguna menekan tombol batal atau menekan tombol Escape saat dalam mode edit, THE TodoList SHALL membatalkan perubahan dan mengembalikan teks Task ke nilai sebelumnya tanpa menyimpan apapun.
8. WHEN pengguna menandai sebuah Task sebagai selesai, THE TodoList SHALL menampilkan teks Task tersebut dengan garis coret (strikethrough) dan opacity 50%.
9. WHEN pengguna menandai kembali sebuah Task yang sudah selesai, THE TodoList SHALL menghapus garis coret dan mengembalikan opacity ke 100%, menandakan Task kembali ke status belum selesai.
10. WHEN pengguna menghapus sebuah Task, THE TodoList SHALL menghapus Task tersebut dari daftar secara permanen.
11. WHEN terjadi perubahan pada daftar Task (tambah, edit, tandai, atau hapus), THE TodoList SHALL menyimpan keseluruhan daftar Task ke Local_Storage secara otomatis.
12. THE TodoList SHALL menggunakan satu kunci penyimpanan tetap di Local_Storage (tidak berubah selama siklus hidup aplikasi) yang tidak dibagikan dengan komponen lain.
13. IF Local_Storage tidak dapat diakses saat halaman dimuat, THEN THE TodoList SHALL menampilkan daftar Task kosong tanpa menampilkan pesan error teknis kepada pengguna.

---

### Requirement 4: Quick Links

**User Story:** Sebagai pengguna, saya ingin menyimpan dan mengakses tautan ke website favorit saya langsung dari dashboard, sehingga saya dapat membuka website yang sering dikunjungi dengan cepat.

#### Acceptance Criteria

1. THE QuickLinks SHALL menampilkan semua Link yang tersimpan di Local_Storage saat halaman dimuat.
2. WHEN pengguna mengisi nama dan URL, lalu menekan tombol tambah Link, THE QuickLinks SHALL menambahkan Link baru ke dalam daftar dan menyimpan daftar terbaru ke Local_Storage secara otomatis.
3. IF pengguna mencoba menambahkan Link dengan nama atau URL yang kosong, THEN THE QuickLinks SHALL menolak penambahan, tidak membuat Link baru, dan menampilkan pesan kesalahan yang menunjukkan field mana yang kosong.
4. IF pengguna mencoba menambahkan Link dengan nama yang melebihi 100 karakter, THEN THE QuickLinks SHALL menolak penambahan dan menampilkan pesan kesalahan yang menunjukkan batas maksimal karakter.
5. IF pengguna memasukkan URL yang tidak dimulai dengan "http://" atau "https://", THEN THE QuickLinks SHALL menambahkan awalan "https://" secara otomatis sebelum menyimpan Link.
6. WHEN pengguna mengklik sebuah Link, THE QuickLinks SHALL membuka URL yang tersimpan di tab baru browser.
7. WHEN pengguna menghapus sebuah Link, THE QuickLinks SHALL menghapus Link tersebut dari daftar secara permanen dan menyimpan daftar terbaru ke Local_Storage secara otomatis.
8. THE QuickLinks SHALL memuat data Link dari Local_Storage menggunakan kunci penyimpanan yang unik dan konsisten untuk mencegah konflik data dengan komponen lain.
9. IF total jumlah Link yang tersimpan telah mencapai 20 Link, THEN THE QuickLinks SHALL menolak penambahan Link baru dan menampilkan pesan yang menginformasikan batas maksimal telah tercapai.

---

### Requirement 5: Persistensi Data dan Pemulihan State

**User Story:** Sebagai pengguna, saya ingin data saya tetap tersimpan saat saya menutup dan membuka kembali browser, sehingga saya tidak perlu memasukkan data berulang kali.

#### Acceptance Criteria

1. WHEN pengguna memuat ulang atau membuka kembali halaman Dashboard, THE Dashboard SHALL memulihkan seluruh data Task (jumlah, teks, status) dan Link (URL, nama) dari Local_Storage sesuai dengan kondisi terakhir sebelum halaman ditutup.
2. WHEN terjadi perubahan pada data Task atau Link, THE Dashboard SHALL menyimpan data terbaru ke Local_Storage sebelum operasi dianggap selesai.
3. IF Local_Storage tidak mengandung data Task, THEN THE TodoList SHALL menampilkan daftar Task kosong tanpa menampilkan pesan error kepada pengguna.
4. IF Local_Storage tidak mengandung data Link, THEN THE QuickLinks SHALL menampilkan daftar Link kosong tanpa menampilkan pesan error kepada pengguna.
5. IF data yang tersimpan di Local_Storage rusak atau tidak dapat di-parse (contoh: format JSON tidak valid), THEN THE Dashboard SHALL menggunakan data default kosong dan tidak menampilkan pesan error teknis seperti nama fungsi, stack trace, atau kode error kepada pengguna.

---

### Requirement 6: Kompatibilitas dan Performa

**User Story:** Sebagai pengguna, saya ingin dashboard bekerja dengan lancar di browser modern yang saya gunakan, sehingga saya dapat menggunakannya tanpa hambatan teknis.

#### Acceptance Criteria

1. THE Dashboard SHALL berfungsi tanpa JavaScript error, tanpa rendering yang rusak, dan tanpa fitur yang tidak dapat diakses pada versi stable terbaru dari Chrome, Firefox, Edge, dan Safari.
2. WHEN halaman dimuat, THE Dashboard SHALL menampilkan seluruh antarmuka dalam keadaan fully interactive (semua tombol, input, dan link dapat digunakan) dalam waktu kurang dari 2 detik pada koneksi lokal.
3. WHEN pengguna berinteraksi dengan komponen apapun (menambah Task, mengklik Link, mengoperasikan timer), THE Dashboard SHALL menampilkan perubahan visual yang terlihat dalam waktu kurang dari 100 milidetik setelah aksi pengguna.
4. THE Dashboard SHALL menampilkan seluruh konten dan kontrol yang dapat digunakan tanpa scroll horizontal pada layar dengan lebar minimal 320 piksel hingga 2560 piksel.
5. WHEN Dashboard dijalankan melalui protokol file://, THE Dashboard SHALL memuat seluruh aset, mengaktifkan semua fitur, dan mengakses data Local_Storage yang tersimpan tanpa memerlukan server.
