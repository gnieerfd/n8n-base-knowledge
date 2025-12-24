# Implementasi n8n Sederhana Menggunakan pada google spreadsheet

## Pendahuluan

Saya mempelajari dan mengimplementasikan sebuah workflow otomatis menggunakan **n8n**.  
Project ini bersifat **non-komersial**, sehingga seluruh tools yang digunakan adalah **gratis dan open-source**.

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

Alur kerja secara umum:

---

## Implementasi Workflow n8n

Workflow dibangun menggunakan beberapa node utama:

### 1. Manual Trigger  
Digunakan untuk melakukan simulasi dan pengujian workflow secara manual.

---

### 2. Set Node – Input Pertanyaan  
Node ini digunakan untuk menyimpan pertanyaan dalam format JSON agar dapat diteruskan ke node berikutnya.

Contoh data:
```json
{
  "pertanyaan": "Apa itu BigAssistant?"
}
```
### 3. Google Sheets – Get row(s)

Node ini berfungsi untuk mencari data di sheet knowledge_base berdasarkan kolom pertanyaan.

Jika pertanyaan ditemukan, node akan mengembalikan field jawaban. 

### 4. IF Node – Cek Jawaban

Node IF digunakan untuk mengecek apakah field jawaban tidak kosong.

TRUE → Jawaban ditemukan

FALSE → Jawaban tidak ditemukan

Logika ini tidak menggunakan perhitungan jumlah data, melainkan langsung memvalidasi isi jawaban.

### 5. Set Node – Jawaban dari Knowledge Base

Jika jawaban ditemukan, sistem akan mengembalikan jawaban dari Google Sheets.

### 6. Set Node – Jawaban Tidak Ditemukan

Jika jawaban tidak ditemukan, sistem akan mengembalikan pesan:

"Maaf, informasi tidak ditemukan dalam basis pengetahuan."

### 7. Log-Interaksi
Node ini mencatat setiap interaksi ke Google Sheets untuk keperluan analisis di masa mendatang.

---

## Hasil dan Pengujian
Workflow berhasil:
- Mendeteksi pertanyaan yang tersedia di basis pengetahuan
- Memberikan jawaban yang sesuai
- Memberikan respon default ketika data tidak ditemukan
- Menjaga data pertanyaan tetap terbawa di setiap kondisi (TRUE / FALSE)
Hal ini membuktikan bahwa n8n dapat digunakan untuk membangun sistem asisten sederhana dengan pendekatan low-code.