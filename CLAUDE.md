# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PILAR** (Peduli Laut dan Pesisir) is a full-stack monorepo for a beach/coastal cleanup volunteer platform. It consists of two separate projects:

- `pilar-backend/` — NestJS REST API running on port **3001**
- `pilar-frontend/` — Next.js web app running on port **3000**

## Commands

All commands must be run from within the respective subdirectory.

### Backend (`cd pilar-backend`)

```bash
npm run start:dev      # Development server with watch mode
npm run build          # Compile TypeScript to dist/
npm run start:prod     # Run compiled production build
npm run lint           # ESLint with auto-fix
npm run format         # Prettier formatting
npm run test           # Unit tests (Jest)
npm run test:watch     # Jest watch mode
npm run test:cov       # Coverage report
npm run test:e2e       # End-to-end tests
npx prisma migrate dev # Run DB migrations
npx prisma generate    # Regenerate Prisma client
npx prisma studio      # Visual DB browser
```

### Frontend (`cd pilar-frontend`)

```bash
npm run dev            # Development server
npm run build          # Next.js production build
npm run start          # Start production server
npm run lint           # ESLint
```

## Architecture

### Backend (NestJS)

Feature-based module structure — each feature has its own module, controller, and service under `src/`:

- **`auth/`** — JWT strategy, Passport guards, login/register endpoints, `@GetUser()` decorator
- **`events/`** — Event CRUD (admin creates, users view)
- **`pendaftaran/`** — Volunteer registration/enrollment with approve/reject flow
- **`dokumentasi/`** — Photo uploads for events (via Supabase storage)
- **`sampah/`** — Waste tracking records per event
- **`laporan/`** — Reporting
- **`sertifikat/`** — Certificate generation for approved registrations
- **`prisma/`** — Shared `PrismaService` injectable across all modules

**Key globals:**
- API prefix: `/api` on all routes
- Global `ValidationPipe` with `whitelist: true, transform: true`
- CORS enabled for `localhost:3000`
- JWT auth via `@UseGuards(JwtAuthGuard)`

### Frontend (Next.js App Router)

- **`app/`** — Pages using the App Router; server components by default, `'use client'` where interactivity is needed
- **`components/layout/`** — `Sidebar`, `DashboardLayout` wrapping authenticated pages
- **`components/ui/`** — Reusable UI primitives
- **`lib/api.ts`** — Axios instance that auto-attaches `Authorization: Bearer <token>` from localStorage and redirects to `/login` on 401
- **`lib/store.ts`** — Zustand auth store persisted to localStorage (user object + token)

**Auth flow:** Login → JWT stored in localStorage + cookie → Axios interceptor attaches header → 401 triggers automatic redirect to `/login`.

### Database (Prisma + PostgreSQL)

Schema defined in `pilar-backend/prisma/schema.prisma`. Core models:

- **`User`** — role: `USER | ADMIN`, with bio, photo, phone
- **`Event`** — status: `UPCOMING | ONGOING | DONE`, with quota tracking
- **`Pendaftaran`** — User↔Event join with status `PENDING | APPROVED | REJECTED`, includes health info (JSON)
- **`Dokumentasi`** — Photos with captions, linked to Event
- **`Sampah`** — Waste entries (type + kg) per Event
- **`Sertifikat`** — One-to-one with approved Pendaftaran

## Tech Stack Summary

| Layer | Technology |
|---|---|
| Backend framework | NestJS 11 |
| ORM | Prisma + PostgreSQL |
| File storage | Supabase |
| Auth | JWT + Passport.js |
| Frontend framework | Next.js 16 (React 19) |
| Styling | TailwindCSS |
| State management | Zustand |
| Forms | React Hook Form + Zod |
| HTTP client | Axios |
| Validation (backend) | class-validator / class-transformer |

## UI Design System (Premium Upgrade)

All pages use a cohesive premium design language with inline styles (no globals.css modifications). Key design patterns:

### Design Tokens
- **Primary gradient:** `linear-gradient(135deg, #0ea5e9, #0369a1)` (ocean blue)
- **Background gradient:** `linear-gradient(135deg, #fdfaf5, #f0f7ff)` (warm-to-cool)
- **Border color:** `rgba(14,165,233,0.06)` (ultra-subtle blue tint)
- **Card shadow:** `0 4px 16px rgba(0,0,0,0.02)` (barely-there depth)
- **Hover shadow:** `0 12px 32px rgba(14,165,233,0.12)` (elevated blue glow)
- **Border radius:** `18-20px` for cards, `10-12px` for buttons, `8-10px` for nav items

### Animation Patterns
- **Entrance:** `_dashFade` / `_sertFade` etc. — `translateY(14px)` + opacity fade, staggered via `animation-delay`
- **Hover cards:** `translateY(-3px to -6px)` + enhanced shadow via CSS class transitions
- **Hover list items:** `translateX(4px)` subtle rightward slide
- **Hover buttons:** `translateY(-2px)` + glow shadow
- **Sidebar nav:** `translateX(3px)` hover slide with `_sidebarFadeIn` stagger per item
- All transitions use `cubic-bezier(0.4,0,0.2,1)` for natural motion

### Component Patterns
- **Stat cards:** Icon in gradient pill + label above large number, decorative corner orb
- **Section headers:** Accent color label (uppercase, letter-spaced) above serif/bold heading
- **Status badges:** Gradient background matching status color, rounded pill shape
- **Progress bars:** `linear-gradient(to right, #38bdf8, #0369a1)` with glow shadow
- **Active nav items:** Blue gradient background + dot indicator + icon highlight
- **Empty states:** Centered icon in gradient circle + descriptive text

### Sidebar Navigation
- **Width:** `240px` (was 220px)
- Includes icons for each nav item via inline SVG
- User section shows online status indicator (green dot with glow)
- Menu label divider between user info and nav items
- Sertifikat menu added for USER role (between Dashboard and Profil)
- LandingSidebar also includes Sertifikat link for non-admin users

### Pages Enhanced
| Page | File | Key Changes |
|---|---|---|
| Landing | `app/page.tsx` | Animated hero with gradient text, floating orbs, CTA buttons, premium event cards with date overlay badge, icon-enriched stat cards |
| Login | `app/login/page.tsx` | Custom styled inputs with focus glow, animated left panel, gradient CTA button with hover lift |
| Register | `app/register/page.tsx` | Consistent with login, custom inputs, animated decorations |
| User Dashboard | `app/dashboard/page.tsx` | Icon stat cards with gradient accents, hover-sliding list items, section hover shadows |
| Admin Dashboard | `app/dashboard/admin/page.tsx` | Gradient add-event button, progress bars in table, row hover highlights |
| Event Detail | `app/events/[id]/page.tsx` | Image hover zoom, icon-backed info rows, gradient top accent on registration card, enhanced CTA |
| Sertifikat | `app/sertifikat/page.tsx` | Gradient icon badges, top accent stripe on certificate cards, glow animation on empty state |
| Profile | `app/profile/page.tsx` | Avatar overlay with backdrop blur, gradient stat pills, custom form inputs |
| Settings | `app/settings/page.tsx` | Icon section headers, animated toggle switches, danger zone with warning icon |
| Admin Events | `app/dashboard/admin/events/page.tsx` | Progress bars per event card, enhanced filter pills, icon-enriched metadata |
| Sidebar | `components/layout/Sidebar.tsx` | Icons per nav item, active glow dot, gradient background, staggered entrance animation |
| LandingSidebar | `components/layout/LandingSidebar.tsx` | Glass-morphism overlay, gradient user card, premium close button |
