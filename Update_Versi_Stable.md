# NGINX VERSI STABLE

Bisa. Untuk Ubuntu 24.04, kamu cukup mengganti repository `mainline` menjadi repository Stable resmi Nginx. Konfigurasi pin priority `900` yang sudah kamu buat tetap bisa digunakan.

Menurut dokumentasi resmi Nginx, repository Stable menggunakan URL `https://nginx.org/packages/ubuntu`.
nginx.org

## 1. Ganti repository ke Stable

Jalankan:

bash

```
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/ubuntu noble nginx" \
  | sudo tee /etc/apt/sources.list.d/nginx.list
```

Perintah ini mengganti isi file repository yang sebelumnya menunjuk ke `mainline`.

## 2. Update daftar paket

bash

```
sudo apt update
apt policy nginx
```

Pastikan URL repository yang muncul adalah:

text

```
https://nginx.org/packages/ubuntu noble/nginx
```

Versi kandidat seharusnya mengikuti cabang Stable, bukan lagi cabang Mainline. Nomor versi pastinya bergantung pada rilis Stable yang tersedia saat itu.

## 3. Upgrade Nginx ke versi Stable

Simulasikan terlebih dahulu:

bash

```
sudo apt-get -s install nginx
```

Jika hasilnya aman, lanjutkan:

bash

```
sudo apt install nginx
```

Lalu periksa:

bash

```
nginx -v
sudo nginx -t
```

## 4. Pastikan pembaruan berikutnya tetap dari Stable

Setelah konfigurasi selesai, pembaruan rutin bisa dilakukan dengan:

bash

```
sudo apt update
sudo apt upgrade
```

APT akan mengambil versi Stable terbaru yang tersedia dari repository tersebut. Pembaruan tidak otomatis terpasang kecuali kamu juga mengatur mekanisme pembaruan otomatis.

Catatan: Stable bukan berarti tidak mendapat pembaruan keamanan. Cabang ini tetap menerima perbaikan kritis dan keamanan yang diperlukan.

NGINX Documentation
