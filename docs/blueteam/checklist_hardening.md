| KETENTUAN HARDENING WAJIB |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| No | Kategori | Kontrol | Checklist | Bukti | Ketentuan |
| 1.0 | Access & Authentication | Nonaktifkan login root langsung | TRUE | https://docs.google.com/document/d/1Arlg000cuF_QwnHMuAeEAZ5zim3Rj0Hrakm03siFmDw/edit?usp=sharing | Screenshot config file (before/after) + hasil percobaan login gagal setelah hardening |
|  |  | SSH key-based auth (disable password auth) | TRUE | https://docs.google.com/document/d/1ZE4BerdBVXjAqsPFkMVWSjno5jW1iyPlMvQtpKrqpeo/edit?usp=sharing |  |
|  |  | Ganti port default; fail2ban / rate-limit percobaan login | TRUE | https://docs.google.com/document/d/19cLGv3b_y3rWoDEq850v4OuLJmWA-mSkpvL-f2H7DtM/edit?usp=sharing |  |
|  |  | MFA pada dashboard monitoring | FALSE |  |  |
| 2.0 | Service & Attack Surface | Nonaktifkan service/port yang tidak dipakai | TRUE | https://docs.google.com/document/d/1GWNxE2TOF4mTewCqkcKzfRFtQDSWYyrM1DPDnh_GW7c/edit?usp=sharing | Output nmap scan before vs  after (jumlah port terbuka  berkurang) |
|  |  | Firewall (ufw/iptables) whitelist port perlu saja | TRUE | https://docs.google.com/document/d/1ofta7-eDC0SlGr8x18q1vcayIQrMD3Rrcwz2SAuVFhw/edit?usp=sharing |  |
|  |  | Update & patch OS/paket ke versi aman | TRUE | https://docs.google.com/document/d/11lyycjBhD7Jg2Wq9J73RvB3Rp4hsIl1_l6fknAjTK-0/edit?usp=sharing |  |
|  |  | Hapus akun/kredensial default | FALSE |  |  |
| 3.0 | Data & Application | Enkripsi/kredensial tidak hardcoded (env var/secrets) | TRUE | https://docs.google.com/document/d/1iFvborawE8K2d3Du3wLNbTYeBmW7uevmz2npbMY6amM/edit?tab=t.0 | Snippet config/kode + log yang menunjukkan kontrol aktif |
|  |  | Validasi input pada endpoint API/MQTT topic | FALSE |  |  |
|  |  | Least privilege pada user DB | FALSE |  |  |
|  |  | Logging aktivitas mencurigakan diaktifkan | TRUE | https://docs.google.com/document/d/1iFvborawE8K2d3Du3wLNbTYeBmW7uevmz2npbMY6amM/edit?tab=t.gwl8g07lhvku |  |