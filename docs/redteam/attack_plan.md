# Attack Plan — Red Team Fase 2

Target: `172.20.4.2` | Sumber: `172.20.4.4` | Monitoring: `172.20.4.3`
Hasil scan 22 Okt 2026: 22/open OpenSSH 10.0p2; 9090/open Apache Tomcat (ThingsBoard, HTTP 200); 5432/open PostgreSQL 9.6+.

| # | Kategori | Teknik | Tools | Target |
|---|----------|--------|-------|--------|
| 1 | Reconnaissance | Port/service scanning, OS fingerprinting, enumerasi banner | nmap, Wireshark | 172.20.4.2 (full TCP) |
| 2 | Exploitation / Akses | Brute-force login SSH + PostgreSQL, cek kredensial default, cek CVE OpenSSH 10.0p2 | Hydra, Nmap NSE (ssh-brute, pgsql-brute), Metasploit | 172.20.4.2:22, 172.20.4.2:5432 |
| 3 | Web/API/Protokol IoT | Enum endpoint, uji SQLi/XSS terkontrol, uji auth tanpa token, cek Tomcat manager | Burp Suite / OWASP ZAP, curl, Nikto | 172.20.4.2:9090 |


