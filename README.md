# eBPF-project
eBPF-based Network Intrusion Monitor

---

## 🔥 Gambaran Singkat Proyek

### 📛 Nama Proyek (boleh kamu ganti):
"KERNELWATCH: eBPF Realtime Network Intrusion Monitor"

---

## Latar belakang

Keamanan jaringan merupakan aspek kritis dalam sistem informasi modern. Ancaman seperti port scanning, brute-force login, dan flood traffic (DNS, HTTP) dapat menjadi langkah awal serangan yang lebih besar. Sayangnya, banyak solusi deteksi intrusi (Intrusion Detection System / IDS) bersifat berat, sulit di-deploy, atau memerlukan agen tambahan di setiap host.

eBPF (Extended Berkeley Packet Filter), teknologi dari kernel Linux, menawarkan pendekatan baru yang ringan, fleksibel, dan efisien dalam memantau dan mengolah event di dalam sistem. Dengan eBPF, sistem dapat memproses trafik secara langsung dari dalam kernel tanpa perlu modifikasi kernel atau overhead besar.

---

## Rumusan masalah

- Bagaimana membangun sistem monitoring jaringan berbasis eBPF untuk deteksi intrusi ringan (lightweight IDS)?
- Apakah eBPF dapat digunakan untuk mendeteksi aktivitas berbahaya secara real-time tanpa mengganggu performa host?
- Bagaimana sistem dapat mengirim alert dan mencatat log aktivitas mencurigakan secara efisien?

---

## 🎯 Tujuan:
- Membangun sistem monitoring jaringan real-time berbasis eBPF
- Mendeteksi aktivitas mencurigakan seperti port scanning, DNS flood, brute-force login
- Logging event secara efisien langsung dari kernel
- Mengirim alert ke dashboard / Telegram
- Bisa dikembangkan jadi IDS/IPS super ringan

---

## Ruang Lingkup
✅ Termasuk:
-Pemrograman eBPF untuk event jaringan
- Daemon pengguna (Python/Go) untuk notifikasi/log
- Simulasi serangan (nmap, hydra, dns flood)
- Analisis performa ringan (CPU/RAM usage)

🚫 Tidak termasuk:
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


## 🔬 Hal yang Harus Diuji
| Pengujian	                          | Tujuan                                      |
|-------------------------------------|---------------------------------------------|
|✅ Packet capture (via eBPF)	        | Pastikan data benar ditangkap di kernel     |
|✅ Event detection (port scan, dst)	| Validasi logika deteksi berjalan            |
|✅ Resource usage	                  | Harus tetap ringan (low CPU/RAM)            |
|✅ Alert accuracy	                  | Tidak false positive/negative berlebihan    |
|✅ Integrasi alert                   | Notifikasi berhasil ke Telegram/Slack/Log   |
|✅ Multi-host monitoring (opsional)	| Kalau deploy ke beberapa VM via agent       |

---

## 🛠️ Skill yang Relevan
|Skill	                                       | Level yang Dibutuhkan |
|----------------------------------------------|-----------------------|
|Linux & Networking (iptables, netstat, TCP/IP)|	Menengah–Lanjut      |
|eBPF (BCC/XDP/BPFTrace)                       |	Menengah             |
|Python/Go (backend agent)                     |	Menengah             |
|Monitoring Tools (Grafana, Prometheus)        |	Dasar–Menengah       |
|Bash                                          | scripting	Dasar      |
|Cybersecurity (IDS/IPS konsep)                |	Menengah             |

---

## 🕸️ Possible Topology (Simplified)

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

### 📦 Komponen:
- eBPF agent → menangkap & proses event
- Python/Go daemon → handle event & kirim notifikasi
- Grafana/Prometheus/SQLite (opsional) → dashboard visualisasi
- Simulasi serangan → `Nmap`, `Hydra`, `DNS flood tools`

---

## 🧱 State Awal Proyek

- Target OS: Ubuntu 22.04 (host/VM)
- Tools utama:
  - `bcc`, `bpftrace`, `bpftool`
  - `Python` untuk scripting (bisa diganti `Go`)
  - Simulasi serangan pakai `nmap`, `hydra`, `hping3`
- Belum ada: log parser, alert system, rule engine
- Rencana: bangun base eBPF script, lalu sambungkan ke user-space agent
- Target MVP: Deteksi port scanning → alert → log 

---

## 🗂️ Struktur Folder & File Awal

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

### 📌 Penjelasan Cepat
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

## 📚 Penelitian/Proyek Sejenis (Relevan)
Judul Proyek / Paper:
- A Real-time Intrusion Detection System using eBPF and XDP
- Design and Implementation of Lightweight IDS Using eBPF
- Kernel-Level Packet Filtering with eBPF for Security Audit
- eBPF as a Foundation for Next-Gen Intrusion Detection
- Efficient Traffic Analysis in Cloud using eBPF Framework

---

## 🧠 Plus: Potensi Pengembangan
| Fitur Lanjutan	                 | Value Tambahan                          |
|----------------------------------|-----------------------------------------|
|ML for anomaly detection	         | Buat auto-learning IDS (AI-powered)     |
|Web dashboard custom	             | Visualisasi event real-time             |
|Plugin-based rule system	         | Bisa load rule baru tanpa restart agent |
|eBPF map exporter to DB	         | Bisa query traffic stats historis       |
|Mode "stealth logging"	           | Log tanpa terlihat oleh attacker        |
|Alert context-aware	             |Misal: hanya alert kalau lebih dari 3x scan/detik|

---

## Penutup
Proyek ini diharapkan menghasilkan prototipe IDS ringan berbasis eBPF, yang dapat berjalan di sistem Linux modern tanpa overhead besar. Dengan kemampuan mendeteksi aktivitas mencurigakan secara real-time langsung dari kernel, proyek ini membuka potensi solusi keamanan yang efisien dan fleksibel untuk lingkungan server, cloud, dan edge computing.

---



