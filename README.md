# Turnstile, cf_clearance, & AWS WAF Token Solver

> Terinspirasi dari [SGAHSCAJASCJ/Turnstile-Solver](https://github.com/SGAHSCAJASCJ/Turnstile-Solver)

Solusi pemecahan CAPTCHA Cloudflare Turnstile, cf_clearance, dan AWS WAF Token berkinerja tinggi yang dibangun dengan **FastAPI** dan teknologi browser asinkron (**Camoufox**), menyediakan layanan RESTful API yang siap dipakai.

---

## ✨ Fitur Utama

- **3 Endpoint Solver**: `/turnstile`, `/clearance`, `/aws-token`
- **Auto Install & Fetch**: Dependensi Python dan Camoufox diinstall otomatis saat pertama kali jalan.
- **Konfigurasi via `config.json`**: Semua setting dapat diatur dari file atau prompt interaktif.
- **Proxy Rotation**: Dukungan proxy per-instance browser dengan rotasi round-robin.
- **Forced Cleanup**: Cleanup memori berkala paksa untuk kestabilan server di VPS ber-RAM kecil.
- **Mode Headless & GUI**: Kompatibel untuk dijalankan via Terminal/VPS (`xvfb`) maupun RDP.

---

## 🚀 Instalasi & Setup (Khusus VPS Baru)

**Catatan:** *Masalah instalasi berulang (`camoufox fetch` error atau browser dependensi) di VPS baru biasanya disebabkan karena cache data browser yang belum lengkap.* Script versi terbaru sudah memperbaiki deteksi versi camoufox otomatis.

Langkah-langkah yang **paling disarankan** di VPS Linux (Ubuntu/Debian) baru:

```bash
# 1. Update system & Install system dependencies browser
sudo apt update -y && sudo apt upgrade -y
sudo apt install xvfb -y
sudo apt install libasound2 -y
sudo apt install python3 -y
sudo apt install python3-pip -y
sudo apt install python3-venv -y

# 2. Buat & Aktifkan virtual environment (sangat disarankan)
python3 -m venv venv
source venv/bin/activate

# 3. Clone Repository
git clone https://github.com/najibyahya/Turnstile-Solver
cd Turnstile-Solver

# 4. Install dependensi Python dasar
pip install fastapi==0.95.2 uvicorn "camoufox[fetch]" loguru psutil playwright

# 5. FETCH & INSTALL DEPENDENCY MANUAL (LAKUKAN SEKALI SAJA)
# Ini mencegah masalah "Version information not found" & "browser dependencies"
python3 -m camoufox fetch
python3 -m playwright install-deps

# 6. Jalankan Server
python3 api_server.py
```

> **INFO:** Pada percobaan berikutnya, Anda cukup melakukan:
> ```bash
> source venv/bin/activate
> xvfb-run -a python3 api_server.py
> ```

---

## ⚙️ Konfigurasi (`config.json`)

Ketika pertama kali dijalankan, script akan membuat `config.json`. Anda bisa mengubahnya langsung:

```json
{
    "headless":      true,
    "thread":        2,
    "page_count":    1,
    "proxy_support": false,
    "proxy_file":    "proxies.txt",
    "host":          "0.0.0.0",
    "port":          8000,
    "debug":         false,
    "cleanup_interval_minutes": 10
}
```

| Parameter | Tipe | Default | Keterangan |
|---|---|---|---|
| `headless` | bool | `true` | Browser berjalan tanpa tampilan GUI |
| `thread` | int | `2` | Jumlah instance browser (disarankan max = jumlah core CPU) |
| `page_count` | int | `1` | Jumlah tab/halaman per browser |
| `proxy_support` | bool | `false` | Status penggunaan daftar proxy dari `proxies.txt` |
| `cleanup_interval_minutes` | int | `10` | Jeda waktu sistem merefresh/membersihkan memori browser |

---

## 🌐 Format Proxy (`proxies.txt`)

Jika `proxy_support` dihidupkan, tambahkan proxy pada file `proxies.txt` (satu proxy tiap baris).
Format yang didukung:
```text
http://ip:port
http://user:pass@ip:port
socks5://user:pass@ip:port
```

---

## 📖 Endpoint Dokumentasi API

Solver ini bekerja dengan cara asynchronous (membuat antrean tugas). Masing-masing endpoint memblokir proses sampai token sukses diambil atau dikembalikan dalam status gagal.

### 1. Endpoint Task Pembuatan

| Task | Endpoint | Parameter yang dibutuhkan |
|---|---|---|
| Turnstile | `GET /turnstile` | `url` (Target URL), `sitekey` (Turnstile Key) |
| cf_clearance | `GET /clearance` | `url` (Target URL), `timeout` (opsional batas detik) |
| AWS WAF | `GET /aws-token` | `url` (Target URL), `timeout` (opsional batas detik) |

**Contoh Response Sukses (202 Accepted):**
```json
{
  "task_id": "8a31e...b41",
  "status": "accepted"
}
```

### 2. Endpoint Polling Hasil (`GET /result?id=<task_id>`)

Anda wajb melakukan **polling request** ke endpoint ini tiap (minimal) 1 detik menggunakan `task_id` dari pembuatan task di atas sampai `status` bernilai `success` atau `error`.

**Contoh Response Sukses dari Turnstile:**
```json
{
  "status": "success",
  "elapsed_time": 2.431,
  "value": "0.AbCdEf..."
}
```

**Contoh Response Sukses dari cf_clearance/AWS WAF:**
```json
{
  "status": "success",
  "elapsed_time": 3.102,
  "user_agent": "Mozilla/5.0 ...",
  "cookies": "cf_clearance=abcdef...;",
  "cf_clearance": "abcdef..."
}
```

**Kode Status HTTP:**
- `200` = Sukses.
- `202` = Sedang diproses, terus lakukan request GET /result.
- `404` = Task ID kadaluarsa atau tidak ditemukan.
- `408` = Time Out (> 5 Menit memutar).
- `500`/`422` = Eror di internal atau Captcha gagal disolve.

---

## ❗ Troubleshooting

| Masalah | Solusi |
|---|---|
| `Version information not found at /root/.cache/camoufox/version.json` | Script sudah di-update untuk mencegah ini. Jika masih terjadi, jalankan manual `python3 -m camoufox fetch`. |
| Berulang menginstall `playwright install-deps` | Pastikan Anda memiliki hak *sudo/root* jika package diinstall via sistem, dan jalankan perintah install pip menggunakan Virtual Environment (`venv`), lalu install manual via `python3 -m playwright install-deps` |
| `Pool halaman berisi 0 / task tak selesai` | RAM penuh. Kurangi jumlah `thread` dan pastikan RAM longgar minimal ±300 MB per-thread. |

---

## 📄 Lisensi
MIT License — Lihat [LICENSE](LICENSE).

<div align="center">
<b>⚡ Performa Tinggi &nbsp;|&nbsp; 🚀 Multi Solver &nbsp;|&nbsp; 🛡️ Camoufox Powered</b>
</div>
