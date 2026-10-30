# LAPORAN GANGGUAN NETWORK

periksa routing table
```
ip -br addr
```
```
ip -br link
```
```
ip route
```
```
ip route get 8.8.8.8
```
```
ip rule show
```
Pengujian - Lihat paket yang keluar dan masuk
```
sudo tcpdump -ni ens3 'arp or icmp'
```
Satu pengujian pembanding yang sangat berguna
```
ping -4 -c 5 1.1.1.1
ip route get 1.1.1.1
```
Langkah berikutnya: pastikan paket keluar dari server
Ini pemeriksaan paling penting sekarang.

Di server .21, buka terminal pertama dan jalankan:
```
sudo tcpdump -ni ens3 -vv 'icmp and (host 1.1.1.1 or host 8.8.8.8)'
```

Baik. Berarti kondisi kita sekarang:

- Platform: Cloud.
- Akses gateway: melalui tim network.
- Server `.99`: normal mengakses internet.
- Server `.21`: bisa berkomunikasi dengan gateway, tetapi tidak bisa mengakses IP publik melalui ICMP maupun TCP.

Dengan kondisi ini, saya menyarankan jangan melakukan perubahan konfigurasi Debian dulu. Kita perlu meminta tim network melakukan pemeriksaan pada jalur egress server `.21`.

## 1. Dugaan penyebab paling kuat

Urutan yang perlu diperiksa:

1. Firewall atau ACL yang membatasi trafik keluar berdasarkan IP sumber `192.168.101.21`.
2. SNAT/NAT tidak berlaku untuk IP `.21`.
3. Kebijakan egress pada virtual network atau cloud port berbeda.
4. Routing balik dari jaringan upstream menuju `.21` bermasalah.

DNS kemungkinan juga terdampak karena akses DNS keluar tidak berhasil, bukan berarti DNS adalah akar masalahnya.

## 2. Pesan untuk tim network

Saya buatkan pesan teknis yang bisa langsung Anda kirimkan.

Menulis

Halo Tim Network,

Mohon bantu investigasi konektivitas internet pada server berikut:

**Server bermasalah**

- IP: `192.168.101.21`
- Gateway: `192.168.101.1`
- Interface: `ens3`
- MAC: `fa:16:3e:f1:68:d5`

**Server pembanding (normal)**

- IP: `192.168.101.99`
- Gateway: `192.168.101.1`

Kedua server berada pada jaringan yang sama dan menggunakan gateway yang sama. Server `.99` dapat mengakses internet secara normal, sedangkan server `.21` tidak bisa.

Hasil investigasi dari server `.21`:

1. Ping gateway `192.168.101.1` berhasil, 0% packet loss.
2. ARP gateway berhasil dan status neighbor `REACHABLE`.
3. Default route sudah benar: `default via 192.168.101.1 dev ens3`
4. Ping ke `1.1.1.1` dan `8.8.8.8` gagal, 100% packet loss.
5. Koneksi HTTPS ke `1.1.1.1:443` mengalami connection timeout.
6. DNS lookup `google.com` mengalami resolving timeout.
7. Hasil tcpdump pada interface `ens3` menunjukkan ICMP Echo Request dari `192.168.101.21` benar-benar keluar, tetapi tidak ada Echo Reply yang kembali.
8. Firewall iptables INPUT dan OUTPUT menggunakan policy ACCEPT tanpa aturan tambahan.

Mohon dilakukan pengecekan dan perbandingan konfigurasi server `.21` dengan `.99`, khususnya:

- Egress firewall dan ACL berdasarkan source IP.
- SNAT/MASQUERADE untuk `192.168.101.21`.
- Cloud network port security dan kebijakan egress.
- Routing dan return path.
- NAT gateway dan external network.
- Apakah ada kebijakan isolasi atau pembatasan khusus pada IP `192.168.101.21`.

Mohon juga dilakukan packet capture untuk memastikan apakah trafik dari `.21` diterima oleh gateway dan diteruskan ke jaringan eksternal.

Terima kasih.

## 3. Sambil menunggu tim network, lakukan satu pemeriksaan dari Debian

Jalankan di `.21`:

bash

```
ip route get 1.1.1.1
cat /etc/resolv.conf
ip neigh show
```

Kemudian jalankan:

bash

```
curl -4 -Iv --connect-timeout 5 https://1.1.1.1
```

Simpan hasilnya sebagai bukti tambahan.

## 4. Permintaan khusus kepada tim network

Jika tim network bisa melakukan packet capture, minta mereka menguji tiga titik:

| Titik pemeriksaan | Yang harus dibuktikan |
| --- | --- |
| Gateway interface LAN | Apakah paket dari `.21` diterima? |
| Gateway interface WAN | Apakah paket diteruskan dan sudah di-SNAT? |
| Gateway interface WAN (arah balik) | Apakah response dari internet kembali? |

Ini akan membantu menemukan titik putus tanpa menebak-nebak.

Satu hal yang ingin saya tekankan: karena `.99` normal dan `.21` gagal, padahal gateway dan subnet sama, jangan buru-buru mengubah konfigurasi Debian `.21`. Kita sudah punya cukup bukti untuk meminta investigasi jaringan secara spesifik.

Jika tim network memberikan hasil pengecekan firewall, NAT, atau packet capture, kirimkan ke sini. Saya bisa bantu membaca hasilnya dan menentukan perbaikan berikutnya.
