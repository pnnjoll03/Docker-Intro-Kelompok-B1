# Laporan Praktikum Modul 1: Docker Installation

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

Sebutkan minimal 3 perbedaan antara Virtual Machine dan Container.

- VM memiliki OS sendiri termasuk kernelnya masing masing berjalan di atas hypervisor sedangkan container tidak, berbagi satu kernel yang sama dengan Host OS, jadi gak perlu install OS penuh di setiap contaianernya.
- Vm berukuran besar dibandingkan dengan container karena VM berisi seluruh file OS nya sedangkan container hanay berisi aplikasi dan library yang dibutuhkan saja.
- VM bootingnya lebih lama dibandingkan dengan container karena boot OS dari awal sedangkan container dapat boot secara cepat karena dijalankan diatas kernel host yang sudah aktif
- VM memiliki isolasi yang tinggi karena setiap VM terpisah secara fisik dilevel CPUnya dan container isolasinya tingkat proses karena berbagi kernel yang sama.

### Soal 2

Apa fungsi dari containerd dan runc dalam arsitektur Docker?

- Containerd, runtime tingkat tinggi yang mengatur lifecycle container seperti create, start, stop, delete, mengelola image, mengatur snapshot file-system, menjadi perantara antara domain engine dan runtime rendah
- Runc, komponen tingkat rendah yang berfungsi sebagai eksekutor seperti menjalankan container sesuai stander Open Container Initiative (OCI), berinteraksi dengan kernel linux, membuat environment isolated untuk container

### Soal 3

Mengapa Docker membutuhkan kernel Linux? Bagaimana Docker Desktop di Windows mengatasi hal ini?

Karena menggunakan dua fitur utama yang ada di kernel linux yaitu namespaces, fitur yang bertanggung jawab atas isolasi, memastikan sebuah container memiliki pandangan sendiri terhadap sistem, seperti jaringan, proses (PID), dan mount point, sehingga tidak bisa melihat atau mengganggu container lain. Dan control groups, fitur berfungsi untuk pembatasa sumber daya, mengatur berapa banyak CPU, memori (RAM), dan bandwidth yang boleh digunakan oleh sebuah container agar tidak menghabiskan seluruh sumber daya host.

   Karena Windows memiliki kernel yang berbeda dengan Linux, Docker Desktop menggunakan teknologi virtualisasi untuk menyediakan "rumah" bagi kernel Linux di dalam sistem Windows:

- WSL 2 (Windows Subsystem for Linux): Ini adalah metode yang paling umum digunakan saat ini. Windows menjalankan kernel Linux asli yang sangat ringan di dalam sebuah Virtual Machine (VM) khusus yang dikelola oleh Microsoft. Docker Desktop kemudian berjalan di atas kernel Linux tersebut.
- Hyper-V VM: Sebelum adanya WSL 2, Docker Desktop menggunakan hypervisor Hyper-V untuk membuat sebuah Virtual Machine Linux (biasanya berbasis distro ringan seperti Alpine atau Moby) yang berjalan di latar belakang.
- Docker Engine: Di dalam VM atau WSL 2 tersebut, barulah Docker Engine dapat berinteraksi dengan kernel Linux untuk menjalankan container.

### Soal 4

Apa keuntungan layered filesystem pada Docker Image?

efisiensi dalam penyimpanan dan distribusi data. Poin poin keuntungannya :

- **Efisiensi Penyimpanan (Storage Efficiency)**: Karena image bersifat *read-only* dan tersusun atas lapisan-lapisan, lapisan yang sama tidak perlu disimpan berkali-kali.
- **Kecepatan Distribusi (Fast Distribution)**: Saat  melakukan docker pull atau push, Docker hanya akan mengirimkan lapisan yang belum ada di host tujuan. Inilah alasan kenapa proses *download* image kedua biasanya jauh lebih cepat jika menggunakan *base image* yang sama.
- **Optimasi Proses Build (Build Cache)**: Docker menggunakan mekanisme *caching* pada setiap layer. Jika  mengubah isi index.html tapi tidak mengubah Dockerfile, Docker hanya akan membangun ulang layer yang berubah, sementara layer instalasi Nginx akan diambil dari *cache*.
- **Penghematan Penggunaan Memori**: Saat menjalankan banyak container dari image yang sama, kernel Linux dapat berbagi lapisan *read-only* yang sama di memori (RAM), sehingga penggunaan sumber daya menjadi sangat minim.
- **Pemisahan Konten yang Jelas**: Seperti yang terlihat pada perintah docker image history yang jalankan, bisa melihat dengan jelas instruksi mana yang menambah ukuran image (seperti COPY atau RUN) dan mana yang hanya berupa metadata.

### Soal 5

Jelaskan perbedaan antara docker run dan docker exec.

Docker run digunakan ketika kita ingin membuat dan menjalakan container baru dari sebuh image sedangkan docker exec untuk menjalankan perintah baru didalam container yang sudah aktif.

---

## Screenshot

### 1. docker version â€” versi Client dan Server

![image1](image/Laporan%20Modul%201/image1.png)

### 2. sudo systemctl status docker â€” service active (running)

![image2](image/Laporan%20Modul%201/image2.png)

### 3. docker run hello-world â€” pesan sukses lengkap

![image3](image/Laporan%20Modul%201/image3.png)

### 4. docker images â€” daftar image yang sudah di-pull

![image4](image/Laporan%20Modul%201/image4.png)

### 5. docker ps â€” container nginx berjalan dengan port mapping

![image5](image/Laporan%20Modul%201/image5.png)

### 6. Browser mengakses http://localhost:8080 â€” halaman Nginx default

![image6](image/Laporan%20Modul%201/image6.png)

### 7. docker build \-t pens-web:1.0 . â€” proses build berhasil (step terakhir)

![image7](image/Laporan%20Modul%201/image7.png)

### 8. Browser mengakses http://localhost:9090 â€” halaman custom PENS

![image8](image/Laporan%20Modul%201/image8.png)

---

## B. Post Lab

### Soal 1

Bandingkan output docker image history nginx dengan docker image history pens-web:1.0. Layer mana saja yang di-share?

Layer yang di-share adalah :

- COPY docker-entrypoint.sh / \# buildkit
- COPY 10-listen-on-ipv6-by-default.sh /docker-entrypoint.d/
- COPY 15-local-resolvers.envsh /docker-entrypoint.d/
- COPY 20-envsubst-on-templates.sh /docker-entrypoint.d/
- COPY 30-tune-worker-processes.sh /docker-entrypoint.d/
- ENTRYPOINT \["/[docker-entrypoint.sh](http://docker-entrypoint.sh)"\]
- EXPOSE 80 (atau EXPOSE map\[80/tcp:{}\])
- STOPSIGNAL SIGQUIT
- CMD \["nginx" "-g" "daemon off;"\]

### Soal 2

Apa yang terjadi pada data di dalam container setelah container dihapus dengan docker rm? Bagaimana solusinya?

Seluruh data yang ada di dalama writable layer container tersebut akan hilang secara permanen.

### Soal 3

Jelaskan perbedaan antara EXPOSE di Dockerfile dan flag \-p pada docker run. Apakah EXPOSE cukup untuk membuat port dapat diakses dari host?

EXPOSE dalam Dockerfile (Dokumentasi & Metadata) , Instruksi EXPOSE berfungsi sebagai sarana komunikasi antara pembuat image dan pengguna image.

- Fungsi Utama: Memberi tahu Docker dan pengguna bahwa aplikasi di dalam container mendengarkan (*listen*) pada port tertentu.
- Efek Ke Host: Instruksi ini tidak membuka akses ke mesin host. Port tersebut tetap tertutup bagi siapa pun di luar jaringan internal Docker.
- Penggunaan Internal: Sangat berguna untuk fitur *inter-container communication* atau ketika menggunakan flag \-P (P besar) yang akan memetakan semua port yang di-EXPOSE ke port acak di host.

  Flag \-p pada docker run (Publish/Mapping), Flag \-p (atau \--publish) adalah perintah eksekusi yang benar-benar melakukan konfigurasi pada sistem jaringan host .

- Fungsi Utama: Memetakan port di mesin host (laptop/VM) ke port di dalam container.
- Format: \[port-host\]:\[port-container\]. Contoh: \-p 8080:80 berarti permintaan yang masuk ke localhost:8080 di browser akan diteruskan ke port 80 di dalam container.
- Konektivitas: Inilah satu-satunya cara manual untuk membuat layanan di dalam container dapat diakses secara publik melalui IP host atau localhost.

  Apakah EXPOSE cukup untuk membuat port dapat diakses dari host?

  Tidak Jika hanya menulis EXPOSE 80 tanpa menyertakan flag \-p saat menjalankan container, tidak akan bisa membuka aplikasi tersebut melalui browser di mesin host. EXPOSE hanya bertindak sebagai "label" atau petunjuk. tetap membutuhkan flag \-p untuk membuat "jembatan" yang menghubungkan port fisik di laptop/VM dengan port virtual di dalam container.

### Soal 4

Mengapa menggunakan tag spesifik (misal nginx:1.26) lebih baik daripada nginx:latest untuk production?

Karena demi kepastian versi semisal kita pakai yang nginx:latest jadi bersifat dinamis dan sulit untuk di rollback karena kita gak bisa melacak versi mana yang lebih stabil, lalu Memungkinkan tim IT untuk melakukan pemindaian keamanan (*vulnerability scanning*) pada versi yang tetap dan memastikan versi tersebut sudah melalui tahap pengujian internal.

### Soal 5

Berapa ukuran image alpine:3.20 dibanding ubuntu:22.04? Apa trade-off menggunakan Alpine?

**Ukuran image alpine:3.20 :**

   **![image9](image/Laporan%20Modul%201/image9.png)**

   **Dan ukuran ubuntu:22.04 :**

   **![image10](image/Laporan%20Modul%201/image10.png)**

   Konsekuensi menggunakan alpine :

- Kompatibilitas Library (musl vs glibc): Alpine menggunakan m
- usl libc, sedangkan kebanyakan aplikasi Linux standar dibuat untuk glibc. Hal ini bisa menyebabkan aplikasi (seperti Python atau Node.js) gagal berjalan atau butuh konfigurasi rumit.
- Waktu Build yang Lebih Lama: Karena perbedaan *library*, beberapa paket tidak bisa diunduh dalam bentuk biner siap pakai dan harus dikompilasi dari awal (*compile from source*), yang justru memperlambat proses *build*.
- Alat Debugging Terbatas: Untuk menjaga ukurannya tetap kecil, Alpine tidak menyertakan perintah dasar seperti curl, ping, atau dig secara bawaan.  harus menginstalnya secara manual jika ingin melakukan *troubleshooting*.
- Ekosistem Package Manager: Alpine menggunakan apk yang koleksi paketnya mungkin tidak selengkap atau se-mutakhir apt milik Ubunt**u.**

---

<!-- Gambar -->

