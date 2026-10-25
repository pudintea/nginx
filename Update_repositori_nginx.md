# Repository Nginx
```
apt-cache policy nginx
```
Hasilnya :
```
nginx:
  Installed: 1.24.0-2ubuntu7.18
  Candidate: 1.24.0-2ubuntu7.18
  Version table:
 *** 1.24.0-2ubuntu7.18 500
        500 http://mirror.biznetgio.com/ubuntu noble-updates/main amd64 Packages
        500 http://mirror.biznetgio.com/ubuntu noble-security/main amd64 Packages
        100 /var/lib/dpkg/status
     1.24.0-2ubuntu7 500
        500 http://mirror.biznetgio.com/ubuntu noble/main amd64 Packages
```

Dari output kamu, penyebabnya sudah jelas: Nginx masih berasal dari repository bawaan Ubuntu 24.04 (Noble), bukan repository resmi `nginx.org`.

Paket Ubuntu memang masih menyediakan Nginx `1.24.0-2ubuntu7.18` untuk Noble. Jadi, `apt upgrade` saja tidak akan menaikkan versi tersebut ke rilis Stable NGINX yang lebih baru.

Details of package nginx in noble-updates

+1

## Langkah berikutnya

Kalau backup pada langkah sebelumnya sudah dilakukan, ikuti langkah berikut secara berurutan.

### 1. Pastikan repository resmi belum ada atau sudah ada

bash

```
cat /etc/apt/sources.list.d/nginx.list 2>/dev/null
```

Jika belum ada, tambahkan repository Stable resmi untuk Ubuntu Noble:

bash

```
sudo apt update
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

Unduh signing key:

bash

```
curl -fsSL https://nginx.org/keys/nginx_signing.key \
  -o /tmp/nginx_signing.key

gpg --show-keys --with-fingerprint /tmp/nginx_signing.key
```

Verifikasi fingerprint resmi:

text

```
573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62
```

Jika cocok, daftarkan key:

bash

```
sudo gpg --dearmor \
  -o /usr/share/keyrings/nginx-archive-keyring.gpg \
  /tmp/nginx_signing.key
```

Lalu tambahkan repository:

bash

```
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/ubuntu noble nginx" \
  | sudo tee /etc/apt/sources.list.d/nginx.list
```

### 2. Prioritaskan paket dari NGINX

bash

```
printf 'Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n' \
  | sudo tee /etc/apt/preferences.d/99nginx
```

### 3. Cek kandidat versi

bash

```
sudo apt update
apt-cache policy nginx
```

Pastikan `Candidate` berubah ke versi dari `nginx.org`. Jangan lanjut jika kandidat masih berasal dari mirror Ubuntu.

Jika sudah benar, simulasi perubahan terlebih dahulu:

bash

```
sudo apt-get -s install nginx
```

Periksa apakah ada paket yang akan dihapus atau perubahan dependensi yang tidak diharapkan.

### 4. Upgrade dan verifikasi

Jika simulasi aman:

bash

```
sudo apt install nginx
sudo nginx -t
sudo systemctl reload nginx
nginx -v
```

Periksa juga layanan:

bash

```
sudo systemctl status nginx --no-pager
```

Dokumentasi resmi: [NGINX Linux packages](<https://nginx.org/en/linux_packages.html?utm_source=chatgpt.com>).

Penting: repository resmi dapat menggunakan susunan paket dan konfigurasi yang berbeda dari paket Ubuntu. Sebelum upgrade pada server produksi, periksa konfigurasi dan modul tambahan yang terpasang agar tidak ada modul yang hilang.

Setelah kamu menjalankan `sudo apt update` dan `apt-cache policy nginx`, kirimkan output terbarunya. Saya bisa membantu memastikan kandidat versinya benar sebelum kamu melakukan upgrade.
