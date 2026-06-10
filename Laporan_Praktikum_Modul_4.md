# Laporan Praktikum Modul 4: PostgreSQL & Database Management

**Disusun oleh:**
- Muhammad Mahdavikia Abdillah (31246000032)
- Diaz Raharjo Muliasmara (31246000042)
- Panji Nayaka Reswara (3124600057)

**Kelas:** 2 D4 IT B

**Dosen Pembimbing:**
Dr Ferry Astika Saputra ST, M.Sc

---

**MATA KULIAH WORKSHOP ADMINISTRASI JARINGAN**
**DEPARTEMEN TEKNIK INFORMATIKA & KOMPUTER**
**POLITEKNIK ELEKTRONIKA NEGERI SURABAYA**

---

## A. Pra Lab

### Soal 1

Apa fungsi file/folder /docker-entrypoint-initdb.d/ di image PostgreSQL?

Jawab:
   Folder /docker-entrypoint-initdb.d/ adalah direktori khusus untuk mengeksekusi skrip SQL atau Shell secara otomatis untuk membangun skema, membuat pengguna, atau mengisi data awal (seeding) sebelum database mulai melayani koneksi. Sistem ini hanya berjalan satu kali pada proses inisialisasi volume data yang kosong, guna memastikan bahwa konfigurasi dasar database terbentuk secara konsisten tanpa intervensi manual.

### Soal 2

Mengapa POSTGRES\_PASSWORD wajib diset? Apa risikonya jika tidak ada password?

Jawab:
   Variabel POSTGRES\_PASSWORD wajib disetel untuk mengaktifkan skema otentikasi dasar yang melindungi akses administratif ke database. Tanpa kata sandi, PostgreSQL akan berada dalam kondisi terbuka, sehingga siapa pun yang memiliki akses ke jaringan tersebut dapat masuk sebagai superuser untuk mencuri, memodifikasi, atau menghapus seluruh data secara permanen.

### Soal 3

Jelaskan perbedaan antara pg\_dump format custom (-Fc) dan format SQL plain text.

Jawab:
   Format plain text menghasilkan file teks mentah berisi perintah SQL yang dapat dibaca manusia, sehingga mudah diedit secara manual dan bisa dijalankan pada klien database apa pun tanpa alat khusus; namun, ukuran filenya cenderung besar dan proses pemulihannya lebih lambat karena harus dibaca baris demi baris.
   format custom (-Fc) adalah format biner yang dikompresi secara otomatis, menghasilkan ukuran file yang jauh lebih kecil dan efisien dalam penyimpanan. Keunggulan utamanya adalah kemampuan untuk dipulihkan secara paralel menggunakan pg\_restore, serta fleksibilitas untuk memilih tabel tertentu saja yang ingin dipulihkan dari satu file cadangan yang sama.

### Soal 4

Apa itu shared\_buffers dan mengapa perlu disesuaikan untuk container?

Jawab:
   parameter konfigurasi PostgreSQL yang menentukan jumlah memori (RAM) yang didedikasikan untuk menyimpan data dari tabel dan indeks secara sementara agar lebih cepat diakses tanpa harus terus-menerus membaca dari disk.Penyesuaian parameter ini sangat krusial di lingkungan container karena batasan memori yang dialokasikan pada Docker seringkali berbeda dengan spesifikasi server fisik. Jika nilai shared\_buffers dibiarkan pada angka bawaan yang biasanya sangat kecil akan mengakibatkan performa menjadi lambat. Sebaliknya, menyetel nilai yang terlalu besar tanpa memperhitungkan batas memori container dapat memicu Out Of Memory (OOM) Killer dari sistem operasi yang akan menghentikan paksa container tersebut

### Soal 5

Mengapa data PostgreSQL harus disimpan di Docker Volume, bukan di container layer?

Jawab:
   Data PostgreSQL harus disimpan di Docker Volume karena container layer bersifat sementara (ephemeral), di mana semua data yang tersimpan di dalamnya akan terhapus secara permanen jika container dihentikan atau diperbarui. Dengan menggunakan Docker Volume, data dipisahkan dari siklus hidup container sehingga tetap aman tersimpan di host meskipun container dihapus atau diganti dengan versi image yang lebih baru.

---

## Screenshot

### 1. docker compose ps â€” db dan pgadmin running \+ healthy

![image1](image/Laporan%20Modul%204/image1.png)

### 2. psql connect \+ \\dt app.\* â€” list tabel

![image2](image/Laporan%20Modul%204/image2.png)

### 3. SELECT \* FROM app.mahasiswa â€” data sample

![image3](image/Laporan%20Modul%204/image3.png)

### 4. Query JOIN nilai â€” output tabel gabungan

![image4](image/Laporan%20Modul%204/image4.png)

### 5. pgAdmin4 login \+ server connection

![image5](image/Laporan%20Modul%204/image5.png)

### 6. pgAdmin4 â€” tabel view/edit data

![image6](image/Laporan%20Modul%204/image6.png)

### 7. pg\_dump output â€” backup file terbuat

![image7](image/Laporan%20Modul%204/image7.png)

### 8. pg\_restore \+ SELECT â€” data berhasil di-restore

![image8](image/Laporan%20Modul%204/image8.png)

### 9. pg\_stat\_activity â€” koneksi aktif

![image9](image/Laporan%20Modul%204/image9.png)

### 10. PostgreSQL log â€” isi log file

![image10](image/Laporan%20Modul%204/image10.png)

---

## B. Post Lab

### Soal 1

Jalankan docker compose down lalu docker compose up \-d. Apakah data mahasiswa masih ada? Buktikan.

Jawab:
   Data mahasiswa tetap ada, dengan catatan kamu telah memetakan direktori data PostgreSQL ke sebuah Docker Volume dalam file konfigurasi docker-compose.yml milikmu.
   Hal ini terjadi karena perintah docker compose down memang menghentikan dan menghapus container, namun ia tidak menghapus volume yang terikat padanya kecuali kamu menambahkan flag khusus (-v).
   **![image11](image/Laporan%20Modul%204/image11.png)**

### Soal 2

Jalankan docker compose down \-v lalu docker compose up \-d. Apa yang terjadi? Apakah init script dijalankan ulang?

![image12](image/Laporan%20Modul%204/image12.png)

### Soal 3

Bandingkan ukuran file backup format custom vs SQL. Mana yang lebih kecil dan mengapa?

![image13](image/Laporan%20Modul%204/image13.png)
   Jawab:
   Format yang lebih kecil sql
   Alasan:
   Format Custom memiliki overhead metadata (informasi tambahan untuk restore) yang ukurannya lebih besar daripada penghematan kompresinya jika isi datanya masih sedikit/kecil. Sementara format SQL hanya berisi teks perintah murni tanpa tambahan data biner tersebut.

### Soal 4

Buat query yang menampilkan mahasiswa yang belum memiliki nilai di semester apapun.

Jawab:
   **![image14](image/Laporan%20Modul%204/image14.png)**

### Soal 5

Jelaskan peran user app\_reader yang dibuat di init script. Apa bedanya dengan labuser?

Jawab:
   Peran app\_reader:
   User ini berperan sebagai pengguna dengan hak akses terbatas (Read-Only). Fungsinya adalah untuk membaca data dari database tanpa bisa mengubah, menambah, atau menghapus data tersebut (hanya memiliki izin SELECT). Biasanya digunakan oleh aplikasi pelaporan atau data visualization demi keamanan data.
    Peran labuser:
   User ini umumnya berperan sebagai Owner/Superuser atau pengguna dengan hak akses penuh (Read-Write). labuser memiliki izin untukmelakukan operasi DML (Data Manipulation Language) dan DDL (Data Definition Language) secara lengkap, seperti INSERT, UPDATE, DELETE, hingga CREATE TABLE. User ini bertanggung jawab penuh atas manajemen struktur dan isi database.

---

<!-- Gambar -->

