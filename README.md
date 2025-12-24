# BigAssistant - Knowledge Retrieval Engine (Phase 1: Foundation)

[![n8n](https://img.shields.io/badge/Orchestrator-n8n-FF6C37?style=flat&logo=n8n)](https://n8n.io/)
[![Docker](https://img.shields.io/badge/Infrastructure-Docker-2496ED?style=flat&logo=docker)](https://www.docker.com/)
[![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-34A853?style=flat&logo=google-sheets)](https://www.google.com/sheets/about/)

Project ini adalah implementasi fase pertama dari **n8n**, sebuah sistem asisten pintar berbasis otomasi workflow. Fokus utama fase ini adalah membangun **Knowledge Retrieval Engine** yang robust dengan mekanisme **Audit Trail** yang terstruktur.

Project ini dikembangkan sebagai bagian dari program magang mahasiswa Teknik Komputer, Universitas Brawijaya.

---

## 🚀 Fitur Utama

- **Automated Knowledge Retrieval**: Pengambilan informasi otomatis dari basis data internal.
- **Robust Exception Handling**: Mekanisme jalur logika (True/False) untuk menangani ketersediaan data (Fallback Mechanism).
- **Comprehensive Audit Trail**: Pencatatan setiap interaksi (waktu, pertanyaan, jawaban) ke dalam log untuk keperluan monitoring dan evaluasi.
- **Persistent Infrastructure**: Berjalan di atas kontainer Docker untuk memastikan stabilitas environment.

---

## 🛠️ Stack Teknologi

| Komponen | Teknologi | Peran |
| :--- | :--- | :--- |
| **Orchestrator** | n8n | Mengatur logika alur data (Workflow Engine). |
| **Infrastructure** | Docker & Docker Compose | Virtualisasi environment n8n. |
| **Database** | Google Sheets | Penyimpanan basis pengetahuan (Knowledge Base). |
| **Audit Logging** | Google Sheets | Pencatatan rekam jejak interaksi (Monitoring). |

---

## 📊 Arsitektur Sistem & Data

### 1. Alur Kerja (Logical Flow)
1. **Trigger**: Manual input pertanyaan melalui node `Edit Fields`.
2. **Search**: Query data ke spreadsheet `knowledge_base` berdasarkan kolom `pertanyaan`.
3. **Logic**: Node `IF` mengecek variabel `jawaban`.
    - **True Path**: Mengembalikan data jawaban asli dari Sheets.
    - **False Path**: Mengalihkan ke pesan fallback: *"Maaf, informasi tidak ditemukan dalam basis pengetahuan."*
4. **Final Log**: Semua hasil akhir dicatat ke spreadsheet `log_interaksi`.

### 2. Skema Data (Schema)
**Knowledge Base:**
- `pertanyaan`: Kunci pencarian (String).
- `jawaban`: Informasi respons (String).

**Log Interaksi (Audit Trail):**
- `waktu`: Timestamp interaksi (Asia/Jakarta).
- `pertanyaan`: Input mentah dari pengguna.
- `jawaban`: Output yang diberikan sistem.

---

## ⚙️ Cara Instalasi & Menjalankan

### 1. Prasyarat
- Docker & Docker Compose terinstal di mesin lokal.

### 2. Menjalankan Infrastruktur
Clone file `docker-compose.yml` lalu jalankan perintah:
```bash
docker-compose up -d
```

```bash
Akses dashboard n8n di: http://localhost:5678
```
### 3. Konfigurasi Google Sheets
Buat dua spreadsheet di Google Sheets:
- `knowledge_base`: Untuk menyimpan data pertanyaan dan jawaban.
- `log_interaksi`: Untuk mencatat setiap interaksi.
Pastikan Anda telah memiliki kredensial OAuth 2.0 atau Service Account di Google Cloud Console untuk menghubungkan n8n dengan Google Sheets.


