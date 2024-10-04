Jasa Setting Mikrotik, OLT, Switch Manage, Web-Server, Mail-Server, DNS Server, Monitoring server, CPANEL, dah yang ingin dibuatin server apa aja deh, bisa hubungi nomor hp saya +62 822-3348-3221 (Purwanto)

---

## **Panduan Instalasi Ookla Server di Ubuntu 24.04**

### 1. **Download Script Ookla Server**
Unduh script instalasi Ookla Server dengan perintah berikut:
```bash
wget https://install.speedtest.net/ooklaserver/ooklaserver.sh
```

### 2. **Memberikan Izin Eksekusi pada Script**
Setelah script terunduh, berikan izin eksekusi:
```bash
chmod a+x ooklaserver.sh
```

### 3. **Menjalankan, Menghentikan, dan Merestart Ookla Server**
Gunakan perintah berikut untuk mengelola service Ookla Server:
- **Start**: 
  ```bash
  ./ooklaserver.sh start
  ```
- **Restart**:
  ```bash
  ./ooklaserver.sh restart
  ```
- **Stop**:
  ```bash
  ./ooklaserver.sh stop
  ```

### 4. **Install Web Server, PHP, dan Unzip**
Untuk mendukung operasi fallback, instal Nginx atau Apache, PHP, dan Unzip:
- Untuk **Nginx**:
  ```bash
  sudo apt -y install nginx-full php8.3-fpm unzip
  ```
- Untuk **Apache**:
  ```bash
  sudo apt -y install apache2 php libapache2-mod-php unzip
  ```

### 5. **Pindah ke Direktori Web Root**
Masuk ke direktori `/var/www/html` untuk mengunduh fallback file:
```bash
cd /var/www/html
```

### 6. **Download Fallback Ookla**
Unduh file fallback yang diperlukan untuk Ookla Server:
```bash
wget https://install.speedtest.net/httplegacy/http_legacy_fallback.zip
```

### 7. **Unzip Fallback File**
Ekstrak file yang telah diunduh dan cek isi direktori:
```bash
unzip http_legacy_fallback.zip
```
Hasil ekstraksi akan berada di direktori `speedtest/`. Lihat hasilnya dengan:
```bash
ls -lh
```

### 8. **Atur Konfigurasi Virtual Host**
Buat atau edit file konfigurasi virtual host sesuai domain/subdomain Anda. Contoh konfigurasi untuk Nginx:

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Untuk Apache, buat file konfigurasi di `/etc/apache2/sites-available/`:

```apache
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html

    <Directory /var/www/html/speedtest>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Aktifkan vhost dan restart web server:
- **Nginx**:
  ```bash
  sudo systemctl restart nginx
  ```
- **Apache**:
  ```bash
  sudo a2ensite your_vhost.conf
  sudo systemctl restart apache2
  ```

### 9. **Install Certbot untuk SSL**
Gunakan Certbot untuk mengamankan koneksi dengan SSL:
```bash
sudo apt install certbot python3-certbot-nginx
```
Untuk Nginx:
```bash
sudo certbot --nginx -d example.com
```
Untuk Apache:
```bash
sudo certbot --apache -d example.com
```

### 10. **Atur SSL pada Konfigurasi Ookla**
Buka dokumentasi Ookla di [Advanced Server Daemon Configuration](https://support.ookla.com/hc/en-us/articles/234577948-Advanced-Server-Daemon-Configuration) untuk melakukan pengaturan SSL pada Ookla Server.

### 11. **Tes Installasi Ookla Server di Browser**
Buka browser dan arahkan ke:
- `http://example.com:8080`
- `http://example.com/speedtest/upload.php`

Pastikan halaman "It worked!" muncul sebagai tanda server berhasil terinstall.

### 12. **Tes Konfigurasi dengan Host Tester**
Gunakan Host Tester Ookla untuk menguji konfigurasi server:
- Buka: [https://speedtest.net/host-tester](https://speedtest.net/host-tester)

### 13. **Buat Service untuk Jalankan Server di Reboot**
Buat perintah agar Ookla Server berjalan saat server reboot. Misalnya, buat file systemd untuk Ookla Server:
1. Buat file unit di `/etc/systemd/system/ooklaserver.service`:
```ini
[Unit]
Description=Ookla Server Daemon
After=network.target

[Service]
Type=simple
User=ooklauser
ExecStart=/full_path_to_your_Ookla_Server_Daemon/OoklaServer --daemon
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
2. Reload systemd dan aktifkan service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable ooklaserver
```
Service ini akan berjalan otomatis saat server reboot.

### 14. **Registrasi Akun Ookla dan Submit Server**
Setelah semuanya siap, buat akun Ookla dan submit server Anda di [https://account.ookla.com/register/servers?step=1](https://account.ookla.com/register/servers?step=1).
Tunggu review dari tim Ookla sekitar 48 jam.

---

Dengan mengikuti panduan di atas, Ookla Server Anda akan terpasang dan siap untuk digunakan.
