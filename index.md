---
layout: default
title: Implementasi n8n dengan Google Spreadsheet
description: Dokumentasi workflow automation sederhana menggunakan n8n dan Google Sheets
---

# Implementasi n8n Sederhana Menggunakan Google Spreadsheet

## Pendahuluan

Saya mempelajari dan mengimplementasikan sebuah workflow otomatis menggunakan **n8n**. Project ini bersifat **non-komersial**, sehingga seluruh tools yang digunakan adalah **gratis dan open-source**.

Tujuan utama saya dalam project ini adalah:
- Memahami konsep dasar automation workflow
- Mengimplementasikan use case sederhana namun nyata
- Mendokumentasikan proses secara runtut dan dapat dipahami

---

## Apa itu n8n?

**n8n** adalah tools *workflow automation* berbasis *node* yang memungkinkan kita menghubungkan berbagai layanan tanpa harus menulis kode kompleks.

Secara sederhana, n8n bekerja seperti ini:
- Setiap **node** memiliki satu tugas spesifik
- Data mengalir dari satu node ke node lain
- Alur logika ditentukan oleh kondisi (IF / ELSE)

Keunggulan n8n:
- Open-source dan gratis
- Bisa dijalankan secara lokal menggunakan Docker
- Mendukung integrasi dengan banyak layanan, termasuk Google Sheets

Dalam project ini, n8n digunakan sebagai **otak utama dari Automation versi sederhana**.

---

## Gambaran Umum Project

Project ini bertujuan untuk membuat **Automation sederhana** yang dapat:
1. Menerima sebuah pertanyaan
2. Mencari jawaban dari basis pengetahuan (Google Sheets)
3. Memberikan jawaban jika ditemukan
4. Memberikan pesan default jika jawaban tidak tersedia

Google Sheets berperan sebagai **Knowledge Base**, sedangkan n8n mengatur alur logika dan pengambilan keputusan.

---

## Arsitektur Sistem

Komponen utama:
- **n8n** → Workflow automation
- **Google Sheets** → Basis pengetahuan & log interaksi
- **Docker Compose** → Menjalankan n8n secara lokal

### Diagram Arsitektur

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│   Manual    │────▶│  n8n Workflow│────▶│ Google Sheets   │
│   Trigger   │     │   Processing │     │ (Knowledge Base)│
└─────────────┘     └──────────────┘     └─────────────────┘
                            │
                            ▼
                    ┌──────────────┐
                    │   Response   │
                    │   (Jawaban)  │
                    └──────────────┘
```

---

## Setup Docker

Berikut adalah setup Docker untuk menjalankan n8n secara lokal:

![Docker Setup](./images/docker-setup.png)
*Screenshot: Docker Compose menjalankan n8n*

**Struktur file:**
```
project/
├── docker-compose.yml
├── .env
└── n8n-data/
```

---

## Konfigurasi Google Sheets API

Sebelum mengimplementasikan workflow, kita perlu mengkonfigurasi Google Sheets API.

![Google API Console](./images/google-api-config.png)
*Screenshot: Konfigurasi Google Sheets API di Google Cloud Console*

**Langkah-langkah:**
1. Buat project di Google Cloud Console
2. Enable Google Sheets API
3. Buat Service Account
4. Download credentials JSON
5. Share Google Sheets dengan email service account

---

## Struktur Google Spreadsheet

### Sheet 1: Knowledge Base

Spreadsheet digunakan sebagai basis pengetahuan dengan struktur:

| pertanyaan | jawaban |
|------------|---------|
| Apa itu BigAssistant? | BigAssistant adalah sistem otomasi sederhana... |
| Bagaimana cara menggunakan n8n? | n8n dapat digunakan dengan membuat workflow... |

![Knowledge Base Sheet](./images/spreadsheet-knowledge-base.png)
*Screenshot: Google Sheets - Knowledge Base*

### Sheet 2: Log Interaksi

Sheet kedua untuk mencatat log interaksi:

| timestamp | pertanyaan | jawaban | status |
|-----------|------------|---------|---------|
| 2024-12-24 10:00 | Apa itu n8n? | n8n adalah... | found |
| 2024-12-24 10:05 | Test? | Tidak ditemukan | not_found |

![Log Interaksi Sheet](./images/spreadsheet-log.png)
*Screenshot: Google Sheets - Log Interaksi*

---

## Implementasi Workflow n8n

Workflow dibangun menggunakan beberapa node utama:

![n8n Workflow Overview](./images/n8n-workflow-full.png)
*Screenshot: Tampilan lengkap workflow n8n*

### 1. Manual Trigger

Digunakan untuk melakukan simulasi dan pengujian workflow secara manual.

![Manual Trigger Node](./images/node-manual-trigger.png)
*Screenshot: Konfigurasi Manual Trigger*

---

### 2. Set Node – Input Pertanyaan

Node ini digunakan untuk menyimpan pertanyaan dalam format JSON agar dapat diteruskan ke node berikutnya.

Contoh data:
```json
{
  "pertanyaan": "Apa itu BigAssistant?"
}
```

![Set Input Node](./images/node-set-input.png)
*Screenshot: Konfigurasi Set Node untuk input pertanyaan*

---

### 3. Google Sheets – Get Row(s)

Node ini berfungsi untuk mencari data di sheet `knowledge_base` berdasarkan kolom pertanyaan.

Jika pertanyaan ditemukan, node akan mengembalikan field jawaban.

**Konfigurasi:**
- **Operation**: Lookup
- **Document ID**: [ID Google Sheets Anda]
- **Sheet Name**: knowledge_base
- **Lookup Column**: pertanyaan
- **Lookup Value**: `{{ $json.pertanyaan }}`

![Google Sheets Get Rows](./images/node-google-sheets-get.png)
*Screenshot: Konfigurasi Google Sheets - Get Row(s)*

---

### 4. IF Node – Cek Jawaban

Node IF digunakan untuk mengecek apakah field jawaban tidak kosong.

**Kondisi:**
- **TRUE** → Jawaban ditemukan
- **FALSE** → Jawaban tidak ditemukan

Logika ini tidak menggunakan perhitungan jumlah data, melainkan langsung memvalidasi isi jawaban.

**Expression:**
```javascript
{{ $json.jawaban !== undefined && $json.jawaban !== null && $json.jawaban !== '' }}
```

![IF Node Configuration](./images/node-if-condition.png)
*Screenshot: Konfigurasi IF Node untuk pengecekan jawaban*

---

### 5. Set Node – Jawaban dari Knowledge Base

Jika jawaban ditemukan, sistem akan mengembalikan jawaban dari Google Sheets.

```json
{
  "pertanyaan": "{{ $('Set Input').item.json.pertanyaan }}",
  "jawaban": "{{ $json.jawaban }}",
  "status": "found"
}
```

![Set Node Found](./images/node-set-found.png)
*Screenshot: Set Node untuk jawaban yang ditemukan*

---

### 6. Set Node – Jawaban Tidak Ditemukan

Jika jawaban tidak ditemukan, sistem akan mengembalikan pesan:

```json
{
  "pertanyaan": "{{ $('Set Input').item.json.pertanyaan }}",
  "jawaban": "Maaf, informasi tidak ditemukan dalam basis pengetahuan.",
  "status": "not_found"
}
```

![Set Node Not Found](./images/node-set-not-found.png)
*Screenshot: Set Node untuk jawaban tidak ditemukan*

---

### 7. Google Sheets – Log Interaksi

Node ini mencatat setiap interaksi ke Google Sheets untuk keperluan analisis di masa mendatang.

**Konfigurasi:**
- **Operation**: Append
- **Document ID**: [ID Google Sheets Anda]
- **Sheet Name**: log_interaksi
- **Data to Send**: 
  - timestamp: `{{ $now.toISO() }}`
  - pertanyaan: `{{ $json.pertanyaan }}`
  - jawaban: `{{ $json.jawaban }}`
  - status: `{{ $json.status }}`

![Google Sheets Log](./images/node-google-sheets-log.png)
*Screenshot: Konfigurasi Google Sheets untuk logging*

---

## Hasil dan Pengujian

### Test Case 1: Pertanyaan Ditemukan

**Input:**
```json
{
  "pertanyaan": "Apa itu BigAssistant?"
}
```

**Output:**
```json
{
  "pertanyaan": "Apa itu BigAssistant?",
  "jawaban": "BigAssistant adalah sistem otomasi sederhana...",
  "status": "found"
}
```

![Test Found](./images/test-result-found.png)
*Screenshot: Hasil test untuk pertanyaan yang ditemukan*

---

### Test Case 2: Pertanyaan Tidak Ditemukan

**Input:**
```json
{
  "pertanyaan": "Siapa presiden Indonesia 2030?"
}
```

**Output:**
```json
{
  "pertanyaan": "Siapa presiden Indonesia 2030?",
  "jawaban": "Maaf, informasi tidak ditemukan dalam basis pengetahuan.",
  "status": "not_found"
}
```

![Test Not Found](./images/test-result-not-found.png)
*Screenshot: Hasil test untuk pertanyaan tidak ditemukan*

---

## Kesimpulan

Workflow berhasil:
- ✅ Mendeteksi pertanyaan yang tersedia di basis pengetahuan
- ✅ Memberikan jawaban yang sesuai
- ✅ Memberikan respon default ketika data tidak ditemukan
- ✅ Menjaga data pertanyaan tetap terbawa di setiap kondisi (TRUE / FALSE)
- ✅ Mencatat semua interaksi ke log untuk analisis

Hal ini membuktikan bahwa n8n dapat digunakan untuk membangun sistem asisten sederhana dengan pendekatan low-code.

---

## Struktur Folder Screenshot

Untuk menampilkan screenshot dengan benar, buat struktur folder berikut di repository GitHub Anda:

```
repository/
├── index.md (file ini)
└── images/
    ├── docker-setup.png
    ├── google-api-config.png
    ├── spreadsheet-knowledge-base.png
    ├── spreadsheet-log.png
    ├── n8n-workflow-full.png
    ├── node-manual-trigger.png
    ├── node-set-input.png
    ├── node-google-sheets-get.png
    ├── node-if-condition.png
    ├── node-set-found.png
    ├── node-set-not-found.png
    ├── node-google-sheets-log.png
    ├── test-result-found.png
    └── test-result-not-found.png
```

---

## Repository & Kontak

- **GitHub Repository**: [Link ke repository Anda]
- **Author**: [Nama Anda]
- **License**: MIT

---

*Dokumentasi ini dibuat sebagai bagian dari pembelajaran automation workflow menggunakan n8n dan Google Sheets.*