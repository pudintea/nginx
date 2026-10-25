# KONFIGURASI REVERSE PROXY NGINX 1.30.6 STABLE

FIle ada di 
/etc/nginx/conf.d/
```
cd /etc/nginx/conf.d/
```
Buat File :
```
nano web.pudin.eu.org
```

Masukan :
```
server {
    listen 80;
    server_name web.pudin.eu.org;

    # Redirect HTTP ke HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    http2 on;
    server_name web.pudin.eu.org;

    # Sertifikat SSL dari Cloudflare
    ssl_certificate     /etc/cloudflare/pudin.eu.org.pem;
    ssl_certificate_key /etc/cloudflare/pudin.eu.org.key;

    # Daftar IPv4 Cloudflare
    set_real_ip_from 173.245.48.0/20;
    set_real_ip_from 103.21.244.0/22;
    set_real_ip_from 103.22.200.0/22;
    set_real_ip_from 103.31.4.0/22;
    set_real_ip_from 141.101.64.0/18;
    set_real_ip_from 108.162.192.0/18;
    set_real_ip_from 190.93.240.0/20;
    set_real_ip_from 188.114.96.0/20;
    set_real_ip_from 197.234.240.0/22;
    set_real_ip_from 198.41.128.0/17;
    set_real_ip_from 162.158.0.0/15;
    set_real_ip_from 104.16.0.0/13;
    set_real_ip_from 104.24.0.0/14;
    set_real_ip_from 172.64.0.0/13;
    set_real_ip_from 131.0.72.0/22;

    # Daftar IPv6 Cloudflare
    set_real_ip_from 2400:cb00::/32;
    set_real_ip_from 2606:4700::/32;
    set_real_ip_from 2803:f800::/32;
    set_real_ip_from 2405:b500::/32;
    set_real_ip_from 2405:8100::/32;
    set_real_ip_from 2a06:98c0::/29;
    set_real_ip_from 2c0f:f248::/32;

    # Header spesifik dari Cloudflare yang memuat IP asli pengunjung
    real_ip_header CF-Connecting-IP;

    # Optimasi Keamanan SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # Hilangkan informasi server dari header
    server_tokens off;
    proxy_hide_header Server;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
        # Ganti port 3000 dengan port tempat aplikasi backend Anda berjalan
        proxy_pass http://127.0.0.1:3000;

        # Meneruskan informasi klien ke server backend
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```


Mengelola Nginx setelah instalasi
==================================
Perintah yang paling sering digunakan:

# Memeriksa konfigurasi
```
sudo nginx -t
```
# Reload konfigurasi tanpa menghentikan layanan
```
sudo nginx -s reload
sudo systemctl reload nginx
```
# Restart layanan
```
sudo systemctl restart nginx
```
# Melihat log error secara langsung
```
sudo tail -f /var/log/nginx/error.log
```
# Memperbarui paket dari repository
```
sudo apt update && sudo apt install --only-upgrade nginx
```

## BY PUDIN SAEPUDIN
