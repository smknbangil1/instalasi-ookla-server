Jasa Setting Mikrotik, OLT, Switch Manage, Web-Server, Mail-Server, DNS Server, Monitoring server, CPANEL, dah yang ingin dibuatin server apa aja deh, bisa hubungi nomor hp saya +62 822-3348-3221 (Purwanto)

---

## **Panduan Instalasi OoklaServer di Ubuntu 24.04**

### **Prasyarat**
1. **Buat A-Record Subdomain**
   Pastikan Anda membuat A-Record untuk subdomain yang mengarah ke IP Public server Anda. Contohnya: `speedtest.domainanda.com`.

2. **Konfigurasi Firewall**
   Jalankan perintah berikut untuk membuka port yang diperlukan:
   ```bash
   sudo ufw allow 22
   sudo ufw allow 80
   sudo ufw allow 443
   sudo ufw allow 8080
   sudo ufw allow 5060
   sudo ufw enable
   sudo ufw reload
   ```

---

### **Langkah Instalasi**

#### **1. Unduh Installer**
   Jalankan perintah berikut untuk mengunduh skrip installer:
   ```bash
   wget https://install.speedtest.net/ooklaserver/ooklaserver.sh
   ```

#### **2. Berikan Hak Akses Eksekusi**
   Ubah hak akses skrip agar dapat dieksekusi:
   ```bash
   chmod +x ooklaserver.sh
   ```

#### **3. Eksekusi Instalasi**
   Jalankan skrip instalasi:
   ```bash
   sudo ./ooklaserver.sh install
   ```

#### **4. Edit Konfigurasi**
   Buka file konfigurasi:
   ```bash
   sudo nano OoklaServer.properties
   ```

   Ubah dan sesuaikan konfigurasi menjadi seperti berikut:
   ```
   OoklaServer.useIPv6 = true
   OoklaServer.allowedDomains = *.ookla.com, *.speedtest.net
   OoklaServer.enableAutoUpdate = true
   OoklaServer.ssl.useLetsEncrypt = true
   ```
   **Catatan**: Sertifikat SSL akan diterbitkan setelah server didaftarkan dan direview oleh tim Ookla.

   Untuk pengaturan lanjutan, Anda dapat merujuk ke dokumentasi resmi:
   [Advanced Server Daemon Configuration](https://support.ookla.com/hc/en-us/articles/234577948-Advanced-Server-Daemon-Configuration)

---

### **Manajemen OoklaServer Daemon**
Masuk ke direktori instalasi OoklaServer:
```bash
cd <directory_instalasi_ooklaserver>
```

Gunakan perintah berikut untuk manajemen layanan:
- Menjalankan:
  ```bash
  sudo ./ooklaserver.sh start
  ```
- Menghentikan:
  ```bash
  sudo ./ooklaserver.sh stop
  ```
- Merestart:
  ```bash
  sudo ./ooklaserver.sh restart
  ```

Untuk melihat bantuan, gunakan:
```bash
./ooklaserver.sh -h
```

---

### **Verifikasi Awal**
1. Akses server dengan membuka URL berikut di browser:
   ```
   http://speedtest.domainanda.com:8080
   ```

2. Gunakan alat host-tester Ookla untuk pemeriksaan lebih lanjut:
   [Speedtest Host Tester](https://www.speedtest.net/host-tester)  
   Masukkan URL berikut:
   ```
   speedtest.domainanda.com:8080
   ```
   **Catatan**: Jika terdapat bagian yang "failed", lakukan perbaikan sebelum mendaftarkan server Anda sebagai host.

---

### **Mengatasi Masalah Upload (Failed Upload Test)**

#### **1. Instal Web Server dan PHP**
Untuk mengatasi masalah upload, Anda perlu menginstal web server (seperti Nginx) dan PHP. Contoh konfigurasi virtual host Nginx adalah sebagai berikut:
```nginx
server {
    listen 80;
    listen 443 ssl;

    server_name speedtest.domainanda.com;

    # Redirect HTTP ke HTTPS
    if ($scheme = http) {
        return 301 https://$host$request_uri;
    }

    # Sertifikat SSL
    ssl_certificate /etc/letsencrypt/live/speedtest.domainanda.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/speedtest.domainanda.com/privkey.pem;

    # Direktori Root
    root /var/www/html;
    index index.php index.html index.htm;

    # Log File
    access_log /var/log/nginx/speedtest.access.log;
    error_log /var/log/nginx/speedtest.error.log;

    # PHP Configuration
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    client_max_body_size 1000M;

    # File Statis
    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### **2. Unduh dan Pasang HTTP Legacy Fallback**
Unduh dan ekstrak file HTTP Legacy Fallback:
```bash
wget http://install.speedtest.net/httplegacy/http_legacy_fallback.zip
unzip http_legacy_fallback.zip
```
Pindahkan hasil ekstraksi ke direktori web server:
```bash
mv http_legacy_fallback /var/www/html/speedtest
```

Akses URL berikut untuk memastikan semuanya bekerja:
```
https://speedtest.domainanda.com/speedtest/upload.php
```

---

### **Verifikasi Final**
Gunakan URL berikut untuk memverifikasi konfigurasi server Anda:
[Ookla Host Tester](https://www.ookla.com/host-tester)

---

### **Pendaftaran Server**
Jika semua tes telah berhasil, Anda dapat mendaftarkan server Anda untuk menjadi host resmi Speedtest Ookla. Proses ini akan direview dalam 2 hari kerja. Setelah disetujui, server Anda akan aktif sebagai host Speedtest.

--- 

systemd:
```systemd
[Unit]
Description=OoklaServer Daemon
After=network.target

[Service]
Type=forking
ExecStart=/ookla/ooklaserver.sh start
WorkingDirectory=/ookla
User=root
Group=root
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---
