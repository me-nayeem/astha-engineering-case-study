# Astha Engineering & Solutions — B2B Industrial Automation Platform

A production web platform built for **Astha Engineering & Solutions Ltd.**, an authorised importer and distributor of industrial automation equipment (PLCs, HMIs, VFDs, servo drives/motors) in Bangladesh. The platform replaces a static/offline sales process with a full digital catalog, a bilingual (English/Bangla) storefront, a quote-request workflow in place of a shopping cart, a repair-job intake system, and an internal admin panel for managing the entire product catalog and customer pipeline.

**Live site:** [asthaengineering.com](https://www.asthaengineering.com)

---

## 1. About the project

Astha Engineering sells and services industrial automation hardware — PLCs, HMIs, VFDs, servo motors/drives — from brands like Mitsubishi Electric, Siemens, Delta, Panasonic, Yaskawa, ABB, LS Electric, and Weintek. Their business model is B2B and quote-driven rather than transactional: customers browse a catalog, then reach out via a quote request or WhatsApp rather than checking out through a cart.

This project digitizes that entire workflow:

- A public storefront where factories, textile mills, and industrial clients can browse 500+ products by category, brand, and stock status.
- A **quote-request** system that replaces "add to cart / checkout" — customers request a quote per product, and the request lands directly in the admin's queue.
- A **repair-job intake** system for the company's board-level repair service (PLC, HMI, SMPS, VFD/inverter repair), tracked through a status pipeline from received to delivered.
- A bilingual experience (English / Bangla) across both UI copy and catalog content, since the customer base spans both languages.
- A full **admin dashboard** for non-technical staff to manage products, categories, brands, customers, repair jobs, and site settings without touching code.

## 2. What I built

I designed and implemented the system end-to-end — data model, REST API, authentication, admin tooling, and the customer-facing UI:

- **Product catalog** with category/brand/condition/stock filtering, featured products, image galleries (multi-image per product via Cloudinary), and structured bilingual spec tables (key/value specs per product).
- **Quote request pipeline**: public quote submission tied to a product, with status tracking (`PENDING → VIEWED → RESPONDED`) and an admin inbox to manage and respond to leads.
- **Repair job tracking**: public repair-request form (works for guests and logged-in customers) with a full status workflow (`RECEIVED → DIAGNOSING → REPAIRING → COMPLETED → DELIVERED`), technician notes, and estimated turnaround.
- **Authentication & accounts**: JWT-based auth with refresh tokens, bcrypt password hashing, forgot/reset-password flow, and a customer dashboard (profile, saved/wishlisted products, inquiry history).
- **Role-based admin panel**: a separate `ADMIN` role and route tree (`/admin/*`) for managing products, categories, brands, customers, repair jobs, and global site settings — gated by server-side middleware and client-side route guards.
- **Bilingual content model**: nearly every content field in the database (product names, descriptions, specs, category/brand copy) is stored as `_en` / `_bn` pairs, with `i18next` driving the UI language switch.
- **SEO & discoverability**: metadata via `react-helmet-async`, `robots.txt`, and a generated `sitemap.xml`.
- **Operational hardening**: rate limiting, Helmet security headers, a CORS allowlist scoped to the production domains, and centralized error handling on the API.

## 3. Architecture

The system is a decoupled client/server application: a Vite-built React SPA talking to a REST API over JSON, backed by PostgreSQL via Prisma, with image assets offloaded to Cloudinary and transactional email via Gmail/Nodemailer.

```
┌───────────────────────────────┐        HTTPS / JSON (axios)        ┌───────────────────────────────┐
│            CLIENT              │  ────────────────────────────▶   │            SERVER              │
│  React 19 + Vite (SPA)         │                                   │  Node.js + Express 5           │
│  ─────────────────────────     │  ◀────────────────────────────   │  ─────────────────────────     │
│  • React Router v7 routing     │                                   │  helmet · cors · rate-limit    │
│  • TanStack Query (server      │                                   │  express.json · morgan         │
│    state / caching)            │                                   │                                │
│  • Zustand (auth store)        │                                   │  Routes: auth · products ·     │
│  • React Hook Form + Zod       │                                   │  categories · brands · quotes ·│
│  • shadcn/ui + Radix + Tailwind│                                   │  repairs · wishlist · upload · │
│  • i18next (en / bn)           │                                   │  settings · admin              │
│  • Public / Dashboard / Admin  │                                   │                                │
│    layouts + route guards      │                                   │  JWT auth (access + refresh)   │
└────────────────┬────────────────┘                                  │  bcrypt password hashing       │
                 │                                                   │  Zod request validation        │
                 │ media URLs                                       └────────────────┬────────────────┘
                 ▼                                                                    │ Prisma ORM
┌───────────────────────────────┐                                                     ▼
│          CLOUDINARY            │                                   ┌───────────────────────────────┐
│  Product images & brand        │                                   │          PostgreSQL             │
│  logos (via Multer memory      │                                   │  users · refresh_tokens ·      │
│  upload → Cloudinary API)      │                                   │  products · categories ·       │
└───────────────────────────────┘                                   │  brands · product_images ·     │
                                                                      │  product_specs · wishlists ·   │
┌───────────────────────────────┐                                   │  quote_requests · repair_jobs ·│
│       NODEMAILER (Gmail)       │  ◀──────── password reset,         │  site_settings                 │
│  Transactional email           │            notification emails     └───────────────────────────────┘
└───────────────────────────────┘

Deployment: Client → Vercel (SPA rewrite via vercel.json)  |  Server → Node host  |  DB → PostgreSQL (Neon-compatible via @prisma/adapter-neon)
```

**Data model highlights** (PostgreSQL via Prisma): `User` (with `Role` enum `ADMIN`/`CUSTOMER`), `RefreshToken`, `Category`, `Brand`, `Product` (with `ProductCondition` and `StockStatus` enums, indexed on category/brand/condition/stock/featured for fast filtering), `ProductImage`, `ProductSpec`, `Wishlist`, `QuoteRequest` (with `QuoteStatus` pipeline), `RepairJob` (with `RepairStatus` pipeline), and `SiteSetting` for admin-editable config.

## 4. Screenshots

| Home | Featured products |
|---|---|
| ![Home page](public/gitImage/astha%20home.png) | ![Featured products](public/gitImage/astha%20featuress.png) |

| Product catalog & filters | About |
|---|---|
| ![Product listing](public/gitImage/astha%20products.png) | ![About page](public/gitImage/astha%20about.png) |

| Repair services | Contact |
|---|---|
| ![Repair services](public/gitImage/astha%20repair.png) | ![Contact page](public/gitImage/astha%20contact%20.png) |

## 5. Tech stack

**Frontend**
- React 19 + Vite
- React Router v7 (lazy-loaded routes)
- TanStack Query (server-state caching) + Zustand (auth state)
- React Hook Form + Zod (form validation)
- shadcn/ui + Radix UI + Tailwind CSS v4
- i18next / react-i18next (English + Bangla)
- react-helmet-async (SEO metadata), yet-another-react-lightbox (image galleries), Sonner / react-hot-toast (notifications)

**Backend**
- Node.js + Express 5
- PostgreSQL + Prisma ORM (with `@prisma/adapter-pg` / `@prisma/adapter-neon`)
- JWT (access + refresh tokens) + bcryptjs for authentication
- Zod for request validation, express-async-handler for async error handling
- Multer + Cloudinary for image uploads and storage
- Nodemailer (Gmail transport) for transactional email
- Helmet, CORS allowlisting, express-rate-limit for API hardening

**Tooling & deployment**
- ESLint + Prettier
- Prisma Migrate for schema migrations, Prisma Studio for data inspection
- Vercel (client, SPA), Node host (API), PostgreSQL (Neon-compatible)

## 6. Key features at a glance

- Filterable, paginated product catalog (category, brand, condition, stock status, featured)
- Full bilingual content (English / Bangla) across UI and database
- Quote-request workflow in place of a cart/checkout — matches the real B2B sales process
- Repair-job intake with a full status pipeline and technician notes
- JWT authentication with refresh tokens, password reset, and role-based access control
- Admin dashboard for products, categories, brands, customers, repair jobs, and site settings
- Customer wishlist / saved products and inquiry history
- Cloudinary-backed multi-image product galleries
- SEO-ready: metadata via `react-helmet-async`, `robots.txt`, `sitemap.xml`
- Hardened API: Helmet, CORS allowlist, rate limiting, centralized error handling

---

*This repository is a case-study snapshot documenting the architecture of the production [Astha Engineering & Solutions](https://www.asthaengineering.com) platform and my role in building it.*
