# panduan-rustdesk-pribadi
Tutorial: Self-Host Server RustDesk di DigitalOcean Menggunakan Docker

Panduan ini akan memandu Anda melalui proses lengkap untuk menyiapkan server RustDesk pribadi Anda di DigitalOcean. Dengan server pribadi, Anda akan mendapatkan koneksi remote desktop yang lebih cepat, stabil, dan sepenuhnya privat.

Tutorial ini dibuat pada 20 Agustus 2025 dan menggunakan versi terbaru dari tools yang relevan pada tanggal tersebut.
📋 Prasyarat

Sebelum memulai, pastikan Anda memiliki:

    Akun DigitalOcean: Idealnya, gunakan kredit dari GitHub Student Developer Pack agar gratis.

    Sebuah Nama Domain (Opsional, Sangat Direkomendasikan): Menggunakan domain (misal: remote.perusahaananda.com) jauh lebih mudah dan aman daripada menggunakan alamat IP. Tutorial ini akan menggunakan domain.

    Pemahaman Dasar Terminal/SSH: Anda harus tahu cara terhubung ke server menggunakan terminal (di macOS/Linux) atau PuTTY (di Windows).

🚀 Langkah-langkah Instalasi
Langkah 1: Membuat Droplet (Server) di DigitalOcean

Droplet adalah server virtual tempat kita akan menginstal server RustDesk.

    Login ke akun DigitalOcean Anda.

    Klik tombol "Create" lalu pilih "Droplets".

    Pilih Region: Pilih lokasi server yang paling dekat dengan Anda dan pengguna Anda. Untuk Indonesia, Singapura (Singapore) adalah pilihan terbaik untuk latensi terendah.

    Pilih OS: Pilih Ubuntu 22.04 (LTS) x64.

    Pilih Tipe Droplet: Pilih "Basic" plan.

    Pilih CPU: Pilih Droplet termurah, biasanya Regular with SSD seharga ~$4-6/bulan. Ini sudah lebih dari cukup untuk server RustDesk.

    Pilih Autentikasi:

        Sangat direkomendasikan: Pilih SSH Key. Ini jauh lebih aman. Anda perlu menambahkan kunci publik SSH Anda ke DigitalOcean.

        Alternatif: Pilih Password. Buat password root yang sangat kuat.

    Klik "Create Droplet". Tunggu beberapa menit hingga server Anda siap. Setelah selesai, DigitalOcean akan menampilkan alamat IP publik server Anda. Salin alamat IP ini.

Langkah 2: Mengarahkan Domain ke IP Droplet (Opsional)

Jika Anda menggunakan domain, ini saatnya mengarahkannya ke server baru Anda.

    Pergi ke penyedia domain Anda (misalnya Namecheap, GoDaddy, dll).

    Masuk ke pengaturan DNS untuk domain Anda.

    Buat "A Record" baru dengan konfigurasi berikut:

        Host/Name: rustdesk (atau subdomain lain yang Anda inginkan, misal: remote).

        Value/Points to: Tempelkan alamat IP Droplet yang Anda salin dari Langkah 1.

        TTL: Biarkan default.

    Simpan perubahan. Perubahan DNS mungkin butuh beberapa menit hingga beberapa jam untuk menyebar.

    Catatan: Mulai sekarang, setiap kali Anda melihat <IP_ANDA_ATAU_DOMAIN>, gantilah dengan alamat IP Droplet Anda atau domain yang baru saja Anda konfigurasikan (misal: rustdesk.domainanda.com).

Langkah 3: Menghubungkan ke Droplet via SSH

Buka terminal di komputer Anda dan hubungkan ke server.

ssh root@<IP_ANDA_ATAU_DOMAIN>

Jika ini pertama kalinya, Anda akan diminta untuk menerima fingerprint server. Ketik yes dan tekan Enter.
Langkah 4: Menginstal Docker dan Docker Compose

Setelah masuk ke server, kita akan menginstal Docker yang akan mengelola server RustDesk kita.

    Update daftar paket server:

    apt update && apt upgrade -y

    Instal paket-paket prasyarat:

    apt install -y apt-transport-https ca-certificates curl software-properties-common

    Tambahkan GPG Key resmi dari Docker:

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr-share/keyrings/docker-archive-keyring.gpg

    Tambahkan repositori Docker:

    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

    Instal Docker Engine dan Docker Compose:

    apt update
    apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

    Verifikasi instalasi:

    docker --version

    Anda akan melihat versi Docker yang terinstal.

Langkah 5: Membuat dan Menjalankan Server RustDesk

Sekarang bagian intinya. Kita akan membuat file konfigurasi untuk server RustDesk.

    Buat sebuah direktori untuk server Anda:

    mkdir rustdesk_server && cd rustdesk_server

    Buat file konfigurasi docker-compose.yml:

    nano docker-compose.yml

    Salin dan tempel konfigurasi berikut ke dalam editor nano:

    version: '3'

    services:
      hbbs:
        container_name: hbbs
        image: rustdesk/rustdesk-server:latest
        command: hbbs -r <IP_ANDA_ATAU_DOMAIN>
        volumes:
          - ./data:/root
        network_mode: "host"
        depends_on:
          - hbbr
        restart: unless-stopped

      hbbr:
        container_name: hbbr
        image: rustdesk/rustdesk-server:latest
        command: hbbr
        volumes:
          - ./data:/root
        network_mode: "host"
        restart: unless-stopped

        PENTING: Ganti <IP_ANDA_ATAU_DOMAIN> dengan alamat IP atau domain Anda. Ini adalah satu-satunya bagian yang perlu Anda ubah.

    Simpan dan tutup file: Tekan Ctrl + X, lalu Y, lalu Enter.

    Jalankan server RustDesk!

    docker-compose up -d

    Perintah ini akan mengunduh image yang diperlukan dan menjalankan server di background. Server Anda sekarang sudah aktif!

Langkah 6: Konfigurasi Klien RustDesk

Langkah terakhir adalah memberitahu aplikasi RustDesk di komputer Anda untuk menggunakan server pribadi Anda, bukan server publik.

    Buka aplikasi RustDesk di komputer Anda (dan di komputer yang ingin Anda remote).

    Di sebelah ID Anda, klik menu titik tiga (...) dan pilih "ID/Relay Server".

    ID Server: Masukkan <IP_ANDA_ATAU_DOMAIN> Anda.

    Relay Server: Masukkan <IP_ANDA_ATAU_DOMAIN> Anda (isi sama dengan ID Server).

    Key: Bagian ini penting untuk keamanan. Kembali ke terminal SSH Anda dan jalankan perintah berikut untuk melihat log hbbs dan menemukan kunci publik Anda:

    docker-compose logs hbbs

    Cari baris yang terlihat seperti "Public key: <kunci_panjang_acak>". Salin rangkaian karakter acak tersebut.

    Tempelkan kunci publik tadi ke kolom Key di klien RustDesk.

    Klik OK.

Lakukan konfigurasi yang sama di semua perangkat yang ingin Anda hubungkan melalui server pribadi Anda. Jika berhasil, Anda akan melihat status "Ready" dengan centang hijau di bagian bawah aplikasi.
✅ Selamat!

Anda sekarang memiliki server remote desktop RustDesk yang andal, cepat, dan sepenuhnya privat, berjalan di atas infrastruktur DigitalOcean. Semua koneksi Anda kini terenkripsi dan melewati server milik Anda sendiri.
Tips Tambahan: Keamanan Firewall

Untuk keamanan ekstra, Anda bisa mengaktifkan firewall dasar (ufw) untuk hanya mengizinkan port yang dibutuhkan oleh RustDesk dan SSH.

ufw allow 22/tcp          # Port untuk SSH
ufw allow 21115/tcp       # RustDesk
ufw allow 21116/tcp       # RustDesk
ufw allow 21116/udp       # RustDesk
ufw allow 21117/tcp       # RustDesk
ufw allow 21118/tcp       # RustDesk
ufw allow 21119/tcp       # RustDesk
ufw enable              # Aktifkan Firewall
