# grass 


### 1. **Konfigurasi dan File JSON**:
   - **`config.json`** digunakan untuk memuat konfigurasi yang dapat disesuaikan, seperti `proxy_retry_limit` (batas percobaan ulang untuk proxy) dan `reload_interval` (interval waktu untuk memuat ulang proxy setelah kegagalan).
   - Jika file ini tidak ditemukan, skrip menggunakan nilai default yang telah ditentukan dalam kode.

### 2. **Fungsi Auto-Update dari GitHub**:
   - Fungsi `auto_update_script()` memungkinkan skrip untuk memeriksa apakah ada pembaruan terbaru di repositori GitHub.
   - Jika repositori di-clone menggunakan Git, skrip akan meminta untuk melakukan `git pull` untuk menarik pembaruan terbaru.
   - Jika tidak, peringatan akan ditampilkan, dan skrip akan berhenti.

### 3. **Pemeriksaan Kode Aktivasi**:
   - Fungsi `check_activation_code()` meminta pengguna untuk memasukkan kode aktivasi yang benar (dalam hal ini, "UJICOBA").
   - Skrip hanya akan melanjutkan jika kode yang dimasukkan benar, meningkatkan lapisan keamanan untuk penggunaan skrip.

### 4. **Pengelolaan Proxy dengan Pembatasan Ulang**:
   - **Daftar proxy** dimuat dari file `local_proxies.txt` dan digunakan untuk membuat koneksi WebSocket menggunakan proxy SOCKS5.
   - **`proxy_retry_limit`** menentukan berapa banyak percobaan ulang yang dilakukan jika proxy gagal.
   - Setelah mencoba semua proxy dalam daftar, skrip akan memuat ulang daftar proxy jika semua gagal, berdasarkan interval yang ditentukan di `reload_interval`.

### 5. **WebSocket Connection**:
   - **`connect_to_wss()`** adalah fungsi utama untuk mengelola koneksi WebSocket melalui proxy. Menggunakan proxy yang ditentukan dalam daftar proxy dan mencoba untuk membuka koneksi WebSocket ke server yang dituju.
   - Skrip menggunakan SSL context untuk menghubungkan secara aman dan menyertakan beberapa header HTTP custom untuk meniru perilaku browser.
   - Fungsi **ping/pong** untuk menjaga koneksi tetap aktif dengan server dengan mengirimkan paket **ping** dan menunggu balasan **pong**.
   - Koneksi WebSocket juga menangani pesan "AUTH" dan "PONG" untuk otentikasi dan menjaga koneksi tetap hidup.

### 6. **Manajemen Kegagalan Proxy**:
   - **Proxy yang gagal** akan dicatat dalam daftar `proxy_failures`, dan proxy yang gagal akan disimpan dalam file `successful_proxies.txt` setelah mencoba kembali.
   - Jika semua proxy gagal, skrip akan menunggu `reload_interval` sebelum mencoba lagi.

### 7. **Penggunaan User-Agent Acak**:
   - **User-Agent** dihasilkan secara acak menggunakan library `fake_useragent` untuk menghindari deteksi oleh server.
   - Setiap koneksi WebSocket akan menggunakan User-Agent yang berbeda untuk memberikan kesan bahwa itu berasal dari perangkat yang berbeda.

### 8. **Manajemen Sumber Daya dengan Semaphore**:
   - **`asyncio.Semaphore(100)`** digunakan untuk membatasi jumlah koneksi WebSocket simultan, yang membantu dalam pengelolaan sumber daya dan menghindari kelebihan beban sistem.
   - Dengan ini, skrip dapat menjalankan hingga 100 koneksi paralel secara efisien tanpa membebani sistem atau server secara berlebihan.

### 9. **Distribusi Beban dan Skalabilitas**:
   - **`asyncio.gather()`** digunakan untuk mengeksekusi banyak tugas secara bersamaan, meningkatkan performa dan skalabilitas.
   - Skrip akan memproses semua proxy dalam daftar secara paralel, meningkatkan efisiensi dalam penggunaan waktu dan sumber daya.
   - Skrip dapat dengan mudah disesuaikan untuk menangani jumlah proxy yang lebih besar, jika diperlukan.

### 10. **Manajemen Log dan Penggunaan `loguru`**:
   - **`loguru`** digunakan untuk manajemen log yang canggih, memungkinkan pencatatan pesan dengan tingkat log yang berbeda (info, warn, error).
   - Log yang lebih jelas dan terstruktur membantu dalam pemecahan masalah dan pemantauan jalannya skrip.
   - Penggunaan warna di log juga membantu membedakan status dan kesalahan, mempermudah identifikasi masalah.

### 11. **Peningkatan Efisiensi Memori**:
   - **Asyncio** dan **semaphore** membantu dalam pengelolaan memori dengan memastikan bahwa hanya sejumlah tugas tertentu yang berjalan dalam waktu yang sama.
   - Menggunakan `asyncio.gather` dan pengelolaan koneksi WebSocket yang efisien memungkinkan memori dan sumber daya lainnya digunakan dengan optimal.

### 12. **Penyimpanan dan Pembaruan Daftar Proxy**:
   - Proxy yang berhasil digunakan akan disimpan dalam file `data/successful_proxies.txt`, dan proxy yang gagal akan dihindari dalam percakapan berikutnya.
   - Skrip secara otomatis mengelola daftar proxy dan memperbarui file untuk memastikan hanya proxy yang berfungsi yang digunakan.

### 13. **Pengelolaan Timeout dan Retry**:
   - Koneksi WebSocket mengimplementasikan **timeout** untuk menunggu balasan dari server. Jika tidak ada balasan dalam waktu tertentu, koneksi akan dicoba lagi setelah beberapa detik.
   - Skrip menggunakan **backoff eksponensial** untuk menghindari percakapan berulang dengan server yang sama jika ada kesalahan.

### 14. **Fungsi Pengelolaan Proyek**:
   - Menggunakan sistem yang mempermudah memperbarui dan memperbaiki skrip melalui pengunduhan otomatis pembaruan GitHub, menjaga agar skrip tetap up-to-date tanpa intervensi manual.

### 15. **Pengelolaan Koneksi WebSocket**:
   - Skrip memanfaatkan **multi-channel WebSocket** dengan `asyncio.gather()` untuk menjalankan banyak koneksi secara bersamaan.
   - Setiap koneksi WebSocket mengirimkan dan menerima pesan secara asinkron, menjaga koneksi tetap responsif tanpa membebani server.

### Kesimpulan:
Skrip ini telah dirancang dengan sangat baik untuk menangani banyak proxy SOCKS5 secara paralel, menjaga koneksi WebSocket tetap hidup, mengelola kegagalan, dan menyimpan log dengan canggih. Dengan fitur auto-update dan manajemen proxy yang efisien, skrip ini sangat scalable dan dapat diadaptasi untuk menangani berbagai ukuran beban dan konfigurasi proxy.

Skrip ini juga mengoptimalkan performa, penggunaan memori, dan distribusi beban dengan menggunakan asyncio, semaphore, dan berbagai teknik manajemen sumber daya lainnya. Jadi, skrip ini sudah sangat siap untuk digunakan dalam skala besar.

Jika Anda siap, Anda bisa langsung menjalankan skrip tersebut dan memantau log untuk memastikan semuanya berjalan dengan baik.
