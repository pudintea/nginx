# INSTALL FAIL2BAN UBUNTU - SSH

### 1. Instalasi Fail2ban

Jalankan perintah ini di terminal server Anda (berbasis Ubuntu/Debian):

```bash
sudo apt update
sudo apt install fail2ban -y

```

Aktifkan agar Fail2ban otomatis berjalan saat server di-*restart*:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

```

### 2. Konfigurasi Keamanan (Jail)

Jangan pernah mengedit file bawaan `jail.conf` karena akan tertimpa saat *update*. Buat salinan konfigurasi baru bernama `jail.local`:

```bash
sudo nano /etc/fail2ban/jail.local

```

Masukkan kode berikut ke dalam file tersebut. Kode ini akan memblokir penyerang SSH selama 24 jam jika mereka salah memasukkan *password* 3 kali berturut-turut.

```ini
[DEFAULT]
# IP yang tidak akan pernah diblokir (masukkan IP statis rumah/kantor Anda jika ada, pisahkan dengan spasi)
ignoreip = 127.0.0.1/8 ::1
# Rentang waktu pengecekan (10 menit)
findtime = 10m
# Waktu hukuman default (1 jam)
bantime = 1h
# Maksimal percobaan gagal default
maxretry = 5

[sshd]
enabled = true
port = ssh
filter = sshd
# Sesuaikan logpath jika menggunakan OS selain Ubuntu/Debian (misal CentOS: /var/log/secure)
logpath = /var/log/auth.log
# Aturan ketat untuk SSH
maxretry = 3
bantime = 24h

```

Simpan file dan keluar dari editor (`Ctrl+O`, `Enter`, lalu `Ctrl+X`).

### 3. Terapkan dan Pantau Status

Muat ulang layanan Fail2ban untuk membaca konfigurasi baru:

```bash
sudo systemctl restart fail2ban

```

Untuk melihat apakah Fail2ban sudah berjalan dan memantau penjara (*jail*) SSH:

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd

```

Nantinya, `status sshd` akan menampilkan daftar IP penyerang yang sedang diblokir.

### 4. Mengamankan Akses Web (Rekomendasi)

Alih-alih menggunakan Fail2ban untuk NGINX, amankan web dari *bruteforce* langsung melalui *dashboard* Cloudflare Anda:

1. Buka menu **Security** > **WAF** di Cloudflare.
2. Buat aturan (*Custom Rule*) untuk menantang (*Managed Challenge*) atau memblokir akses ke halaman login sensitif (misalnya URI Path berisi `/login` atau `/admin`) jika datang dari negara selain Indonesia atau memiliki *Threat Score* tinggi.
3. Gunakan menu **Security** > **Bots** dan aktifkan **Bot Fight Mode** untuk menahan *bruteforce* otomatis.

### Tambahan, Penanganan error
fail2ban.configreader   [35085]: WARNING 'allowipv6' not defined in 'Definition'. Using default one: 'auto'

```
sudo nano /etc/fail2ban/fail2ban.conf
```

Cari bagian:
```
[Definition]
```

Tambahkan parameter berikut di bagian tersebut jika belum ada:
```
allowipv6 = auto
```

Simpan, lalu periksa konfigurasi dan restart Fail2ban:
```
sudo fail2ban-client -t
```
```
sudo systemctl restart fail2ban
```
```
sudo systemctl status fail2ban
```

