# Panduan lengkap instal Nginx Stable dari nginx.org di Ubuntu 24.04

Jika kamu membeli VPS baru dengan Ubuntu 24.04 LTS yang masih kosong,
cara terbaik adalah memasang Nginx langsung dari repository resmi `nginx.org`, bukan dari repository bawaan Ubuntu.

## Dokumentasi
- (Ubuntu)[https://nginx.org/en/linux_packages.html#Ubuntu]
- (Debian)[https://nginx.org/en/linux_packages.html#Debian]

## Langkah 1 — Update sistem VPS

Login ke VPS menggunakan SSH:

bash

```
ssh root@IP_VPS_KAMU
```

Jika kamu menggunakan user biasa yang memiliki akses `sudo`, gunakan user tersebut.

Update dan upgrade paket Ubuntu:

bash

```
sudo apt update
sudo apt upgrade -y
sudo reboot
```

Jika VPS reboot, login kembali melalui SSH.

Pastikan sistem memang Ubuntu 24.04:

bash

```
cat /etc/os-release
```

Periksa codename:

bash

```
. /etc/os-release
echo "$VERSION_CODENAME"
```

Output yang diharapkan:

text

```
noble
```

Mengapa ini penting? Repository harus sesuai dengan versi Ubuntu. Untuk Ubuntu 24.04, kita menggunakan `noble`, bukan `jammy` atau `resolute`.

## Langkah 2 — Instal dependensi

Jalankan:

bash

```
sudo apt update
sudo apt install -y curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

Fungsi masing-masing:

- `curl`: mengunduh kunci GPG resmi.
- `gnupg2`: memproses dan memverifikasi kunci.
- `ca-certificates`: memverifikasi sertifikat TLS/HTTPS.
- `lsb-release`: membantu mengenali versi distribusi.
- `ubuntu-keyring`: menyediakan kunci arsip Ubuntu.

## Langkah 3 — Tambahkan kunci GPG resmi Nginx

Repository resmi menggunakan tanda tangan digital agar APT bisa memverifikasi keaslian paket.

Unduh dan simpan kunci:

bash

```
curl -fsSL https://nginx.org/keys/nginx_signing.key \
  | sudo gpg --dearmor --yes \
  -o /usr/share/keyrings/nginx-archive-keyring.gpg
```

Periksa fingerprint kunci:

bash

```
gpg --show-keys --with-fingerprint \
  /usr/share/keyrings/nginx-archive-keyring.gpg
```

Dokumentasi Nginx mencantumkan beberapa kunci penandatanganan. Pastikan fingerprint yang ditampilkan sesuai dengan yang tercantum dalam [dokumentasi resmi Nginx](<https://nginx.org/en/linux_packages.html?utm_source=chatgpt.com>). Jangan lanjutkan jika fingerprint tidak cocok.


Mengapa menggunakan file keyring khusus? Karena konfigurasi repository berikutnya akan memakai `signed-by`, sehingga kunci tersebut digunakan untuk memverifikasi paket dari sumber yang ditentukan tanpa perlu menambahkan kunci ke penyimpanan tepercaya global APT.

## Langkah 4 — Tambahkan repository Nginx Stable

Ini merupakan langkah terpenting.

Buat file repository:

bash

```
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/ubuntu noble nginx" \
  | sudo tee /etc/apt/sources.list.d/nginx.list
```

Perhatikan URL berikut:

Stable — gunakan ini

[https://nginx.org/packages/ubuntu](<https://nginx.org/packages/ubuntu?utm_source=chatgpt.com>)

Mainline — jangan gunakan untuk konfigurasi ini

[https://nginx.org/packages/mainline/ubuntu](<https://nginx.org/packages/mainline/ubuntu?utm_source=chatgpt.com>)

Kamu tidak perlu menambahkan kata `stable` ke dalam URL. Repository `packages/ubuntu` adalah jalur Stable, sedangkan `packages/mainline/ubuntu` adalah jalur Mainline.

nginx.org

Periksa isi file:

bash

```
cat /etc/apt/sources.list.d/nginx.list
```

Seharusnya tampil:

text

```
deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/ubuntu noble nginx
```

## Langkah 5 — Atur prioritas repository

Buat konfigurasi pinning:

bash

```
sudo tee /etc/apt/preferences.d/99nginx <<'EOF'
Package: *
Pin: origin nginx.org
Pin: release o=nginx
Pin-Priority: 900
EOF
```

Konfigurasi ini membuat APT memprioritaskan paket dari `nginx.org` dibandingkan paket dengan prioritas standar Ubuntu, selama paket tersebut cocok dengan aturan pinning.

Verifikasi:

bash

```
cat /etc/apt/preferences.d/99nginx
```

Mengapa perlu pinning?

Karena VPS tetap menggunakan repository Ubuntu untuk paket sistem lainnya. Pinning membantu memastikan bahwa ketika ada paket Nginx dari Ubuntu dan Nginx resmi, APT lebih memilih paket dari repository yang kita inginkan.

Catatan: pinning tersebut berlaku untuk semua paket yang memenuhi aturan, bukan hanya paket bernama `nginx`. Untuk instalasi khusus ini biasanya tidak menjadi masalah, tetapi jika kamu kelak menambahkan paket lain dari `nginx.org`, periksa kandidat paketnya juga. Konfigurasi ini mengikuti panduan resmi Nginx.

nginx.org

## Langkah 6 — Update indeks dan periksa kandidat versi

Sekarang jalankan:

bash

```
sudo apt update
```

Kemudian:

bash

```
apt policy nginx
```

Periksa dua bagian berikut:

- `Candidate`: versi yang akan dipilih APT.
- `Version table`: daftar versi dan repository asalnya.

Kandidat seharusnya berasal dari:

text

```
https://nginx.org/packages/ubuntu noble/nginx
```

Versi Stable yang tersedia akan berubah seiring waktu. Jadi, jangan mengunci panduan ini pada nomor versi tertentu.

Jika kandidat masih berasal dari `archive.ubuntu.com`, `mirror.biznetgio.com`, atau mirror Ubuntu lainnya, periksa kembali file repository dan hasil `apt policy nginx` sebelum instalasi.

## Langkah 7 — Instal Nginx

Setelah repository dan kandidat versi benar, jalankan:

bash

```
sudo apt install nginx
```

Pada VPS baru yang benar-benar belum memasang Nginx, proses ini akan memasang paket dari repository resmi. Kamu tidak perlu menghapus paket Nginx Ubuntu terlebih dahulu jika memang belum pernah memasangnya.

Setelah selesai, aktifkan layanan:

bash

```
sudo systemctl enable --now nginx
```

Periksa status:

bash

```
sudo systemctl status nginx --no-pager
```

Status yang diharapkan:

text

```
Active: active (running)
```

## Langkah 8 — Verifikasi instalasi

Jalankan ketiga perintah berikut:

bash

```
nginx -v
```

bash

```
sudo nginx -t
```

bash

```
apt policy nginx
```

Hasil yang diharapkan:

- Nginx menampilkan nomor versi yang terpasang.
- Pengujian konfigurasi menyatakan `syntax is ok` dan `test is successful`.
- Kandidat paket berasal dari `nginx.org/packages/ubuntu`.

Kamu juga dapat menguji HTTP lokal:

bash

```
curl -I http://127.0.0.1
```

Jika Nginx berjalan dan konfigurasi default masih aktif, biasanya akan terlihat respons HTTP seperti `200 OK`.

Untuk mengaksesnya dari komputer lain, buka `http://IP_VPS_KAMU` setelah memastikan port TCP 80 diizinkan oleh firewall VPS dan firewall penyedia cloud.

## Langkah 9 — Kelola pembaruan Stable ke depannya

Setelah instalasi berhasil, kamu tidak perlu mengulang semua langkah di atas setiap kali ada rilis baru.

Cukup jalankan secara berkala:

bash

```
sudo apt update
sudo apt upgrade
```

APT akan membaca metadata repository yang dikonfigurasi dan memasang pembaruan sesuai kebijakan paket yang berlaku.

Untuk melihat apakah ada versi baru Nginx:

bash

```
apt policy nginx
```

Untuk melihat versi yang sedang berjalan:

bash

```
nginx -v
```

Jika ingin melihat paket apa saja yang akan diperbarui sebelum menginstalnya:

bash

```
sudo apt list --upgradable
```

Penting: perintah `apt upgrade` dapat memperbarui paket sistem lain, bukan hanya Nginx. Untuk memperbarui Nginx secara khusus, kamu dapat menjalankan:

bash

```
sudo apt install --only-upgrade nginx
```

Perintah ini tidak menjamin pembaruan otomatis di latar belakang; kamu tetap perlu menjalankannya atau mengatur mekanisme pembaruan otomatis.

## Langkah 10 — Jika ingin pembaruan otomatis

Ubuntu dapat menggunakan `unattended-upgrades` untuk memasang pembaruan tanpa harus selalu menjalankan perintah secara manual.

Instal paketnya:

bash

```
sudo apt install unattended-upgrades
```

Periksa konfigurasi:

bash

```
cat /etc/apt/apt.conf.d/50unattended-upgrades
```

Namun, jangan langsung menganggap Nginx Stable dari `nginx.org` otomatis masuk ke daftar sumber pembaruan yang diizinkan. Konfigurasi `Unattended-Upgrade::Origins-Pattern` perlu diperiksa dan disesuaikan agar paket dari repository Nginx dapat diproses. Kamu juga perlu mempertimbangkan apakah ingin semua pembaruan dipasang otomatis atau hanya pembaruan keamanan.

Untuk VPS produksi, pendekatan yang lebih aman sering kali adalah menjadwalkan pembaruan, memeriksa perubahan paket, lalu melakukan upgrade pada waktu pemeliharaan.


Kesimpulan: untuk VPS baru, yang perlu kamu kelola hanya repository Stable, kunci GPG, aturan prioritas, dan pembaruan paket. Setelah repository Stable dipasang dengan benar, kamu tidak perlu mengunduh Nginx secara manual setiap kali ada versi baru. APT akan mengambil versi terbaru yang tersedia dari jalur Stable ketika kamu menjalankan pembaruan.

Sumber utama: [NGINX — Linux packages resmi](<https://nginx.org/en/linux_packages.html?utm_source=chatgpt.com>).
