Untuk Ubuntu 24.04 LTS, kamu bisa memindahkan repository Nginx dari repository bawaan Ubuntu (`archive.ubuntu.com`) ke repository resmi Nginx (`nginx.org`). Dengan begitu, paket Nginx akan mendapatkan pembaruan dari repository resmi Nginx saat kamu menjalankan `apt update` dan `apt upgrade`.

Ada dua pilihan repository resmi:

- Mainline — fitur dan pembaruan terbaru lebih cepat tersedia. Cocok jika ingin Nginx selalu mengikuti rilis terbaru.
- Stable — lebih berfokus pada stabilitas dan perubahan yang lebih konservatif.

Jika tujuanmu adalah mengikuti perkembangan Nginx terbaru, pilih Mainline.

## 1. Periksa instalasi Nginx saat ini

Sebelum mengubah repository, simpan backup konfigurasi dan periksa sumber paket yang digunakan.

bash

```
nginx -v
apt policy nginx
sudo nginx -t
sudo cp -a /etc/nginx "/etc/nginx.backup-$(date +%F)"
```

Pastikan `nginx -t` menunjukkan `syntax is ok` dan `test is successful`. Jika kamu memakai modul pihak ketiga atau konfigurasi khusus, perhatikan kompatibilitasnya saat berpindah paket.

## 2. Tambahkan repository resmi nginx.org

Langkah berikut mengikuti panduan resmi Nginx untuk Ubuntu 24.04 (`noble`).

nginx.org

+1

### A. Instal dependensi

bash

```
sudo apt update
sudo apt install -y curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

### B. Impor kunci GPG resmi

bash

```
curl -fsSL https://nginx.org/keys/nginx_signing.key \
  | sudo gpg --dearmor --yes \
  -o /usr/share/keyrings/nginx-archive-keyring.gpg
```

Verifikasi fingerprint kunci:

bash

```
gpg --show-keys --with-fingerprint \
  /usr/share/keyrings/nginx-archive-keyring.gpg
```

Dokumentasi resmi mencantumkan beberapa fingerprint kunci Nginx. Pastikan kunci yang ditampilkan cocok dengan fingerprint yang dipublikasikan Nginx sebelum melanjutkan. [Dokumentasi resmi Nginx](<https://nginx.org/en/linux_packages.html?utm_source=chatgpt.com>).

### C. Pilih repository Mainline

Jalankan perintah berikut untuk Ubuntu 24.04:

bash

```
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/ubuntu noble nginx" \
  | sudo tee /etc/apt/sources.list.d/nginx.list
```

Jika lebih mengutamakan stabilitas, ganti URL `packages/mainline/ubuntu` dengan `packages/ubuntu`.

### D. Prioritaskan paket dari nginx.org

Buat aturan agar APT memilih paket dari repository resmi Nginx daripada paket bawaan Ubuntu:

bash

```
sudo tee /etc/apt/preferences.d/99nginx <<'EOF'
Package: *
Pin: origin nginx.org
Pin: release o=nginx
Pin-Priority: 900
EOF
```

## 3. Perbarui dan migrasikan paket Nginx

Perbarui indeks paket terlebih dahulu:

bash

```
sudo apt update
apt policy nginx
```

Periksa bagian `Candidate` dan daftar `Version table`. Pastikan versi kandidat berasal dari `nginx.org/packages/mainline/ubuntu`.

Setelah itu, jalankan:

bash

```
sudo apt install nginx
```

APT mungkin meminta konfirmasi untuk mengganti atau mengubah paket yang sudah terpasang. Baca daftar perubahan sebelum menyetujuinya.

Jika muncul masalah dependensi atau penghapusan paket yang tidak diharapkan, jangan lanjutkan secara paksa. Periksa hasil simulasi terlebih dahulu:

bash

```
sudo apt-get -s install nginx
```

## 4. Verifikasi hasil migrasi

bash

```
apt policy nginx
nginx -v
sudo nginx -t
systemctl status nginx --no-pager
```

Periksa juga sumber versi paket:

bash

```
apt-cache policy nginx
```

Pastikan versi kandidat dan versi terpasang sudah sesuai dengan repository resmi yang dipilih.

## 5. Apakah Nginx akan selalu update otomatis?

Setelah repository ditambahkan, perintah berikut akan mengambil pembaruan terbaru yang tersedia:

bash

```
sudo apt update
sudo apt upgrade
```

Namun, menambahkan repository tidak otomatis berarti Nginx langsung diperbarui begitu versi baru dirilis. Jika ingin pembaruan otomatis, kamu bisa mengonfigurasi `unattended-upgrades` dan memeriksa kebijakan pembaruan paketnya.

Perlu diingat bahwa Mainline mengikuti perkembangan rilis lebih cepat, sedangkan Stable lebih cocok untuk lingkungan produksi yang memprioritaskan perubahan lebih konservatif.

NGINX Documentation

Catatan penting: Migrasi paket dapat mengubah tata letak file konfigurasi, modul, atau perilaku layanan dibandingkan paket Ubuntu. Pastikan virtual host, TLS/SSL, reverse proxy, dan modul tambahan tetap berfungsi setelah migrasi.

## Lihat kode dibawah ini
```
ginx:
  Installed: 1.24.0-2ubuntu7.18
  Candidate: 1.31.8-1~noble
  Version table:
     1.31.8-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.31.7-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.31.6-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.31.5-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.31.4-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.31.3-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.31.2-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.31.1-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.31.0-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.8-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.7-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.6-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.5-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.4-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.3-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.2-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.1-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.29.0-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.27.5-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.27.4-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.27.3-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.27.2-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.27.1-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.27.0-2~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.27.0-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
     1.25.5-1~noble 900
        900 https://nginx.org/packages/mainline/ubuntu noble/nginx amd64 Packages
 *** 1.24.0-2ubuntu7.18 500
        500 http://mirror.biznetgio.com/ubuntu noble-updates/main amd64 Packages
        500 http://mirror.biznetgio.com/ubuntu noble-security/main amd64 Packages
        100 /var/lib/dpkg/status
     1.24.0-2ubuntu7 500
        500 http://mirror.biznetgio.com/ubuntu noble/main amd64 Packages
```

Konfigurasimu sudah benar. ✅ Repository resmi `nginx.org` sudah terdeteksi dan diprioritaskan dengan pin priority `900`.

Dari output kamu:

- Versi terpasang: `1.24.0-2ubuntu7.18` — dari repository Ubuntu.
- Versi kandidat: `1.31.8-1~noble` — dari repository resmi Nginx Mainline.
- Priority 900: repository `nginx.org` diprioritaskan daripada repository Ubuntu yang memiliki priority `500`.

## Langkah berikutnya: upgrade Nginx

Sebelum melakukan upgrade, simulasikan dahulu:

bash

```
sudo apt-get -s install nginx
```

Jika tidak ada penghapusan paket penting atau konflik dependensi yang tidak diharapkan, jalankan:

bash

```
sudo apt update
sudo apt install nginx
```

Kemudian verifikasi:

bash

```
nginx -v
apt policy nginx
sudo nginx -t
```

Jika berhasil, versi Nginx yang terpasang seharusnya mengikuti kandidat dari `nginx.org`.

Perhatian: Nginx akan berpindah dari paket Ubuntu ke paket resmi Nginx. Pastikan konfigurasi virtual host, sertifikat SSL, reverse proxy, dan modul tambahan tetap kompatibel. Jika server ini melayani trafik produksi, lakukan upgrade pada waktu pemeliharaan.

## Agar selalu mendapat pembaruan

Setelah migrasi berhasil, setiap kali menjalankan:

bash

```
sudo apt update && sudo apt upgrade
```

APT akan memeriksa versi terbaru yang tersedia dari repository yang sudah dikonfigurasi.

Perlu dibedakan: ini membuat Nginx siap menerima pembaruan, bukan otomatis menginstalnya saat rilis baru muncul. Untuk instalasi otomatis, kamu perlu mengaktifkan dan memeriksa konfigurasi `unattended-upgrades`.
