# NGINX VERSI STABLE UNTUK UBUNTU 24.04

Bisa. Untuk Ubuntu 24.04, kamu cukup mengganti repository `mainline` menjadi repository Stable resmi Nginx. Konfigurasi pin priority `900` yang sudah kamu buat tetap bisa digunakan.

Menurut dokumentasi resmi Nginx, repository Stable menggunakan URL `https://nginx.org/packages/ubuntu`.
nginx.org


## 1. Download dan simpan GPG key resmi Nginx
```
sudo apt install -y curl gnupg2 ca-certificates lsb-release ubuntu-keyring
curl -fsSL https://nginx.org/keys/nginx_signing.key \
  | sudo gpg --dearmor -o /usr/share/keyrings/nginx-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/nginx-archive-keyring.gpg
```
## 2. Verifikasi GPG key
```
gpg --show-keys --with-fingerprint /usr/share/keyrings/nginx-archive-keyring.gpg
```
Pastikan terdapat fingerprint:
```
8540A6F18833A80E9C1653A42FD21310B49F6B46
```
Sesuaikan fingerprint tersebut. 

## 3. Pastikan repository menggunakan keyring tersebut
```
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/ubuntu noble nginx" \
| sudo tee /etc/apt/sources.list.d/nginx.list
```
## 4. Prioritaskan paket dari nginx.org
Buat aturan agar APT memilih paket dari repository resmi Nginx daripada paket bawaan Ubuntu:
```
sudo tee /etc/apt/preferences.d/99nginx <<'EOF'
Package: *
Pin: origin nginx.org
Pin: release o=nginx
Pin-Priority: 900
EOF
```

## 5. Update repository
```
sudo apt clean
sudo apt update
```

## 6. Upgrade Nginx ke versi Stable

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

## 7. Pastikan pembaruan berikutnya tetap dari Stable

Setelah konfigurasi selesai, pembaruan rutin bisa dilakukan dengan:

bash

```
sudo apt update
sudo apt upgrade
```

APT akan mengambil versi Stable terbaru yang tersedia dari repository tersebut. Pembaruan tidak otomatis terpasang kecuali kamu juga mengatur mekanisme pembaruan otomatis.

Catatan: Stable bukan berarti tidak mendapat pembaruan keamanan. Cabang ini tetap menerima perbaikan kritis dan keamanan yang diperlukan.

NGINX Documentation
