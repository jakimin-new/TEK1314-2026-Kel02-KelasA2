# Proyek Simulasi Jaringan - Cyber Range

## Anggota Kelompok 2 Kelas A
1. Dzaky Naufal Putra 
Raditio (J0404241020) as Blue Team
2. Siti Syafa Laela (J0404241053) as Lead
3. Gde Saputra 
Pramana Artha(J0404241107) as Red Team
4. Muhamad Arif Dermawan (J0404241134) as Red Team

## Deskripsi Skenario

Skenario yang dipilih adalah **Web & Service Exploitation (DVWA + SSH Brute Force)**.

Skenario ini mensimulasikan proses eksploitasi terhadap sebuah target server yang berjalan di atas **Metasploitable 2**, dengan dua celah keamanan utama yang diuji:

- **Port 80/TCP (HTTP)** – Eksploitasi terhadap aplikasi web rentan **DVWA (Damn Vulnerable Web Application)**.
- **Port 22/TCP (SSH)** – Pengujian keamanan melalui **weak password testing / brute force**.

**Alasan pemilihan OS:** Metasploitable 2 dipilih karena hemat penggunaan RAM dan sudah menyediakan service yang rentan secara bawaan (built-in), sehingga tidak memerlukan konfigurasi manual yang rumit untuk membangun environment pengujian.

## Struktur Repositori

```
docs/
└── design/
    ├── topology.png   # Desain jaringan (disusun oleh Blue Team)
    └── ip_plan.md      # Tabel Hostname, IP Address, dan OS yang direncanakan
README.md
```

## Ringkasan Topologi

Jaringan terbagi menjadi beberapa zona yang dipisahkan oleh firewall/router pada **Intermediary Zone**:

- **Attacker Zone** – berisi node Kali Linux / CyberOps untuk melakukan serangan.
- **DMZ Zone** – berisi target server (Web/DB) yang menjalankan Metasploitable 2.
- **Internal Zone** – berisi server dan workstation internal.
- **Monitoring Zone** – berisi Security Onion Node untuk memantau trafik.
- **VPN Tunneling** – menghubungkan client ke Intermediary Zone secara aman.

Detail pembagian IP Address untuk setiap node dapat dilihat pada [`docs/design/ip_plan.md`](docs/design/ip_plan.md).
