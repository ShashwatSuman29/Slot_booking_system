# Slot Booking System

A modern and scalable Slot Booking Web Application built using **Next.js**, **TypeScript**, **Node.js/Express**, **MongoDB**, and **Redis**.

The application supports secure authentication, real-time slot availability, distributed locking to prevent double bookings, optimistic UI updates, and a production-ready scalable architecture.

---

## Features

- JWT Authentication & RBAC
- Slot Booking & Cancellation
- Redis Distributed Locking
- MongoDB Transactions
- Optimistic UI Updates
- React Query + Zustand
- Admin & User APIs
- Rate Limiting & Security Middleware
- Responsive Dashboard UI
- Production Deployment Ready

---

## Tech Stack

### Frontend
- Next.js App Router
- TypeScript
- Tailwind CSS
- TanStack Query
- Zustand

### Backend
- Node.js
- Express.js
- MongoDB + Mongoose
- Redis
- JWT Authentication
- Zod Validation

---

## Project Structure

```bash
slot-booking-app/
├── backend/
├── frontend/
└── README.md
```

---

## Environment Variables

### Backend (`backend/.env`)

```env
PORT=5000

MONGO_URI=

REDIS_URL=

JWT_SECRET=
JWT_REFRESH_SECRET=

CORS_ORIGIN=http://localhost:3000
```

### Frontend (`frontend/.env.local`)

```env
NEXT_PUBLIC_API_URL=http://localhost:5000/api/v1
```

---

## Installation

### Clone Repository

```bash
git clone <repo-url>

cd slot-booking-app
```

---

## Backend Setup

```bash
cd backend

npm install

npm run dev
```

Backend runs on:

```bash
http://localhost:5000
```

---

## Frontend Setup

```bash
cd frontend

npm install

npm run dev
```

Frontend runs on:

```bash
http://localhost:3000
```

---

## Production Build

### Backend

```bash
npm run build
npm start
```

### Frontend

```bash
npm run build
npm start
```

---

## Docker

```bash
docker build -t booking-api .

docker run -p 5000:5000 booking-api
```

---

## API Routes

### Auth
```http
POST /api/v1/auth/register
POST /api/v1/auth/login
```

### Slots
```http
GET /api/v1/slots
POST /api/v1/slots
```

### Bookings
```http
POST /api/v1/bookings
PATCH /api/v1/bookings/:id/cancel
```

---

## Deployment

### Frontend
- Vercel

### Backend
- Railway
- Render
- AWS ECS

---

## Future Improvements

- Google Calendar Integration
- Stripe Payments
- Email Notifications
- SMS Reminders
- WebSocket Real-Time Updates

---

## License

MIT License
