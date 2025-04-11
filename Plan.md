# eBPF-project
eBPF-based Network Intrusion Monitor

---

## Gambaran Singkat Proyek

### Nama Proyek:
"KernelWatch: eBPF Realtime Network Intrusion Monitor"

### Latar belakang

Keamanan jaringan merupakan aspek kritis dalam sistem informasi modern. Ancaman seperti port scanning, brute-force login, dan flood traffic (DNS, HTTP) dapat menjadi langkah awal serangan yang lebih besar. Sayangnya, banyak solusi deteksi intrusi (Intrusion Detection System / IDS) bersifat berat, sulit di-deploy, atau memerlukan agen tambahan di setiap host.

eBPF (Extended Berkeley Packet Filter), teknologi dari kernel Linux, menawarkan pendekatan baru yang ringan, fleksibel, dan efisien dalam memantau dan mengolah event di dalam sistem. Dengan eBPF, sistem dapat memproses trafik secara langsung dari dalam kernel tanpa perlu modifikasi kernel atau overhead besar.

### Rumusan masalah

- Bagaimana membangun sistem monitoring jaringan berbasis eBPF untuk deteksi intrusi ringan (lightweight IDS)?
- Apakah eBPF dapat digunakan untuk mendeteksi aktivitas berbahaya secara real-time tanpa mengganggu performa host?
- Bagaimana sistem dapat mengirim alert dan mencatat log aktivitas mencurigakan secara efisien?

### Tujuan:
- Membangun sistem monitoring jaringan real-time berbasis eBPF
- Mendeteksi aktivitas mencurigakan seperti port scanning, DNS flood, brute-force login
- Logging event secara efisien langsung dari kernel
- Mengirim alert ke dashboard / Telegram
- Bisa dikembangkan jadi IDS/IPS super ringan

### Ruang Lingkup
Termasuk:
- Pemrograman eBPF untuk event jaringan
- Daemon pengguna (Python/Go) untuk notifikasi/log
- Simulasi serangan (nmap, hydra, dns flood)
- Analisis performa ringan (CPU/RAM usage)

Tidak termasuk:
- Sistem Machine Learning untuk klasifikasi intrusi
- Integrasi dengan distributed monitoring (multi-host IDS)

---

## Metodologi
| Tahapan	          | Aktivitas                                        |
|-------------------|--------------------------------------------------|
| Studi Literatur	  | Mempelajari eBPF, IDS, tools monitoring modern   |
| Desain Sistem	    | Arsitektur, flow event, struktur folder proyek   |
| Implementasi eBPF	| Menulis program deteksi (XDP/BCC) di kernel      |
| User-space Agent	| Daemon Python untuk baca log + alert             |
| Simulasi Serangan	| Menggunakan nmap, hydra, dan testing tools       |
| Evaluasi	        | Ukur akurasi deteksi, latency, dan resource usage|


## Hal yang Harus Diuji
| Pengujian	                          | Tujuan                                      |
|-------------------------------------|---------------------------------------------|
|Packet capture (via eBPF)	        | Pastikan data benar ditangkap di kernel     |
|Event detection (port scan, dst)	| Validasi logika deteksi berjalan            |
|Resource usage	                  | Harus tetap ringan (low CPU/RAM)            |
|Alert accuracy	                  | Tidak false positive/negative berlebihan    |
|Integrasi alert                   | Notifikasi berhasil ke Telegram/Slack/Log   |
|Multi-host monitoring (opsional)	| Kalau deploy ke beberapa VM via agent       |

---

## Skill yang Relevan
|Skill	                                       | Level yang Dibutuhkan |
|----------------------------------------------|-----------------------|
|Linux & Networking (iptables, netstat, TCP/IP)|	Menengah–Lanjut      |
|eBPF (BCC/XDP/BPFTrace)                       |	Menengah             |
|Python/Go (backend agent)                     |	Menengah             |
|Monitoring Tools (Grafana, Prometheus)        |	Dasar–Menengah       |
|Bash                                          | scripting	Dasar      |
|Cybersecurity (IDS/IPS konsep)                |	Menengah             |

---

## Possible Topology (Simplified)

       ┌────────────┐
       │  Attacker  │
       └────┬───────┘
            │     (Simulated attack: Nmap, SSH brute)
     ┌──────▼─────┐
     │ eBPF Host  │ <== eBPF program jalan di kernel
     │ (Ubuntu VM)│
     └──────┬─────┘
            │
     ┌──────▼─────┐
     │  Backend   │ (bisa SSH, DNS, Web)
     └────────────┘

### Komponen:
- eBPF agent → menangkap & proses event
- Python/Go daemon → handle event & kirim notifikasi
- Grafana/Prometheus/SQLite (opsional) → dashboard visualisasi
- Simulasi serangan → `Nmap`, `Hydra`, `DNS flood tools`

---

## State Awal Proyek

- Target OS: Ubuntu 22.04 (host/VM)
- Tools utama:
  - `bcc`, `bpftrace`, `bpftool`
  - `Python` untuk scripting (bisa diganti `Go`)
  - Simulasi serangan pakai `nmap`, `hydra`, `hping3`
- Belum ada: log parser, alert system, rule engine
- Rencana: bangun base eBPF script, lalu sambungkan ke user-space agent
- Target MVP: Deteksi port scanning → alert → log 

---

## Struktur Folder & File Awal

```
kernelwatch/
├── ebpf/                      # eBPF program (BCC, XDP, dsb)
│   ├── portscan_monitor.c     # BPF C code untuk deteksi port scanning
│   ├── dns_flood_detector.c   # Deteksi DNS flood (opsional)
│   └── headers.h              # Struct parsing paket (jika dibutuhkan)
│
├── agent/                     # User-space daemon (Python/Go)
│   ├── main.py                # Daemon utama, pembaca event dari eBPF
│   ├── alert.py               # Pengiriman alert (Telegram, email, log)
│   ├── rules.py               # Engine rule (threshold, whitelist, dll)
│   └── config.yaml            # Konfigurasi (token, IP penting, port)
│
├── scripts/                   # Tools untuk testing dan setup
│   ├── attack_sim.sh          # Simulasi serangan (nmap, hydra, dll)
│   └── setup_dev_env.sh       # Installer otomatis dependensi & BCC
│
├── dashboards/                # (Opsional) Dashboard & visualisasi
│   ├── prometheus.yml         # Konfigurasi Prometheus
│   └── grafana.json           # Template dashboard Grafana
│
├── data/                      # Output & log
│   ├── event_log.jsonl        # Log event (format JSON Lines)
│   └── metrics.db             # SQLite database (opsional)
│
├── docs/                      # Dokumentasi proyek
│   ├── arch-diagram.png       # Diagram arsitektur/topologi
│   └── README.md              # Dokumentasi utama proyek
│
├── .gitignore                 # File/folder yang diabaikan oleh Git
└── requirements.txt           # Dependensi Python (bcc, requests, dll)
```

### Penjelasan Cepat
|Folder / File  |	Fungsi                                                                |
|---------------|-----------------------------------------------------------------------|
|ebpf/	        | Tempat program eBPF yang akan dikompilasi & attach ke kernel          |
|agent/	        | Daemon yang akan baca event dari kernel & lakukan sesuatu (alert/log) |
|scripts/	      | Tools bantu buat testing, install dep, dll                            |
|dashboards/	  | Integrasi monitoring opsional (kalau kamu pakai Prometheus/Grafana)   |
|data/	        | Tempat nyimpen log & database mini kalau kamu tracking traffic        |
|docs/	        | Penjelasan teknis, arsitektur, proposal singkat                       |
|requirements.txt |	Python dependencies yang bisa kamu pip install -r                   |

---

## Penelitian/Proyek Sejenis (Relevan)
Judul Proyek / Paper:
- A Real-time Intrusion Detection System using eBPF and XDP
- Design and Implementation of Lightweight IDS Using eBPF
- Kernel-Level Packet Filtering with eBPF for Security Audit
- eBPF as a Foundation for Next-Gen Intrusion Detection
- Efficient Traffic Analysis in Cloud using eBPF Framework

---

## Plus: Potensi Pengembangan
| Fitur Lanjutan	                 | Value Tambahan                          |
|----------------------------------|-----------------------------------------|
|ML for anomaly detection	         | Buat auto-learning IDS (AI-powered)     |
|Web dashboard custom	             | Visualisasi event real-time             |
|Plugin-based rule system	         | Bisa load rule baru tanpa restart agent |
|eBPF map exporter to DB	         | Bisa query traffic stats historis       |
|Mode "stealth logging"	           | Log tanpa terlihat oleh attacker        |
|Alert context-aware	             |Misal: hanya alert kalau lebih dari 3x scan/detik|
|Integrasi ke SIEM (via syslog atau API) |

---

## Strategi Eksekusi Proyek

### 1. Fokus ke Rencana Awal (MVP / Minimum Viable Product)
Ini tahap buat ngebuktiin sistemnya kerja, sekecil apapun. Termasuk:
- Menulis 1 eBPF program (misal: deteksi port scan)
- Bikin agent Python yang baca event
- Logging event ke file (cukup text dulu)
- (Opsional) Kirim alert sederhana ke Telegram/log

Tujuannya:
🔹 Dapet hasil awal yang bisa diuji
🔹 Bisa kamu tunjukin ke dosen/pembimbing sebagai "proyek udah jalan"
🔹 Dasar untuk pengembangan selanjutnya

### 2. Pengembangan Bertahap (Versi Pro)
Setelah dasar beres & stabil, kamu bisa mulai tambah fitur keren yang:
- Bikin proyek kamu beda dari yang lain
- Tambah nilai akademis & teknisnya
- Contoh alur pengembangan:

```
MVP
│
├── [v1] Port scan detector + logger
├── [v2] Tambah deteksi DNS flood / brute-force
├── [v3] Kirim notifikasi (Telegram / syslog)
├── [v4] Export data ke SQLite / dashboard
├── [v5] Integrasi SIEM / Rule plugin / CLI dashboard
└── [v6] Tambah ML sederhana untuk prediksi
```

--- 

## Jadi, Step Awal :
- Setup environment --> Ubuntu VM, install BCC, Python deps
- Bikin eBPF script basic --> Misalnya: tangkap koneksi TCP ke banyak port dalam 3 detik
- Bikin Python agent --> Baca hasil dari eBPF, simpen ke file
- Tes pakai nmap --> Simulasi scanning → cek apakah event muncul

---

## KERNELWATCH: Technical To-Do List

### Setup & Persiapan Awal
- [ ] Install Ubuntu 22.04 (bare-metal / VM / WSL2)
- [ ] Install dependency: `bcc`, `bpftool`, `python3`, `pip`, `net-tools`
- [ ] Setup folder struktur proyek (`kernelwatch/` + subfolder)
- [ ] Buat virtualenv Python (opsional)
- [ ] Install Python package: `bcc`, `pyyaml`, `requests`

### Simulasi Serangan (Untuk Testing)
- [ ] Install `nmap`, `hydra`, `hping3`
- [ ] Buat script `scripts/attack_sim.sh` untuk otomatisasi tes
- [ ] Jalankan basic `nmap` scan ke host target

### eBPF Program (MVP)
- [ ] Buat file `ebpf/portscan_monitor.c`
- [ ] Tulis eBPF XDP/BCC program untuk capture koneksi TCP SYN
- [ ] Tambah threshold basic: >5 koneksi ke port berbeda dalam 3 detik = suspicious
- [ ] Compile & attach eBPF ke interface (via BCC/Python)

### Python User-Space Agent
- [ ] Buat file `agent/main.py` untuk baca event dari eBPF
- [ ] Buat file `agent/alert.py` untuk logging + (opsional) Telegram alert
- [ ] Tambahkan basic `config.yaml` (interface, alert toggle, dst)
- [ ] Tambah sistem log ke `data/event_log.jsonl`
- [ ] Tambahkan console print ketika ada aktivitas mencurigakan

### Testing MVP
- [ ] Jalankan script `attack_sim.sh` → amati output eBPF + log
- [ ] Cek apakah port scan terdeteksi dengan benar
- [ ] Coba tuning threshold detection & delay

### Pengembangan Tahap 2 (Optional/Future)
- [ ] Deteksi DNS flood atau brute-force SSH
- [ ] Integrasi notifikasi Telegram/email
- [ ] Export log ke SQLite atau Prometheus
- [ ] Buat dashboard (Grafana / CLI / Web)
- [ ] Tambahkan plugin sistem rule (misal `rules.yaml`)
- [ ] Rancang integrasi dengan SIEM (via syslog atau API)
- [ ] Tambah basic anomaly detection (ML, time pattern)

### Dokumentasi
- [ ] Buat `README.md` isi overview proyek
- [ ] Tambahkan arsitektur diagram ke `docs/`
- [ ] Buat log harian/tahapan di `docs/dev-log.md`
- [ ] Rancang laporan proposal akhir proyek (jika tugas akhir)

---

## Penutup
Proyek ini diharapkan menghasilkan prototipe IDS ringan berbasis eBPF, yang dapat berjalan di sistem Linux modern tanpa overhead besar. Dengan kemampuan mendeteksi aktivitas mencurigakan secara real-time langsung dari kernel, proyek ini membuka potensi solusi keamanan yang efisien dan fleksibel untuk lingkungan server, cloud, dan edge computing.

---

