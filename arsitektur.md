# Arsitektur KosKu Platform

> **Versi:** 1.0
> **Status:** Draft — Arsitektur Teknis untuk Platform Manajemen Kos (Mamikos-like)

---

## Daftar Isi

1. [High-Level System Architecture](#1-high-level-system-architecture)
2. [Project Structure (Monorepo)](#2-project-structure-monorepo)
3. [Tech Stack](#3-tech-stack)
4. [Frontend Architecture](#4-frontend-architecture)
5. [Backend Architecture](#5-backend-architecture)
6. [Database Schema](#6-database-schema)
7. [API Design (RESTful)](#7-api-design-restful)
8. [Authentication & Authorization Flow](#8-authentication--authorization-flow)
9. [Real-time Architecture](#9-real-time-architecture)
10. [Payment Flow](#10-payment-flow)
11. [Background Jobs (Workers)](#11-background-jobs-workers)
12. [Third-Party Integrations](#12-third-party-integrations)
13. [Deployment Architecture](#13-deployment-architecture)
14. [CI/CD Pipeline](#14-cicd-pipeline)
15. [Monitoring & Observability](#15-monitoring--observability)
16. [Security](#16-security)
17. [Phased Development Plan](#17-phased-development-plan)

---

## 1. High-Level System Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                            CLIENT LAYER                            │
│                                                                    │
│  ┌─────────────────────────┐   ┌─────────────────────────────────┐│
│  │   Tenant Web App        │   │   Owner Web App                 ││
│  │   (Next.js)             │   │   (Next.js)                     ││
│  │   ├─ Search & Browse    │   │   ├─ Dashboard & Analytics      ││
│  │   ├─ Booking & Payment  │   │   ├─ Property/Room Management   ││
│  │   ├─ Chat (WS)          │   │   ├─ Booking Management         ││
│  │   ├─ Reviews            │   │   ├─ Billing & Payment Tracking ││
│  │   ├─ MamiPoin/Promo     │   │   ├─ Ads & Subscription         ││
│  │   └─ Akun & Settings    │   │   ├─ Chat (WS)                  ││
│  └──────────┬──────────────┘   │   └─ Laporan Keuangan           ││
│             │                  └──────────┬───────────────────────┘│
│             └──────────────────┬──────────┘                       │
│                                │ HTTPS / WSS                      │
│                    ┌───────────▼────────────┐                     │
│                    │   CDN / Cloudflare      │                     │
│                    │   (Static assets, SSL)  │                     │
│                    └───────────┬────────────┘                     │
└────────────────────────────────┼──────────────────────────────────┘
                                 │
┌────────────────────────────────┼──────────────────────────────────┐
│                         GATEWAY LAYER                             │
│                    ┌───────────▼────────────┐                     │
│                    │   API Gateway           │                     │
│                    │   (Nginx / Traefik)     │                     │
│                    │   ├─ Rate Limiting      │                     │
│                    │   ├─ Request Routing    │                     │
│                    │   ├─ API Versioning     │                     │
│                    │   └─ Load Balancing     │                     │
│                    └───────────┬────────────┘                     │
└────────────────────────────────┼──────────────────────────────────┘
                                 │
┌────────────────────────────────┼──────────────────────────────────┐
│                        SERVICE LAYER                              │
│                    ┌───────────▼────────────┐                     │
│                    │   Backend (Express.js)  │                     │
│                    │   Modular Monolith      │                     │
│                    │                         │                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │
│  │ Auth Module  │  │ User Module  │  │ Property     │            │
│  │ ├─ Login/Reg │  │ ├─ Profile   │  │ Module       │            │
│  │ ├─ JWT/RBAC  │  │ ├─ Verifikasi│  │ ├─ CRUD Kos  │            │
│  │ ├─ OAuth     │  │ ├─ Dokumen   │  │ ├─ Kamar     │            │
│  │ └─ Sessions  │  │ └─ Blacklist │  │ ├─ Fasilitas │            │
│  └──────────────┘  └──────────────┘  │ ├─ Harga     │            │
│  ┌──────────────┐  ┌──────────────┐  │ └─ VirtualTour│            │
│  │ Booking      │  │ Payment      │  └──────────────┘            │
│  │ Module       │  │ Module       │  ┌──────────────┐            │
│  │ ├─ Ajukan    │  │ ├─ Invoice   │  │ Chat Module  │            │
│  │ ├─ Konfirmasi│  │ ├─ Midtrans  │  │ ├─ Real-time  │            │
│  │ ├─ Kontrak   │  │ ├─ Refund    │  │ ├─ Broadcast  │            │
│  │ └─ Waitlist  │  │ └─ Riwayat   │  │ └─ File Share │            │
│  └──────────────┘  │ └─ Rekening  │  └──────────────┘            │
│  ┌──────────────┐  └──────────────┘  ┌──────────────┐            │
│  │ Review       │  ┌──────────────┐  │ Notification │            │
│  │ Module       │  │ Ads & Subs   │  │ Module       │            │
│  │ ├─ Rating    │  │ Module       │  │ ├─ WA (Evo)  │            │
│  │ ├─ Ulasan    │  │ ├─ GoldPlus  │  │ ├─ Email     │            │
│  │ └─ Report    │  │ ├─ MamiAds   │  │ ├─ In-App    │            │
│  └──────────────┘  │ ├─ MamiPrime │  │ └─ Reminder  │            │
│  ┌──────────────┐  │ └─ MamiPoin  │  └──────────────┘            │
│  │ Promo/Voucher│  └──────────────┘  ┌──────────────┐            │
│  │ Module       │  ┌──────────────┐  │ Admin Module │            │
│  │ ├─ Diskon    │  │ Reporting    │  │ ├─ Verifikasi │            │
│  │ ├─ Kode voc  │  │ Module       │  │ ├─ Kelola     │            │
│  │ └─ Eksklusif │  │ ├─ Revenue   │  │ ├─ Moderation │            │
│  └──────────────┘  │ ├─ Occupancy │  │ └─ Support    │            │
│                     │ ├─ Pajak     │  └──────────────┘            │
│                     │ └─ Export    │                               │
│                     └──────────────┘                               │
│                    ┌───────────▼────────────┐                     │
│                    │   Background Workers    │                     │
│                    │   (BullMQ + Redis)      │                     │
│                    │   ├─ Generate Tagihan   │                     │
│                    │   ├─ Kirim Reminder     │                     │
│                    │   ├─ Expire Booking     │                     │
│                    │   ├─ Update Statistik   │                     │
│                    │   └─ Cleanup Session    │                     │
│                    └────────────────────────┘                     │
└──────────────────────────────────────────────────────────────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
┌─────────▼──────────┐  ┌───────▼───────┐  ┌───────────▼────────┐
│   PostgreSQL        │  │   Redis       │  │   Cloudflare R2     │
│   (Primary DB)      │  │   ├─ Cache    │  │   (Object Storage)  │
│                     │  │   ├─ Session  │  │   ├─ Foto Properti  │
│   ├─ Users          │  │   ├─ Queue    │  │   ├─ Dokumen Kontrak│
│   ├─ Properties     │  │   └─ Pub/Sub  │  │   ├─ Virtual Tour   │
│   ├─ Rooms          │  │               │  │   └─ Avatar User    │
│   ├─ Bookings       │  └───────────────┘  └────────────────────┘
│   ├─ Payments       │
│   ├─ Chats          │
│   ├─ Reviews        │
│   └─ Transactions   │
└─────────────────────┘
```

---

## 2. Project Structure (Monorepo)

```
kosan/
│
├── apps/
│   ├── web-tenant/              # Next.js — Tenant-facing SPA
│   │   ├── app/
│   │   │   ├── (auth)/          # login, register
│   │   │   ├── (main)/          # homepage, search, detail
│   │   │   ├── booking/         # booking flow
│   │   │   ├── payment/         # payment flow
│   │   │   ├── chat/            # real-time chat
│   │   │   ├── profile/         # akun & settings
│   │   │   └── admin/           # (opsional) admin panel
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/            # API client (axios/fetch)
│   │   ├── stores/              # Zustand (client state)
│   │   └── lib/                 # utils, helpers
│   │
│   ├── web-owner/               # Next.js — Owner-facing dashboard
│   │   ├── app/
│   │   │   ├── (auth)/
│   │   │   ├── dashboard/       # overview, charts
│   │   │   ├── properties/      # kelola properti
│   │   │   ├── rooms/           # kelola kamar
│   │   │   ├── bookings/        # kelola booking
│   │   │   ├── tenants/         # data penyewa
│   │   │   ├── billing/         # tagihan & pembayaran
│   │   │   ├── reports/         # laporan keuangan
│   │   │   ├── ads/             # GoldPlus, MamiAds, MamiPrime
│   │   │   └── settings/        # pengaturan akun & bisnis
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── stores/
│   │   └── lib/
│   │
│   └── api/                     # Express.js — Backend REST API
│       ├── prisma/
│       │   ├── schema.prisma    # Database schema
│       │   └── migrations/
│       ├── src/
│       │   ├── main.ts          # Entry point (Express app)
│       │   ├── config/          # env, constants, app config
│       │   ├── common/          # shared middleware, utils
│       │   │   ├── middleware/
│       │   │   │   ├── auth.ts          # JWT verification
│       │   │   │   ├── rbac.ts          # role-based access
│       │   │   │   ├── validate.ts      # Zod validation
│       │   │   │   ├── upload.ts        # Multer / R2 upload
│       │   │   │   ├── rate-limit.ts    # rate limiter
│       │   │   │   └── error-handler.ts # global error handler
│       │   │   ├── errors/
│       │   │   ├── types/
│       │   │   └── utils/
│       │   ├── modules/         # Modular monolith
│       │   │   ├── auth/
│       │   │   │   ├── auth.controller.ts
│       │   │   │   ├── auth.service.ts
│       │   │   │   ├── auth.validation.ts
│       │   │   │   ├── auth.routes.ts
│       │   │   │   └── auth.test.ts
│       │   │   ├── user/
│       │   │   │   └── ...
│       │   │   ├── property/
│       │   │   ├── room/
│       │   │   ├── booking/
│       │   │   ├── payment/
│       │   │   ├── chat/
│       │   │   ├── review/
│       │   │   ├── notification/
│       │   │   ├── subscription/
│       │   │   ├── ads/
│       │   │   ├── promo/
│       │   │   ├── loyalty/     # MamiPoin
│       │   │   ├── report/
│       │   │   ├── survey/
│       │   │   ├── waitlist/
│       │   │   ├── admin/
│       │   │   └── blog/
│       │   ├── workers/         # BullMQ workers (background jobs)
│       │   │   ├── billing.worker.ts
│       │   │   ├── reminder.worker.ts
│       │   │   └── report.worker.ts
│       │   └── integrations/    # Third-party services
│       │       ├── payment/     # Midtrans / Xendit
│       │       ├── whatsapp/    # Evolution API
│       │       ├── email/       # Nodemailer / Resend
│       │       ├── storage/     # S3 / R2
│       │       ├── maps/        # Google Maps / Mapbox
│       │       └── auth/        # Google OAuth, Apple OAuth
│       ├── Dockerfile
│       ├── jest.config.ts
│       └── package.json
│
├── packages/                    # Shared libraries
│   ├── shared-types/            # TypeScript interfaces (shared frontend-backend)
│   │   ├── src/
│   │   │   ├── user.ts
│   │   │   ├── property.ts
│   │   │   ├── booking.ts
│   │   │   └── ...
│   │   ├── tsconfig.json
│   │   └── package.json
│   ├── validations/             # Zod schemas (shared frontend-backend)
│   │   ├── src/
│   │   │   ├── auth.schema.ts
│   │   │   ├── property.schema.ts
│   │   │   └── ...
│   │   └── package.json
│   └── eslint-config/           # Shared ESLint config
│       ├── base.js
│       ├── next.js
│       ├── node.js
│       └── package.json
│
├── docker/
│   ├── docker-compose.yml       # Postgres + Redis + API (development)
│   ├── docker-compose.prod.yml  # Production compose overrides
│   ├── Dockerfile.api
│   └── nginx.conf               # API Gateway config
│
├── .github/
│   └── workflows/
│       ├── ci.yml               # Lint, test, build
│       └── deploy.yml           # Deploy to VPS / Vercel
│
├── .env.example
├── .gitignore
├── turbo.json                   # Turborepo config
├── pnpm-workspace.yaml          # pnpm monorepo
├── package.json                 # Root workspace
├── tsconfig.base.json           # Base TS config
└── README.md
```

---

## 3. Tech Stack

| Layer | Teknologi | Justifikasi |
|-------|-----------|-------------|
| **Frontend Tenant** | Next.js 14 (App Router) + TypeScript | SEO untuk halaman search, SSR/ISR, satu framework |
| **Frontend Owner** | Next.js 14 (App Router) + TypeScript | Dashboard, client-side rendering, reusable components |
| **Styling** | Tailwind CSS | Utility-first, cepat develop, konsisten |
| **State (Server)** | TanStack Query (React Query) | Caching, background refetch, optimistic updates |
| **State (Client)** | Zustand | Ringan, zero boilerplate, persist middleware |
| **Form + Validation** | React Hook Form + Zod | Performant, validasi skema shared dengan backend |
| **Charts** | Recharts | Ringan, React-native, cukup untuk dashboard |
| **Table** | TanStack Table | Complex table, sorting, filtering untuk owner dashboard |
| **Backend** | Express.js + TypeScript | Mature, simple, ecosystem besar, community luas |
| **ORM** | Prisma | Type-safe, migration auto, relation management, studio GUI |
| **DB** | PostgreSQL 16 | Reliabel, JSONB untuk data fleksibel, PostGIS nanti |
| **Cache / Queue** | Redis + BullMQ | Job queue untuk WA/email reminder, cache, pub/sub untuk chat |
| **Auth** | JWT (access 15m + refresh 30d) + OAuth (Google, Apple) | Stateless, secure, token rotation |
| **Password** | Argon2id | Gold standard untuk hashing password |
| **Payment Gateway** | Midtrans (Core API) | #1 di Indonesia, support banyak metode (GoPay, QRIS, transfer, kartu) |
| **WA Integration** | Evolution API | Open source, QR session, self-host, no monthly fee |
| **Email** | Nodemailer (dev) / Resend (prod) | Transaksional email, reliable delivery |
| **Storage** | Cloudflare R2 | S3-compatible, no egress fee, global CDN |
| **Real-time** | Socket.IO | Chat & notifikasi real-time, auto-reconnect, room support |
| **Maps** | Mapbox GL JS | Better performance than Google Maps, customizable |
| **Monitoring** | Sentry (error tracking) + Pino (logging) | Observability end-to-end |
| **CI/CD** | GitHub Actions | Free untuk public repo, integrated dengan GitHub |
| **Container** | Docker | Backend + Redis, reproducible deployment |
| **Frontend Hosting** | Vercel | Optimal untuk Next.js, automatic ISR, edge functions |
| **Backend Hosting** | VPS (DigitalOcean / Linode) | Docker compose, full control, biaya terjangkau |
| **DB Hosting** | Supabase / Neon (managed PostgreSQL) | Automatic backup, point-in-time recovery, connection pooling |
| **Package Manager** | pnpm | Fast, disk efficient, native monorepo support |
| **Monorepo Tool** | Turborepo | Parallel build, caching, dependency graph |
| **Code Quality** | ESLint + Prettier + Husky + lint-staged | Konsistensi kode, auto-format |
| **Testing** | Jest + Supertest (backend), Vitest + Testing Library (frontend) | Unit test, integration test, E2E |

---

## 4. Frontend Architecture

### 4.1 Route Structure

#### Tenant App (`web-tenant`)

```
app/
├── page.tsx                          # Homepage (search bar, banner, rekomendasi)
├── layout.tsx                        # Root layout (Navbar, Footer)
│
├── (auth)/
│   ├── login/
│   │   └── page.tsx                  # Login — Pencari Kos / Pemilik Kos
│   ├── register/
│   │   └── page.tsx                  # Register — Pencari Kos / Pemilik Kos
│   ├── forgot-password/
│   │   └── page.tsx
│   └── reset-password/
│       └── page.tsx
│
├── (main)/
│   ├── search/
│   │   └── page.tsx                  # Hasil pencarian (list + map view)
│   ├── properties/
│   │   └── [id]/
│   │       └── page.tsx              # Detail properti
│   ├── blog/
│   │   ├── page.tsx                  # List artikel
│   │   └── [slug]/
│   │       └── page.tsx              # Detail artikel
│   ├── help/
│   │   └── page.tsx                  # Pusat bantuan
│   └── page.tsx                      # Redirect ke homepage
│
├── booking/
│   ├── [roomId]/
│   │   └── page.tsx                  # Form ajukan sewa
│   └── [bookingId]/
│       ├── confirmation/
│       │   └── page.tsx              # Konfirmasi booking
│       └── contract/
│           └── page.tsx              # Kontrak digital
│
├── payment/
│   ├── [paymentId]/
│   │   └── page.tsx                  # Halaman bayar
│   ├── [paymentId]/success/
│   │   └── page.tsx                  # Payment success
│   └── methods/
│       └── page.tsx                  # Kelola metode bayar
│
├── chat/
│   ├── page.tsx                      # List percakapan
│   └── [chatId]/
│       └── page.tsx                  # Room chat
│
├── survey/
│   └── page.tsx                      # Jadwalkan / lihat survei
│
├── favorites/
│   └── page.tsx                      # Wishlist
│
├── waitlist/
│   └── page.tsx                      # Daftar tunggu
│
├── reviews/
│   └── page.tsx                      # Riwayat review
│
├── loyalty/
│   └── page.tsx                      # MamiPoin
│
├── promos/
│   └── page.tsx                      # Promo & voucher
│
└── profile/
    ├── page.tsx                      # Profil & settings
    ├── bookings/
    │   └── page.tsx                  # Riwayat sewa
    └── settings/
        └── page.tsx                  # Pengaturan akun
```

#### Owner App (`web-owner`)

```
app/
├── page.tsx                          # Redirect ke dashboard
├── layout.tsx                        # Root layout (Sidebar, Navbar)
│
├── (auth)/
│   ├── login/
│   │   └── page.tsx
│   ├── register/
│   │   └── page.tsx
│   └── ...                           # Sama seperti tenant auth
│
├── dashboard/
│   └── page.tsx                      # Overview: occupancy, revenue, insights
│
├── properties/
│   ├── page.tsx                      # List properti
│   ├── new/
│   │   └── page.tsx                  # Daftarkan properti baru
│   └── [id]/
│       ├── page.tsx                  # Detail properti
│       ├── edit/
│       │   └── page.tsx              # Edit properti
│       ├── rooms/
│       │   ├── page.tsx              # List kamar
│       │   ├── new/
│       │   │   └── page.tsx          # Tambah kamar
│       │   └── [roomId]/
│       │       └── page.tsx          # Edit kamar / atur harga
│       ├── photos/
│       │   └── page.tsx              # Kelola foto & virtual tour
│       └── stats/
│           └── page.tsx              # Statistik iklan properti
│
├── bookings/
│   ├── page.tsx                      # List semua booking
│   └── [id]/
│       └── page.tsx                  # Detail & manage booking
│
├── tenants/
│   ├── page.tsx                      # Data penyewa
│   ├── active/
│   │   └── page.tsx                  # Penyewa aktif
│   └── history/
│       └── page.tsx                  # Riwayat penyewa
│
├── billing/
│   ├── page.tsx                      # Semua tagihan
│   ├── generate/
│   │   └── page.tsx                  # Generate tagihan
│   ├── [paymentId]/
│   │   └── page.tsx                  # Detail tagihan
│   └── history/
│       └── page.tsx                  # Riwayat pembayaran
│
├── contracts/
│   ├── page.tsx                      # List kontrak
│   └── [id]/
│       └── page.tsx                  # Detail & sign kontrak
│
├── reports/
│   ├── revenue/
│   │   └── page.tsx                  # Laporan pendapatan
│   ├── occupancy/
│   │   └── page.tsx                  # Laporan okupansi
│   ├── expenses/
│   │   └── page.tsx                  # Laporan pengeluaran
│   └── export/
│       └── page.tsx                  # Export laporan
│
├── subscription/
│   ├── page.tsx                      # Langganan GoldPlus
│   └── history/
│       └── page.tsx                  # Riwayat subscription
│
├── ads/
│   ├── page.tsx                      # MamiAds
│   └── new/
│       └── page.tsx                  # Buat iklan baru
│
├── mami-prime/
│   └── page.tsx                      # MamiPrime
│
├── broadcast/
│   └── page.tsx                      # Broadcast chat (GoldPlus)
│
├── competitors/
│   └── page.tsx                      # Cek properti sekitar
│
├── services/
│   ├── photo/
│   │   └── page.tsx                  # MamiFoto
│   └── virtual-tour/
│       └── page.tsx                  # MamiTour
│
├── promo/
│   └── page.tsx                      # Kelola promo
│
├── chat/
│   ├── page.tsx                      # List percakapan
│   └── [chatId]/
│       └── page.tsx                  # Room chat
│
└── settings/
    ├── page.tsx                      # Profil & pengaturan akun
    ├── business/
    │   └── page.tsx                  # Informasi bisnis
    ├── payout/
    │   └── page.tsx                  # Rekening penampungan
    ├── notifications/
    │   └── page.tsx                  # Pengaturan notifikasi
    └── security/
        └── page.tsx                  # Keamanan & password
```

### 4.2 Component Architecture

```
components/
├── ui/                               # Design system primitives
│   ├── Button/
│   ├── Input/
│   ├── Select/
│   ├── Modal/
│   ├── Card/
│   ├── Badge/
│   ├── Avatar/
│   ├── Skeleton/
│   ├── Toast/
│   ├── Table/
│   ├── Tabs/
│   └── ...
│
├── layout/                           # Layout components
│   ├── Navbar.tsx
│   ├── Footer.tsx
│   ├── Sidebar.tsx                   # Owner dashboard sidebar
│   ├── MobileNav.tsx
│   └── Breadcrumb.tsx
│
├── feature/                          # Feature-specific components
│   ├── search/
│   │   ├── SearchBar.tsx
│   │   ├── SearchFilters.tsx
│   │   ├── PropertyCard.tsx
│   │   ├── PropertyList.tsx
│   │   ├── MapView.tsx
│   │   └── SearchSort.tsx
│   ├── property/
│   │   ├── PropertyGallery.tsx
│   │   ├── VirtualTour.tsx
│   │   ├── RoomList.tsx
│   │   ├── RoomCard.tsx
│   │   ├── ReviewSection.tsx
│   │   └── ReviewCard.tsx
│   ├── booking/
│   │   ├── BookingForm.tsx
│   │   ├── BookingSummary.tsx
│   │   ├── BookingStatus.tsx
│   │   └── ContractViewer.tsx
│   ├── payment/
│   │   ├── PaymentMethod.tsx
│   │   ├── InvoiceCard.tsx
│   │   ├── PaymentTimer.tsx
│   │   └── ReceiptUpload.tsx
│   ├── chat/
│   │   ├── ChatList.tsx
│   │   ├── ChatRoom.tsx
│   │   ├── ChatMessage.tsx
│   │   ├── ChatInput.tsx
│   │   └── FileUpload.tsx
│   ├── dashboard/
│   │   ├── StatCard.tsx
│   │   ├── RevenueChart.tsx
│   │   ├── OccupancyChart.tsx
│   │   ├── BookingTable.tsx
│   │   └── InsightCard.tsx
│   ├── billing/
│   │   ├── InvoiceTable.tsx
│   │   ├── PaymentRow.tsx
│   │   ├── ReminderButton.tsx
│   │   └── TagihanForm.tsx
│   └── ...
│
├── shared/                           # Shared business components
│   ├── AuthGuard.tsx                 # Protect routes by role
│   ├── RoleSwitcher.tsx              # Switch antara tenant/owner
│   ├── NotificationBell.tsx
│   ├── SearchableSelect.tsx
│   ├── FileDropzone.tsx
│   ├── Pagination.tsx
│   ├── EmptyState.tsx
│   ├── ErrorBoundary.tsx
│   ├── LoadingSpinner.tsx
│   └── ConfirmDialog.tsx
│
└── providers/
    ├── AuthProvider.tsx
    ├── QueryProvider.tsx
    ├── SocketProvider.tsx
    └── ThemeProvider.tsx
```

### 4.3 State Management Strategy

```
┌─────────────────────────────────────────────────────────┐
│                   STATE LAYER                            │
│                                                          │
│  SERVER STATE (TanStack Query)                           │
│  ├─ Data dari API: properties, bookings, payments, etc   │
│  ├─ Caching otomatis + stale-while-revalidate            │
│  ├─ Optimistic updates untuk actions (booking, favorite) │
│  └─ Invalidasi query setelah mutation                    │
│                                                          │
│  CLIENT STATE (Zustand)                                  │
│  ├─ UI state: sidebar open/close, modal, active tab      │
│  ├─ Search filters sementara (sebelum submit)            │
│  ├─ Map view state (center, zoom, selected marker)       │
│  └─ Form draft (progress booking)                        │
│                                                          │
│  AUTH STATE (Auth.js / NextAuth + Zustand persist)       │
│  ├─ User session, token, role                            │
│  ├─ Access token di memory                               │
│  └─ Refresh token di HttpOnly cookie                     │
│                                                          │
│  REAL-TIME STATE (Socket.IO + Zustand)                   │
│  ├─ Active chat rooms                                    │
│  ├─ Unread message counts                                │
│  └─ Online status                                        │
└─────────────────────────────────────────────────────────┘
```

### 4.4 Data Fetching Strategy

| Kebutuhan | Metode | Keterangan |
|-----------|--------|------------|
| Halaman publik (homepage, search) | SSR + TanStack Query Hydrate | SEO penting |
| Detail properti | ISR (revalidate tiap 5 menit) | SEO + data real-time cukup |
| Dashboard owner | Client-side (CSR) | Data private, perlu auth |
| Chat | WebSocket (Socket.IO) | Real-time |
| Form booking/payment | Client-side mutate | Interaktif, butuh feedback cepat |
| Search results | CSR + debounce | Filter dinamis |

---

## 5. Backend Architecture

### 5.1 Module Structure (Modular Monolith)

Setiap module mengikuti pattern yang konsisten:

```
modules/auth/
├── auth.controller.ts     # Handler HTTP (req, res)
├── auth.service.ts        # Business logic
├── auth.repository.ts     # Prisma queries (data access)
├── auth.validation.ts     # Zod schemas
├── auth.routes.ts         # Express router
├── auth.types.ts          # Types specific to this module
└── __tests__/
    ├── auth.controller.test.ts
    └── auth.service.test.ts
```

### 5.2 Middleware Pipeline

```
Request
  │
  ▼
[1. Helmet]                  ─── Headers keamanan (XSS, CSP, etc.)
  │
  ▼
[2. CORS]                    ─── Allow specific origins
  │
  ▼
[3. Rate Limiter]            ─── Per IP / per endpoint
  │
  ▼
[4. Body Parser]             ─── JSON + URL-encoded
  │
  ▼
[5. Logger (Pino)]           ─── Structured logging
  │
  ▼
[6. Auth (JWT Verify)]       ─── Extract user dari token (optional untuk public routes)
  │
  ▼
[7. RBAC]                    ─── Cek role (untuk protected routes)
  │
  ▼
[8. Validator (Zod)]         ─── Validasi request body/params/query
  │
  ▼
[9. Controller]              ─── Route handler → call service
  │
  ▼
[10. Service]                ─── Business logic
  │
  ▼
[11. Repository]             ─── Prisma query
  │
  ▼
[12. Response]               ─── JSON response + status code
```

### 5.3 Error Handling Strategy

```typescript
// common/errors/AppError.ts
class AppError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string,
    public details?: unknown
  ) {
    super(message);
  }
}

// common/errors/NotFoundError.ts
class NotFoundError extends AppError {
  constructor(resource: string) {
    super(404, 'NOT_FOUND', `${resource} tidak ditemukan`);
  }
}

// common/errors/ValidationError.ts
class ValidationError extends AppError {
  constructor(errors: ZodError) {
    super(400, 'VALIDATION_ERROR', 'Data tidak valid', errors.flatten());
  }
}

// common/errors/UnauthorizedError.ts
class UnauthorizedError extends AppError {
  constructor(message = 'Silakan login terlebih dahulu') {
    super(401, 'UNAUTHORIZED', message);
  }
}

// common/errors/ForbiddenError.ts
class ForbiddenError extends AppError {
  constructor(message = 'Anda tidak memiliki akses') {
    super(403, 'FORBIDDEN', message);
  }
}
```

Response format konsisten:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Data tidak valid",
    "details": {
      "fieldErrors": {
        "email": ["Email tidak valid"],
        "password": ["Password minimal 8 karakter"]
      }
    }
  }
}
```

Sukses response:

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

### 5.4 Logging Strategy (Pino)

```typescript
// common/logger.ts
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport:
    process.env.NODE_ENV === 'development'
      ? { target: 'pino-pretty', options: { colorize: true } }
      : undefined,
  redact: ['req.headers.authorization', 'req.body.password', 'req.body.passwordConfirmation'],
});

export default logger;
```

---

## 6. Database Schema

### 6.1 Entity Relationship Diagram (Text)

```
┌──────────────┐     ┌──────────────┐     ┌───────────────┐
│    users     │1──M│   sessions    │     │   owners      │
│──────────────│     │──────────────│     │───────────────│
│ id           │     │ id           │     │ id            │
│ email        │     │ user_id      │1───M│ user_id       │1:1
│ phone        │     │ refresh_token│     │ business_name │
│ password_hash│     │ expires_at   │     │ business_phone│
│ name         │     │ user_agent   │     │ address       │
│ avatar_url   │     │ ip_address   │     │ payout_account│
│ role         │     │ revoked_at   │     │ subscription  │
│ email_verified│    │ created_at   │     │ goldplus_until│
│ phone_verified│    └──────────────┘     └───────┬───────┘
│ identity_veri│                                  │
│ created_at   │                                  │1:M
│ updated_at   │     ┌──────────────┐            │
│ deleted_at   │     │  properties  │◄───────────┘
└──────┬───────┘     │──────────────│
       │1:M          │ id           │     ┌───────────────┐
       │             │ owner_id     │     │   rooms       │
       ▼             │ name         │     │───────────────│
┌──────────────┐     │ type         │1:M  │ id            │
│ notifications│    │ category     │◄────│ property_id   │
│──────────────│     │ description  │     │ name          │
│ id           │     │ address      │     │ capacity      │
│ user_id      │     │ lat          │     │ price_monthly │
│ type         │     │ lng          │     │ price_yearly  │
│ title        │     │ facilities   │[J]  │ price_daily   │
│ body         │     │ rules        │[J]  │ min_advance   │
│ data         │[J]  │ photos       │[J]  │ size_m2       │
│ is_read      │     │ virtual_tour │     │ floor         │
│ read_at      │     │ status       │     │ status        │
│ created_at   │     │ total_views  │     │ created_at    │
└──────────────┘     │ total_fav    │     └───────┬───────┘
                     │ created_at   │             │1:M
                     │ updated_at   │             │
                     │ verified_at  │             ▼
                     └──────┬───────┘     ┌───────────────┐
                            │1:M          │  bookings     │
                            │             │───────────────│
                            ▼             │ id            │
                     ┌──────────────┐     │ room_id       │
                     │   reviews    │     │ tenant_id     │
                     │──────────────│     │ status        │
                     │ id           │     │ start_date    │
                     │ booking_id   │1:1  │ end_date      │
                     │ user_id      │◄────│ duration_months│
                     │ property_id  │     │ base_price    │
                     │ rating       │     │ service_fee   │
                     │ comment      │     │ deposit       │
                     │ response     │     │ total_amount  │
                     │ created_at   │     │ contract_url  │
                     └──────────────┘     │ contract_signd│
                                          │ created_at    │
                     ┌──────────────┐     │ updated_at    │
                     │  payments    │     └───────────────┘
                     │──────────────│              │1:M
                     │ id           │              │
                     │ booking_id   │◄─────────────┘
                     │ tenant_id    │
                     │ type         │     ┌───────────────┐
                     │ amount       │     │   chats       │
                     │ paid_amount  │     │───────────────│
                     │ due_date     │     │ id            │
                     │ paid_at      │     │ property_id   │(optional)
                     │ status       │     │ sender_id     │
                     │ method       │     │ receiver_id   │
                     │ external_id  │     │ message_type  │
                     │ invoice_url  │     │ content       │
                     │ receipt_url  │     │ file_url      │
                     │ created_at   │     │ read_at       │
                     │ updated_at   │     │ created_at    │
                     └──────────────┘     └───────────────┘

┌──────────────┐     ┌──────────────┐     ┌───────────────┐
│ subscriptions│    │  mami_ads    │     │  mami_prime   │
│──────────────│     │──────────────│     │───────────────│
│ id           │     │ id           │     │ id            │
│ owner_id     │     │ owner_id     │     │ owner_id      │
│ package      │     │ property_id  │     │ property_id   │
│ start_date   │     │ type         │     │ multiplier    │
│ end_date     │     │ start_date   │     │ expires_at    │
│ amount       │     │ end_date     │     │ created_at    │
│ payment_id   │     │ budget       │     └───────────────┘
│ status       │     │ impressions  │
│ created_at   │     │ clicks       │     ┌───────────────┐
└──────────────┘     │ status       │     │ loyalty_pts  │
                     │ created_at   │     │───────────────│
┌──────────────┐     └──────────────┘     │ id            │
│   promos     │                          │ user_id       │
│──────────────│     ┌──────────────┐     │ points        │
│ id           │     │   surveys    │     │ type         │
│ code         │     │──────────────│     │ reference_type│
│ type         │     │ id           │     │ reference_id  │
│ value        │     │ user_id      │     │ created_at    │
│ max_discount │     │ property_id  │     └───────────────┘
│ min_transact │     │ room_id      │
│ start_date   │     │ schedule_date│     ┌───────────────┐
│ end_date     │     │ schedule_time│     │  waitlist     │
│ usage_limit  │     │ status       │     │───────────────│
│ used_count   │     │ notes        │     │ id            │
│ applicable   │[J]  │ owner_id     │     │ user_id       │
│ created_at   │     │ created_at   │     │ room_id       │
└──────────────┘     └──────────────┘     │ status        │
                                           │ created_at    │
[J] = JSONB column                         └───────────────┘
```

### 6.2 Prisma Schema Highlights

```prisma
// Prisma schema — konsep utama

enum UserRole {
  TENANT
  OWNER
  ADMIN
}

model User {
  id               String    @id @default(cuid())
  email            String?   @unique
  phone            String?   @unique
  passwordHash     String
  name             String
  avatarUrl        String?
  role             UserRole  @default(TENANT)
  emailVerifiedAt  DateTime?
  phoneVerifiedAt  DateTime?
  identityVerified Boolean   @default(false)
  createdAt        DateTime  @default(now())
  updatedAt        DateTime  @updatedAt
  deletedAt        DateTime?

  sessions     Session[]
  reviews      Review[]
  notifications Notification[]
  chatsSent    Chat[]       @relation("Sender")
  chatsRecv    Chat[]       @relation("Receiver")
  bookings     Booking[]    @relation("Tenant")
  payments     Payment[]
  surveys      Survey[]
  waitlists    Waitlist[]
  loyaltyPoints LoyaltyPoint[]
  owner        Owner?
}

model Owner {
  id              String    @id @default(cuid())
  userId          String    @unique
  businessName    String?
  businessPhone   String?
  address         String?
  payoutAccount   String?   // Bank account / ewallet
  payoutMethod    String?   // BCA, MANDIRI, GoPay, etc
  subscription    String    @default(FREE) // FREE, GOLDPLUS
  goldplusUntil   DateTime?
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt

  user        User         @relation(fields: [userId], references: [id])
  properties  Property[]
  subscriptions Subscription[]
  ads         MamiAds[]
  mamiPrimes  MamiPrime[]
  promo       Promo[]
  surveys     Survey[]     @relation("Owner")
}

model Property {
  id            String   @id @default(cuid())
  ownerId       String
  name          String
  type          String   // KOS, APARTEMEN
  category      String   // PUTRA, PUTRI, CAMPUR
  description   String?
  address       String
  lat           Float
  lng           Float
  facilities    Json     @default("[]")
  rules         Json     @default("[]")
  photos        Json     @default("[]")
  virtualTourUrl String?
  status        String   @default(PENDING) // PENDING, ACTIVE, INACTIVE, VERIFIED
  totalViews    Int      @default(0)
  totalFavorites Int     @default(0)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  verifiedAt    DateTime?

  owner     Owner     @relation(fields: [ownerId], references: [id])
  rooms     Room[]
  reviews   Review[]
  chats     Chat[]
  surveys   Survey[]
  ads       MamiAds[]
  mamiPrime MamiPrime[]
  promos    Promo[]   @relation("PromoProperties")
  waitlists Waitlist[]
}

model Room {
  id              String   @id @default(cuid())
  propertyId      String
  name            String
  capacity        Int      @default(1)
  priceMonthly    Decimal  @db.Decimal(12, 2)
  priceYearly     Decimal? @db.Decimal(12, 2)
  priceDaily      Decimal? @db.Decimal(12, 2)
  minAdvanceMonths Int     @default(1)
  sizeM2          Float?
  floor           Int?
  status          String   @default(AVAILABLE) // AVAILABLE, OCCUPIED, MAINTENANCE, RENOVATION
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  property  Property  @relation(fields: [propertyId], references: [id])
  bookings  Booking[]
  surveys   Survey[]
  waitlists Waitlist[]
}

model Booking {
  id              String   @id @default(cuid())
  roomId          String
  tenantId        String
  status          String   @default(PENDING) // PENDING, APPROVED, ACTIVE, COMPLETED, CANCELLED, REJECTED
  startDate       DateTime
  endDate         DateTime
  durationMonths  Int
  basePrice       Decimal  @db.Decimal(12, 2)
  serviceFee      Decimal  @db.Decimal(12, 2) @default(0)
  deposit         Decimal  @db.Decimal(12, 2) @default(0)
  totalAmount     Decimal  @db.Decimal(12, 2)
  contractUrl     String?
  contractSignedAt DateTime?
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  room     Room      @relation(fields: [roomId], references: [id])
  tenant   User      @relation("Tenant", fields: [tenantId], references: [id])
  payments Payment[]
  review   Review?
}

model Payment {
  id          String   @id @default(cuid())
  bookingId   String
  tenantId    String
  type        String   @default(SEWA) // SEWA, DEPOSIT, SERVICE_FEE, PENALTY
  amount      Decimal  @db.Decimal(12, 2)
  paidAmount  Decimal  @db.Decimal(12, 2) @default(0)
  dueDate     DateTime
  paidAt      DateTime?
  status      String   @default(UNPAID) // UNPAID, PARTIAL, PAID, OVERDUE, REFUNDED
  method      String?  // TRANSFER, GOPAY, QRIS, CARD
  externalId  String?  // Midtrans transaction id
  invoiceUrl  String?
  receiptUrl  String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  booking Booking @relation(fields: [bookingId], references: [id])
  tenant  User    @relation(fields: [tenantId], references: [id])
}

// ... models lainnya mengikuti pola yang sama
```

---

## 7. API Design (RESTful)

### 7.1 API Conventions

| Aturan | Detail |
|--------|--------|
| **Base URL** | `https://api.kosku.app/v1` |
| **Format** | JSON (Content-Type: application/json) |
| **Auth** | `Authorization: Bearer <access_token>` |
| **Pagination** | `?page=1&limit=20` — Response includes `meta` |
| **Filtering** | `?status=active&minPrice=500000&maxPrice=2000000` |
| **Sorting** | `?sort=price:asc,createdAt:desc` |
| **Fields** | `?fields=id,name,price` (partial response) |
| **Include** | `?include=owner,rooms` (relasi) |
| **Search** | `?q=kata kunci` (full-text search via PostgreSQL) |
| **Versioning** | URL-based (`/v1/`) |
| **Rate Limit** | 100 req/min per IP (public), 300 req/min per user (authenticated) |

### 7.2 Endpoints

#### Auth

| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| POST | `/auth/register` | Register user baru (tenant/owner) | No |
| POST | `/auth/login` | Login, return access + refresh token (cookie) | No |
| POST | `/auth/refresh` | Refresh access token via HttpOnly cookie | Cookie |
| POST | `/auth/logout` | Revoke refresh token + clear cookie | Yes |
| POST | `/auth/oauth/google` | Google OAuth login/register | No |
| POST | `/auth/oauth/apple` | Apple OAuth login/register | No |
| POST | `/auth/forgot-password` | Kirim email reset password | No |
| POST | `/auth/reset-password` | Reset password dengan token | No |
| POST | `/auth/verify-email` | Verifikasi email | No |
| POST | `/auth/verify-phone` | Verifikasi no HP via OTP | No |

#### User

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| GET | `/users/me` | Profile sendiri | Yes | All |
| PATCH | `/users/me` | Update profile | Yes | All |
| PUT | `/users/me/password` | Ubah password | Yes | All |
| DELETE | `/users/me` | Hapus akun | Yes | All |
| POST | `/users/me/verify-identity` | Upload KTP verifikasi | Yes | All |
| GET | `/users/me/notifications` | List notifikasi (paginated) | Yes | All |
| PATCH | `/users/me/notifications/{id}/read` | Mark one as read | Yes | All |
| PATCH | `/users/me/notifications/read-all` | Mark all as read | Yes | All |
| GET | `/users/me/settings` | Get notification settings | Yes | All |
| PATCH | `/users/me/settings` | Update notification settings | Yes | All |
| GET | `/users/{id}` | Get user profile (public) | No | - |
| POST | `/users/{id}/report` | Report user | Yes | All |

#### Properties (Tenant-facing)

| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| GET | `/properties` | Search properti (pagination, filter, sort, map bounds) | No |
| GET | `/properties/nearby` | Properti terdekat (lat, lng, radius) | No |
| GET | `/properties/recommended` | Recommended properties | No |
| GET | `/properties/{id}` | Detail properti + rooms | No |
| GET | `/properties/{id}/reviews` | Reviews properti (paginated) | No |
| GET | `/properties/{id}/similar` | Properti serupa | No |

#### Properties (Owner-facing)

| Method | Endpoint | Deskripsi | Role |
|--------|----------|-----------|------|
| POST | `/owner/properties` | Daftar properti baru | Owner |
| GET | `/owner/properties` | List properti milik owner | Owner |
| GET | `/owner/properties/{id}` | Detail properti milik owner | Owner |
| PATCH | `/owner/properties/{id}` | Edit properti | Owner |
| DELETE | `/owner/properties/{id}` | Hapus properti (soft delete) | Owner |
| POST | `/owner/properties/{id}/photos` | Upload foto | Owner |
| DELETE | `/owner/properties/{id}/photos/{photoId}` | Hapus foto | Owner |
| POST | `/owner/properties/{id}/virtual-tour` | Upload VR 360° | Owner |
| PATCH | `/owner/properties/{id}/verify` | Ajukan verifikasi properti | Owner |

#### Rooms

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| GET | `/properties/{id}/rooms` | List kamar properti | No | - |
| GET | `/rooms/{id}` | Detail kamar | No | - |
| POST | `/owner/properties/{id}/rooms` | Tambah kamar | Yes | Owner |
| PATCH | `/owner/rooms/{id}` | Edit kamar | Yes | Owner |
| DELETE | `/owner/rooms/{id}` | Hapus kamar | Yes | Owner |
| PATCH | `/owner/rooms/{id}/status` | Ubah status kamar | Yes | Owner |
| PATCH | `/owner/rooms/{id}/price` | Ubah harga | Yes | Owner |

#### Bookings

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| POST | `/rooms/{id}/book` | Ajukan sewa | Yes | Tenant |
| GET | `/bookings` | List booking user | Yes | All |
| GET | `/bookings/{id}` | Detail booking | Yes | All |
| PATCH | `/owner/bookings/{id}/approve` | Terima booking | Yes | Owner |
| PATCH | `/owner/bookings/{id}/reject` | Tolak booking | Yes | Owner |
| PATCH | `/bookings/{id}/cancel` | Batalkan booking | Yes | Tenant/Owner |
| POST | `/bookings/{id}/contract` | Upload kontrak | Yes | Owner |
| POST | `/bookings/{id}/contract/sign` | Sign kontrak digital | Yes | Both |
| POST | `/bookings/{id}/extend` | Perpanjang sewa | Yes | Tenant |
| GET | `/owner/bookings` | List booking owner | Yes | Owner |

#### Payments

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| GET | `/payments` | Riwayat pembayaran user | Yes | All |
| GET | `/payments/{id}` | Detail tagihan + invoice | Yes | All |
| POST | `/payments/{id}/pay` | Bayar (create Midtrans transaction) | Yes | Tenant |
| GET | `/payments/methods` | Metode bayar tersedia | No | - |
| POST | `/payments/webhook/midtrans` | Webhook Midtrans (ip whitelist) | No | - |
| POST | `/payments/{id}/upload-receipt` | Upload bukti transfer manual | Yes | Tenant |
| POST | `/payments/{id}/verify` | Verifikasi pembayaran manual | Yes | Owner/Admin |
| POST | `/payments/{id}/remind` | Kirim reminder pembayaran | Yes | Owner |
| GET | `/payments/{id}/invoice` | Download invoice PDF | Yes | All |

#### Chat

| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| GET | `/chats` | List percakapan (grouped by property/user) | Yes |
| GET | `/chats/{chatId}/messages` | List pesan (paginated, cursor-based) | Yes |
| POST | `/chats/{chatId}/messages` | Kirim pesan teks | Yes |
| POST | `/chats/{chatId}/messages/file` | Kirim pesan file/foto | Yes |
| PATCH | `/chats/{chatId}/read` | Mark as read | Yes |
| DELETE | `/chats/{chatId}/messages/{msgId}` | Delete message | Yes |
| POST | `/owner/chat/broadcast` | Broadcast chat (GoldPlus) | Owner |
| GET | `/chats/unread-count` | Total unread messages | Yes |

#### Reviews

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| POST | `/bookings/{id}/review` | Tulis review | Yes | Tenant |
| GET | `/properties/{id}/reviews` | List review (paginated, sorted) | No | - |
| PATCH | `/reviews/{id}` | Edit review | Yes | Tenant |
| DELETE | `/reviews/{id}` | Hapus review | Yes | Tenant |
| POST | `/reviews/{id}/response` | Respon owner ke review | Yes | Owner |

#### Surveys

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| POST | `/surveys` | Jadwalkan survei | Yes | Tenant |
| GET | `/surveys` | List survei user | Yes | All |
| PATCH | `/surveys/{id}` | Update jadwal (reschedule/cancel) | Yes | Tenant |
| PATCH | `/owner/surveys/{id}/confirm` | Konfirmasi jadwal survei | Yes | Owner |
| PATCH | `/owner/surveys/{id}/complete` | Tandai survei selesai | Yes | Owner |

#### Waitlist

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| POST | `/rooms/{id}/waitlist` | Daftar waitlist | Yes | Tenant |
| GET | `/waitlist` | List waitlist user | Yes | Tenant |
| DELETE | `/waitlist/{id}` | Hapus dari waitlist | Yes | Tenant |

#### Favorites

| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| GET | `/favorites` | List favorit | Yes |
| POST | `/properties/{id}/favorite` | Tambah ke favorit | Yes |
| DELETE | `/properties/{id}/favorite` | Hapus dari favorit | Yes |

#### Loyalty (MamiPoin)

| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| GET | `/loyalty/points` | Total poin + riwayat | Yes |
| GET | `/loyalty/history` | Riwayat transaksi poin | Yes |
| POST | `/loyalty/redeem` | Tukar poin | Yes |
| GET | `/loyalty/rewards` | Daftar reward yang bisa ditukar | Yes |

#### Promos & Vouchers

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| GET | `/promos` | List promo aktif | No | - |
| POST | `/promos/validate` | Validasi kode voucher | Yes | Tenant |
| POST | `/owner/promos` | Buat promo baru | Yes | Owner |
| GET | `/owner/promos` | List promo owner | Yes | Owner |
| PATCH | `/owner/promos/{id}` | Edit promo | Yes | Owner |
| DELETE | `/owner/promos/{id}` | Hapus promo | Yes | Owner |

#### Subscriptions (GoldPlus)

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| GET | `/subscriptions/plans` | List paket GoldPlus | No | - |
| POST | `/subscriptions` | Beli/subscribe GoldPlus | Yes | Owner |
| GET | `/subscriptions/active` | Cek subscription aktif | Yes | Owner |
| GET | `/subscriptions/history` | Riwayat subscription | Yes | Owner |

#### MamiAds

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| GET | `/ads/plans` | List paket iklan | No | - |
| POST | `/ads` | Buat iklan baru | Yes | Owner |
| GET | `/ads` | List iklan owner | Yes | Owner |
| GET | `/ads/{id}` | Detail iklan | Yes | Owner |
| GET | `/ads/{id}/stats` | Statistik iklan (impressions, clicks) | Yes | Owner |
| PATCH | `/ads/{id}/pause` | Pause iklan | Yes | Owner |
| PATCH | `/ads/{id}/resume` | Resume iklan | Yes | Owner |

#### MamiPrime

| Method | Endpoint | Deskripsi | Auth | Role |
|--------|----------|-----------|------|------|
| POST | `/mami-prime` | Beli MamiPrime | Yes | Owner |
| GET | `/mami-prime/active` | Cek MamiPrime aktif | Yes | Owner |

#### Owner Dashboard

| Method | Endpoint | Deskripsi | Role |
|--------|----------|-----------|------|
| GET | `/owner/dashboard` | Ringkasan: properties, occupancy, revenue, booking requests | Owner |
| GET | `/owner/stats/views` | Statistik views per properti (daily/weekly/monthly) | Owner |
| GET | `/owner/stats/bookings` | Statistik booking (conversion rate) | Owner |
| GET | `/owner/stats/chats` | Statistik chat masuk | Owner |
| GET | `/owner/revenue` | Pendapatan (period filter: daily/monthly/yearly) | Owner |
| GET | `/owner/revenue/summary` | Revenue breakdown per properti | Owner |
| GET | `/owner/reports/export` | Export laporan (CSV/Excel/PDF) | Owner |
| GET | `/owner/competitors` | Cek properti sekitar (harga, occupancy) | Owner |

#### Admin

| Method | Endpoint | Deskripsi | Role |
|--------|----------|-----------|------|
| GET | `/admin/properties/pending` | Properti butuh verifikasi | Admin |
| PATCH | `/admin/properties/{id}/verify` | Verifikasi / tolak properti | Admin |
| GET | `/admin/users` | List users (filter by role, status) | Admin |
| GET | `/admin/users/{id}` | Detail user | Admin |
| PATCH | `/admin/users/{id}/ban` | Ban / suspend user | Admin |
| PATCH | `/admin/users/{id}/unban` | Unban user | Admin |
| GET | `/admin/reports` | List laporan user | Admin |
| PATCH | `/admin/reports/{id}` | Action laporan (dismiss, warn, ban) | Admin |
| GET | `/admin/dashboard` | Admin overview stats | Admin |
| GET | `/admin/payments` | All payments (monitoring fraud) | Admin |
| GET | `/admin/transactions` | All platform transactions | Admin |

#### Blog

| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| GET | `/blog` | List artikel (paginated) | No |
| GET | `/blog/{slug}` | Detail artikel | No |
| POST | `/admin/blog` | Buat artikel | Admin |
| PATCH | `/admin/blog/{id}` | Edit artikel | Admin |
| DELETE | `/admin/blog/{id}` | Hapus artikel | Admin |

#### Support

| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| GET | `/help/faq` | FAQ list | No |
| POST | `/help/ticket` | Buat tiket bantuan | Yes |
| GET | `/help/tickets` | List tiket user | Yes |
| POST | `/help/contact` | Form kontak CS | No |

---

## 8. Authentication & Authorization Flow

### 8.1 JWT Token Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                        TOKEN STRATEGY                            │
│                                                                   │
│  ACCESS TOKEN                                                     │
│  ├─ Expiry: 15 menit                                              │
│  ├─ Storage: Memory (Zustand state)                               │
│  ├─ Transport: Authorization: Bearer <token>                      │
│  └─ Payload: { userId, role, jti }                                │
│                                                                   │
│  REFRESH TOKEN                                                    │
│  ├─ Expiry: 30 hari                                               │
│  ├─ Storage: HttpOnly Cookie (secure, sameSite=strict, path=/auth)│
│  ├─ Transport: Cookie otomatis via browser                        │
│  ├── Hash stored di Redis: `sess:{userId}:{jti}`                  │
│  └─ Payload: { userId, role, jti, tokenFamily }                   │
│                                                                   │
│  TOKEN ROTATION                                                   │
│  ├─ Setiap refresh: old refresh token di-revoke, issue baru       │
│  ├─ Detect compromised: jika refresh token lama digunakan lagi   │
│  │   → revoke ALL tokens untuk user tersebut                      │
│  └─ Token family tracking di Redis                                │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Login Flow

```
[Client]                          [Server]
   │                                 │
   │  POST /auth/login               │
   │  { email, password }            │
   │────────────────────────────────►│
   │                                 │
   │                                 ├── Verifikasi email exists
   │                                 ├── Argon2id verify password
   │                                 ├── Generate accessToken (15m)
   │                                 ├── Generate refreshToken (30d)
   │                                 ├── Store hash di Redis
   │                                 │   Key: sess:{userId}:{jti}
   │                                 │   Val: { hash, userAgent, ip }
   │                                 │
   │◄────────────────────────────────┤
   │  Set-Cookie: refreshToken=...   │
   │  (HttpOnly, Secure, SameSite)   │
   │  Response: {                    │
   │    user: { id, name, role },    │
   │    accessToken: "eyJ..."        │
   │  }                              │
```

### 8.3 Token Refresh Flow

```
[Client]                          [Server]
   │                                 │
   │  Request API → 401              │
   │◄────────────────────────────────┤
   │                                 │
   │  Interceptor detect 401         │
   │  Call POST /auth/refresh        │
   │  (Cookie otomatis terkirim)     │
   │────────────────────────────────►│
   │                                 │
   │                                 ├── Read refreshToken dari cookie
   │                                 ├── Verify JWT signature
   │                                 ├── Check expiry
   │                                 ├── Hash token, cari di Redis
   │                                 ├── Rotate: delete old, create new
   │                                 ├── Issue new accessToken
   │                                 │
   │◄────────────────────────────────┤
   │  Set-Cookie: refreshToken=new   │
   │  Response: { newAccessToken }   │
   │                                 │
   │  Retry original request         │
   │  with new accessToken           │
```

### 8.4 RBAC (Role-Based Access Control)

```typescript
// common/middleware/rbac.ts
const RoleHierarchy = {
  TENANT: ['tenant'],
  OWNER: ['owner'],
  ADMIN: ['admin', 'owner', 'tenant'],
} as const;

function requireRole(...roles: UserRole[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    const userRole = req.user.role;
    if (!roles.some(r => RoleHierarchy[r].includes(userRole))) {
      throw new ForbiddenError('Anda tidak memiliki akses ke resource ini');
    }
    next();
  };
}

// Usage:
// router.get('/owner/properties', requireAuth, requireRole('OWNER'), handler);
// router.get('/bookings', requireAuth, requireRole('TENANT', 'OWNER'), handler);
```

### 8.5 Multi-Tenant Data Isolation

```typescript
// Setiap query owner-scoped wajib filter by ownerId dari JWT
// common/middleware/tenant-isolation.ts

function tenantIsolation() {
  return (req: Request, res: Response, next: NextFunction) => {
    // Inject owner filter ke req untuk digunakan di repository
    req.tenantFilter = { ownerId: req.user.userId };
    next();
  };
}

// Repository layer:
// const properties = await prisma.property.findMany({
//   where: { ...req.tenantFilter, ...additionalFilters }
// });
```

---

## 9. Real-time Architecture

### 9.1 WebSocket Architecture (Socket.IO)

```
┌─────────────────────────────────────────────────────────────────┐
│                    WEBSOCKET ARCHITECTURE                        │
│                                                                   │
│  [Tenant Browser]    [Owner Browser]         [Backend]           │
│       │                    │                     │               │
│       │──── WS Connect ────►                    │               │
│       │                    │──── WS Connect ────►               │
│       │                    │                     │               │
│       │  Auth: send JWT   │                     │               │
│       │◄─── auth ok ──────┤◄─── auth ok ────────│               │
│       │                    │                     │               │
│       │                    │                     │               │
│       │── Join room ──────►│                     │               │
│       │  room: user:{id}   │                     │               │
│       │                    │                     │               │
│       │                    │── Join room ───────►│               │
│       │                    │  room: owner:{id}   │               │
│       │                    │                     │               │
│       │                    │                     │               │
│  ──── Chat Flow ──────────│─────────────────────│───────────    │
│       │                    │                     │               │
│       │── Send Message ─────────────────────────►│               │
│       │  { chatId, text }  │                     │               │
│       │                    │                     ├── Save to DB  │
│       │                    │                     ├── Emit ke     │
│       │                    │                     │   room:       │
│       │◄── new_message ────┤◄── new_message ─────│   chat:{id}   │
│       │                    │                     │               │
│       │                    │                     ├── Queue WA    │
│       │                    │                     │   notif (if   │
│       │                    │                     │   offline)    │
│       │                    │                     │               │
│  ──── Notification Flow ──│─────────────────────│───────────    │
│       │                    │                     │               │
│       │                    │                     │── Queue job   │
│       │                    │                     │  (payment     │
│       │                    │                     │   reminder)   │
│       │                    │                     │               │
│       │◄── notification ───┤◄── notification ───│               │
│       │  { type, title }   │  { type, title }   │               │
│       │                    │                     │               │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 Redis Pub/Sub for Horizontal Scaling

```
[API Instance 1]              [Redis]              [API Instance 2]
      │                         │                       │
      │── publish ─────────────►│                       │
      │  channel: chat:room123  │                       │
      │                         │── broadcast ────────►│
      │                         │  to subscribers       │
      │                         │                       │
      │                         │                       │── emit Socket.IO
      │                         │                       │   to connected clients
```

---

## 10. Payment Flow

### 10.1 Midtrans Integration Flow

```
[Tenant]                   [Backend]                    [Midtrans]
   │                          │                             │
   │  POST /payments/{id}/pay │                             │
   │─────────────────────────►│                             │
   │                          │                             │
   │                          ├── Create Payment record     │
   │                          │   (status: PENDING)         │
   │                          │                             │
   │                          ├── POST /charger API         │
   │                          │   transaction_details: {    │
   │                          │     order_id: "KOS-{id}",   │
   │                          │     gross_amount: 500000    │
   │                          │   }                         │
   │                          │   customer_details: {...}   │
   │                          │────────────────────────────►│
   │                          │                             │
   │                          │◄─── Response ──────────────│
   │  {                        │   token: "abc123"          │
   │    redirect_url: "https://│   redirect_url: "..."      │
   │      app.midtrans.com/..."│                             │
   │  }                        │                             │
   │◄─────────────────────────┤                             │
   │                          │                             │
   │── Redirect ke Midtrans ──┼───────────────────────────►│
   │  (Pilih metode bayar)     │                             │
   │                          │                             │
   │                          │                             │
   │── Selesai bayar ─────────┼─────────────────────────────│
   │                          │                             │
   │                          │◄── HTTP POST ──────────────│
   │                          │   Webhook: transaction_status
   │                          │   order_id: "KOS-{id}"      │
   │                          │   transaction_status:       │
   │                          │    "settlement"             │
   │                          │                             │
   │                          ├── Verify signature key      │
   │                          ├── Update Payment → PAID     │
   │                          ├── Update Booking → ACTIVE   │
   │                          ├── Notify owner (WA + in-app)│
   │                          ├── Notify tenant (in-app)    │
   │                          ├── Queue: generate next      │
   │                          │   billing schedule          │
   │                          │                             │
   │                          │◄── Response 200 OK ────────│
   │                          │                             │
   │  Status: paid             │                             │
   │◄─────────────────────────┤                             │
```

### 10.2 Payment States

```
PENDING ───► Tenant bayar ───► Midtrans process ───► PAID
  │                                                    │
  │                                                    ├── Booking → ACTIVE
  │                                                    ├── Next billing schedule
  │                                                    └── Notify owner + tenant
  │
  ├── Expired (24 jam) ──► EXPIRED ──► Booking → CANCELLED
  │
  └── Failed ──► FAILED ──► Tenant bisa retry

PAID ──► Refund request ──► REFUNDED ──► Booking → CANCELLED

OVERDUE (after due_date) ──► Reminder H+1, H+3, H+7
                                   │
                                   └── Tenant bayar ──► PAID
```

---

## 11. Background Jobs (Workers)

### 11.1 BullMQ Queue Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      QUEUE ARCHITECTURE                         │
│                                                                   │
│  [Express App]             [Redis]              [Worker Process] │
│       │                     │                       │           │
│       │── add job ─────────►│                       │           │
│       │  queue: billing     │                       │           │
│       │  data: {roomId,     │                       │           │
│       │         month}      │                       │           │
│       │                     │── job available ─────►│           │
│       │                     │                       │           │
│       │                     │                       ├── Process │
│       │                     │                       ├── Generate│
│       │                     │                       │   invoice │
│       │                     │                       ├── Send WA │
│       │                     │                       │   reminder│
│       │                     │                       ├── Mark    │
│       │                     │                       │   done    │
│       │                     │◄── job complete ──────│           │
│       │                     │                       │           │
└─────────────────────────────────────────────────────────────────┘
```

### 11.2 Job Definitions

| Queue | Job | Trigger | Schedule | Deskripsi |
|-------|-----|---------|----------|-----------|
| `billing` | `generate-monthly` | Cron | Tgl 1 setiap bulan | Generate tagihan sewa untuk semua active booking |
| `reminder` | `payment-h-7` | Cron | H-7 dari due_date | WA: "Jangan lupa bayar tagihan" |
| `reminder` | `payment-h-3` | Cron | H-3 dari due_date | WA: "Tagihan segera jatuh tempo" |
| `reminder` | `payment-h` | Cron | H due_date | WA: "Tagihan hari ini jatuh tempo" |
| `reminder` | `payment-h+3` | Cron | H+3 overdue | WA: "Tagihan sudah melewati jatuh tempo" |
| `reminder` | `payment-h+7` | Cron | H+7 overdue | WA: "Peringatan terakhir" |
| `reminder` | `survey-h-1` | Cron | H-1 survei | WA: "Ingat jadwal survei besok" |
| `reminder` | `survey-2h` | Cron | 2 jam sebelum survei | WA: "Ingat jadwal survei 2 jam lagi" |
| `booking` | `expire-pending` | Cron | Every 30 min | Expire booking pending > 24 jam |
| `booking` | `auto-checkout` | Cron | Setiap hari | Cek booking yang habis masa sewa → COMPLETED |
| `stats` | `update-views` | Cron | Every 15 min | Proses & aggregate view logs |
| `stats` | `update-dashboard` | Cron | Setiap jam | Refresh cache dashboard owner |
| `cleanup` | `expire-sessions` | Cron | Setiap 6 jam | Hapus session Redis yang expired |
| `notification` | `send-immediate` | Event | Real-time | Kirim WA/Email notifikasi (di-trigger action) |
| `report` | `generate-monthly-report` | Cron | Tgl 1 | Generate laporan bulanan untuk owner |

### 11.3 Worker Implementation Pattern

```typescript
// workers/billing.worker.ts
import { Worker } from 'bullmq';
import { redisConnection } from '../config/redis';
import { BillingService } from '../modules/billing/billing.service';

const billingWorker = new Worker(
  'billing',
  async (job) => {
    const { type, data } = job;

    switch (type) {
      case 'generate-monthly':
        await BillingService.generateMonthlyBills(data.month, data.year);
        break;

      case 'generate-single':
        await BillingService.generateBillForBooking(data.bookingId);
        break;
    }
  },
  { connection: redisConnection, concurrency: 5 }
);

billingWorker.on('completed', (job) => {
  logger.info(`Job ${job.id} completed: ${job.name}`);
});

billingWorker.on('failed', (job, err) => {
  logger.error(`Job ${job.id} failed: ${err.message}`, { job, error: err });
});
```

---

## 12. Third-Party Integrations

### 12.1 Integration Map

```
┌─────────────────────────────────────────────────────────────────┐
│                   THIRD-PARTY INTEGRATIONS                       │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Midtrans    │  │  Evolution    │  │  Google       │          │
│  │  (Payment)   │  │  API (WA)     │  │  OAuth        │          │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤          │
│  │ Core API     │  │ QR Login     │  │ Google Login  │          │
│  │ Webhook      │  │ Send Message │  │ Profile API   │          │
│  │ Snap UI      │  │ Webhook Msg  │  └──────────────┘          │
│  │ Refund API   │  │ Instance Mng │  ┌──────────────┐          │
│  └──────────────┘  └──────────────┘  │  Apple        │          │
│  ┌──────────────┐  ┌──────────────┐  │  Sign In      │          │
│  │  Resend      │  │  Cloudflare  │  ├──────────────┤          │
│  │  (Email)     │  │  R2 (Storage)│  │ Apple Login   │          │
│  ├──────────────┤  ├──────────────┤  └──────────────┘          │
│  │ SMTP Relay   │  │ S3 API       │  ┌──────────────┐          │
│  │ Templates    │  │ Public URL   │  │  Mapbox      │          │
│  │ Webhooks     │  │ Signed URL   │  ├──────────────┤          │
│  └──────────────┘  └──────────────┘  │ Geocoding    │          │
│  ┌──────────────┐  ┌──────────────┐  │ Map SDK      │          │
│  │  Sentry      │  │  Upstash     │  │ Directions   │          │
│  │  (Monitor)   │  │  Redis       │  └──────────────┘          │
│  ├──────────────┤  ├──────────────┤                             │
│  │ Error Track  │  │ Managed Redis│                             │
│  │ Performance  │  │ Auto Backup  │                             │
│  │ Replay       │  └──────────────┘                             │
│  └──────────────┘                                               │
└─────────────────────────────────────────────────────────────────┘
```

### 12.2 Evolution API (WhatsApp)

```
┌─────────────────────────────────────────────────────────────┐
│                   EVOLUTION API STRATEGY                    │
│                                                             │
│  Deployment:                                                │
│  ┌────────────────────────────────────────────────┐        │
│  │  Docker container (evolution-api/evolution-api)│        │
│  │  Port: 8080                                    │        │
│  │  Env: AUTHENTICATION_API_KEY=xxx               │        │
│  │       DATABASE_ENABLED=true                    │        │
│  │       DATABASE_CONNECTION_URI=postgresql://... │        │
│  └────────────────────────────────────────────────┘        │
│                                                             │
│  Flow:                                                      │
│  1. Owner scan QR via web-owner → instance created          │
│  2. Instance connected → status: CONNECTED                  │
│  3. Worker send message via Evolution API:                  │
│     POST /message/sendText/{instanceName}                   │
│     { "number": "6281xxx", "text": "Pesan..." }             │
│  4. Webhook incoming → Evolution kirim POST ke backend      │
│     untuk pesan masuk dari tenant via WA                    │
│                                                             │
│  Security:                                                  │
│  - API Key di env variable                                  │
│  - Webhook signature verification                           │
│  - Instance per owner (isolated)                            │
│  - Auto-reconnect jika disconnect                           │
└─────────────────────────────────────────────────────────────┘
```

### 12.3 Midtrans Payment

```typescript
// integrations/payment/midtrans.ts
import Midtrans from 'midtrans-client';

const snap = new Midtrans.Snap({
  isProduction: process.env.NODE_ENV === 'production',
  serverKey: process.env.MIDTRANS_SERVER_KEY,
  clientKey: process.env.MIDTRANS_CLIENT_KEY,
});

const core = new Midtrans.CoreApi({
  isProduction: process.env.NODE_ENV === 'production',
  serverKey: process.env.MIDTRANS_SERVER_KEY,
  clientKey: process.env.MIDTRANS_CLIENT_KEY,
});

interface CreateTransactionParams {
  orderId: string;
  amount: number;
  customer: {
    firstName: string;
    lastName?: string;
    email: string;
    phone: string;
  };
  items: Array<{
    id: string;
    name: string;
    price: number;
    quantity: number;
  }>;
}

export async function createTransaction(params: CreateTransactionParams) {
  const transaction = await snap.createTransaction({
    transaction_details: {
      order_id: params.orderId,
      gross_amount: params.amount,
    },
    customer_details: {
      first_name: params.customer.firstName,
      last_name: params.customer.lastName,
      email: params.customer.email,
      phone: params.customer.phone,
    },
    item_details: params.items,
  });

  return {
    token: transaction.token,
    redirectUrl: transaction.redirect_url,
  };
}
```

---

## 13. Deployment Architecture

### 13.1 Production Architecture

```
                         Production
                         ─────────
                    ┌─────────────────────────┐
                    │      Cloudflare          │
                    │  ├─ DNS (kosku.app)      │
                    │  ├─ CDN (static assets)  │
                    │  └─ WAF (security)       │
                    └────────┬────────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
    ┌─────────▼────┐  ┌─────▼──────┐  ┌────▼────────┐
    │  Vercel      │  │  Vercel    │  │  VPS / DO   │
    │  web-tenant  │  │  web-owner │  │  Docker     │
    │  .kosku.app  │  │  .kosku.app│  │             │
    │              │  │            │  │  ┌────────┐ │
    │  Next.js SSR │  │  Next.js   │  │  │ Nginx  │ │
    │  ISR         │  │  CSR       │  │  │ Proxy  │ │
    └──────────────┘  └────────────┘  │  └───┬────┘ │
                                       │      │      │
                                       │  ┌───▼────┐ │
                                       │  │ API x2 │ │
                                       │  │ (Docker)│ │
                                       │  └────────┘ │
                                       │  ┌────────┐ │
                                       │  │ Redis   │ │
                                       │  └────────┘ │
                                       │  ┌────────┐ │
                                       │  │ Worker  │ │
                                       │  └────────┘ │
                                       └──────┬──────┘
                                              │
                    ┌─────────────────────────┼─────────┐
                    │                         │         │
           ┌────────▼──────┐         ┌───────▼─────┐   │
           │  Supabase /   │         │  Upstash    │   │
           │  Neon (PG)   │         │  Redis      │   │
           │  Managed DB  │         │  Managed    │   │
           └───────────────┘         └─────────────┘   │
                                         ┌─────────────▼┐
                                         │  Cloudflare  │
                                         │  R2          │
                                         │  (Storage)   │
                                         └──────────────┘
```

### 13.2 Development Environment

```yaml
# docker/docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    ports:
      - '5432:5432'
    environment:
      POSTGRES_USER: kosku
      POSTGRES_PASSWORD: kosku_dev
      POSTGRES_DB: kosku
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'

  api:
    build:
      context: ../apps/api
      dockerfile: Dockerfile.dev
    ports:
      - '3001:3001'
    environment:
      DATABASE_URL: postgresql://kosku:kosku_dev@postgres:5432/kosku
      REDIS_URL: redis://redis:6379
    volumes:
      - ../apps/api/src:/app/src
    depends_on:
      - postgres
      - redis

  mailhog:
    image: mailhog/mailhog
    ports:
      - '1025:1025'   # SMTP
      - '8025:8025'   # Web UI

volumes:
  pgdata:
```

### 13.3 Environment Variables

```bash
# .env.example

# App
NODE_ENV=development
PORT=3001
API_URL=http://localhost:3001
TENANT_APP_URL=http://localhost:3000
OWNER_APP_URL=http://localhost:3002
CORS_ORIGINS=http://localhost:3000,http://localhost:3002

# Database
DATABASE_URL=postgresql://kosku:kosku_dev@localhost:5432/kosku

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_ACCESS_SECRET=your-access-secret-min-32-chars
JWT_REFRESH_SECRET=your-refresh-secret-min-32-chars
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=30d

# Midtrans
MIDTRANS_SERVER_KEY=SB-Mid-server-xxx
MIDTRANS_CLIENT_KEY=SB-Mid-client-xxx
MIDTRANS_MERCHANT_ID=xxx
MIDTRANS_IS_PRODUCTION=false

# Cloudflare R2
R2_ACCOUNT_ID=xxx
R2_ACCESS_KEY_ID=xxx
R2_SECRET_ACCESS_KEY=xxx
R2_BUCKET_NAME=kosku-uploads
R2_PUBLIC_URL=https://cdn.kosku.app

# Evolution API
EVOLUTION_API_URL=http://localhost:8080
EVOLUTION_API_KEY=xxx

# Resend (Email)
RESEND_API_KEY=re_xxx
EMAIL_FROM=noreply@kosku.app

# Google OAuth
GOOGLE_CLIENT_ID=xxx
GOOGLE_CLIENT_SECRET=xxx

# Apple OAuth
APPLE_CLIENT_ID=xxx
APPLE_TEAM_ID=xxx
APPLE_KEY_ID=xxx
APPLE_PRIVATE_KEY=xxx

# Mapbox
MAPBOX_ACCESS_TOKEN=pk.xxx

# Sentry
SENTRY_DSN=https://xxx@xxx.ingest.sentry.io/xxx

# Logging
LOG_LEVEL=info

# Rate Limiting
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX=100
```

---

## 14. CI/CD Pipeline

### 14.1 GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm lint

  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: kosku
          POSTGRES_PASSWORD: kosku_test
          POSTGRES_DB: kosku_test
        ports:
          - 5432:5432
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm db:migrate
      - run: pnpm test
        env:
          DATABASE_URL: postgresql://kosku:kosku_test@localhost:5432/kosku_test
          REDIS_URL: redis://localhost:6379

  build:
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm build
```

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build & push Docker image
        run: |
          docker build -t ghcr.io/kosku/api:latest -f apps/api/Dockerfile .
          docker push ghcr.io/kosku/api:latest
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /opt/kosku
            docker compose pull api
            docker compose up -d --scale api=2 api
            docker system prune -f

  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy Tenant App to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_TENANT_PROJECT_ID }}
          vercel-args: '--prod'
          working-directory: apps/web-tenant
      - name: Deploy Owner App to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_OWNER_PROJECT_ID }}
          vercel-args: '--prod'
          working-directory: apps/web-owner
```

---

## 15. Monitoring & Observability

### 15.1 Logging Strategy

| Level | Kapan | Data |
|-------|-------|------|
| `error` | Exception, 5xx, external service failure | Full error stack, request context |
| `warn` | Rate limit exceeded, invalid input, retry | Request info, warning detail |
| `info` | Business events (booking created, payment success) | Entity ID, action, duration |
| `debug` | Development only | Query detail, timing, payload |

### 15.2 Metrics to Track

| Kategori | Metrik | Tools |
|----------|--------|-------|
| **API Performance** | Response time (p50, p95, p99), request rate, error rate | Sentry, Prometheus |
| **Business** | Registrations, bookings, payments, revenue, occupancy | Custom dashboard |
| **Queue** | Job count, processing time, failure rate | BullMQ Dashboard |
| **Database** | Query performance, connection pool, slow queries | Prisma Metrics |
| **Infrastructure** | CPU, memory, disk, network | Docker stats, VPS monitoring |

### 15.3 Sentry Setup

```typescript
// common/middleware/sentry.ts
import * as Sentry from '@sentry/node';
import { nodeProfilingIntegration } from '@sentry/profiling-node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  integrations: [nodeProfilingIntegration()],
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  profilesSampleRate: 0.2,
});

// Express error handler → Otomatis capture ke Sentry
app.use(Sentry.Handlers.errorHandler());
```

---

## 16. Security

### 16.1 Security Checklist

| Area | Implementasi |
|------|--------------|
| **Password** | Argon2id (memory=64MB, iterations=3, parallelism=4) |
| **JWT** | RS256 signature, 15m access, 30d refresh, rotation |
| **Cookies** | HttpOnly, Secure, SameSite=Strict, Path=/auth |
| **CORS** | Whitelist specific origins (not wildcard) |
| **Headers** | Helmet: CSP, X-Frame-Options, X-Content-Type-Options, etc |
| **Rate Limit** | 100/min public, 300/min authenticated, stricter per endpoint |
| **SQL Injection** | Prisma ORM (parameterized queries) |
| **XSS** | Helmet CSP + React default escaping |
| **CSRF** | Double Submit Cookie pattern + SameSite cookies |
| **File Upload** | Validate MIME type, max size (10MB), scan virus |
| **R2 Access** | Signed URLs untuk private files, public bucket untuk public |
| **Input Validation** | Zod schemas di semua endpoints |
| **Auth Bypass** | RBAC middleware di setiap protected route |
| **Secrets** | Environment variables, never in code, rotate regularly |
| **Dependencies** | Dependabot auto-update, `npm audit` in CI |

### 16.2 Rate Limiting Strategy

```typescript
// common/middleware/rate-limit.ts
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

// Global
const globalLimiter = rateLimit({
  store: new RedisStore({ client: redisClient }),
  windowMs: 60 * 1000,  // 1 menit
  max: 100,              // 100 request per IP
  standardHeaders: true,
  legacyHeaders: false,
});

// Auth endpoints (prevent brute force)
const authLimiter = rateLimit({
  store: new RedisStore({ client: redisClient }),
  windowMs: 15 * 60 * 1000,  // 15 menit
  max: 10,                    // 10 attempts per IP
  message: { success: false, error: { code: 'RATE_LIMIT', message: 'Terlalu banyak percobaan. Coba lagi 15 menit.' } },
});

// API per user (authenticated)
const apiLimiter = rateLimit({
  store: new RedisStore({ client: redisClient }),
  windowMs: 60 * 1000,
  max: 300,
  keyGenerator: (req) => req.user?.userId || req.ip,
});

// Payment endpoints
const paymentLimiter = rateLimit({
  store: new RedisStore({ client: redisClient }),
  windowMs: 60 * 1000,
  max: 10,  // 10 payment attempts per menit
  keyGenerator: (req) => req.user.userId,
});
```

### 16.3 File Upload Security

```typescript
// common/middleware/upload.ts
import multer from 'multer';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import crypto from 'crypto';

const ALLOWED_MIME_TYPES = [
  'image/jpeg',
  'image/png',
  'image/webp',
  'application/pdf',
  'video/mp4',
];

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: MAX_FILE_SIZE },
  fileFilter: (_req, file, cb) => {
    if (ALLOWED_MIME_TYPES.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new AppError(400, 'INVALID_FILE_TYPE', 'Tipe file tidak diizinkan'));
    }
  },
});

async function uploadToR2(file: Express.Multer.File, folder: string) {
  const ext = file.originalname.split('.').pop();
  const key = `${folder}/${crypto.randomUUID()}.${ext}`;

  await s3Client.send(new PutObjectCommand({
    Bucket: process.env.R2_BUCKET_NAME,
    Key: key,
    Body: file.buffer,
    ContentType: file.mimetype,
  }));

  return `${process.env.R2_PUBLIC_URL}/${key}`;
}
```

---

## 17. Phased Development Plan

### Phase 1 — Foundation & MVP (Weeks 1-10)

| Week | Milestone | Deliverables |
|------|-----------|--------------|
| 1 | **Setup** | Monorepo (pnpm + turborepo), ESLint, Prettier, Husky, Docker Compose, Prisma schema |
| 2 | **Auth** | Register, Login, Logout, Refresh Token, RBAC, Google OAuth |
| 3 | **User Profile** | Profile CRUD, Avatar upload, Notification settings |
| 4 | **Property CRUD** | Create/Edit property, Upload photos, Room management (owner) |
| 5 | **Search** | Property search (filter, sort, pagination), Property detail page (public) |
| 6 | **Booking** | Ajukan sewa (tenant), Approve/Reject (owner), Booking status flow |
| 7 | **Payment** | Manual payment (upload receipt), Verify payment (owner), Payment history |
| 8 | **Dashboard** | Owner dashboard (occupancy, revenue, booking requests) |
| 9 | **WhatsApp** | Evolution API setup, QR scan, Basic notification (booking confirm) |
| 10 | **Integration & Polish** | End-to-end testing, Bug fixes, Deployment setup |

### Phase 2 — Core Features (Weeks 11-18)

| Week | Milestone | Deliverables |
|------|-----------|--------------|
| 11 | **Payment Gateway** | Midtrans integration, Snap UI, Webhook handler, Auto status update |
| 12 | **Real-time Chat** | Socket.IO setup, Chat UI (tenant + owner), File sharing |
| 13 | **Digital Contract** | Contract upload, E-signature, Contract history |
| 14 | **Tenant Management** | Tenant list, Tenant history, Blacklist, Tenant screening |
| 15 | **Billing Automation** | Auto generate monthly bills, Bill tracking, Overdue status |
| 16 | **Payment Reminders** | WA reminders (H-7, H-3, H, H+3, H+7), Email fallback |
| 17 | **Financial Reports** | Revenue report, Occupancy report, Export (CSV/Excel/PDF) |
| 18 | **Map View** | Mapbox integration, Map view search, Nearby properties |

### Phase 3 — Engagement (Weeks 19-26)

| Week | Milestone | Deliverables |
|------|-----------|--------------|
| 19 | **Reviews & Ratings** | Write review, Rating system, Owner response |
| 20 | **Survey Scheduling** | Schedule visit, Confirm/Cancel survey, Reminder notifications |
| 21 | **Waitlist** | Join waitlist, Auto-notify when available |
| 22 | **Favorites** | Add/remove favorites, Wishlist page |
| 23 | **MamiPoin** | Points system, Earn points, Redeem rewards, Tier system |
| 24 | **Promos & Vouchers** | Create promo (owner), Voucher codes, Validate & apply |
| 25 | **Notifications Hub** | In-app notification center, Push notification, Notification preferences |
| 26 | **Blog & Help** | Blog CRUD (admin), FAQ, Help center, Contact form |

### Phase 4 — Monetization (Weeks 27-32)

| Week | Milestone | Deliverables |
|------|-----------|--------------|
| 27 | **GoldPlus** | Subscription plans, Buy subscription, Feature gating |
| 28 | **MamiAds** | Ad packages, Create ad, Ad placement, Impression/click tracking |
| 29 | **MamiPrime** | Search ranking boost, Buy prime, Expiry handling |
| 30 | **Broadcast Chat** | Send bulk message (GoldPlus), Template messages |
| 31 | **Competitor Analysis** | Nearby property prices, Occupancy comparison, Market insights |
| 32 | **Owner Services** | MamiFoto (photo service), MamiTour (virtual tour), Professional photography booking |

### Phase 5 — Admin & Scale (Weeks 33-40)

| Week | Milestone | Deliverables |
|------|-----------|--------------|
| 33 | **Admin Panel** | User management, Property verification, Report moderation |
| 34 | **Admin Dashboard** | Platform stats, Transaction monitoring, Fraud detection |
| 35 | **Performance** | Caching strategy, Query optimization, CDN tuning, Load testing |
| 36 | **Security Audit** | Penetration testing, Dependency audit, Security hardening |
| 37 | **Virtual Tour 360°** | VR viewer integration, Upload & process 360° photos |
| 38 | **Advanced Analytics** | Owner insights, Market trends, Pricing suggestions |
| 39 | **i18n & A11y** | Multi-language support, Accessibility compliance |
| 40 | **Launch** | Production deployment, Monitoring setup, Documentation, Handover |

---

## Appendix

### A. Naming Conventions

| Konteks | Convention | Contoh |
|---------|------------|--------|
| **Files** | kebab-case | `auth.controller.ts`, `property-card.tsx` |
| **Components** | PascalCase | `PropertyCard`, `SearchFilters` |
| **Functions** | camelCase | `getUserById`, `createBooking` |
| **API Routes** | kebab-case | `/owner/properties/pending` |
| **Database Tables** | snake_case plural | `users`, `properties`, `bookings` |
| **DB Columns** | snake_case | `owner_id`, `business_name`, `created_at` |
| **ENV Variables** | UPPER_SNAKE | `DATABASE_URL`, `JWT_ACCESS_SECRET` |
| **Git Branches** | `{type}/{description}` | `feat/payment-gateway`, `fix/booking-expiry` |
| **Commits** | Conventional Commits | `feat: add Midtrans payment integration` |

### B. Git Strategy

```
main          ─── ● ─── ● ─── ● ─── ● ─── (production)
                    \         /
develop       ────── ● ─── ● ─── ● ─── (integration)
                      \   /
feat/payment  ──────── ● ─── ● ─── (feature branch)
```

- `main`: Production-ready, protected, direct push tidak diizinkan
- `develop`: Integration branch, base untuk feature branches
- `feat/*`: Feature branches, merge via PR ke develop
- `fix/*`: Bugfix branches
- `chore/*`: Dependency updates, tooling

Commit format: `type(scope): description` (Conventional Commits)

### C. Testing Strategy

| Layer | Tools | Target Coverage |
|-------|-------|-----------------|
| **Unit (Backend)** | Jest + Supertest | Service layer, validation, helpers → 80%+ |
| **Integration (Backend)** | Jest + Supertest | API endpoints, database queries → 70%+ |
| **Unit (Frontend)** | Vitest + Testing Library | Components, hooks, utils → 70%+ |
| **E2E** | Playwright | Critical flows (register, booking, payment) → 3-5 flows |

### D. Documentation

| Dokumen | Tools | Update |
|---------|-------|--------|
| **API Docs** | Swagger (OpenAPI 3.0) | Auto-generate dari Zod schemas via `zod-to-json-schema` |
| **Architecture** | `arsitektur.md` (file ini) | Perubahan signifikan |
| **Setup Guide** | `README.md` | Setup instructions, env vars, commands |
| **PRD** | `prd.md` | Product requirements & feature list |
| **Database** | Prisma Studio + `schema.prisma` | Schema documentation via comments |
| **Runbook** | Internal wiki | Deployment, incident response, common issues |

---

> **Dokumen ini hidup.** Arsitktur akan berevolusi seiring development. Setiap keputusan arsitektural signifikan harus dicatat sebagai ADR (Architecture Decision Record) di folder `docs/adr/`.
