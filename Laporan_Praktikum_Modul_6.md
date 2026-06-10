# Laporan Praktikum Modul 6: Monitoring dengan Prometheus & Grafana

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

Jelaskan perbedaan model pull-based (Prometheus) dan push-based (Fluent Bit) dalam pengumpulan data.

**Perbedaan mendasar antara model *pull-based* dan *push-based* terletak pada siapa yang menginisiasi pengiriman data dalam ekosistem monitoring atau logging. Dalam model pull-based yang digunakan oleh Prometheus, server monitoring berperan aktif sebagai pihak yang melakukan *polling* atau "menarik" data secara berkala dari target (seperti *exporter* atau aplikasi) melalui endpoint HTTP yang telah ditentukan; metode ini memberikan kontrol penuh kepada server pusat untuk mengatur ritme pengumpulan data, mempermudah deteksi ketika target mati (*down*), namun memerlukan konfigurasi *service discovery* agar server mengetahui lokasi setiap target. Sebaliknya, dalam model push-based yang diterapkan oleh Fluent Bit, agen yang terpasang pada sisi klien berperan sebagai pihak yang "mendorong" atau mengirimkan log dan data secara proaktif ke sistem pengumpulan pusat (seperti *backend* log aggregation); pendekatan ini sangat efisien untuk lingkungan *ephemeral* atau sistem yang sering berganti alamat IP karena klien tidak perlu menunggu instruksi dari server, serta meminimalisir beban pada server pusat karena pengiriman data diatur oleh agen, meskipun metode ini dapat menyulitkan deteksi kegagalan klien karena server tidak tahu kapan seharusnya data dikirimkan.**

### Soal 2

Apa itu PromQL? Berikan contoh query untuk menghitung rata-rata CPU usage dalam 5 menit terakhir.

**PromQL (Prometheus Query Language) adalah bahasa kueri fungsional yang dirancang khusus untuk memanipulasi data deret waktu (*time-series data*) yang tersimpan dalam Prometheus, di mana setiap kueri bekerja berdasarkan dimensi label dan nilai numerik yang tercatat dari waktu ke waktu. Dengan PromQL, pengguna dapat melakukan agregasi, transformasi matematis, serta pemfilteran data secara presisi untuk kebutuhan pemantauan performa maupun pembuatan *alerting*. Untuk menghitung rata-rata penggunaan CPU dalam lima menit terakhir, Anda dapat menggunakan fungsi `rate()` yang dikombinasikan dengan fungsi `avg()` seperti pada contoh berikut: `avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))`. Dalam kueri tersebut, `rate()` menghitung kecepatan perubahan per detik dari metrik CPU yang *idle* dalam rentang waktu lima menit, kemudian `avg()` merata-ratakan nilai tersebut di seluruh instans yang ada, sehingga memberikan gambaran konsumsi CPU rata-rata yang lebih akurat dibandingkan sekadar melihat data *snapshot* sesaat.**

### Soal 3

Mengapa cAdvisor membutuhkan akses ke /var/run/docker.sock dan /sys?

**cAdvisor membutuhkan akses ke direktori `/var/run/docker.sock` dan `/sys` karena ia berperan sebagai pengumpul metrik *low-level* yang harus berkomunikasi langsung dengan *runtime* kontainer dan kernel sistem operasi untuk mendapatkan data performa yang akurat. Dengan mengakses `docker.sock`, cAdvisor dapat berkomunikasi dengan API Docker untuk mengidentifikasi kontainer mana saja yang sedang berjalan, melacak siklus hidupnya, serta mendapatkan metadata terkait kontainer tersebut secara *real-time*. Sementara itu, akses ke direktori `/sys` (khususnya `/sys/fs/cgroup`) sangat krusial karena di sinilah kernel Linux menempatkan informasi statistik *cgroups* (control groups) yang mencatat penggunaan sumber daya seperti CPU, memori, dan I/O disk untuk setiap proses atau grup proses; tanpa akses ke *cgroups* melalui jalur sistem ini, cAdvisor tidak akan mampu membaca metrik penggunaan sumber daya yang spesifik di tingkat *containerized environment*.**

### Soal 4

Apa keuntungan Grafana provisioning (file YAML) dibanding konfigurasi manual via UI?

**Penggunaan *provisioning* berbasis file YAML di Grafana menawarkan keuntungan signifikan dibandingkan konfigurasi manual melalui UI, terutama terkait dengan aspek Infrastructure as Code (IaC) dan konsistensi lingkungan. Dengan menggunakan file YAML, seluruh konfigurasi *dashboard*, *datasource*, dan *alerting* dapat disimpan dalam sistem kontrol versi seperti Git, yang memungkinkan Anda melakukan *tracking* perubahan, melakukan *code review* sebelum *deployment*, serta melakukan *rollback* dengan mudah jika terjadi kesalahan konfigurasi. Metode ini juga sangat krusial dalam lingkungan *automated pipeline* atau *CI/CD*, di mana Anda dapat melakukan *deploy* infrastruktur monitoring secara otomatis ke berbagai *environment* (seperti *staging* ke *production*) tanpa harus mengonfigurasi ulang secara manual yang rentan terhadap kesalahan manusia (*human error*) dan ketidakkonsistenan antar-instans.**

   **Selain itu, *provisioning* via file YAML memungkinkan konfigurasi bersifat *immutable* dan *idempotent*, artinya sistem akan selalu memastikan kondisi *dashboard* atau *datasource* sesuai dengan definisi file tersebut setiap kali Grafana melakukan *restart* atau pembaruan, sehingga menghilangkan risiko konfigurasi "hantu" yang tertinggal dari sesi *editing* manual di masa lalu. Sementara UI lebih cocok untuk eksperimen atau *troubleshooting* cepat, pendekatan file YAML jauh lebih superior untuk menjaga integritas sistem dalam skala besar karena memungkinkan standarisasi konfigurasi di seluruh tim.**

### Soal 5

Jelaskan perbedaan antara Gauge, Counter, dan Histogram dalam Prometheus metrics.

**Dalam Prometheus, perbedaan utama antara Counter, Gauge, dan Histogram terletak pada sifat data yang mereka simpan dan bagaimana data tersebut diolah untuk memberikan informasi yang bermakna. Counter adalah metrik kumulatif yang nilainya hanya bisa bertambah atau diatur ulang ke nol saat *restart* aplikasi; tipe ini sangat cocok untuk menghitung kejadian yang bersifat akumulatif, seperti jumlah permintaan HTTP yang diterima atau total kesalahan sistem. Gauge merepresentasikan nilai numerik yang dapat naik maupun turun secara bebas, menjadikannya metrik yang ideal untuk memantau keadaan yang bersifat fluktuatif seperti penggunaan memori, jumlah koneksi aktif, atau suhu ruangan. Sementara itu, Histogram jauh lebih kompleks karena ia tidak hanya menghitung jumlah total pengamatan, tetapi juga mengelompokkan nilai-nilai tersebut ke dalam "bucket" yang telah ditentukan sebelumnya dan mencatat total jumlah observasi serta nilai akumulatif dari semua observasi; struktur ini memungkinkan Anda untuk menghitung statistik krusial seperti kuantil (persentil) dan distribusi durasi permintaan, yang tidak mungkin didapatkan hanya dari data rata-rata sederhana.**

* **Counter: Menunjukkan "berapa banyak" (selalu meningkat).**
* **Gauge: Menunjukkan "berapa saat ini" (bisa naik-turun).**
* **Histogram: Menunjukkan "sebaran atau distribusi" (untuk analisis latensi dan durasi).**

---

## Screenshot

### 1. Screenshot 1

![image1](image/Laporan%20Modul%206/image1.png)

### 2. Screenshot 2

![image2](image/Laporan%20Modul%206/image2.png)

### 3. Screenshot 3

![image3](image/Laporan%20Modul%206/image3.png)

### 4. Screenshot 4

![image4](image/Laporan%20Modul%206/image4.png)

### 5. Screenshot 5

![image5](image/Laporan%20Modul%206/image5.png)

### 6. Screenshot 6

![image6](image/Laporan%20Modul%206/image6.png)

### 7. Screenshot 7

![image7](image/Laporan%20Modul%206/image7.png)

### 8. Screenshot 8

![image8](image/Laporan%20Modul%206/image8.png)

### 9. Screenshot 9

![image9](image/Laporan%20Modul%206/image9.png)

### 10. Screenshot 10

![image10](image/Laporan%20Modul%206/image10.png)

### 11. Screenshot 11

![image11](image/Laporan%20Modul%206/image11.png)

### 12. Screenshot 12

![image12](image/Laporan%20Modul%206/image12.png)

### 13. Screenshot 13

![image13](image/Laporan%20Modul%206/image13.png)

### 14. Screenshot 14

![image14](image/Laporan%20Modul%206/image14.png)

---

## B. Post Lab

### Soal 1

Dari dashboard Container Metrics, container mana yang paling banyak menggunakan CPU dan memory? Mengapa?

**Container dengan penggunaan CPU dan memori tertinggi dapat dilihat pada dashboard Container Metrics di Grafana. Berdasarkan hasil monitoring, container yang memiliki penggunaan sumber daya terbesar umumnya adalah container yang menjalankan proses yang lebih intensif seperti Prometheus, cAdvisor, Fluent Bit, atau container yang sedang menjalankan stress test. Hal ini terjadi karena container tersebut secara terus-menerus melakukan pengumpulan metrik, pemrosesan data log, serta menjalankan berbagai proses backend yang membutuhkan sumber daya komputasi lebih besar dibandingkan container lainnya. Semakin banyak proses yang dijalankan dan semakin tinggi aktivitas pada container tersebut, maka penggunaan CPU dan memori juga akan meningkat.**

### Soal 2

Saat stress test berjalan, berapa persen CPU usage yang terukur di Grafana? Bandingkan dengan output top atau htop di host.

**Saat menjalankan *stress test*, perbedaan angka persentase CPU antara Grafana dan utilitas lokal seperti `top` atau `htop` sering kali muncul akibat perbedaan metodologi kalkulasi dan interval waktu pengumpulan data. Grafana, melalui Prometheus, umumnya menampilkan hasil rata-rata (*average*) dari *rate* penggunaan CPU selama rentang waktu tertentu, seperti 1 menit atau 5 menit, sehingga nilainya cenderung tampak lebih halus (*smooth*) dan tidak menangkap lonjakan tajam (*spikes*) sesaat yang terjadi di balik layar. Sebaliknya, `top` atau `htop` memberikan tampilan *real-time* dengan *sampling* yang jauh lebih cepat (biasanya setiap 1-3 detik), sehingga angka yang ditampilkan mencerminkan beban CPU yang aktual terjadi pada detik tersebut. Sebagai contoh, jika *stress test* memicu lonjakan CPU hingga 100% secara sporadis namun cepat, `top` akan langsung menunjukkan angka 100%, sementara grafik di Grafana mungkin hanya memperlihatkan kenaikan moderat karena nilai tersebut telah dirata-ratakan dengan periode *idle* di antara lonjakan. Selain itu, perlu diingat bahwa alat seperti `htop` sering kali menghitung total beban sistem termasuk *I/O wait* dan *interrupts* dengan cara yang berbeda dengan metrik yang dikirim oleh `node_exporter` ke Prometheus, yang sering kali membedakan secara ketat antara kategori *user*, *system*, dan *idle* berdasarkan statistik *cgroups* atau `/proc/stat`.**

### Soal 3

Buat query PromQL yang menampilkan 3 container dengan memory usage tertinggi. Tunjukkan query dan hasilnya.

**Untuk querynya ini topk(3, container\_memory\_usage\_bytes{name\!=""}/1024/1024)**

   **![image15](image/Laporan%20Modul%206/image15.png)**

### Soal 4

Dari dashboard Log Analytics, berapa rasio ERROR vs INFO log dalam 1 jam terakhir? Apakah ini normal untuk aplikasi production?

**![image16](image/Laporan%20Modul%206/image16.png)**

   **Berdasarkan hasil pantauan saya di *dashboard* Log Analytics untuk satu jam terakhir, rasio antara log *error* dan *info* saat ini menunjukkan angka nol pada keduanya. Saya sedang melakukan investigasi apakah kondisi ini wajar karena memang sedang tidak ada trafik yang masuk ke sistem, atau justru ini indikasi bahwa sistem *logging* saya sedang mengalami kendala teknis. Mengingat ini adalah lingkungan *production*, saya tidak bisa mengabaikan potensi kegagalan pada *log appender* atau masalah pada *collector* yang menyebabkan data tidak terkirim. Saat ini, prioritas saya adalah memastikan apakah *instrumentation* di aplikasi masih berjalan dengan benar atau ada filter pada kueri yang salah, karena dalam kondisi operasional normal, minimal harus ada log aktivitas *info* yang masuk sebagai bukti bahwa aplikasi saya tetap *up* dan melayani *request*.**

### Soal 5

Jika Prometheus container dihapus dan dibuat ulang (tanpa menghapus volume prom-data), apakah data historis metrik masih ada? Buktikan.

**ya, data historis metrik Anda akan tetap ada dan aman, karena penyimpanan data Prometheus dilakukan di luar *container* itu sendiri melalui mekanisme *persistent volume*. Ketika Anda menghapus *container*, hanya *runtime environment* dan memori *container* tersebut yang hilang, namun file basis data (TSDB) yang tersimpan di *volume* tetap utuh di *host* atau *storage* yang terpasang. Saat Anda membuat ulang *container* dan melakukan *mounting* ulang ke direktori `prom-data` yang sama, Prometheus akan secara otomatis melakukan pemindaian (*recovery*) pada *write-ahead log* (WAL) dan indeks data yang ada di *volume* tersebut untuk melanjutkan operasi dari titik terakhir sebelum *container* lama dihapus.**

   **![image17](image/Laporan%20Modul%206/image17.png)**

---

<!-- Gambar -->

