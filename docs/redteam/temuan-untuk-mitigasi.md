# Ringkasan Temuan Red Team — untuk Mitigasi Blue Team

**Tanggal serangan:** 26 November 2026
**Target:** `172.20.4.2` (replika ThingsBoard) | **Sumber:** Attacker Node Kali
**Skenario:** S1 Recon, S2 Exploitation (SSH+PostgreSQL), S3 Web/API (:80/:9090)
**Bukti lengkap:** `docs/redteam/evidence/` (10 file + index)

## Temuan yang PERLU mitigasi (4)

| # | Temuan | Detail | Kategori | Prioritas |
|---|--------|--------|----------|-----------|
| 1 | **Tidak ada TLS** | `https://172.20.4.2:9090` gagal handshake; seluruh trafik dashboard — termasuk kredensial login — berjalan plaintext HTTP di :80 dan :9090 | C — enkripsi | TINGGI |
| 2 | **Swagger UI publik** | `/swagger-ui/index.html` bisa diakses tanpa login. Versi aplikasi terbaca: **ThingsBoard 4.2.1.1**. Peta API lengkap terekspos (admin, audit-log, ai-model, dll.) | C — information disclosure | SEDANG |
| 3 | **Password policy lemah** | `/api/noauth/userPasswordPolicy` (200 tanpa auth): minimum 6 karakter, tanpa wajib digit/uppercase/special, allowWhitespaces=true, forceUserToResetPasswordIfNotValid=false, tanpa expiration/reuse | A — access & auth | SEDANG |
| 4 | **Security headers absen di static frontend + /actuator terekspos** | Halaman utama :80/:9090 tanpa `X-Frame-Options` & `X-Content-Type-Options` (respons `/api/*` sudah benar). `/actuator` menjawab 200 (hanya self+info, tapi sebaiknya ditutup) | C — application | RENDAH |

## Rekomendasi mitigasi per temuan

1. **TLS**: aktifkan HTTPS di Tomcat (self-signed certificate cukup untuk lab) atau tempatkan reverse proxy TLS di depan :80/:9090. Minimal: dokumentasikan keputusan bila lab tidak mengizinkan.
2. **Swagger**: batasi `/swagger-ui/**` + `/v3/api-docs/**` dengan auth (mis. basic auth IP whitelist) atau nonaktifkan di environment produksi-like.
3. **Password policy**: naikkan `minimumLength` ke ≥8, wajibkan `minimumDigits` + `minimumUppercaseLetters`, aktifkan `passwordExpirationPeriodDays`, set `forceUserToResetPasswordIfNotValid=true`.
4. **Headers + actuator**: tambah `X-Frame-Options: DENY` + `X-Content-Type-Options: nosniff` di layer static (respons /api/* jangan diubah), tutup atau proteksi `/actuator/**`.

## Pertahanan yang SUDAH terverifikasi baik (7) — pertahankan

1. SSH :2245 key-only — password auth ditolak di level protokol (hydra: `does not support password authentication`)
2. Kredensial default ThingsBoard diganti (`Invalid username or password`)
3. Kredensial default PostgreSQL diganti + password tidak di top-100 dunia
4. Rate-limit/throttle aktif di PostgreSQL — burst diputus, pelan dibatasi ~1 detik/percobaan
5. Semua endpoint API sensitif terkunci JWT; WS telemetry menolak subscribe tanpa AUTH (`1008 policy violation`)
6. Endpoint token (activate/reset) respons seragam — tahan enumerasi
7. Tomcat manager dimask, HTTP method terbatas GET/HEAD/OPTIONS, UDP tertutup

## Verifikasi pasca-mitigasi (Tahap 5 — Red Team akan ulangi)

Setelah mitigasi terpasang, Red Team akan menjalankan ulang dan mengharapkan:
```
curl -i https://172.20.4.2:9090            → handshake TLS berhasil
curl -i http://172.20.4.2:9090/swagger-ui/index.html → 401/403 atau redirect login
curl http://172.20.4.2:9090/api/noauth/userPasswordPolicy → policy diperketat
nikto -h http://172.20.4.2:9090           → tidak lagi melaporkan 2 header hilang
curl -i http://172.20.4.2:9090/actuator   → 404/401
```
Hasil ulangan dicatat di `docs/purple-team-log.md` sebagai penutup Purple Team Loop.
