# Allo Inventory Reservation System

A full-stack inventory reservation platform built as part of the Allo Engineering Take-Home Exercise.

This application solves the checkout race-condition problem in multi-warehouse inventory systems by introducing temporary stock reservations.

When a customer proceeds to checkout, inventory units are reserved for a limited period (10 minutes). If payment succeeds, the reservation is confirmed and stock is permanently allocated. If payment fails or the reservation expires, the reserved inventory is released automatically.



---

## Problem Statement

In high-traffic systems, multiple customers may try to purchase the same inventory item simultaneously.

If stock is reduced only after payment:

- Multiple customers may purchase the same physical inventory
- Overselling occurs
- Refunds become necessary

If stock is reduced immediately after adding to cart:

- Inventory appears unavailable
- Cart abandonment causes false stock depletion

This system solves the problem using temporary inventory reservations.

---

## Features

### Inventory Management

- Products
- Warehouses
- Inventory per warehouse
- Available stock calculation
- Reserved stock tracking

### Reservation System

- Reserve inventory
- Confirm reservation
- Release reservation
- Automatic reservation expiration

### Frontend

- Product listing page
- Warehouse stock display
- Reserve button
- Reservation page
- Live countdown timer
- Confirm purchase button
- Cancel button
- Real-time UI updates
- Error handling for:
  - 409 Conflict
  - 410 Gone

### Backend

- REST API implementation
- Transaction-safe reservation handling
- Concurrency protection
- PostgreSQL database
- Prisma ORM

### Bonus Features

- Redis-based idempotency support
- Retry-safe APIs

---

## Tech Stack

### Frontend

- Next.js (App Router)
- TypeScript
- Tailwind CSS
- shadcn/ui

### Backend

- Next.js API Routes
- Prisma ORM
- PostgreSQL (Supabase)

### Infrastructure

- Vercel
- Upstash Redis

---


## Database Schema

Main entities:

### Product

- id
- name
- createdAt

### Warehouse

- id
- name
- location
- createdAt

### Inventory

- id
- productId
- warehouseId
- totalUnits
- reservedUnits

### Reservation

- id
- productId
- warehouseId
- quantity
- status
- expiresAt
- createdAt

Reservation Status:

- PENDING
- CONFIRMED
- RELEASED

---

## API Endpoints

| Method | Endpoint | Description |
|----------|------------|-------------|
| GET | /api/products | List products with stock |
| GET | /api/warehouses | List warehouses |
| POST | /api/reservations | Create reservation |
| POST | /api/reservations/:id/confirm | Confirm reservation |
| POST | /api/reservations/:id/release | Release reservation |

---

## Running Locally

### Clone repository

```bash
git clone <your-repository-url>

cd allo-inventory-reservation-system
```

### Install dependencies

```bash
npm install
```

### Create environment variables

Create:

```env
.env
```

Add:

```env
DATABASE_URL=your_pool_connection_url

DIRECT_URL=your_direct_database_url

REDIS_URL=your_upstash_url

REDIS_TOKEN=your_upstash_token
```

### Run Prisma migration

```bash
npx prisma migrate dev
```

### Seed database

```bash
npx prisma db seed
```

### Start development server

```bash
npm run dev
```

Open:

```text
http://localhost:3000
```

---

## Concurrency Strategy

This project prevents race conditions during inventory reservations.

The reservation API uses:

- Prisma transactions
- Database-level consistency checks

Flow:

```text
Check available stock

availableStock = totalUnits - reservedUnits

If availableStock >= quantity:

Increase reservedUnits

Create reservation

Else:

Return 409
```

Why this matters:

If two users attempt to reserve the final inventory unit simultaneously:

- Only one transaction succeeds
- The other receives:

```text
409 Conflict
```

This prevents overselling.

---

## Reservation Expiry

Reservations automatically expire after a configured time window.

Production approach:

- Vercel Cron Job

Flow:

```text
Every minute:

Find expired reservations

Update status:

PENDING → RELEASED

Reduce reservedUnits
```

---

## Idempotency Strategy

API requests support an Idempotency-Key header.

Flow:

```text
Client Request

↓

Check Redis

↓

Existing Key

Return previous response

↓

Otherwise

Process request

Store result
```

This prevents duplicate operations caused by retries.

---

## Trade-offs

Current implementation prioritizes:

- Correctness
- Simplicity
- Readability


---

