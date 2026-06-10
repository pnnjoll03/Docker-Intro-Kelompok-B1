# Laporan Praktikum Modul 3: Web Server & Reverse Proxy

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

Apa keuntungan menjalankan web server di container dibandingkan langsung di host?

- Container berjalan dilingkungannya sendiri. Jika satu container error atau diretas, dampaknya tidak akan langsung mengenai sistem host atau container lain.
- Manajemen Dependency yang baik, kita bisa menjalankan PHP 7.4 di satu container dan PHP 8.2 di container lain pada host yang sama tanpa konflik library.![image1](image/Laporan%20Modul%203/image1.png)![image2](image/Laporan%20Modul%203/image2.png)

### Soal 2

Jelaskan perbedaan document root Apache (/usr/loca/apache2/htdocs/) vs Nginx (/usr/share/nginx/html/)

Document Root adalah direktori utama di dalam server tempat fileâ€ website (seperti index.html) disimpan

- Apache defaultnya disimpan di /usr/local/apache2/htdocs/
- Nginx defaultnya disimpan di /usr/share/nginx/html/

  Cek Apache : docker exec web-apache ls /usr/local/apache2/htdocs/![image3](image/Laporan%20Modul%203/image3.png)

  Cek Nginx : docker exec nginx-web ls /usr/share/nginx/html/

  ![image4](image/Laporan%20Modul%203/image4.png)

### Soal 3

Apa itu SSL Terminator dan mengapa dilakukan di reverse proxy?

SSL Terminator adalah proses dimana enkripsi SSL/TLS dideskripsi di tingkat *intermediary* (seperti Reverse Proxy) sebelum diteruskan ke server backend dalam bentuk HTTP biasa. Mengapa di Reverse Proxy?

- Proses enkripsi/deskripsi itu berat. Dengan melakukan ini di proxy, server backend bisa fokus memproses logika aplikasi saja.
- Sentralisasi, karena kita hanya perlu mengelola sertifikat SSL di satu tempat (di proxy), bukan di setiap server backend.

### Soal 4

Apa perbedaan name-based dan IP-based virtual hosting?

- IP-based Virtual Hosting: setiap website memiliki alamat IP unik. Server membedakan request berdasarkan IP mana saja yang dituju. Contoh website A di 1.1.1.1, Website B di 1.1.1.2.
- Name-based Virtual Hosting: Beberapa website berbagi alamat IP yang sama. Server membedakan request menggunakan header HTTP Host yang dikirim oleh browser. Ini adalah metode yang paling umum digunakan saat ini karena menghemat alamat IP.

### Soal 5

Mengapa self-signed certificate menghasilkan warning di browser?

Browser hanya mempercayai setifikat yang ditandatangani oleh Certificate Authority (CA) resmi. Self-signed certificate adalah sertifkat yang dibuat dan ditandatangani sendiri oleh pemilik server. Karena tidak ada pihak ketiga yangn terpercaya yang memvalidasi identitas server tersebut, browser memberikan peringatan bahwa identitas server tidak dapat diverifikasi untuk mencegah serangan Man-in-the-Middle.

---

## Screenshot

### 1. docker compose ps â€” 4 service running

![image5](image/Laporan%20Modul%203/image5.png)

### 2. curl \-k https://site1.lab:8443 â€” halaman Site 1

![image6](image/Laporan%20Modul%203/image6.png)

### 3. Screenshot 3

![image7](image/Laporan%20Modul%203/image7.png)

### 4. curl \-I http://site1.lab:8080 â€” HTTPâ†’HTTPS redirect 301

![image8](image/Laporan%20Modul%203/image8.png)

### 5. openssl s\_client output â€” detail certificate

![image9](image/Laporan%20Modul%203/image9.png)

### 6. Screenshot 6

![image10](image/Laporan%20Modul%203/image10.png)

### 7. POST /api/visitors â€” response 201

![image11](image/Laporan%20Modul%203/image11.png)

### 8. GET /api/visitors â€” daftar visitor

![image12](image/Laporan%20Modul%203/image12.png)

### 9. Log Nginx per-site

![image13](image/Laporan%20Modul%203/image13.png)

### 10. Log Apache per-site

![image14](image/Laporan%20Modul%203/image14.png)

---

## B. Post Lab

### Soal 1

Bandingkan response header dari Apache vs Nginx. Header apa yang menunjukkan software web server?

**![image15](image/Laporan%20Modul%203/image15.png)![image16](image/Laporan%20Modul%203/image16.png)**

### Soal 2

Jika Nginx proxy down, apakah Apache masih bisa diakses langsung? Bagaimana cara testnya?

Jika Nginx berfungsi sebagai Reverse Proxy (pintu masuk utama) untuk Apach, maka jika Nginx mati, user tidak bisa mengakses Apache melalui alamat proxy tersebut. Namun, jika port Apache (8081) masih dibuka/diekspose ke host, Apache tetap bisa diakses langsung lewat port tersebut

   ![image17](image/Laporan%20Modul%203/image17.png)![image18](image/Laporan%20Modul%203/image18.png)

### Soal 3

Tunjukkan bahwa X-Real-IP header diteruskan dengan benar dari Nginx ke Flask

- Pastikan semua container berjalan

  Untuk memastikan container berjalan gunakan perintah docker compose up \-d![image19](image/Laporan%20Modul%203/image19.png)

- Jalankan perintah uji coba:

  Gunakan perintah curl untuk mengakses API Flask melalui proxy Nginx:**![image20](image/Laporan%20Modul%203/image20.png)**Berdasarkan outputnya Integrasi antara Nginx sebagai Reverse Proxy dan Flask sebagai backend telah terkonfigurasi dengan baik. Sistem mampu menjaga integritas data informasi klien meskipun terdapat lapisan proxy di tengahnya, yang sangat penting untuk kebutuhan logging dan keamanan aplikasi.

### Soal 4

Jelaskan mengapa Flask app perlu terhubung ke dua network (web-net dan db-net).

- Web-net: digunakan agar Nginx (Proxy) bisa berkomunikasi dengan Flask. Network ini â€œterbukaâ€ ke arah luar.
- Db-net: digunakan agar Flask bisa berkomunikasi dengan Database.
- Database tidak boleh terhubung ke web-net. Dengan begitu, jika Nginx diretas, peretas tidak bisa langsung â€œmelihatâ€ atau menyerang Database karena mereke berada di network yang berbeda. Flask bertindak sebagai jembatan yang aman.

### Soal 5

Apa yang terjadi jika file server.key atau server.crt dihapus saat container running?

- Uji Akses HTTPS Awal:

  Pastikan web masih bisa diakses normal![image21](image/Laporan%20Modul%203/image21.png)

- Hapus file sertifikat di dalam container

  Ubah /etc/nginx/certs agar tidak read only![image22](image/Laporan%20Modul%203/image22.png)

  Restart container

  ![image23](image/Laporan%20Modul%203/image23.png)

  Hapus file sertifikat di dalam container

  ![image24](image/Laporan%20Modul%203/image24.png)

- Uji akses setelah penghapusan
- ![image25](image/Laporan%20Modul%203/image25.png)

  Website tetap bisa diakses karena Nginx sudah load isi sertifikat ke dalam memori (RAM) saat pertama kali dijalankan.

- Lakukan reload konfigurasi

  Gunakan perintah docker exec nginx-proxy nginx \-s reload

  ![image26](image/Laporan%20Modul%203/image26.png)

  Terjadi error karena server.key sudah tidak ditemukan di disk, sehingga service tidak bisa memperbarui konfigurasinya.

---

<!-- Gambar -->

