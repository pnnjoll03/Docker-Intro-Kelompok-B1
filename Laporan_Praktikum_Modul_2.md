# Laporan Praktikum Modul 2: Docker Networking & Volume

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

Apa perbedaan default bridge dan user-defined bridge network?

Secara mendasar, ada dua jenis bridge yang sering digunakan: default bridge (biasanya bernama bridge) dan user-defined bridge (yang Anda buat sendiri).

   Berikut adalah perbedaan utama di antara keduanya:

1. DNS Resolution (Resolusi Nama)
- Default Bridge: Kontainer hanya bisa berkomunikasi satu sama lain melalui alamat IP. Jika Anda ingin menghubungkan dua kontainer, Anda harus mengetahui IP masing-masing secara spesifik atau menggunakan opsi \--link (yang sekarang sudah dianggap usang/deprecated).
- User-defined Bridge: Memiliki fitur automatic service discovery. Anda bisa memanggil kontainer lain cukup dengan menggunakan nama kontainer-nya sebagai hostname. Docker secara otomatis akan menerjemahkan nama tersebut menjadi alamat IP yang tepat.
2. Isolasi Keamanan
- Default Bridge: Semua kontainer yang tidak menentukan jaringan saat dijalankan akan dimasukkan ke dalam bridge yang sama. Hal ini bisa berisiko karena aplikasi yang tidak berhubungan secara fungsional tetap berada dalam satu jaringan yang sama.
- User-defined Bridge: Memberikan isolasi yang lebih baik. Hanya kontainer yang secara eksplisit Anda masukkan ke dalam jaringan tersebut yang bisa saling berkomunikasi. Ini mengikuti prinsip *least privilege* dalam keamanan jaringan.
3. Konfigurasi On-the-fly
- Default Bridge: Konfigurasi jaringan ini cenderung kaku. Untuk mengubah pengaturan pada default bridge (seperti MTU atau aturan iptables), Anda seringkali harus me-restart Docker secara keseluruhan.
- User-defined Bridge: Anda bisa membuat, menghapus, dan mengonfigurasi ulang jaringan kapan saja tanpa memengaruhi kontainer yang sedang berjalan di jaringan lain. Anda juga bisa menghubungkan atau memutuskan kontainer dari jaringan ini secara *real-time*.

### Soal 2

Kapan menggunakan Volume vs Bind Mount vs tmpfs?

1. Volumes (Paling Disarankan)Data disimpan di bagian sistem file host yang dikelola sepenuhnya oleh Docker (/var/lib/docker/volumes/).
- Kapan digunakan:
* Berbagi data antar beberapa kontainer yang sedang berjalan.
* Menyimpan data database (MySQL, PostgreSQL, dll) agar tidak hilang saat kontainer dihapus.
* Saat Anda ingin performa I/O yang lebih baik di Docker Desktop (Windows/Mac).
* Ketika Anda ingin mencadangkan (*backup*) atau memindahkan data dengan mudah ke host lain.
2. Bind Mounts, Memetakan folder atau file spesifik dari host (misal: /home/user/project) langsung ke dalam kontainer.
- Kapan digunakan:
* Development: Berbagi kode sumber dari komputer Anda ke kontainer agar perubahan kode langsung terlihat tanpa *rebuild* image (seperti saat menggunakan Hot Reload).
* Berbagi konfigurasi file sistem dari host ke kontainer (misal: /etc/resolv.conf).
* Ketika Anda butuh struktur direktori host yang persis sama dengan yang ada di dalam kontainer.
3. tmpfs Mounts, Data disimpan di memori (RAM) host, bukan di disk (SSD/HDD). Data akan langsung terhapus jika kontainer dimatikan.
- Kapan digunakan:
1. Keamanan: Menyimpan data sensitif (seperti *password* atau *keys*) yang tidak boleh tersimpan secara permanen di disk.
2. Performa Tinggi: Jika aplikasi butuh menulis/membaca file sementara dengan sangat cepat.
3. Menghindari beban tulis pada disk yang bisa memperpendek umur media penyimpanan (seperti pada SD Card di Raspberry Pi).

### Soal 3

Apa yang terjadi pada named volume saat docker compose down? Bagaimana jika pakai flag \-v?

1. Standar: docker compose down

   Secara default, perintah ini hanya menghapus:

- Kontainer yang didefinisikan dalam file Compose.
- Jaringan (Networks) yang dibuat oleh Compose.
- Image (hanya jika menggunakan flag tertentu, tapi standarnya tidak).

  Apa yang terjadi pada Named Volume? Volume tidak akan dihapus. Data Anda tetap aman tersimpan di dalam direktori internal Docker. Saat Anda menjalankan docker compose up lagi, kontainer baru akan langsung tersambung kembali ke volume tersebut dan semua data lama (seperti database atau file upload) tetap tersedia.

2. Menggunakan Flag: docker compose down \-v

   Flag \-v atau \--volumes adalah instruksi "bersih-bersih total". Perintah ini akan menghapus:

- Semua yang dihapus oleh perintah standar (kontainer & network).
- Semua Named Volumes yang didefinisikan di dalam file docker-compose.yml.

  Apa dampaknya? Semua data yang tersimpan di dalam volume tersebut akan terhapus secara permanen dari sistem host. Jika di dalamnya ada data database yang belum di-backup, data tersebut akan hilang dan tidak bisa dikembalikan.

### Soal 4

Apa fungsi depends\_on dan healthcheck di docker-compose.yml?

- depends\_on (Urutan Antrian)

  Fungsinya cuma buat ngatur siapa yang nyala duluan.

* Logika: "Jalankan DB dulu, baru jalankan Backend."
* Masalah: Docker cuma tau DB sudah running (prosesnya nyala), tapi dia nggak tau kalau DB-nya masih *loading* internal. Backend bisa *error* karena DB belum benar-benar siap menerima koneksi.
- healthcheck (Status Kesehatan)

  Fungsinya buat mastiin aplikasi di dalam kontainer sudah siap pakai.

* Logika: Docker bakal ngetes secara berkala (misal tiap 10 detik) pakai perintah seperti curl atau pg\_isready.
* Status: Kontainer bakal punya label khusus: starting \-\> healthy atau unhealthy.

### Soal 5

Mengapa user-defined bridge bisa DNS resolve nama container, sedangkan default bridge tidak?

1. Desain Docker Engine (Embedded DNS), Docker memiliki fitur bernama Embedded DNS Server. Server DNS internal ini bertugas memetakan nama kontainer ke alamat IP secara otomatis.
- User-defined Bridge: Saat Anda membuat network sendiri, Docker secara otomatis mengaktifkan *Embedded DNS* ini di dalam jaringan tersebut. Setiap kontainer yang bergabung akan didaftarkan ke DNS server ini.
- Default Bridge: Jaringan bridge bawaan dibuat sejak versi awal Docker sebelum fitur DNS internal ini ada. Untuk menjaga aplikasi lama tetap berjalan tanpa perubahan perilaku, Docker tidak mengaktifkan DNS resolver pada jaringan ini.
2. Mekanisme Resolusi Nama
- Pada User-defined: Docker bertindak sebagai DNS resolver. Ketika kontainer app memanggil db, ia bertanya ke DNS internal Docker (biasanya di IP 127.0.0.11), dan Docker menjawab dengan IP kontainer db.
- Pada Default: Karena tidak ada DNS resolver, satu-satunya cara kontainer saling mengenali nama adalah melalui file /etc/hosts. Dulu, ini dilakukan secara manual atau menggunakan flag \--link yang sifatnya statis (tidak otomatis terupdate jika kontainer di-restart dengan IP baru).
3. Keamanan dan Isolasi, Docker sengaja memisahkan perilaku ini untuk mendorong pengguna menggunakan User-defined Bridge.
- Di Default Bridge, semua kontainer dari berbagai project bisa saling melihat jika DNS diaktifkan secara global. Ini kurang aman.
- Di User-defined Bridge, DNS hanya bekerja di dalam lingkup jaringan tersebut. Jadi, kontainer web di Network A tidak bisa sembarangan memanggil kontainer db di Network B menggunakan nama, karena mereka berada di "buku telepon" (DNS zone) yang berbeda.

---

## Screenshot

### 1. docker network ls

![image1](image/Laporan%20Modul%202/image1.png)

### 2. ping antar container by nama

![image2](image/Laporan%20Modul%202/image2.png)

### 3. docker volume ls \+ inspect

![image3](image/Laporan%20Modul%202/image3.png)

### 4. Screenshot 4

![image4](image/Laporan%20Modul%202/image4.png)

### 5. Screenshot 5

![image5](image/Laporan%20Modul%202/image5.png)

### 6. Volume sharing antar container

![image6](image/Laporan%20Modul%202/image6.png)

### 7. **Sebelum :

![image7](image/Laporan%20Modul%202/image7.png)

### 8. **Sesudah :

![image8](image/Laporan%20Modul%202/image8.png)

### 9. tmpfs inspect

![image9](image/Laporan%20Modul%202/image9.png)

### 10. docker compose ps

![image10](image/Laporan%20Modul%202/image10.png)

### 11. Screenshot 11

![image11](image/Laporan%20Modul%202/image11.png)

### 12. Screenshot 12

![image12](image/Laporan%20Modul%202/image12.png)

### 13. Screenshot 13

![image13](image/Laporan%20Modul%202/image13.png)

---

## B. Post Lab

### Soal 1

Jalankan docker network inspect lab-frontend. Sebutkan container dan IP masing-masing.

**![image14](image/Laporan%20Modul%202/image14.png)**

### Soal 2

Hapus container lab-db lalu docker compose up \-d lagi. Apakah data PostgreSQL masih ada? Mengapa?

Apakah data PostgreSQL masih ada? **Ya, data tetap ada (aman). Meskipun kontainernya kamu hapus dan buat ulang, database tidak akan kembali kosong.** Mengapa? (Alasan Teknis)

1. **Pemisahan Data (Decoupling) Dalam file docker-compose.yml di modul tersebut, data PostgreSQL disimpan menggunakan Named Volume. Artinya, data tidak disimpan di dalam kontainer, melainkan di folder khusus di sistem operasi (Host) yang dikelola oleh Docker.**
2. **Siklus Hidup (Life-Cycle)**
1. **Kontainer: Bersifat sementara. Kalau dihapus, semua isinya hilang.**
2. **Volume: Bersifat permanen. Ia tidak ikut terhapus meskipun kontainer yang menggunakannya dihancurkan.**
3. **Mounting Otomatis Saat kamu menjalankan docker compose up \-d kembali, Docker akan melihat ada volume bernama db\_data (atau sejenisnya). Docker kemudian secara otomatis mencolokkan kembali (mounting) volume tersebut ke kontainer yang baru.**

### Soal 3

Tunjukkan perbedaan output docker inspect untuk mount type volume vs bind.

**Ini untuk volume**

   **![image15](image/Laporan%20Modul%202/image15.png)**

   **Ini untuk bind![image16](image/Laporan%20Modul%202/image16.png)**

   **Perbedannya untuk volume itu ngarahnya ke docker dan bind itu nyimpen ke laptop saya**

### Soal 4

Jelaskan alur request dari browser â†’ Nginx â†’ Flask â†’ PostgreSQL.

1\. Browser â†’ Nginx (Reverse Proxy)

- **Aksi: Pengguna memasukkan alamat URL (misalnya http://localhost) di browser.**
- **Proses: Browser mengirimkan HTTP Request ke port yang dibuka oleh host (biasanya port 80 atau 443).**
- **Peran Nginx: Nginx bertindak sebagai Reverse Proxy atau pintu gerbang utama. Nginx menerima request tersebut dan meneruskannya (forward) ke layanan backend (Flask).**
- **Kenapa pakai Nginx? Untuk keamanan (menyembunyikan port asli Flask), efisiensi (menangani static files), dan load balancing.**

  2\. Nginx â†’ Flask (Application Server)

- Proses: Nginx meneruskan paket data ke kontainer Flask melalui jaringan internal Docker (biasanya menggunakan port 5000 atau melalui protokol WSGI seperti Gunicorn/uWSGI).
- Peran Flask: Flask adalah Application Tier yang berisi logika bisnis. Flask akan melihat rute (route) mana yang diminta (misalnya /data atau /login).
- Eksekusi: Python menjalankan kode sesuai endpoint yang dipanggil. Jika aplikasi memerlukan data dari database, Flask akan menyiapkan query SQL.

  3\. Flask â†’ PostgreSQL (Database)

- Proses: Flask mengirimkan permintaan data (melalui library seperti psycopg2 atau SQLAlchemy) ke kontainer PostgreSQL pada port 5432 di dalam jaringan internal Docker.
- Peran PostgreSQL: Sebagai Data Tier, PostgreSQL menerima query, mencari atau memproses data pada storage (yang biasanya dihubungkan dengan Docker Volume agar data tidak hilang), dan mengirimkan hasilnya kembali ke Flask.

  4\. Alur Balik (Response), Setelah data didapat:

1. PostgreSQL mengirimkan hasil query ke Flask.
2. Flask mengolah data tersebut (misalnya mengubahnya menjadi JSON atau merender template HTML) dan mengirimkannya kembali ke Nginx.
3. Nginx mengirimkan HTTP Response akhir tersebut kembali ke Browser pengguna.
4. Browser menampilkan hasil akhirnya kepada pengguna.

### Soal 5

Bandingkan ukuran image yang digunakan stack ini. Mana terbesar dan mengapa?

Dalam stack yang dijelaskan pada modul tersebut, image PostgreSQL biasanya menjadi yang paling besar dibandingkan dengan Nginx maupun Flask (berbasis Python-Alpine atau Python-Slim). Besarnya ukuran image PostgreSQL disebabkan karena ia membawa seluruh sistem manajemen database relasional yang kompleks, termasuk mesin pemrosesan query, sistem logging, pustaka manajemen memori, serta dependensi sistem operasi yang cukup lengkap untuk menjamin integritas data. Sementara itu, image Nginx umumnya sangat ringan karena hanya difokuskan pada pengiriman file statis dan proxying, dan image Flask dapat dijaga ukurannya tetap kecil jika menggunakan basis OS minimalis seperti Alpine Linux yang hanya menyertakan pustaka Python esensial. Perbedaan ukuran ini mencerminkan fungsi masing-masing: Nginx sebagai gerbang yang ramping, Flask sebagai logika aplikasi yang spesifik, dan PostgreSQL sebagai infrastruktur penyimpanan data yang masif dan fitur-lengkap.

---

<!-- Gambar -->

