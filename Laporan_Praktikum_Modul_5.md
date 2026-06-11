# Laporan Praktikum Modul 5: Centralized Logging dengan Fluent Bit

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

Mengapa centralized logging penting di lingkungan container?

Di lingkungan container yang dinamis dan terdistribusi seperti Docker atau Kubernetes, centralized logging bukan sekadar fitur tambahan, melainkan sebuah kebutuhan krusial.

   alasan utama mengapa hal ini sangat penting:

1. Masalah Efemeritas (Sifat Sementara Container)Container bersifat *ephemeral*, artinya jika sebuah container *crash*, dihapus, atau di-*restart*, semua data di dalamnyaâ€”termasuk file log lokalâ€”akan ikut hilang secara permanen. Tanpa centralized logging, kamu tidak akan pernah tahu penyebab container tersebut mati karena "kotak hitam" rekam medisnya sudah lenyap.
2. Skalabilitas dan Fragmentasi Log , Dalam arsitektur mikroservis, satu aplikasi bisa berjalan di puluhan container yang tersebar di host berbeda. Sangat tidak efisien jika seorang developer harus masuk ke satu per satu container (docker logs) hanya untuk mencari jejak error. Centralized logging mengumpulkan semua data tersebut ke satu tempat (seperti PostgreSQL yang kamu buat), sehingga pencarian bisa dilakukan dalam satu jendela query.
3.  Korelasi Event Antar-Service, Sebuah error di satu service seringkali disebabkan oleh kegagalan di service lain. Dengan log yang terpusat dan memiliki *timestamp* yang sinkron, kamu bisa melihat urutan kejadian secara kronologis di seluruh ekosistem aplikasi. Misalnya, kamu bisa melihat bahwa payment-service error tepat satu milidetik setelah database-service mengalami *timeout*.4
4. Analisis dan Monitoring Real-Time, Data log dalam bentuk teks mentah sulit diolah. Dengan menarik log ke database pusat, kamu bisa menggunakan alat visualisasi (seperti Metabase atau Grafana) untuk membuat dashboard. Kamu bisa melihat tren jumlah error per jam, mengidentifikasi container mana yang paling sering bermasalah, dan mendeteksi anomali sebelum sistem benar-benar tumbang.
5.  Keamanan dan Audit, Log adalah aset penting untuk audit keamanan. Jika terjadi upaya peretasan, peretas mungkin mencoba menghapus log lokal untuk menghilangkan jejak. Dengan mengirimkan log secara langsung ke server terpisah yang aman, kamu memiliki salinan permanen yang tidak bisa dimanipulasi dari dalam container yang terkompromi.

### Soal 2

Apa perbedaan antara Docker logging driver json-file dan fluentd?

Perbedaan utama antara driver json-file dan fluentd terletak pada cara penyimpanan dan aksesibilitas datanya. Driver json-file adalah standar bawaan Docker yang menyimpan log dalam bentuk file lokal di dalam storage host, sehingga log akan ikut terhapus jika container atau volume terkait dihilangkan dan sulit dianalisis secara kolektif jika terdapat banyak container. Sebaliknya, driver fluentd bertindak sebagai pengirim (*shipper*) yang langsung meneruskan aliran log ke server Fluent-bit atau Fluentd melalui jaringan, sehingga memungkinkan penyimpanan data yang terpusat, persisten, dan mudah diintegrasikan dengan database eksternal seperti PostgreSQL untuk kebutuhan analisis jangka panjang. Sementara json-file lebih cocok untuk pengembangan lokal yang sederhana karena minim konfigurasi, fluentd jauh lebih unggul untuk lingkungan produksi atau tugas modul ini karena mendukung skalabilitas, keamanan data melalui sentralisasi, serta kemampuan untuk memproses metadata container secara otomatis.

### Soal 3

Jelaskan keuntungan menyimpan log di database (PostgreSQL) vs file text.

Menyimpan log di PostgreSQL jauh lebih unggul dibanding file teks karena log jadi bisa di-query menggunakan SQL, sehingga mencari error spesifik di ribuan baris cuma butuh hitungan detik. Selain itu, data log tersusun rapi dalam kolom (structured), memudahkan kita membuat ringkasan statistik seperti jumlah error per jam. Database juga siap divisualisasikan ke dashboard seperti Metabase, serta memiliki fitur auto-cleanup otomatis lewat script, sehingga kamu nggak perlu pusing folder penyimpanan penuh seperti kalau pakai file teks biasa.

### Soal 4

Apa itu structured logging dan mengapa lebih baik daripada plain text log?

Structured logging itu mencatat log dalam format data (seperti JSON), bukan sekadar kalimat cerita (plain text).Kenapa lebih baik?

1. Gampang di-filter: Kamu bisa langsung cari berdasarkan kategori (misal: cari yang user\_id tertentu saja) tanpa ribet.
2. Siap diolah mesin: Komputer nggak perlu "menebak" isi kalimatmu; data langsung masuk ke kolom database dengan rapi.
3. Konsisten: Semua log punya format yang sama, jadi nggak ada lagi beda format antar programmer.

### Soal 5

Mengapa Fluent Bit lebih cocok untuk sidecar/edge collection dibanding Fluentd?

- Jauh Lebih Ringan: Fluent Bit cuma makan RAM sekitar 1â€“2 MB, sedangkan Fluentd butuh 30â€“50 MB. Kalau kamu punya banyak container, Fluent Bit jauh lebih hemat resource.
- Lebih Kencang: Karena dibuat pakai bahasa C, Fluent Bit punya performa lebih tinggi dan CPU yang lebih rendah dibanding Fluentd yang berbasis Ruby.
- Tanpa Ribet: Fluent Bit nggak butuh dependensi tambahan (seperti Ruby gems), jadi ukuran container-nya sangat kecil dan cocok banget buat jadi sidecar (kolektor log yang nempel di tiap aplikasi).

---

## Screenshot

### 1. docker compose ps â€” 5 service running

![image1](image/Laporan%20Modul%205/image1.png)

### 2. docker compose logs fluent-bit â€” Fluent Bit menerima log

![image2](image/Laporan%20Modul%205/image2.png)

### 3. Screenshot 3

![image3](image/Laporan%20Modul%205/image3.png)

### 4. Screenshot 4

![image4](image/Laporan%20Modul%205/image4.png)

### 5. Screenshot 5

![image5](image/Laporan%20Modul%205/image5.png)

### 6. Screenshot 6

![image6](image/Laporan%20Modul%205/image6.png)

### 7. Screenshot 7

![image7](image/Laporan%20Modul%205/image7.png)

### 8. Screenshot 8

![image8](image/Laporan%20Modul%205/image8.png)

### 9. Screenshot 9

![image9](image/Laporan%20Modul%205/image9.png)

### 10. Screenshot 10

![image10](image/Laporan%20Modul%205/image10.png)

### 11. Screenshot 11

![image11](image/Laporan%20Modul%205/image11.png)

### 12. Screenshot 12

![image12](image/Laporan%20Modul%205/image12.png)

---

## B. Post Lab

### Soal 1

Berapa total log yang masuk ke PostgreSQL setelah 5 menit? Tunjukkan distribusi per container dan per level.

**Untuk total log yang masuk setealh 5 menit sebanyak 99 log![image13](image/Laporan%20Modul%205/image13.png)**

   **Distribusi per container ![image14](image/Laporan%20Modul%205/image14.png)**

   **Distribusi per level![image15](image/Laporan%20Modul%205/image15.png)**

### Soal 2

Tulis query SQL yang menampilkan log rate per menit selama 10 menit terakhir.

**![image16](image/Laporan%20Modul%205/image16.png)**

### Soal 3

Apa yang terjadi jika container fluent-bit di-stop? Apakah container lain juga stop? Apakah log hilang?

Jika container fluent-bit di-stop, container aplikasi lainnya akan tetap berjalan karena Docker memperlakukan proses logging secara terpisah dari lifecycle container. Namun, performa aplikasi bisa terganggu atau bahkan mengalami *hang* jika Docker dikonfigurasi dalam mode *blocking* karena ia akan terus menunggu konfirmasi pengiriman log yang tidak kunjung dijawab oleh Fluent-bit. Terkait data, log yang tercipta selama Fluent-bit mati kemungkinan besar akan hilang karena Docker daemon tidak memiliki penyimpanan sementara yang persisten untuk menampung antrean log tersebut. Data log baru akan kembali terekam dan tersimpan di PostgreSQL hanya setelah container Fluent-bit dijalankan kembali dan berhasil melakukan *handshake* ulang dengan driver logging Docker.

### Soal 4

Jelaskan alur sebuah log entry dari log-generator stdout sampai masuk ke tabel container\_logs.

Alur perjalanan sebuah log entry dari log-generator hingga mendarat di tabel container\_logs di PostgreSQL melibatkan beberapa lapisan abstraksi Docker dan mekanisme *buffering*. Berikut adalah tahapannya:

- Penerbitan Log (log-generator): Aplikasi log-generator menulis pesan ke stdout (standard output). Di dalam container, ini hanyalah aliran teks biasa yang biasanya muncul di terminal jika kita menjalankan aplikasi secara interaktif.
- Intersepsi oleh Docker Daemon: Karena container dikonfigurasi menggunakan logging-driver: fluentd, Docker Daemon tidak membuang log tersebut ke file JSON (default), melainkan menangkap aliran stdout tersebut dan membungkusnya ke dalam paket metadata yang berisi *container\_id*, *container\_name*, dan *timestamp*.
- Pengiriman via Protokol Forward: Docker Daemon mengirimkan paket log tersebut melalui protokol Forward (biasanya lewat port 24224\) menuju container Fluent-bit. Di sini, log masih berada dalam format biner yang sangat efisien.
- Pemrosesan di Fluent-bit (Input & Filter): \* Input: Fluent-bit menerima log tersebut melalui plugin in\_forward.
1. Filter (Optional): Jika ada filter, Fluent-bit akan membedah log JSON tersebut, misalnya memisahkan tingkat keparahan (*level*) dari isi pesan (*message*).
- Pengiriman ke Database (Output PGSQL): Plugin out\_pgsql di Fluent-bit menerjemahkan data JSON tadi menjadi perintah SQL INSERT. Berdasarkan konfigurasi *mapping* kolom yang kamu buat, Fluent-bit mencocokkan kunci JSON (seperti source\_container) ke kolom tabel (seperti container\_name).
- Penyimpanan di PostgreSQL: Database PostgreSQL menerima perintah INSERT, menjalankan aturan skema di dalam *schema* logs, dan akhirnya menyimpan data tersebut secara permanen ke dalam baris tabel container\_logs.
-

### Soal 5

Modifikasi LOG\_INTERVAL menjadi 0.5 detik. Berapa log rate per menit yang dihasilkan?

**![image17](image/Laporan%20Modul%205/image17.png)**

---

<!-- Gambar -->

