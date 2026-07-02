# PRD — SaaS Manajemen Kos "KosKu"

# 1. Visi Produk

KosKu adalah platform SaaS manajemen kos yang membantu pemilik kos mengelola operasional secara lebih rapi, otomatis, dan terpusat.

Tujuan utama produk ini bukan menggantikan seluruh proses bisnis kos, tetapi menghilangkan penggunaan buku catatan, Excel, dan chat WhatsApp yang berantakan.

Fokus utama:

* mengurangi pekerjaan administratif,
* meminimalkan keterlambatan pembayaran,
* meningkatkan visibilitas bisnis kos,
* mengotomatisasi proses operasional rutin.

---

# 2. Target Pengguna

## Primary User

* Pemilik kos 10–100 kamar
* Pemilik beberapa properti kos
* Pengelola kos profesional

## Secondary User

* Admin kos
* Penjaga kos
* Staff operasional

## End User

* Penghuni kos

---

# 3. Masalah yang Diselesaikan

## Sebelum Menggunakan KosKu

* Tagihan dicatat manual.
* Sulit mengetahui penghuni yang menunggak.
* Sulit mengetahui kamar kosong.
* Laporan pemasukan tidak jelas.
* Komunikasi tersebar di WhatsApp.
* Sulit melakukan rekap bulanan.

## Setelah Menggunakan KosKu

* Tagihan dibuat otomatis.
* Pengingat pembayaran berjalan otomatis.
* Status kamar terlihat real-time.
* Laporan tersedia otomatis.
* Komunikasi penghuni lebih terorganisir.
* Pemilik mengetahui kondisi bisnis hanya dalam beberapa detik.

---

# 4. Nilai Jual Utama

> Kelola kos tanpa Excel, buku catatan, dan chat yang berantakan.

Pemilik kos dapat mengetahui kondisi bisnisnya dalam kurang dari 30 detik setelah membuka dashboard.

---

# 5. Modul Produk

## Dashboard

Menampilkan:

* Total kamar
* Kamar terisi
* Kamar kosong
* Penghuni aktif
* Pendapatan bulan ini
* Piutang aktif
* Tagihan jatuh tempo
* Tingkat okupansi
* Maintenance aktif

---

## Properti dan Multi Properti

Mendukung:

* Kos A
* Kos B
* Kos C

Semua properti dapat dikelola dari satu dashboard.

---

## Manajemen Kamar

Data:

* Nomor kamar
* Lantai
* Harga sewa
* Fasilitas
* Foto kamar
* Catatan tambahan

Status:

* Kosong
* Terisi
* Booking
* Maintenance
* Renovasi

---

## Manajemen Penghuni

Data:

* Nama
* Nomor telepon
* Nomor identitas
* Nomor kendaraan
* Kontak darurat
* Foto identitas
* Tanggal masuk
* Tanggal keluar

---

## Tagihan Otomatis

Mendukung:

* Bulanan
* Mingguan
* Tahunan

Komponen tagihan:

* Sewa
* Listrik
* Air
* Internet
* Kebersihan
* Parkir
* Biaya tambahan

---

## Pengingat Pembayaran Otomatis

Jadwal:

* H-7
* H-3
* Hari H
* H+3
* H+7

Media:

* WhatsApp
* Email
* SMS

---

## Integrasi WhatsApp

Menggunakan QR Login Session.

Fitur:

* Pengingat pembayaran otomatis
* Konfirmasi pembayaran
* Pengumuman maintenance
* Pengingat kontrak habis
* Broadcast penghuni

---

## Pembayaran

Status:

* Belum dibayar
* Sebagian dibayar
* Lunas
* Menunggak

Metode:

* Transfer Bank
* QRIS
* Tunai
* E-Wallet

---

## Deposit Penghuni

Fitur:

* Deposit awal
* Penggunaan deposit
* Pengembalian deposit

---

## Meter Listrik dan Air

Input:

* Meter sebelumnya
* Meter terbaru

Sistem menghitung penggunaan otomatis.

---

## Riwayat Penghuni

Menyimpan:

* Riwayat pembayaran
* Riwayat pelanggaran
* Riwayat kerusakan
* Riwayat kamar

---

## Inventaris Kamar

Contoh:

* Kasur
* Lemari
* AC
* TV
* Meja
* Kursi

---

## Keluhan dan Maintenance

Keluhan:

* Lampu rusak
* Keran bocor
* AC rusak
* Internet bermasalah

Status:

* Menunggu
* Diproses
* Selesai

---

## Booking Kamar

Calon penghuni dapat:

* melihat kamar kosong,
* melakukan reservasi,
* membayar booking fee.

---

## Portal Penghuni

Penghuni dapat:

* melihat tagihan,
* melihat riwayat pembayaran,
* upload bukti pembayaran,
* membuat tiket keluhan.

---

## Kontrak Digital

Fitur:

* Upload kontrak
* Tanda tangan digital
* Reminder kontrak habis

---

## Laporan dan Analitik

Laporan:

* Pendapatan bulanan
* Pendapatan tahunan
* Tingkat okupansi
* Piutang aktif
* Pendapatan per kamar
* Pendapatan per properti

Analitik:

* Properti paling menguntungkan
* Kamar paling menguntungkan
* Tingkat keterlambatan pembayaran

---

# 6. Prioritas Pengembangan

## Versi 1 (MVP)

* Dashboard
* Properti
* Kamar
* Penghuni
* Tagihan
* Pembayaran
* Reminder
* Laporan

## Versi 2

* Deposit
* Meter listrik
* Meter air
* Maintenance
* Inventaris

## Versi 3

* Booking kamar
* Portal penghuni
* Pembayaran online
* Multi properti

## Versi 4

* Kontrak digital
* Analitik
* Integrasi pihak ketiga

---

# 7. Fitur yang Tidak Menjadi Prioritas

* Marketplace pencarian kos
* Chat internal
* AI chatbot
* Prediksi harga sewa AI
* Sistem akuntansi lengkap

---

# 8. Technology Stack

## Frontend

* Next.js App Router
* TypeScript
* Tailwind CSS
* React Hook Form
* Zod
* TanStack Query
* Zustand
* TanStack Table
* Recharts

Tidak menggunakan UI framework seperti shadcn/ui.

Menggunakan internal design system dan reusable component library milik sendiri.

---

## Backend

* Express.js
* TypeScript
* REST API Architecture

---

## Database

* PostgreSQL
* Prisma ORM

---

## Cache dan Queue

* Redis
* BullMQ

Digunakan untuk:

* Reminder WhatsApp
* Email
* Invoice generation
* Reporting

---

## Notification

* Evolution API
* SMTP Email

---

## Storage

* Cloudflare R2
* S3 Compatible Storage

---

## Monitoring

* Sentry
* Pino Logger

---

## Deployment

* Frontend menggunakan Vercel
* Backend menggunakan Docker
* Database menggunakan PostgreSQL Managed Service

---

# 9. Arsitektur

Menggunakan:

## Modular Monolith Architecture

Bukan microservice.

Alasan:

* lebih sederhana,
* lebih murah,
* lebih mudah dirawat solo founder,
* masih mampu menangani ribuan pelanggan.

---

## Module Backend

* Auth
* Property
* Room
* Tenant
* Billing
* Utility
* Maintenance
* Notification
* Reporting

---

# 10. Security Architecture

## Authentication

* JWT Access Token 15 menit
* Refresh Token 30 hari
* HttpOnly Cookie
* Token Rotation

---

## Password Security

* Argon2id
* Minimum 8 karakter
* Huruf besar
* Huruf kecil
* Angka

---

## Authorization

RBAC:

* Super Admin
* Owner
* Admin
* Staff
* Tenant

---

## Multi Tenant Isolation

Semua query wajib menggunakan owner_id.

Tujuan:

* mencegah kebocoran data antar pelanggan.

---

## API Security

* Helmet
* CORS
* Rate Limiting
* Validation
* API Versioning

---

## Proteksi Serangan

* CSRF Protection
* XSS Protection
* SQL Injection Protection
* Brute Force Protection

---

## File Upload Security

Validasi:

* MIME Type
* Extension
* Ukuran file

Storage menggunakan private bucket.

---

## WhatsApp Security

* Session terenkripsi
* Session dipisahkan per owner
* Session dapat diputus dari dashboard

---

## Secrets Management

Semua secret menggunakan environment variable dan secret manager.

---

# 11. Audit dan Compliance

Audit log mencatat:

* Login
* Logout
* Pembayaran
* Penghapusan invoice
* Perubahan harga
* Perubahan role

---

# 12. Backup Strategy

* Daily Backup
* Weekly Backup
* Monthly Archive

Restore test dilakukan minimal setiap bulan.

---

# 13. Monitoring dan Observability

Monitoring:

* CPU
* RAM
* Database
* Queue
* Error Rate
* Response Time

Health endpoint:

* /health
* /ready
* /live

---

# 14. Performance Strategy

## Database Index

* owner_id
* room_id
* tenant_id
* invoice_id
* due_date
* payment_status

## Pagination

Semua endpoint list wajib menggunakan pagination.

## Caching

Redis digunakan untuk:

* Dashboard
* Statistik
* Session

## Queue

BullMQ digunakan untuk seluruh pekerjaan asynchronous.

---

# 15. DevOps

* Docker
* GitHub Actions
* CI/CD Pipeline
* Environment Separation
* Automated Deployment

Environment:

* Development
* Staging
* Production

---

# 16. Code Quality

* ESLint
* Prettier
* Husky
* lint-staged
* Conventional Commit

---

# 17. Engineering Principles

* SOLID
* DRY
* KISS
* YAGNI
* Clean Architecture
* Repository Pattern
* DTO Pattern
* Dependency Injection
* Feature Based Structure
* Security by Default
* Observability First

---

# 18. Scalability Target

## Phase 1

* 100 owner
* 5.000 penghuni

Infrastructure:

* 4 CPU
* 8 GB RAM

---

## Phase 2

* 1.000 owner
* 50.000 penghuni

Memisahkan:

* Database
* Redis
* Backend

---

## Phase 3

* 10.000 owner
* 500.000 penghuni

Mulai mempertimbangkan pemisahan:

* Notification Service
* Billing Service
* Reporting Service

---

# 19. Prinsip Produk

Bangun sesederhana mungkin, tetapi jangan membangun sesuatu yang akan menyulitkan enam bulan dari sekarang.
