# Attack Plan — Red Team Fase 2

Target: `172.20.4.2` \| Sumber: `172.20.4.4` \| Monitoring: `172.20.4.3` Hasil scan 22 Okt 2026: 22/open OpenSSH 10.0p2; 9090/open Apache Tomcat (ThingsBoard, HTTP 200); 5432/open PostgreSQL 9.6+.

| \# | Kategori | Teknik | Tools | Target |
| :---- | :---- | :---- | :---- | :---- |
| 1 | Reconnaissance | Port/service scanning, OS fingerprinting, enumerasi banner | nmap, Wireshark | 172.20.4.2 (full TCP) |
| 2 | Exploitation / Akses | Brute-force login SSH \+ PostgreSQL, cek kredensial default, cek CVE OpenSSH 10.0p2 | Hydra, Nmap NSE (ssh-brute, pgsql-brute), Metasploit | 172.20.4.2:22, 172.20.4.2:5432 |
| 3 | Web/API/Protokol IoT | Enum endpoint, uji SQLi/XSS terkontrol, uji auth tanpa token, cek Tomcat manager | Burp Suite / OWASP ZAP, curl, Nikto | 172.20.4.2:9090 |

## Perbedaan plan vs eksekusi (26 November 2026\)

Serangan dieksekusi 26 Nov 2026\. Tiga kategori di atas semuanya berjalan. Beberapa hal berubah saat eksekusi, sebagian karena kondisi target berubah, sebagian karena temuan baru membuka jalur lain.

### Yang berubah dari plan

| Plan | Eksekusi | Alasan |
| :---- | :---- | :---- |
| SSH di port 22 | SSH di port 2245 | Full scan 26 Nov menemukan SSH dipindah; port 22 tidak lagi terbuka |
| Hasil scan 22 Okt: 22, 9090, 5432 | 80, 2245, 5432, 9090 | Port 80 (Apache Tomcat) terbuka saat scan ulang 26 Nov |
| Metasploit, Wireshark | Tidak dipakai | Hydra dan NSE menjawab semua pertanyaan brute-force; tidak ada kebutuhan sniffing |
| Burp Suite / OWASP ZAP | Tidak dipakai | Enum endpoint manual dengan curl plus nikto menutup checklist web |
| Uji SQLi/XSS terkontrol | Tidak dilakukan | Versi teridentifikasi 4.2.1.1 via Swagger; CVE-2025-34281 (XSS Image Gallery) sudah tertutup di versi ini |

### Tidak ada di plan tapi dijalankan

Kami menjalankan lima hal tambahan setelah temuan S3 membuka jalan: enumerasi Swagger UI, cek kebijakan password lewat `/api/noauth/userPasswordPolicy`, uji WebSocket telemetry, cek `/actuator`, dan brute PostgreSQL dengan wordlist rockyou (5.000 lalu 100 password terpopuler). Sebagian besar temuan akhir datang dari sini: tidak ada TLS, versi aplikasi bocor, kebijakan password lemah, dan rate-limit aktif.

&nbsp;