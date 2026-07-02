<div align="center">

# Jalur Berlian — Fleet Management System

**Sistem manajemen armada & order pengiriman kontainer**
PT. Jalur Berlian Makassar · Sulawesi Selatan

[![Go](https://img.shields.io/badge/Go-1.25-00ADD8?style=flat-square&logo=go&logoColor=white)](https://github.com/izzahnin/jb-backend)
[![Next.js](https://img.shields.io/badge/Next.js-16-000000?style=flat-square&logo=nextdotjs&logoColor=white)](https://github.com/izzahnin/jb-frontend)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://supabase.com)
[![Redis](https://img.shields.io/badge/Redis-7-DC382D?style=flat-square&logo=redis&logoColor=white)](https://upstash.com)
[![Vercel](https://img.shields.io/badge/Vercel-deployed-000000?style=flat-square&logo=vercel)](https://jalurberlian.vercel.app)

[🌐 Live Demo](https://jalurberlian.vercel.app) · [⚙️ Backend Repo](https://github.com/izzahnin/jb-backend) · [🖥️ Frontend Repo](https://github.com/izzahnin/jb-frontend)

</div>

---

## 🔑 Coba Demo

Buka live demo, login dengan akun berikut — **read-only, tidak bisa mengubah data**.

```
URL   : https://jalurberlian.vercel.app
Login : demo
Pass  : demo1234
```

> Akun `demo` bisa melihat semua halaman dan data. Semua aksi create/edit/delete diblokir di level backend dan frontend.

---

## Tentang Proyek

Jalur Berlian adalah aplikasi web **full-stack production-ready** untuk mengelola operasional armada truk pengiriman kontainer. Dibangun dari nol sebagai proyek portofolio dengan arsitektur dan keamanan setara sistem enterprise.

**Fitur utama:**
- Manajemen Customer, Truck, Driver, dan Order dengan audit trail lengkap
- Sistem Dispatch — buat trip, update status, tracking posisi GPS real-time
- GPS tracking berbasis checkpoint — cache Redis, history polyline di peta
- **Public tracking page** — customer lacak pengiriman hanya dengan nomor order, tanpa login
- RBAC 4 role: `super_admin`, `admin_sales`, `admin_ops`, `demo`
- Rate limiting: login 10 req/60s per IP, GPS throttle 30s/trip
- Auto-logout saat token expired, NPWP auto-formatter, mobile responsive

---

## Screenshots

> *(Tambahkan screenshot setelah push — gunakan drag & drop ke GitHub issue untuk dapat URL gambar)*

| Dashboard Stats | Dispatch + GPS Map |
|:---:|:---:|
| <img width="1920" height="945" alt="ScreenShot Tool -20260702163007" src="https://github.com/user-attachments/assets/3998dd41-c507-4368-8954-179a44078516" /> | <img width="1920" height="945" alt="ScreenShot Tool -20260702163036" src="https://github.com/user-attachments/assets/97ccc8e7-7441-4ca2-a8b4-5ff40b4d56c5" />
 |

| Public Tracking | Mobile View |
|:---:|:---:|
|<img width="1920" height="945" alt="ScreenShot Tool -20260702163121" src="https://github.com/user-attachments/assets/0b7d5ba9-0b7c-46a2-bf8a-828b4fb0efec" /> | <img width="1080" height="2255" alt="Screenshot 2026-07-02 165539" src="https://github.com/user-attachments/assets/20a8d0bc-8ac2-4124-9585-314951f48823" />

 |

---

## Arsitektur

```
Browser / Mobile
      │
      ▼
 Vercel (Next.js 16)          ← Frontend: SSR + Edge Middleware RBAC
      │
      ▼
 Render (Go 1.25 + Gin)       ← REST API: 36 endpoint, JWT auth, rate limiting
      │
      ├──► Supabase (PostgreSQL 15)   ← Data utama, relasional, audit trail
      └──► Upstash (Redis 7)          ← Cache GPS terbaru, rate limit counter
```

**Backend — Clean Architecture:**
```
HTTP Request → Handler → Usecase → Repository → PostgreSQL / Redis
```

**Middleware stack:**
```
AuthMiddleware          → validasi JWT, inject user_id + role
AdminMiddleware         → whitelist role valid
DemoReadOnlyMiddleware  → blokir POST/PATCH/DELETE untuk role demo
RequireRoles(...)       → RBAC granular per endpoint
```

---

## Tech Stack

### Backend [`→ jb-backend`](https://github.com/izzahnin/jb-backend)

| Teknologi | Kegunaan |
|-----------|----------|
| Go 1.25 + Gin v1.12 | Web framework — routing, middleware, JSON binding |
| PostgreSQL 15 | Database utama, audit trail, partitioned locations table |
| Redis 7 | Cache GPS terbaru per trip + rate limiting counter |
| sqlx v1.4 | SQL toolkit — query + struct scanning |
| JWT HS256 (24h) | Autentikasi stateless |
| Docker Compose | Local development — satu perintah nyalakan semua service |

### Frontend [`→ jb-frontend`](https://github.com/izzahnin/jb-frontend)

| Teknologi | Kegunaan |
|-----------|----------|
| Next.js 16 + React 19 | App Router, SSR, TypeScript strict mode |
| Tailwind CSS v4 | Dark navy theme, mobile-first responsive |
| TanStack Table v8 | Tabel dengan sorting, filtering, pagination |
| react-leaflet v5 | Peta interaktif — GPS tracking + map picker koordinat |
| Next.js Middleware | Auth guard + RBAC redirect server-side (Edge Runtime) |

### Infrastructure

| Layanan | Platform |
|---------|----------|
| Frontend | Vercel (auto-deploy dari GitHub) |
| Backend | Render |
| Database | Supabase (PostgreSQL managed, SSL) |
| Cache | Upstash (Redis serverless) |

---

## Domain & State Machine

**Order:** `pending → partial → completed | cancelled`

**Trip:** `pickup → in_transit → delivered | cancelled`

Status hanya bisa maju — dikontrol backend + CHECK constraint PostgreSQL. Driver & Truck dikunci `on_duty` otomatis saat trip aktif.

---

## Stats

| | |
|---|---|
| 36 | REST API Endpoints |
| 8 | Halaman Admin Dashboard |
| 4 | Role Level (super_admin · admin_sales · admin_ops · demo) |
| 7 | Tabel Database |
| ~15 | Foreign Key Constraints |
| 2 | Rate-limited Endpoints (Redis) |

---

## Menjalankan Lokal

```bash
# Backend (Docker — nyalakan PostgreSQL + Redis + API sekaligus)
git clone https://github.com/izzahnin/jb-backend
cd jb-backend
cp .env.local.example .env.local   # isi JWT_SECRET dll
docker-compose up -d --build
# API: http://localhost:8080

# Frontend
git clone https://github.com/izzahnin/jb-frontend
cd jb-frontend
cp .env.local.example .env.local   # NEXT_PUBLIC_API_URL=http://localhost:8080
npm install && npm run dev
# http://localhost:3000
```

---

## Kontak

**Nurul Izzah Nurhidayat** · Makassar, Sulawesi Selatan

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/in/nurul-izzah-nurhidayat-397346289)
[![GitHub](https://img.shields.io/badge/GitHub-Profile-181717?style=flat-square&logo=github)](https://github.com/izzahnin)
