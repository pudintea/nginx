Berikut langkah demi langkah untuk menginstal Nginx dari repository resmi `nginx.org` di Debian, bukan dari repository bawaan Debian.

Metode ini menggunakan repository APT resmi Nginx sehingga instalasi dan pembaruan Nginx dapat dikelola menggunakan `apt`.

## Dokumentasi
- (Ubuntu)[https://nginx.org/en/linux_packages.html#Ubuntu]
- (Debian)[https://nginx.org/en/linux_packages.html#Debian]

## 1. Periksa versi Debian dan arsitektur

Jalankan:

bash

```
cat /etc/os-release
dpkg --print-architecture
```

Catat versi Debian, misalnya Debian 12 (Bookworm) atau Debian 13 (Trixie), serta arsitekturnya seperti `amd64` atau `arm64`.

## 2. Instal paket yang diperlukan

Perbarui daftar paket dan instal alat untuk mengelola repository HTTPS serta kunci GPG.

bash

```
sudo apt update
sudo apt install -y curl gnupg2 ca-certificates lsb-release debian-archive-keyring
```

## 3. Tambahkan GPG signing key resmi Nginx

Buat direktori untuk menyimpan kunci APT:

bash

```
sudo install -d -m 0755 /usr/share/keyrings
```

Unduh kunci penandatanganan dari situs resmi Nginx:

bash

```
curl -fsSL https://nginx.org/keys/nginx_signing.key \
  | sudo gpg --dearmor --yes \
  -o /usr/share/keyrings/nginx-archive-keyring.gpg
```

Atur agar kunci bisa dibaca oleh APT:

bash

```
sudo chmod a+r /usr/share/keyrings/nginx-archive-keyring.gpg
```

Penting: Untuk instalasi server produksi, verifikasi fingerprint kunci melalui dokumentasi resmi Nginx sebelum mempercayainya. Jangan menonaktifkan pemeriksaan tanda tangan APT.

Verifikasi kunci yang sudah diunduh:

bash

```
gpg --dry-run --quiet --no-keyring \
  --import --import-options import-show \
  /usr/share/keyrings/nginx-archive-keyring.gpg
```

Dokumentasi Nginx mencantumkan fingerprint kunci resmi berikut. Pastikan fingerprint tersebut muncul dalam hasil verifikasi.

nginx.org

+1

text

```
8540A6F18833A80E9C1653A42FD21310B49F6B46
573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
9E9BE90EACBCDE69FE9B204CBCDCD8A38D88A2B3
```

Jika tidak cocok, jangan lanjutkan instalasi.

## 4. Tambahkan repository Nginx

Nginx menyediakan dua pilihan channel:

- Stable — pilihan yang disarankan untuk kebanyakan server produksi.
- Mainline — berisi pengembangan fitur terbaru dan rilis lebih cepat.

Saya menyarankan channel Stable.

Jalankan perintah berikut untuk membuat repository secara otomatis berdasarkan codename Debian Anda:

bash

```
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/debian $(lsb_release -cs) nginx" \
  | sudo tee /etc/apt/sources.list.d/nginx.list
```

Contoh hasil repository:

text

```
deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/debian bookworm nginx
```

`bookworm` adalah codename Debian 12. Jika menggunakan Debian 13, hasilnya seharusnya `trixie`.

Repository resmi Nginx saat ini mencantumkan dukungan untuk Debian 12 dan Debian 13 pada arsitektur `amd64` dan `arm64`.

nginx.org

## 5. Prioritaskan paket dari nginx.org

Langkah ini memastikan APT lebih memilih paket Nginx dari repository resmi dibandingkan paket yang disediakan distribusi Debian.

bash

```
sudo tee /etc/apt/preferences.d/99nginx <<'EOF'
Package: *
Pin: origin nginx.org
Pin: release o=nginx
Pin-Priority: 900
EOF
```

## 6. Update repository dan instal Nginx

Perbarui indeks paket:

bash

```
sudo apt update
```

Periksa kandidat versi yang akan dipasang:

bash

```
apt-cache policy nginx
```

Pastikan bagian `Candidate` dan daftar `Version table` menunjukkan versi yang berasal dari `nginx.org/packages/debian`.

Selanjutnya, instal Nginx:

bash

```
sudo apt install -y nginx
```

Jika sebelumnya Anda sudah memasang Nginx dari repository Debian, periksa konfigurasi dan dependensi terlebih dahulu karena perpindahan paket dapat memengaruhi konfigurasi yang ada.

## 7. Jalankan dan aktifkan Nginx

Aktifkan layanan agar berjalan otomatis saat server dinyalakan:

bash

```
sudo systemctl enable --now nginx
```

Periksa statusnya:

bash

```
sudo systemctl status nginx
```

Jika berhasil, status layanan akan menunjukkan:

text

```
Active: active (running)
```

## 8. Verifikasi instalasi

Periksa versi Nginx:

bash

```
nginx -v
```

Periksa konfigurasi:

bash

```
sudo nginx -t
```

Tes respons HTTP lokal:

bash

```
curl -I http://127.0.0.1
```

Jika berhasil, biasanya Anda akan mendapatkan respons `HTTP/1.1 200 OK`.

Anda juga dapat membuka alamat IP server melalui browser:

text

```
http://IP_SERVER_ANDA
```

Jika tidak dapat diakses dari komputer lain, periksa firewall serta aturan jaringan yang mengizinkan koneksi TCP port 80.

## 9. Lokasi file penting

Setelah memasang paket resmi `nginx.org`, struktur konfigurasi umumnya seperti berikut:

| File atau direktori | Fungsi |
| --- | --- |
| `/etc/nginx/nginx.conf` | Konfigurasi utama |
| `/etc/nginx/conf.d/` | Konfigurasi tambahan virtual host |
| `/var/log/nginx/access.log` | Log akses |
| `/var/log/nginx/error.log` | Log error |
| `/usr/share/nginx/html/` | Direktori default untuk file website |

Perlu diperhatikan bahwa struktur paket dari `nginx.org` dapat berbeda dari paket Nginx bawaan Debian, terutama pada direktori konfigurasi dan mekanisme pengelolaan virtual host.

Untuk melihat seluruh file yang disediakan paket:

bash

```
dpkg -L nginx
```

## 10. Mengelola Nginx setelah instalasi

Perintah yang paling sering digunakan:

bash

```
# Memeriksa konfigurasi
sudo nginx -t

# Reload konfigurasi tanpa menghentikan layanan
sudo systemctl reload nginx

# Restart layanan
sudo systemctl restart nginx

# Melihat log error secara langsung
sudo tail -f /var/log/nginx/error.log

# Memperbarui paket dari repository
sudo apt update
sudo apt install --only-upgrade nginx
```

Untuk dokumentasi lengkap, lihat [panduan resmi instalasi paket Linux Nginx](<https://nginx.org/en/linux_packages.html?utm_source=chatgpt.com>).

Saran saya: Jika Anda menggunakan Debian 12 atau 13 untuk web server produksi, gunakan repository `stable` terlebih dahulu.

Supaya langkahnya bisa saya sesuaikan secara tepat, Anda menggunakan Debian versi berapa — Debian 11, 12, atau 13?
