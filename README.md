# Turnstile CAPTCHA & Clearance Solver

> Terinspirasi dari [SGAHSCAJASCJ/Turnstile-Solver](https://github.com/SGAHSCAJASCJ/Turnstile-Solver)

Solusi pemecahan CAPTCHA Cloudflare Turnstile, cf_clearance, dan AWS WAF Token berkinerja tinggi yang dibangun dengan **FastAPI** dan teknologi browser asinkron (**Camoufox**), menyediakan layanan RESTful API yang siap dipakai.

---

## ✨ Fitur

| Fitur | Keterangan |
|---|---|
| 🔄 3 Endpoint Solver | `/turnstile`, `/clearance`, `/aws-token` |
| 📦 Auto Install | Dependensi & browser Camoufox diinstall otomatis |
| 🖥️ VPS Auto Setup | Deteksi & install python, pip, venv, browser deps otomatis di Linux |
| ⚙️ Konfigurasi via `config.json` | Semua setting dari satu file + prompt interaktif |
| 🔁 Proxy Rotation | Dukungan proxy per-thread dengan rotasi round-robin |
| 🧹 Forced Cleanup | Cleanup berkala paksa (interval bisa diatur) meski ada worker aktif |
| 🐛 Mode Debug | Aktifkan/matikan log detail via config |
| 🪟 Windows / Linux / RDP | Kompatibel penuh di semua platform |

---

## 📊 Metrik Performa

| Metrik | Nilai | Keterangan |
|---|---|---|
| Kapasitas Konkurensi | 500+ req/menit | Pool halaman asinkron |
| Rata-rata Waktu Respons | 1,8 – 3 detik | Rata-rata per captcha |
| Tingkat Keberhasilan | 99%+ | Dalam kondisi normal |
| Penggunaan Memori | ~300 MB/halaman | Per instance browser |

---

## 🚀 Mulai Cepat

### Persyaratan
- Python 3.10+
- Windows / Linux / macOS / RDP
- RAM 2 GB+
- Koneksi internet stabil

### Instalasi & Menjalankan

Ada **dua server** yang bisa dijalankan:

| Server | File | Endpoint | Keterangan |
|---|---|---|---|
| **Solver** | `api_server.py` | `/turnstile`, `/clearance`, `/aws-token` | Solver lengkap (3 endpoint) |

```bash
git clone https://github.com/najibyahya/Turnstile-Solver
cd Turnstile-Solver

# Server lengkap (turnstile + clearance + aws-token)
python api_server.py
```

> Script akan otomatis menginstall semua dependensi (`fastapi==0.95.2`, `uvicorn`, `camoufox`, `loguru`, `psutil`) dan mengunduh browser Camoufox jika belum ada.

---

## 🖥️ Setup VPS Linux Baru

Script `api_server.py` sudah memiliki **auto-setup** yang akan mengecek dan menginstall kebutuhan VPS secara otomatis. Namun jika ingin setup manual:

```bash
# 1. Update system
sudo apt update -y && sudo apt upgrade -y

# 2. Install Python 3.10+ (biasanya sudah ada di Ubuntu 22.04+)
sudo apt install python3 python3-pip python3-venv -y

# 3. Buat & aktifkan virtual environment
python3 -m venv venv
source venv/bin/activate

# 4. Install module
pip install fastapi==0.95.2 uvicorn camoufox loguru psutil

# 5. Install browser dependencies
python3 -m playwright install-deps

# 6. Clone & jalankan
git clone https://github.com/najibyahya/Turnstile-Solver
cd Turnstile-Solver
python3 api_server.py
```

> Jika menggunakan `headless: false`, jalankan dengan `xvfb-run -a python3 api_server.py`

---

## ⚙️ Konfigurasi (`config.json`)

Edit file `config.json` sesuai kebutuhan **sebelum** menjalankan script, atau ubah langsung via prompt interaktif saat script berjalan.

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
| `headless` | bool | `true` | Browser berjalan tanpa tampilan (mode server) |
| `thread` | int | `2` | Jumlah instance browser — jangan melebihi jumlah core CPU |
| `page_count` | int | `1` | Jumlah halaman per instance browser |
| `proxy_support` | bool | `false` | Aktifkan penggunaan proxy |
| `proxy_file` | string | `proxies.txt` | Path file daftar proxy |
| `host` | string | `0.0.0.0` | Host binding server |
| `port` | int | `8000` | Port server |
| `debug` | bool | `false` | Tampilkan log DEBUG detail |
| `cleanup_interval_minutes` | int | `10` | Interval cleanup paksa dalam menit |

---

## 🌐 Penggunaan Proxy

### 1. Aktifkan di `config.json`
```json
{
    "proxy_support": true,
    "proxy_file": "proxies.txt"
}
```

### 2. Isi `proxies.txt` (satu proxy per baris)
```
# Tanpa autentikasi
http://203.0.113.10:3128
socks5://203.0.113.20:1080

# Dengan autentikasi
http://username:password@203.0.113.30:8080
socks5://username:password@203.0.113.40:1080
```

Proxy akan dirotasi secara **round-robin** — setiap thread browser mendapat proxy yang berbeda.

---

## 📖 Dokumentasi API

### ➡️ Solve Turnstile CAPTCHA
```http
GET /turnstile?url=https://example.com&sitekey=0x4AAAAAAA...
```

| Parameter | Wajib | Keterangan |
|---|---|---|
| `url` | ✅ | URL halaman tempat Turnstile berada |
| `sitekey` | ✅ | Sitekey Turnstile dari halaman tersebut |
| `action` | ❌ | Nilai `data-action` (opsional) |
| `cdata` | ❌ | Nilai `data-cdata` (opsional) |

---

### ➡️ Get cf_clearance Cookie
```http
GET /clearance?url=https://example.com&timeout=30
```

| Parameter | Wajib | Keterangan |
|---|---|---|
| `url` | ✅ | URL target yang dilindungi Cloudflare |
| `timeout` | ❌ | Waktu tunggu dalam detik (default: 30) |

---

### ➡️ Get AWS WAF Token
```http
GET /aws-token?url=https://example.com&timeout=30
```

| Parameter | Wajib | Keterangan |
|---|---|---|
| `url` | ✅ | URL target yang dilindungi AWS WAF |
| `timeout` | ❌ | Waktu tunggu dalam detik (default: 30) |

---

### ➡️ Ambil Hasil
```http
GET /result?id=<task_id>
```

Semua endpoint solver (`/turnstile`, `/clearance`, `/aws-token`) mengembalikan `task_id`. Gunakan endpoint ini untuk poll hasilnya.

**Respons `/turnstile` sukses:**
```json
{
  "status": "success",
  "elapsed_time": 2.431,
  "value": "0.AbCdEf..."
}
```

**Respons `/clearance` sukses:**
```json
{
  "status": "success",
  "elapsed_time": 3.102,
  "cf_clearance": "abcdef123...",
  "user_agent": "Mozilla/5.0 ...",
  "cookies": "cf_clearance=abcdef123; __cf_bm=xyz..."
}
```

**Respons `/aws-token` sukses:**
```json
{
  "status": "success",
  "elapsed_time": 2.871,
  "aws_waf_token": "eyJhbGci...",
  "user_agent": "Mozilla/5.0 ...",
  "cookies": "aws-waf-token=eyJhbGci...; ..."
}
```

**Kode status HTTP:**
| Kode | Kondisi |
|---|---|
| `200` | Berhasil |
| `202` | Masih diproses / diterima, coba lagi |
| `404` | `task_id` tidak valid atau sudah expired |
| `408` | Timeout (> 5 menit) |
| `422` | Gagal diselesaikan |
| `429` | Server penuh, coba lagi nanti |

---

## 💻 Contoh Penggunaan

### Python
```python
import requests
import time

BASE_URL = "http://localhost:8000"

# ── Solve Turnstile ──
def solve_turnstile(url: str, sitekey: str) -> str:
    res = requests.get(f"{BASE_URL}/turnstile", params={"url": url, "sitekey": sitekey})
    task_id = res.json()["task_id"]
    while True:
        result = requests.get(f"{BASE_URL}/result", params={"id": task_id}).json()
        if result["status"] == "success":
            return result["value"]
        elif result["status"] == "error":
            raise Exception(f"Gagal: {result.get('value')}")
        time.sleep(1)

# ── Get cf_clearance ──
def get_clearance(url: str) -> dict:
    res = requests.get(f"{BASE_URL}/clearance", params={"url": url})
    task_id = res.json()["task_id"]
    while True:
        result = requests.get(f"{BASE_URL}/result", params={"id": task_id}).json()
        if result["status"] == "success":
            return result  # cf_clearance, user_agent, cookies
        elif result["status"] == "error":
            raise Exception(f"Gagal: {result.get('message')}")
        time.sleep(1)

# ── Get AWS WAF Token ──
def get_aws_token(url: str) -> dict:
    res = requests.get(f"{BASE_URL}/aws-token", params={"url": url})
    task_id = res.json()["task_id"]
    while True:
        result = requests.get(f"{BASE_URL}/result", params={"id": task_id}).json()
        if result["status"] == "success":
            return result  # aws_waf_token, user_agent, cookies
        elif result["status"] == "error":
            raise Exception(f"Gagal: {result.get('message')}")
        time.sleep(1)
```

### Node.js
```javascript
const axios = require("axios");

const BASE_URL = "http://localhost:8000";

async function pollResult(taskId) {
  while (true) {
    const { data: result } = await axios.get(`${BASE_URL}/result`, {
      params: { id: taskId },
    });
    if (result.status === "success") return result;
    if (result.status === "error") throw new Error(JSON.stringify(result));
    await new Promise((r) => setTimeout(r, 1000));
  }
}

// Solve Turnstile
async function solveTurnstile(url, sitekey) {
  const { data } = await axios.get(`${BASE_URL}/turnstile`, {
    params: { url, sitekey },
  });
  return pollResult(data.task_id);
}

// Get cf_clearance
async function getClearance(url) {
  const { data } = await axios.get(`${BASE_URL}/clearance`, {
    params: { url },
  });
  return pollResult(data.task_id);
}

// Get AWS WAF Token
async function getAwsToken(url) {
  const { data } = await axios.get(`${BASE_URL}/aws-token`, {
    params: { url },
  });
  return pollResult(data.task_id);
}
```

---

## ❗ Referensi Error

### Error saat startup

| Error | Artinya | Solusi |
|---|---|---|
| `ModuleNotFoundError: No module named 'xxx'` | Dependensi belum terinstall | Jalankan ulang script, atau `pip install xxx --break-system-packages` |
| `CalledProcessError: pip install returned non-zero` | pip diblokir sistem (Ubuntu 22.04+) | `pip install xxx --break-system-packages` atau pakai virtualenv |
| `OSError: [Errno 98] Address already in use` | Port sudah dipakai proses lain | Ganti port di `config.json` atau matikan proses yang memakai port tersebut |

### Error saat runtime (log)

| Log | Artinya | Solusi |
|---|---|---|
| `Percobaan captcha X gagal: Timeout 400ms exceeded` | Captcha belum muncul / lambat load | Normal jika tidak terlalu sering |
| `Pool halaman berhasil diinisialisasi, berisi 0 halaman` | Browser gagal membuat halaman | Cek RAM tersedia, kurangi `thread` atau `page_count` |
| `proxy_support aktif tapi file 'proxies.txt' tidak ditemukan` | File proxy tidak ada | Buat file `proxies.txt` dengan daftar proxy |
| `Server penuh, coba lagi nanti` | Semua slot browser sedang terpakai | Naikkan `thread` atau `page_count`, atau tunggu |

### Kode HTTP response API

| Kode | Artinya |
|---|---|
| `202` | Tugas diterima / masih diproses — poll ulang |
| `400` | Parameter wajib tidak disertakan |
| `404` | `task_id` tidak valid atau sudah expired |
| `408` | Tugas timeout (> 5 menit) |
| `422` | Gagal diselesaikan |
| `429` | Server penuh — semua slot browser terpakai |
| `500` | Error tak terduga di server |

---

## 🔧 Tips Performa

- **`thread`**: Sesuaikan dengan jumlah core CPU. Contoh: 8-core → maksimal `thread: 8`
- **`page_count`**: Mulai dari `1`. Naikkan hanya jika RAM mencukupi (±300 MB per halaman)
- **`cleanup_interval_minutes`**: Turunkan (misal `5`) jika RAM terbatas
- **`debug: false`**: Matikan untuk output bersih di production
- **Gunakan proxy** untuk meningkatkan tingkat keberhasilan di sitekey yang ketat

---

## 📄 Lisensi

MIT License — Lihat file [LICENSE](LICENSE) untuk detail.

---

## 🔗 Kredit & Referensi

- 🧑‍💻 **Owner Asli**: [SGAHSCAJASCJ](https://github.com/SGAHSCAJASCJ/Turnstile-Solver) — Fondasi solver ini
- 🦊 [Camoufox](https://github.com/daijro/camoufox) — Browser anti-deteksi berbasis Firefox
- ⚡ [FastAPI](https://fastapi.tiangolo.com/) — Framework API modern
- ☁️ [Cloudflare Turnstile](https://developers.cloudflare.com/turnstile/) — Dokumentasi resmi Turnstile

---

<div align="center">

**⚡ Performa Tinggi &nbsp;|&nbsp; 🚀 3 Endpoint Solver &nbsp;|&nbsp; 🛡️ Stabil & Andal &nbsp;|&nbsp; 🌐 Proxy Ready &nbsp;|&nbsp; 🖥️ VPS Auto Setup**

</div>
