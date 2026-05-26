# Golden Response: Full-Stack Slot Booking Web Application

> **Scope**: Production-grade, end-to-end implementation covering folder structure, database schema, backend APIs, frontend pages, auth, RBAC, conflict prevention, admin panel, and deployment. Every code block is copy-paste ready.

---

## Table of Contents

1. [Folder Structure](#1-folder-structure)
2. [Environment Variables](#2-environment-variables)
3. [Database Schema (Mongoose)](#3-database-schema-mongoose)
4. [Backend — Auth APIs](#4-backend--auth-apis)
5. [Backend — Slot APIs](#5-backend--slot-apis)
6. [Backend — Booking APIs](#6-backend--booking-apis)
7. [Backend — Admin APIs](#7-backend--admin-apis)
8. [Middleware (Auth + RBAC + Rate Limit)](#8-middleware-auth--rbac--rate-limit)
9. [Frontend — Next.js Setup](#9-frontend--nextjs-setup)
10. [Frontend — Auth Pages](#10-frontend--auth-pages)
11. [Frontend — Dashboard](#11-frontend--dashboard)
12. [Frontend — Slot Selection Page](#12-frontend--slot-selection-page)
13. [Frontend — Booking Confirmation](#13-frontend--booking-confirmation)
14. [Frontend — Admin Panel](#14-frontend--admin-panel)
15. [Frontend — API Client + React Query Hooks](#15-frontend--api-client--react-query-hooks)
16. [Frontend — Shared Components](#16-frontend--shared-components)
17. [Security Considerations](#17-security-considerations)
18. [Scaling Strategy](#18-scaling-strategy)
19. [Deployment Guide](#19-deployment-guide)
20. [API Documentation (Postman-Ready)](#20-api-documentation-postman-ready)

---

## 1. Folder Structure

```
slot-booking/
├── apps/
│   ├── backend/                        # Node.js + Express API
│   │   ├── src/
│   │   │   ├── config/
│   │   │   │   ├── db.ts               # MongoDB connection
│   │   │   │   └── redis.ts            # Redis client (optional)
│   │   │   ├── models/
│   │   │   │   ├── User.model.ts
│   │   │   │   ├── Slot.model.ts
│   │   │   │   └── Booking.model.ts
│   │   │   ├── controllers/
│   │   │   │   ├── auth.controller.ts
│   │   │   │   ├── slot.controller.ts
│   │   │   │   ├── booking.controller.ts
│   │   │   │   └── admin.controller.ts
│   │   │   ├── routes/
│   │   │   │   ├── auth.routes.ts
│   │   │   │   ├── slot.routes.ts
│   │   │   │   ├── booking.routes.ts
│   │   │   │   └── admin.routes.ts
│   │   │   ├── middleware/
│   │   │   │   ├── auth.middleware.ts  # JWT verify
│   │   │   │   ├── rbac.middleware.ts  # Role guard
│   │   │   │   ├── validate.middleware.ts
│   │   │   │   └── rateLimiter.ts
│   │   │   ├── validators/
│   │   │   │   ├── auth.schema.ts      # Zod schemas
│   │   │   │   ├── slot.schema.ts
│   │   │   │   └── booking.schema.ts
│   │   │   ├── services/
│   │   │   │   ├── token.service.ts
│   │   │   │   └── booking.service.ts  # Conflict logic
│   │   │   ├── utils/
│   │   │   │   ├── asyncHandler.ts
│   │   │   │   ├── ApiError.ts
│   │   │   │   ├── ApiResponse.ts
│   │   │   │   └── logger.ts
│   │   │   └── app.ts
│   │   ├── index.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   └── frontend/                       # Next.js 14 App Router
│       ├── app/
│       │   ├── (auth)/
│       │   │   ├── login/page.tsx
│       │   │   └── register/page.tsx
│       │   ├── (dashboard)/
│       │   │   ├── layout.tsx
│       │   │   ├── dashboard/page.tsx
│       │   │   ├── slots/page.tsx
│       │   │   ├── bookings/page.tsx
│       │   │   └── booking/[id]/confirm/page.tsx
│       │   ├── admin/
│       │   │   ├── layout.tsx
│       │   │   ├── page.tsx
│       │   │   ├── slots/page.tsx
│       │   │   └── bookings/page.tsx
│       │   ├── layout.tsx
│       │   └── page.tsx               # Landing
│       ├── components/
│       │   ├── ui/
│       │   │   ├── Button.tsx
│       │   │   ├── Input.tsx
│       │   │   ├── Modal.tsx
│       │   │   ├── Toast.tsx
│       │   │   ├── Skeleton.tsx
│       │   │   └── Badge.tsx
│       │   ├── slots/
│       │   │   ├── SlotCard.tsx
│       │   │   ├── SlotGrid.tsx
│       │   │   └── WeekCalendar.tsx
│       │   ├── bookings/
│       │   │   ├── BookingModal.tsx
│       │   │   └── BookingCard.tsx
│       │   ├── admin/
│       │   │   ├── StatsCard.tsx
│       │   │   ├── BookingsTable.tsx
│       │   │   └── SlotManager.tsx
│       │   └── layout/
│       │       ├── Sidebar.tsx
│       │       ├── Navbar.tsx
│       │       └── AdminSidebar.tsx
│       ├── hooks/
│       │   ├── useSlots.ts
│       │   ├── useBookings.ts
│       │   └── useAuth.ts
│       ├── lib/
│       │   ├── api.ts                 # Axios instance
│       │   ├── queryClient.ts
│       │   └── utils.ts
│       ├── store/
│       │   └── authStore.ts           # Zustand
│       ├── types/
│       │   └── index.ts
│       ├── middleware.ts              # Next.js route protection
│       ├── tailwind.config.ts
│       ├── tsconfig.json
│       └── package.json
└── README.md
```

---

## 2. Environment Variables

### Backend `.env`

```env
# Server
PORT=5000
NODE_ENV=production

# MongoDB
MONGO_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/slot-booking

# JWT
JWT_SECRET=your_super_secret_jwt_key_min_32_chars
JWT_EXPIRES_IN=15m
JWT_REFRESH_SECRET=your_refresh_secret_key
JWT_REFRESH_EXPIRES_IN=7d

# Redis (optional)
REDIS_URL=redis://localhost:6379

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=100

# CORS
ALLOWED_ORIGINS=https://yourdomain.com,http://localhost:3000
```

### Frontend `.env.local`

```env
NEXT_PUBLIC_API_URL=http://localhost:5000/api/v1
NEXT_PUBLIC_APP_NAME=SlotBook
```

---

## 3. Database Schema (Mongoose)

### `src/models/User.model.ts`

```typescript
import mongoose, { Document, Schema } from "mongoose";
import bcrypt from "bcryptjs";

export type UserRole = "USER" | "ADMIN";

export interface IUser extends Document {
  name: string;
  email: string;
  password: string;
  role: UserRole;
  refreshToken?: string;
  createdAt: Date;
  comparePassword(candidate: string): Promise<boolean>;
}

const UserSchema = new Schema<IUser>(
  {
    name: {
      type: String,
      required: [true, "Name is required"],
      trim: true,
      maxlength: [60, "Name cannot exceed 60 characters"],
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, "Invalid email format"],
    },
    password: {
      type: String,
      required: [true, "Password is required"],
      minlength: [8, "Password must be at least 8 characters"],
      select: false, // never returned in queries by default
    },
    role: {
      type: String,
      enum: ["USER", "ADMIN"],
      default: "USER",
    },
    refreshToken: {
      type: String,
      select: false,
    },
  },
  { timestamps: true }
);

// Index for fast email lookup
UserSchema.index({ email: 1 });

// Hash password before save
UserSchema.pre("save", async function (next) {
  if (!this.isModified("password")) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

UserSchema.methods.comparePassword = async function (
  candidate: string
): Promise<boolean> {
  return bcrypt.compare(candidate, this.password);
};

export const User = mongoose.model<IUser>("User", UserSchema);
```

### `src/models/Slot.model.ts`

```typescript
import mongoose, { Document, Schema } from "mongoose";

export interface ISlot extends Document {
  date: string;        // "YYYY-MM-DD"
  startTime: string;   // "HH:MM" 24h
  endTime: string;
  isBooked: boolean;
  isDisabled: boolean; // admin can disable slots
  bookedBy?: mongoose.Types.ObjectId;
  createdAt: Date;
}

const SlotSchema = new Schema<ISlot>(
  {
    date: {
      type: String,
      required: true,
      match: [/^\d{4}-\d{2}-\d{2}$/, "Date must be YYYY-MM-DD"],
    },
    startTime: {
      type: String,
      required: true,
      match: [/^\d{2}:\d{2}$/, "Time must be HH:MM"],
    },
    endTime: {
      type: String,
      required: true,
    },
    isBooked: { type: Boolean, default: false },
    isDisabled: { type: Boolean, default: false },
    bookedBy: { type: Schema.Types.ObjectId, ref: "User" },
  },
  { timestamps: true }
);

// Compound unique: no two identical (date + startTime) slots
SlotSchema.index({ date: 1, startTime: 1 }, { unique: true });
// For fast availability queries
SlotSchema.index({ date: 1, isBooked: 1, isDisabled: 1 });

export const Slot = mongoose.model<ISlot>("Slot", SlotSchema);
```

### `src/models/Booking.model.ts`

```typescript
import mongoose, { Document, Schema } from "mongoose";

export type BookingStatus = "CONFIRMED" | "CANCELLED" | "RESCHEDULED";

export interface IBooking extends Document {
  userId: mongoose.Types.ObjectId;
  slotId: mongoose.Types.ObjectId;
  status: BookingStatus;
  previousSlotId?: mongoose.Types.ObjectId; // for reschedule trail
  cancelledAt?: Date;
  rescheduledAt?: Date;
  createdAt: Date;
}

const BookingSchema = new Schema<IBooking>(
  {
    userId: {
      type: Schema.Types.ObjectId,
      ref: "User",
      required: true,
    },
    slotId: {
      type: Schema.Types.ObjectId,
      ref: "Slot",
      required: true,
    },
    status: {
      type: String,
      enum: ["CONFIRMED", "CANCELLED", "RESCHEDULED"],
      default: "CONFIRMED",
    },
    previousSlotId: { type: Schema.Types.ObjectId, ref: "Slot" },
    cancelledAt: Date,
    rescheduledAt: Date,
  },
  { timestamps: true }
);

// Fast per-user booking lookup
BookingSchema.index({ userId: 1, status: 1 });
// Fast per-slot check
BookingSchema.index({ slotId: 1, status: 1 });

export const Booking = mongoose.model<IBooking>("Booking", BookingSchema);
```

---

## 4. Backend — Auth APIs

### `src/validators/auth.schema.ts`

```typescript
import { z } from "zod";

export const RegisterSchema = z.object({
  name: z.string().min(2).max(60).trim(),
  email: z.string().email().toLowerCase().trim(),
  password: z
    .string()
    .min(8)
    .regex(/[A-Z]/, "Must contain uppercase")
    .regex(/[0-9]/, "Must contain number"),
  role: z.enum(["USER", "ADMIN"]).optional().default("USER"),
});

export const LoginSchema = z.object({
  email: z.string().email().toLowerCase().trim(),
  password: z.string().min(1, "Password required"),
});

export const RefreshSchema = z.object({
  refreshToken: z.string().min(1),
});
```

### `src/services/token.service.ts`

```typescript
import jwt from "jsonwebtoken";

const {
  JWT_SECRET,
  JWT_EXPIRES_IN,
  JWT_REFRESH_SECRET,
  JWT_REFRESH_EXPIRES_IN,
} = process.env;

export interface TokenPayload {
  userId: string;
  role: string;
}

export const generateAccessToken = (payload: TokenPayload): string =>
  jwt.sign(payload, JWT_SECRET!, { expiresIn: JWT_EXPIRES_IN as any });

export const generateRefreshToken = (payload: TokenPayload): string =>
  jwt.sign(payload, JWT_REFRESH_SECRET!, {
    expiresIn: JWT_REFRESH_EXPIRES_IN as any,
  });

export const verifyAccessToken = (token: string): TokenPayload =>
  jwt.verify(token, JWT_SECRET!) as TokenPayload;

export const verifyRefreshToken = (token: string): TokenPayload =>
  jwt.verify(token, JWT_REFRESH_SECRET!) as TokenPayload;
```

### `src/controllers/auth.controller.ts`

```typescript
import { Request, Response } from "express";
import { User } from "../models/User.model";
import {
  generateAccessToken,
  generateRefreshToken,
  verifyRefreshToken,
} from "../services/token.service";
import { asyncHandler } from "../utils/asyncHandler";
import { ApiError } from "../utils/ApiError";
import { ApiResponse } from "../utils/ApiResponse";

// POST /api/v1/auth/register
export const register = asyncHandler(async (req: Request, res: Response) => {
  const { name, email, password, role } = req.body;

  const existing = await User.findOne({ email });
  if (existing) throw new ApiError(409, "Email already registered");

  const user = await User.create({ name, email, password, role });

  const accessToken = generateAccessToken({ userId: user.id, role: user.role });
  const refreshToken = generateRefreshToken({
    userId: user.id,
    role: user.role,
  });

  // Store hashed refresh token (or raw — hashing adds security)
  user.refreshToken = refreshToken;
  await user.save({ validateBeforeSave: false });

  res.status(201).json(
    new ApiResponse(201, {
      user: { id: user.id, name: user.name, email: user.email, role: user.role },
      accessToken,
      refreshToken,
    }, "Registered successfully")
  );
});

// POST /api/v1/auth/login
export const login = asyncHandler(async (req: Request, res: Response) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email }).select("+password +refreshToken");
  if (!user || !(await user.comparePassword(password))) {
    throw new ApiError(401, "Invalid credentials");
  }

  const accessToken = generateAccessToken({ userId: user.id, role: user.role });
  const refreshToken = generateRefreshToken({
    userId: user.id,
    role: user.role,
  });

  user.refreshToken = refreshToken;
  await user.save({ validateBeforeSave: false });

  res.json(
    new ApiResponse(200, {
      user: { id: user.id, name: user.name, email: user.email, role: user.role },
      accessToken,
      refreshToken,
    }, "Logged in")
  );
});

// POST /api/v1/auth/refresh
export const refreshToken = asyncHandler(
  async (req: Request, res: Response) => {
    const { refreshToken: token } = req.body;

    let payload;
    try {
      payload = verifyRefreshToken(token);
    } catch {
      throw new ApiError(401, "Invalid or expired refresh token");
    }

    const user = await User.findById(payload.userId).select("+refreshToken");
    if (!user || user.refreshToken !== token) {
      throw new ApiError(401, "Refresh token mismatch — please log in again");
    }

    const newAccess = generateAccessToken({
      userId: user.id,
      role: user.role,
    });
    const newRefresh = generateRefreshToken({
      userId: user.id,
      role: user.role,
    });

    user.refreshToken = newRefresh;
    await user.save({ validateBeforeSave: false });

    res.json(new ApiResponse(200, { accessToken: newAccess, refreshToken: newRefresh }, "Tokens refreshed"));
  }
);

// POST /api/v1/auth/logout
export const logout = asyncHandler(async (req: Request, res: Response) => {
  const user = await User.findById((req as any).user.userId).select("+refreshToken");
  if (user) {
    user.refreshToken = undefined;
    await user.save({ validateBeforeSave: false });
  }
  res.json(new ApiResponse(200, null, "Logged out"));
});
```

### `src/routes/auth.routes.ts`

```typescript
import { Router } from "express";
import { register, login, refreshToken, logout } from "../controllers/auth.controller";
import { validate } from "../middleware/validate.middleware";
import { RegisterSchema, LoginSchema, RefreshSchema } from "../validators/auth.schema";
import { authMiddleware } from "../middleware/auth.middleware";
import { authLimiter } from "../middleware/rateLimiter";

const router = Router();

router.post("/register", authLimiter, validate(RegisterSchema), register);
router.post("/login", authLimiter, validate(LoginSchema), login);
router.post("/refresh", validate(RefreshSchema), refreshToken);
router.post("/logout", authMiddleware, logout);

export default router;
```

---

## 5. Backend — Slot APIs

### `src/validators/slot.schema.ts`

```typescript
import { z } from "zod";

export const CreateSlotSchema = z.object({
  date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/, "Date must be YYYY-MM-DD"),
  startTime: z.string().regex(/^\d{2}:\d{2}$/, "Time must be HH:MM"),
  endTime: z.string().regex(/^\d{2}:\d{2}$/, "Time must be HH:MM"),
});

export const BulkCreateSlotSchema = z.object({
  slots: z.array(CreateSlotSchema).min(1).max(50),
});

export const GetSlotsQuerySchema = z.object({
  date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional(),
  week: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional(), // ISO week start
});
```

### `src/controllers/slot.controller.ts`

```typescript
import { Request, Response } from "express";
import { Slot } from "../models/Slot.model";
import { asyncHandler } from "../utils/asyncHandler";
import { ApiError } from "../utils/ApiError";
import { ApiResponse } from "../utils/ApiResponse";
import { addDays, format } from "date-fns";

// GET /api/v1/slots?date=YYYY-MM-DD
export const getSlots = asyncHandler(async (req: Request, res: Response) => {
  const { date, week } = req.query as { date?: string; week?: string };

  let filter: Record<string, any> = {};

  if (date) {
    filter.date = date;
  } else if (week) {
    // Return 7 days from the given week start
    const start = new Date(week);
    const dates = Array.from({ length: 7 }, (_, i) =>
      format(addDays(start, i), "yyyy-MM-dd")
    );
    filter.date = { $in: dates };
  } else {
    // Default: today
    filter.date = format(new Date(), "yyyy-MM-dd");
  }

  const slots = await Slot.find(filter).sort({ date: 1, startTime: 1 }).lean();

  res.json(new ApiResponse(200, slots, "Slots fetched"));
});

// POST /api/v1/admin/slots  (admin only — in admin controller)
// GET /api/v1/slots/:id
export const getSlotById = asyncHandler(async (req: Request, res: Response) => {
  const slot = await Slot.findById(req.params.id).lean();
  if (!slot) throw new ApiError(404, "Slot not found");
  res.json(new ApiResponse(200, slot, "Slot fetched"));
});
```

### `src/routes/slot.routes.ts`

```typescript
import { Router } from "express";
import { getSlots, getSlotById } from "../controllers/slot.controller";
import { authMiddleware } from "../middleware/auth.middleware";
import { validate } from "../middleware/validate.middleware";
import { GetSlotsQuerySchema } from "../validators/slot.schema";

const router = Router();

router.use(authMiddleware);
router.get("/", validate(GetSlotsQuerySchema, "query"), getSlots);
router.get("/:id", getSlotById);

export default router;
```

---

## 6. Backend — Booking APIs

### `src/services/booking.service.ts`

```typescript
import mongoose from "mongoose";
import { Slot } from "../models/Slot.model";
import { Booking } from "../models/Booking.model";
import { ApiError } from "../utils/ApiError";

/**
 * Atomically book a slot.
 * Uses findOneAndUpdate with a condition to prevent race conditions
 * without requiring Redis — safe for moderate concurrency.
 * For very high concurrency, wrap in a Mongo transaction.
 */
export const bookSlotAtomically = async (
  slotId: string,
  userId: string
): Promise<InstanceType<typeof Booking>> => {
  const session = await mongoose.startSession();
  session.startTransaction();

  try {
    // Atomically mark slot as booked only if currently available
    const slot = await Slot.findOneAndUpdate(
      { _id: slotId, isBooked: false, isDisabled: false },
      { $set: { isBooked: true, bookedBy: userId } },
      { new: true, session }
    );

    if (!slot) {
      throw new ApiError(409, "This slot is no longer available");
    }

    // Check user doesn't already have an active booking for this slot
    const existing = await Booking.findOne({
      userId,
      slotId,
      status: "CONFIRMED",
    }).session(session);

    if (existing) {
      throw new ApiError(409, "You already have an active booking for this slot");
    }

    const booking = await Booking.create([{ userId, slotId, status: "CONFIRMED" }], {
      session,
    });

    await session.commitTransaction();
    return booking[0];
  } catch (err) {
    await session.abortTransaction();
    throw err;
  } finally {
    session.endSession();
  }
};

/**
 * Cancel a booking — releases the slot.
 */
export const cancelBookingService = async (
  bookingId: string,
  userId: string,
  isAdmin: boolean
): Promise<void> => {
  const booking = await Booking.findById(bookingId);
  if (!booking) throw new ApiError(404, "Booking not found");
  if (!isAdmin && booking.userId.toString() !== userId)
    throw new ApiError(403, "Not authorized");
  if (booking.status === "CANCELLED")
    throw new ApiError(400, "Booking already cancelled");

  const session = await mongoose.startSession();
  session.startTransaction();
  try {
    await Booking.findByIdAndUpdate(
      bookingId,
      { status: "CANCELLED", cancelledAt: new Date() },
      { session }
    );
    await Slot.findByIdAndUpdate(
      booking.slotId,
      { $set: { isBooked: false, bookedBy: null } },
      { session }
    );
    await session.commitTransaction();
  } catch (err) {
    await session.abortTransaction();
    throw err;
  } finally {
    session.endSession();
  }
};

/**
 * Reschedule: cancel old slot, book new one atomically.
 */
export const rescheduleBookingService = async (
  bookingId: string,
  newSlotId: string,
  userId: string
): Promise<InstanceType<typeof Booking>> => {
  const booking = await Booking.findById(bookingId);
  if (!booking) throw new ApiError(404, "Booking not found");
  if (booking.userId.toString() !== userId) throw new ApiError(403, "Not authorized");
  if (booking.status !== "CONFIRMED") throw new ApiError(400, "Only confirmed bookings can be rescheduled");
  if (booking.slotId.toString() === newSlotId) throw new ApiError(400, "New slot must be different");

  const session = await mongoose.startSession();
  session.startTransaction();
  try {
    // Release old slot
    await Slot.findByIdAndUpdate(
      booking.slotId,
      { $set: { isBooked: false, bookedBy: null } },
      { session }
    );

    // Claim new slot atomically
    const newSlot = await Slot.findOneAndUpdate(
      { _id: newSlotId, isBooked: false, isDisabled: false },
      { $set: { isBooked: true, bookedBy: userId } },
      { new: true, session }
    );
    if (!newSlot) throw new ApiError(409, "New slot is no longer available");

    // Update booking record
    const updated = await Booking.findByIdAndUpdate(
      bookingId,
      {
        slotId: newSlotId,
        status: "RESCHEDULED",
        previousSlotId: booking.slotId,
        rescheduledAt: new Date(),
      },
      { new: true, session }
    );

    await session.commitTransaction();
    return updated!;
  } catch (err) {
    await session.abortTransaction();
    throw err;
  } finally {
    session.endSession();
  }
};
```

### `src/validators/booking.schema.ts`

```typescript
import { z } from "zod";

export const CreateBookingSchema = z.object({
  slotId: z.string().length(24, "Invalid slot ID"),
});

export const RescheduleBookingSchema = z.object({
  newSlotId: z.string().length(24, "Invalid slot ID"),
});
```

### `src/controllers/booking.controller.ts`

```typescript
import { Request, Response } from "express";
import { Booking } from "../models/Booking.model";
import {
  bookSlotAtomically,
  cancelBookingService,
  rescheduleBookingService,
} from "../services/booking.service";
import { asyncHandler } from "../utils/asyncHandler";
import { ApiError } from "../utils/ApiError";
import { ApiResponse } from "../utils/ApiResponse";

// POST /api/v1/bookings
export const createBooking = asyncHandler(async (req: Request, res: Response) => {
  const { slotId } = req.body;
  const userId = (req as any).user.userId;

  const booking = await bookSlotAtomically(slotId, userId);

  res.status(201).json(
    new ApiResponse(201, booking, "Slot booked successfully")
  );
});

// GET /api/v1/bookings  (user's own bookings)
export const getMyBookings = asyncHandler(async (req: Request, res: Response) => {
  const userId = (req as any).user.userId;
  const page = Math.max(1, parseInt(req.query.page as string) || 1);
  const limit = Math.min(50, parseInt(req.query.limit as string) || 10);
  const skip = (page - 1) * limit;

  const [bookings, total] = await Promise.all([
    Booking.find({ userId })
      .populate("slotId", "date startTime endTime")
      .sort({ createdAt: -1 })
      .skip(skip)
      .limit(limit)
      .lean(),
    Booking.countDocuments({ userId }),
  ]);

  res.json(
    new ApiResponse(200, {
      bookings,
      pagination: { page, limit, total, pages: Math.ceil(total / limit) },
    }, "Bookings fetched")
  );
});

// GET /api/v1/bookings/:id
export const getBookingById = asyncHandler(async (req: Request, res: Response) => {
  const userId = (req as any).user.userId;
  const booking = await Booking.findById(req.params.id)
    .populate("slotId", "date startTime endTime")
    .lean();

  if (!booking) throw new ApiError(404, "Booking not found");
  if (
    booking.userId.toString() !== userId &&
    (req as any).user.role !== "ADMIN"
  ) {
    throw new ApiError(403, "Not authorized");
  }

  res.json(new ApiResponse(200, booking, "Booking fetched"));
});

// PATCH /api/v1/bookings/:id/cancel
export const cancelBooking = asyncHandler(async (req: Request, res: Response) => {
  const { userId, role } = (req as any).user;
  await cancelBookingService(req.params.id, userId, role === "ADMIN");
  res.json(new ApiResponse(200, null, "Booking cancelled"));
});

// PATCH /api/v1/bookings/:id/reschedule
export const rescheduleBooking = asyncHandler(
  async (req: Request, res: Response) => {
    const { newSlotId } = req.body;
    const userId = (req as any).user.userId;
    const booking = await rescheduleBookingService(
      req.params.id,
      newSlotId,
      userId
    );
    res.json(new ApiResponse(200, booking, "Booking rescheduled"));
  }
);
```

### `src/routes/booking.routes.ts`

```typescript
import { Router } from "express";
import {
  createBooking,
  getMyBookings,
  getBookingById,
  cancelBooking,
  rescheduleBooking,
} from "../controllers/booking.controller";
import { authMiddleware } from "../middleware/auth.middleware";
import { validate } from "../middleware/validate.middleware";
import {
  CreateBookingSchema,
  RescheduleBookingSchema,
} from "../validators/booking.schema";

const router = Router();
router.use(authMiddleware);

router.post("/", validate(CreateBookingSchema), createBooking);
router.get("/", getMyBookings);
router.get("/:id", getBookingById);
router.patch("/:id/cancel", cancelBooking);
router.patch("/:id/reschedule", validate(RescheduleBookingSchema), rescheduleBooking);

export default router;
```

---

## 7. Backend — Admin APIs

### `src/controllers/admin.controller.ts`

```typescript
import { Request, Response } from "express";
import { Slot } from "../models/Slot.model";
import { Booking } from "../models/Booking.model";
import { User } from "../models/User.model";
import { asyncHandler } from "../utils/asyncHandler";
import { ApiError } from "../utils/ApiError";
import { ApiResponse } from "../utils/ApiResponse";

// POST /api/v1/admin/slots — create one or bulk
export const createSlots = asyncHandler(async (req: Request, res: Response) => {
  const { slots } = req.body; // array of { date, startTime, endTime }
  const created = await Slot.insertMany(slots, { ordered: false }).catch((err) => {
    // Handle duplicate key errors gracefully
    if (err.code === 11000)
      throw new ApiError(409, "One or more slots already exist for that time");
    throw err;
  });
  res.status(201).json(new ApiResponse(201, created, `${created.length} slot(s) created`));
});

// PATCH /api/v1/admin/slots/:id/disable
export const disableSlot = asyncHandler(async (req: Request, res: Response) => {
  const slot = await Slot.findByIdAndUpdate(
    req.params.id,
    { isDisabled: true },
    { new: true }
  );
  if (!slot) throw new ApiError(404, "Slot not found");
  res.json(new ApiResponse(200, slot, "Slot disabled"));
});

// PATCH /api/v1/admin/slots/:id/enable
export const enableSlot = asyncHandler(async (req: Request, res: Response) => {
  const slot = await Slot.findByIdAndUpdate(
    req.params.id,
    { isDisabled: false },
    { new: true }
  );
  if (!slot) throw new ApiError(404, "Slot not found");
  res.json(new ApiResponse(200, slot, "Slot enabled"));
});

// GET /api/v1/admin/bookings — all bookings with pagination
export const getAllBookings = asyncHandler(async (req: Request, res: Response) => {
  const page = Math.max(1, parseInt(req.query.page as string) || 1);
  const limit = Math.min(100, parseInt(req.query.limit as string) || 20);
  const skip = (page - 1) * limit;
  const status = req.query.status;

  const filter: Record<string, any> = {};
  if (status) filter.status = status;

  const [bookings, total] = await Promise.all([
    Booking.find(filter)
      .populate("userId", "name email")
      .populate("slotId", "date startTime endTime")
      .sort({ createdAt: -1 })
      .skip(skip)
      .limit(limit)
      .lean(),
    Booking.countDocuments(filter),
  ]);

  res.json(
    new ApiResponse(200, {
      bookings,
      pagination: { page, limit, total, pages: Math.ceil(total / limit) },
    }, "All bookings fetched")
  );
});

// GET /api/v1/admin/analytics
export const getAnalytics = asyncHandler(async (req: Request, res: Response) => {
  const [bookingStats, totalUsers, totalSlots] = await Promise.all([
    Booking.aggregate([
      {
        $group: {
          _id: "$status",
          count: { $sum: 1 },
        },
      },
    ]),
    User.countDocuments({ role: "USER" }),
    Slot.countDocuments(),
  ]);

  const stats = bookingStats.reduce(
    (acc: Record<string, number>, cur) => {
      acc[cur._id.toLowerCase()] = cur.count;
      return acc;
    },
    {}
  );

  res.json(
    new ApiResponse(200, {
      totalUsers,
      totalSlots,
      totalConfirmed: stats.confirmed ?? 0,
      totalCancelled: stats.cancelled ?? 0,
      totalRescheduled: stats.rescheduled ?? 0,
    }, "Analytics fetched")
  );
});

// PATCH /api/v1/admin/bookings/:id/cancel — admin cancel any booking
export const adminCancelBooking = asyncHandler(
  async (req: Request, res: Response) => {
    const { cancelBookingService } = await import("../services/booking.service");
    await cancelBookingService(req.params.id, "", true);
    res.json(new ApiResponse(200, null, "Booking cancelled by admin"));
  }
);
```

### `src/routes/admin.routes.ts`

```typescript
import { Router } from "express";
import {
  createSlots,
  disableSlot,
  enableSlot,
  getAllBookings,
  getAnalytics,
  adminCancelBooking,
} from "../controllers/admin.controller";
import { authMiddleware } from "../middleware/auth.middleware";
import { rbacMiddleware } from "../middleware/rbac.middleware";
import { validate } from "../middleware/validate.middleware";
import { BulkCreateSlotSchema } from "../validators/slot.schema";

const router = Router();
router.use(authMiddleware, rbacMiddleware("ADMIN"));

router.post("/slots", validate(BulkCreateSlotSchema), createSlots);
router.patch("/slots/:id/disable", disableSlot);
router.patch("/slots/:id/enable", enableSlot);
router.get("/bookings", getAllBookings);
router.patch("/bookings/:id/cancel", adminCancelBooking);
router.get("/analytics", getAnalytics);

export default router;
```

---

## 8. Middleware (Auth + RBAC + Rate Limit)

### `src/middleware/auth.middleware.ts`

```typescript
import { Request, Response, NextFunction } from "express";
import { verifyAccessToken } from "../services/token.service";
import { ApiError } from "../utils/ApiError";

export const authMiddleware = (
  req: Request,
  _res: Response,
  next: NextFunction
) => {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith("Bearer ")) {
    return next(new ApiError(401, "No token provided"));
  }

  const token = authHeader.split(" ")[1];
  try {
    const payload = verifyAccessToken(token);
    (req as any).user = payload;
    next();
  } catch (err: any) {
    if (err.name === "TokenExpiredError") {
      return next(new ApiError(401, "Token expired"));
    }
    return next(new ApiError(401, "Invalid token"));
  }
};
```

### `src/middleware/rbac.middleware.ts`

```typescript
import { Request, Response, NextFunction } from "express";
import { ApiError } from "../utils/ApiError";

export const rbacMiddleware =
  (...allowedRoles: string[]) =>
  (req: Request, _res: Response, next: NextFunction) => {
    const user = (req as any).user;
    if (!user || !allowedRoles.includes(user.role)) {
      return next(new ApiError(403, "Access denied"));
    }
    next();
  };
```

### `src/middleware/validate.middleware.ts`

```typescript
import { Request, Response, NextFunction } from "express";
import { ZodSchema, ZodError } from "zod";
import { ApiError } from "../utils/ApiError";

export const validate =
  (schema: ZodSchema, target: "body" | "query" | "params" = "body") =>
  (req: Request, _res: Response, next: NextFunction) => {
    try {
      const parsed = schema.parse(req[target]);
      req[target] = parsed as any;
      next();
    } catch (err) {
      if (err instanceof ZodError) {
        const message = err.errors
          .map((e) => `${e.path.join(".")}: ${e.message}`)
          .join(", ");
        return next(new ApiError(422, message));
      }
      next(err);
    }
  };
```

### `src/middleware/rateLimiter.ts`

```typescript
import rateLimit from "express-rate-limit";

export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10, // 10 attempts per window
  message: { success: false, message: "Too many attempts. Try again later." },
  standardHeaders: true,
  legacyHeaders: false,
});

export const globalLimiter = rateLimit({
  windowMs: parseInt(process.env.RATE_LIMIT_WINDOW_MS || "900000"),
  max: parseInt(process.env.RATE_LIMIT_MAX || "100"),
  standardHeaders: true,
  legacyHeaders: false,
});
```

### `src/utils/asyncHandler.ts`

```typescript
import { Request, Response, NextFunction } from "express";

type AsyncFn = (req: Request, res: Response, next: NextFunction) => Promise<any>;

export const asyncHandler = (fn: AsyncFn) =>
  (req: Request, res: Response, next: NextFunction) =>
    Promise.resolve(fn(req, res, next)).catch(next);
```

### `src/utils/ApiError.ts`

```typescript
export class ApiError extends Error {
  statusCode: number;
  constructor(statusCode: number, message: string) {
    super(message);
    this.statusCode = statusCode;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

### `src/utils/ApiResponse.ts`

```typescript
export class ApiResponse<T> {
  success: boolean;
  statusCode: number;
  data: T | null;
  message: string;

  constructor(statusCode: number, data: T | null, message = "Success") {
    this.success = statusCode < 400;
    this.statusCode = statusCode;
    this.data = data;
    this.message = message;
  }
}
```

### `src/app.ts` (Express App entry)

```typescript
import express, { Request, Response, NextFunction } from "express";
import cors from "cors";
import helmet from "helmet";
import morgan from "morgan";
import { globalLimiter } from "./middleware/rateLimiter";
import { ApiError } from "./utils/ApiError";

import authRoutes from "./routes/auth.routes";
import slotRoutes from "./routes/slot.routes";
import bookingRoutes from "./routes/booking.routes";
import adminRoutes from "./routes/admin.routes";

const app = express();

app.use(helmet());
app.use(cors({ origin: process.env.ALLOWED_ORIGINS?.split(",") }));
app.use(express.json({ limit: "10kb" }));
app.use(morgan("combined"));
app.use(globalLimiter);

// Routes
app.use("/api/v1/auth", authRoutes);
app.use("/api/v1/slots", slotRoutes);
app.use("/api/v1/bookings", bookingRoutes);
app.use("/api/v1/admin", adminRoutes);

// 404 handler
app.use((_req, res) => res.status(404).json({ success: false, message: "Route not found" }));

// Global error handler
app.use((err: any, _req: Request, res: Response, _next: NextFunction) => {
  const statusCode = err instanceof ApiError ? err.statusCode : 500;
  const message =
    err instanceof ApiError
      ? err.message
      : process.env.NODE_ENV === "production"
      ? "Internal server error"
      : err.message;

  res.status(statusCode).json({ success: false, message });
});

export default app;
```

---

## 9. Frontend — Next.js Setup

### `types/index.ts`

```typescript
export type Role = "USER" | "ADMIN";
export type BookingStatus = "CONFIRMED" | "CANCELLED" | "RESCHEDULED";
export type SlotStatus = "available" | "booked" | "disabled";

export interface User {
  id: string;
  name: string;
  email: string;
  role: Role;
}

export interface Slot {
  _id: string;
  date: string;        // "YYYY-MM-DD"
  startTime: string;   // "HH:MM"
  endTime: string;
  isBooked: boolean;
  isDisabled: boolean;
}

export interface Booking {
  _id: string;
  userId: string | User;
  slotId: string | Slot;
  status: BookingStatus;
  createdAt: string;
  cancelledAt?: string;
  rescheduledAt?: string;
}

export interface Pagination {
  page: number;
  limit: number;
  total: number;
  pages: number;
}

export interface ApiResponse<T> {
  success: boolean;
  data: T;
  message: string;
}
```

### `store/authStore.ts`

```typescript
import { create } from "zustand";
import { persist } from "zustand/middleware";
import { User } from "@/types";

interface AuthState {
  user: User | null;
  accessToken: string | null;
  refreshToken: string | null;
  setAuth: (user: User, accessToken: string, refreshToken: string) => void;
  updateTokens: (accessToken: string, refreshToken: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      accessToken: null,
      refreshToken: null,
      setAuth: (user, accessToken, refreshToken) =>
        set({ user, accessToken, refreshToken }),
      updateTokens: (accessToken, refreshToken) =>
        set({ accessToken, refreshToken }),
      logout: () => set({ user: null, accessToken: null, refreshToken: null }),
    }),
    { name: "auth-store" }
  )
);
```

### `lib/api.ts`

```typescript
import axios, { AxiosError, InternalAxiosRequestConfig } from "axios";
import { useAuthStore } from "@/store/authStore";

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  timeout: 10000,
});

// Attach access token
api.interceptors.request.use((config: InternalAxiosRequestConfig) => {
  const token = useAuthStore.getState().accessToken;
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

let isRefreshing = false;
let failedQueue: Array<{
  resolve: (token: string) => void;
  reject: (err: unknown) => void;
}> = [];

const processQueue = (err: unknown, token: string | null = null) => {
  failedQueue.forEach((p) => (err ? p.reject(err) : p.resolve(token!)));
  failedQueue = [];
};

// Auto refresh on 401
api.interceptors.response.use(
  (res) => res,
  async (error: AxiosError) => {
    const original = error.config as InternalAxiosRequestConfig & { _retry?: boolean };

    if (error.response?.status === 401 && !original._retry) {
      if (isRefreshing) {
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then((token) => {
          original.headers.Authorization = `Bearer ${token}`;
          return api(original);
        });
      }

      original._retry = true;
      isRefreshing = true;

      const { refreshToken, updateTokens, logout } = useAuthStore.getState();

      try {
        const { data } = await axios.post(
          `${process.env.NEXT_PUBLIC_API_URL}/auth/refresh`,
          { refreshToken }
        );
        const newAccess = data.data.accessToken;
        const newRefresh = data.data.refreshToken;
        updateTokens(newAccess, newRefresh);
        processQueue(null, newAccess);
        original.headers.Authorization = `Bearer ${newAccess}`;
        return api(original);
      } catch (err) {
        processQueue(err);
        logout();
        window.location.href = "/login";
        return Promise.reject(err);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);

export default api;
```

### `lib/queryClient.ts`

```typescript
import { QueryClient } from "@tanstack/react-query";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30 * 1000,         // 30s
      gcTime: 5 * 60 * 1000,        // 5min
      retry: 1,
      refetchOnWindowFocus: true,
    },
    mutations: {
      retry: 0,
    },
  },
});
```

---

## 10. Frontend — Auth Pages

### `app/(auth)/login/page.tsx`

```tsx
"use client";
import { useState } from "react";
import { useRouter } from "next/navigation";
import Link from "next/link";
import api from "@/lib/api";
import { useAuthStore } from "@/store/authStore";
import { Button } from "@/components/ui/Button";
import { Input } from "@/components/ui/Input";
import { useToast } from "@/components/ui/Toast";

export default function LoginPage() {
  const router = useRouter();
  const setAuth = useAuthStore((s) => s.setAuth);
  const { showToast } = useToast();
  const [form, setForm] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [loading, setLoading] = useState(false);

  const validate = () => {
    const e: Record<string, string> = {};
    if (!form.email) e.email = "Email is required";
    else if (!/\S+@\S+\.\S+/.test(form.email)) e.email = "Invalid email";
    if (!form.password) e.password = "Password is required";
    setErrors(e);
    return Object.keys(e).length === 0;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!validate()) return;
    setLoading(true);
    try {
      const { data } = await api.post("/auth/login", form);
      setAuth(data.data.user, data.data.accessToken, data.data.refreshToken);
      showToast("Welcome back!", "success");
      router.push(data.data.user.role === "ADMIN" ? "/admin" : "/dashboard");
    } catch (err: any) {
      const msg = err?.response?.data?.message ?? "Login failed";
      showToast(msg, "error");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="w-full max-w-md bg-white rounded-2xl shadow-lg p-8">
        <h1 className="text-2xl font-bold text-gray-900 mb-6">Sign In</h1>
        <form onSubmit={handleSubmit} className="space-y-4" noValidate>
          <Input
            label="Email"
            type="email"
            value={form.email}
            onChange={(e) => setForm((f) => ({ ...f, email: e.target.value }))}
            error={errors.email}
            autoComplete="email"
          />
          <Input
            label="Password"
            type="password"
            value={form.password}
            onChange={(e) => setForm((f) => ({ ...f, password: e.target.value }))}
            error={errors.password}
            autoComplete="current-password"
          />
          <Button type="submit" loading={loading} className="w-full">
            Sign In
          </Button>
        </form>
        <p className="mt-4 text-center text-sm text-gray-600">
          No account?{" "}
          <Link href="/register" className="text-blue-600 hover:underline">
            Register
          </Link>
        </p>
      </div>
    </div>
  );
}
```

---

## 11. Frontend — Dashboard

### `hooks/useSlots.ts`

```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import api from "@/lib/api";
import { Slot } from "@/types";

export const SLOTS_KEY = (date?: string, week?: string) =>
  ["slots", date, week].filter(Boolean);

export const useSlots = (params: { date?: string; week?: string }) =>
  useQuery<Slot[]>({
    queryKey: SLOTS_KEY(params.date, params.week),
    queryFn: async () => {
      const { data } = await api.get("/slots", { params });
      return data.data;
    },
    enabled: !!(params.date || params.week),
  });
```

### `hooks/useBookings.ts`

```typescript
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import api from "@/lib/api";
import { Booking } from "@/types";
import { useToast } from "@/components/ui/Toast";

export const BOOKINGS_KEY = ["bookings"];

export const useMyBookings = (page = 1) =>
  useQuery({
    queryKey: [...BOOKINGS_KEY, page],
    queryFn: async () => {
      const { data } = await api.get("/bookings", { params: { page } });
      return data.data as { bookings: Booking[]; pagination: any };
    },
  });

export const useCreateBooking = () => {
  const qc = useQueryClient();
  const { showToast } = useToast();

  return useMutation({
    mutationFn: (slotId: string) => api.post("/bookings", { slotId }),
    onSuccess: () => {
      qc.invalidateQueries({ queryKey: BOOKINGS_KEY });
      qc.invalidateQueries({ queryKey: ["slots"] });
      showToast("Slot booked successfully!", "success");
    },
    onError: (err: any) => {
      showToast(
        err?.response?.data?.message ?? "Booking failed",
        "error"
      );
    },
  });
};

export const useCancelBooking = () => {
  const qc = useQueryClient();
  const { showToast } = useToast();

  return useMutation({
    mutationFn: (id: string) => api.patch(`/bookings/${id}/cancel`),
    onMutate: async (id) => {
      // Optimistic update
      await qc.cancelQueries({ queryKey: BOOKINGS_KEY });
      const prev = qc.getQueryData(BOOKINGS_KEY);
      qc.setQueryData([...BOOKINGS_KEY, 1], (old: any) => ({
        ...old,
        bookings: old?.bookings?.map((b: Booking) =>
          b._id === id ? { ...b, status: "CANCELLED" } : b
        ),
      }));
      return { prev };
    },
    onError: (_err, _id, ctx) => {
      qc.setQueryData(BOOKINGS_KEY, ctx?.prev);
      showToast("Cancellation failed", "error");
    },
    onSettled: () => {
      qc.invalidateQueries({ queryKey: BOOKINGS_KEY });
      qc.invalidateQueries({ queryKey: ["slots"] });
      showToast("Booking cancelled", "success");
    },
  });
};

export const useRescheduleBooking = () => {
  const qc = useQueryClient();
  const { showToast } = useToast();

  return useMutation({
    mutationFn: ({ id, newSlotId }: { id: string; newSlotId: string }) =>
      api.patch(`/bookings/${id}/reschedule`, { newSlotId }),
    onSuccess: () => {
      qc.invalidateQueries({ queryKey: BOOKINGS_KEY });
      qc.invalidateQueries({ queryKey: ["slots"] });
      showToast("Booking rescheduled!", "success");
    },
    onError: (err: any) => {
      showToast(err?.response?.data?.message ?? "Reschedule failed", "error");
    },
  });
};
```

### `app/(dashboard)/dashboard/page.tsx`

```tsx
"use client";
import { useState } from "react";
import { useMyBookings } from "@/hooks/useBookings";
import { BookingCard } from "@/components/bookings/BookingCard";
import { Skeleton } from "@/components/ui/Skeleton";
import { useAuthStore } from "@/store/authStore";

export default function DashboardPage() {
  const user = useAuthStore((s) => s.user);
  const [page, setPage] = useState(1);
  const { data, isLoading, isError } = useMyBookings(page);

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold text-gray-900">
          Welcome back, {user?.name} 👋
        </h1>
        <p className="text-gray-500 mt-1">Manage your upcoming appointments</p>
      </div>

      <section>
        <h2 className="text-lg font-semibold mb-4">Your Bookings</h2>

        {isError && (
          <div className="p-4 bg-red-50 rounded-lg text-red-600">
            Failed to load bookings.{" "}
            <button
              className="underline"
              onClick={() => window.location.reload()}
            >
              Retry
            </button>
          </div>
        )}

        {isLoading && (
          <div className="grid gap-4">
            {[1, 2, 3].map((i) => <Skeleton key={i} className="h-24 w-full" />)}
          </div>
        )}

        {!isLoading && data?.bookings.length === 0 && (
          <div className="text-center py-12 text-gray-400">
            No bookings yet. Head to{" "}
            <a href="/slots" className="text-blue-600 underline">
              Slot Selection
            </a>{" "}
            to book your first appointment.
          </div>
        )}

        {!isLoading && (
          <div className="grid gap-4">
            {data?.bookings.map((b) => (
              <BookingCard key={b._id} booking={b} />
            ))}
          </div>
        )}

        {/* Pagination */}
        {data && data.pagination.pages > 1 && (
          <div className="flex gap-2 mt-4 justify-center">
            <button
              disabled={page === 1}
              onClick={() => setPage((p) => p - 1)}
              className="px-3 py-1 rounded border disabled:opacity-40"
            >
              ← Prev
            </button>
            <span className="px-3 py-1 text-sm text-gray-600">
              {page} / {data.pagination.pages}
            </span>
            <button
              disabled={page === data.pagination.pages}
              onClick={() => setPage((p) => p + 1)}
              className="px-3 py-1 rounded border disabled:opacity-40"
            >
              Next →
            </button>
          </div>
        )}
      </section>
    </div>
  );
}
```

---

## 12. Frontend — Slot Selection Page

### `components/slots/WeekCalendar.tsx`

```tsx
"use client";
import { addDays, format, startOfWeek, isSameDay } from "date-fns";

interface WeekCalendarProps {
  selectedDate: Date;
  onDateSelect: (date: Date) => void;
}

export function WeekCalendar({ selectedDate, onDateSelect }: WeekCalendarProps) {
  const weekStart = startOfWeek(selectedDate, { weekStartsOn: 1 });
  const days = Array.from({ length: 7 }, (_, i) => addDays(weekStart, i));

  return (
    <div className="grid grid-cols-7 gap-1 bg-white rounded-xl shadow p-3">
      {days.map((day) => {
        const isSelected = isSameDay(day, selectedDate);
        const isToday = isSameDay(day, new Date());
        const isPast = day < new Date(new Date().setHours(0, 0, 0, 0));

        return (
          <button
            key={day.toISOString()}
            onClick={() => !isPast && onDateSelect(day)}
            disabled={isPast}
            className={`
              flex flex-col items-center p-2 rounded-lg transition-all
              ${isPast ? "opacity-30 cursor-not-allowed" : "hover:bg-blue-50 cursor-pointer"}
              ${isSelected ? "bg-blue-600 text-white hover:bg-blue-600" : ""}
              ${isToday && !isSelected ? "ring-2 ring-blue-300" : ""}
            `}
          >
            <span className="text-xs font-medium uppercase tracking-wide">
              {format(day, "EEE")}
            </span>
            <span className="text-lg font-bold mt-1">{format(day, "d")}</span>
          </button>
        );
      })}
    </div>
  );
}
```

### `components/slots/SlotCard.tsx`

```tsx
"use client";
import { Slot } from "@/types";

interface SlotCardProps {
  slot: Slot;
  onSelect: (slot: Slot) => void;
  selected?: boolean;
}

export function SlotCard({ slot, onSelect, selected }: SlotCardProps) {
  const status = slot.isDisabled
    ? "disabled"
    : slot.isBooked
    ? "booked"
    : "available";

  const styles: Record<string, string> = {
    available: selected
      ? "border-blue-600 bg-blue-600 text-white"
      : "border-gray-200 hover:border-blue-400 bg-white cursor-pointer",
    booked: "border-red-200 bg-red-50 text-red-400 cursor-not-allowed",
    disabled: "border-gray-100 bg-gray-50 text-gray-300 cursor-not-allowed",
  };

  const labels: Record<string, string> = {
    available: "Available",
    booked: "Booked",
    disabled: "Unavailable",
  };

  return (
    <div
      role={status === "available" ? "button" : undefined}
      tabIndex={status === "available" ? 0 : undefined}
      onClick={() => status === "available" && onSelect(slot)}
      onKeyDown={(e) => e.key === "Enter" && status === "available" && onSelect(slot)}
      className={`
        border-2 rounded-xl p-4 transition-all duration-150 select-none
        ${styles[status]}
      `}
      aria-label={`${slot.startTime}–${slot.endTime}: ${labels[status]}`}
    >
      <p className="font-semibold text-sm">
        {slot.startTime} – {slot.endTime}
      </p>
      <span
        className={`text-xs mt-1 inline-block px-2 py-0.5 rounded-full font-medium
          ${status === "available" && !selected ? "bg-green-100 text-green-700" : ""}
          ${status === "booked" ? "bg-red-100 text-red-600" : ""}
          ${status === "disabled" ? "bg-gray-100 text-gray-400" : ""}
          ${selected ? "bg-white/20 text-white" : ""}
        `}
      >
        {labels[status]}
      </span>
    </div>
  );
}
```

### `app/(dashboard)/slots/page.tsx`

```tsx
"use client";
import { useState, useCallback } from "react";
import { format } from "date-fns";
import { WeekCalendar } from "@/components/slots/WeekCalendar";
import { SlotCard } from "@/components/slots/SlotCard";
import { BookingModal } from "@/components/bookings/BookingModal";
import { Skeleton } from "@/components/ui/Skeleton";
import { useSlots } from "@/hooks/useSlots";
import { Slot } from "@/types";

export default function SlotsPage() {
  const [selectedDate, setSelectedDate] = useState(new Date());
  const [selectedSlot, setSelectedSlot] = useState<Slot | null>(null);
  const [showModal, setShowModal] = useState(false);

  const dateStr = format(selectedDate, "yyyy-MM-dd");
  const { data: slots, isLoading, isError } = useSlots({ date: dateStr });

  const handleSlotSelect = useCallback((slot: Slot) => {
    setSelectedSlot(slot);
    setShowModal(true);
  }, []);

  return (
    <div className="space-y-6 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900">Book a Slot</h1>

      <WeekCalendar
        selectedDate={selectedDate}
        onDateSelect={setSelectedDate}
      />

      <div>
        <p className="text-sm text-gray-500 mb-3">
          Showing slots for{" "}
          <span className="font-semibold text-gray-800">
            {format(selectedDate, "EEEE, MMMM d")}
          </span>
        </p>

        {isError && (
          <p className="text-red-500 text-sm">
            Failed to load slots. Please try again.
          </p>
        )}

        {isLoading && (
          <div className="grid grid-cols-3 gap-3">
            {Array.from({ length: 9 }).map((_, i) => (
              <Skeleton key={i} className="h-20 w-full" />
            ))}
          </div>
        )}

        {!isLoading && slots?.length === 0 && (
          <div className="text-center py-12 text-gray-400">
            No slots available for this date.
          </div>
        )}

        {!isLoading && slots && slots.length > 0 && (
          <div className="grid grid-cols-2 sm:grid-cols-3 gap-3">
            {slots.map((slot) => (
              <SlotCard
                key={slot._id}
                slot={slot}
                selected={selectedSlot?._id === slot._id}
                onSelect={handleSlotSelect}
              />
            ))}
          </div>
        )}
      </div>

      {showModal && selectedSlot && (
        <BookingModal
          slot={selectedSlot}
          onClose={() => {
            setShowModal(false);
            setSelectedSlot(null);
          }}
        />
      )}
    </div>
  );
}
```

---

## 13. Frontend — Booking Confirmation

### `components/bookings/BookingModal.tsx`

```tsx
"use client";
import { Slot } from "@/types";
import { useCreateBooking } from "@/hooks/useBookings";
import { format } from "date-fns";
import { Button } from "@/components/ui/Button";

interface Props {
  slot: Slot;
  onClose: () => void;
}

export function BookingModal({ slot, onClose }: Props) {
  const { mutate: book, isPending } = useCreateBooking();

  const handleConfirm = () => {
    book(slot._id, { onSuccess: onClose });
  };

  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center bg-black/50"
      role="dialog"
      aria-modal="true"
      aria-labelledby="booking-modal-title"
    >
      <div className="bg-white rounded-2xl shadow-xl w-full max-w-md mx-4 p-6">
        <h2
          id="booking-modal-title"
          className="text-xl font-bold text-gray-900 mb-4"
        >
          Confirm Booking
        </h2>

        <div className="bg-blue-50 rounded-xl p-4 mb-6 space-y-2">
          <div className="flex justify-between text-sm">
            <span className="text-gray-500">Date</span>
            <span className="font-semibold">
              {format(new Date(slot.date), "MMMM d, yyyy")}
            </span>
          </div>
          <div className="flex justify-between text-sm">
            <span className="text-gray-500">Time</span>
            <span className="font-semibold">
              {slot.startTime} – {slot.endTime}
            </span>
          </div>
        </div>

        <div className="flex gap-3">
          <Button
            variant="outline"
            onClick={onClose}
            disabled={isPending}
            className="flex-1"
          >
            Cancel
          </Button>
          <Button
            onClick={handleConfirm}
            loading={isPending}
            className="flex-1"
          >
            Confirm
          </Button>
        </div>
      </div>
    </div>
  );
}
```

### `components/bookings/BookingCard.tsx`

```tsx
"use client";
import { Booking, Slot } from "@/types";
import { format } from "date-fns";
import { useCancelBooking } from "@/hooks/useBookings";
import { Badge } from "@/components/ui/Badge";
import { Button } from "@/components/ui/Button";

interface Props { booking: Booking }

const STATUS_COLOR: Record<string, string> = {
  CONFIRMED: "green",
  CANCELLED: "red",
  RESCHEDULED: "yellow",
};

export function BookingCard({ booking }: Props) {
  const slot = booking.slotId as Slot;
  const { mutate: cancel, isPending } = useCancelBooking();

  return (
    <div className="border rounded-xl p-4 bg-white flex items-center justify-between gap-4">
      <div>
        <p className="font-semibold text-gray-900">
          {format(new Date(slot.date), "MMMM d, yyyy")}
        </p>
        <p className="text-sm text-gray-500">
          {slot.startTime} – {slot.endTime}
        </p>
        <Badge color={STATUS_COLOR[booking.status]} className="mt-1">
          {booking.status}
        </Badge>
      </div>
      {booking.status === "CONFIRMED" && (
        <Button
          variant="outline"
          size="sm"
          loading={isPending}
          onClick={() => cancel(booking._id)}
          className="text-red-600 border-red-200 hover:bg-red-50"
        >
          Cancel
        </Button>
      )}
    </div>
  );
}
```

---

## 14. Frontend — Admin Panel

### `app/admin/page.tsx`

```tsx
"use client";
import { useQuery } from "@tanstack/react-query";
import api from "@/lib/api";
import { StatsCard } from "@/components/admin/StatsCard";
import { Skeleton } from "@/components/ui/Skeleton";

export default function AdminDashboard() {
  const { data, isLoading } = useQuery({
    queryKey: ["admin-analytics"],
    queryFn: async () => {
      const { data } = await api.get("/admin/analytics");
      return data.data;
    },
  });

  if (isLoading) return <Skeleton className="h-40 w-full" />;

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold text-gray-900">Admin Dashboard</h1>
      <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
        <StatsCard label="Total Users" value={data?.totalUsers ?? 0} icon="👤" />
        <StatsCard label="Total Slots" value={data?.totalSlots ?? 0} icon="📅" />
        <StatsCard label="Confirmed" value={data?.totalConfirmed ?? 0} icon="✅" color="green" />
        <StatsCard label="Cancelled" value={data?.totalCancelled ?? 0} icon="❌" color="red" />
      </div>
    </div>
  );
}
```

### `components/admin/BookingsTable.tsx`

```tsx
"use client";
import { useState } from "react";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import api from "@/lib/api";
import { Booking, Slot, User } from "@/types";
import { format } from "date-fns";
import { Badge } from "@/components/ui/Badge";
import { Button } from "@/components/ui/Button";

export function AdminBookingsTable() {
  const [page, setPage] = useState(1);
  const [statusFilter, setStatusFilter] = useState("");
  const qc = useQueryClient();

  const { data, isLoading } = useQuery({
    queryKey: ["admin-bookings", page, statusFilter],
    queryFn: async () => {
      const { data } = await api.get("/admin/bookings", {
        params: { page, limit: 20, ...(statusFilter && { status: statusFilter }) },
      });
      return data.data as { bookings: Booking[]; pagination: any };
    },
  });

  const { mutate: cancelBooking, isPending: cancelling } = useMutation({
    mutationFn: (id: string) => api.patch(`/admin/bookings/${id}/cancel`),
    onSuccess: () => qc.invalidateQueries({ queryKey: ["admin-bookings"] }),
  });

  return (
    <div>
      <div className="flex gap-2 mb-4 flex-wrap">
        {["", "CONFIRMED", "CANCELLED", "RESCHEDULED"].map((s) => (
          <button
            key={s}
            onClick={() => { setStatusFilter(s); setPage(1); }}
            className={`px-3 py-1 text-sm rounded-full border ${
              statusFilter === s
                ? "bg-blue-600 text-white border-blue-600"
                : "bg-white border-gray-200"
            }`}
          >
            {s || "All"}
          </button>
        ))}
      </div>

      <div className="overflow-x-auto rounded-xl border">
        <table className="w-full text-sm">
          <thead className="bg-gray-50">
            <tr>
              {["User", "Date", "Time", "Status", "Booked At", "Actions"].map(
                (h) => (
                  <th key={h} className="px-4 py-3 text-left font-semibold text-gray-600">
                    {h}
                  </th>
                )
              )}
            </tr>
          </thead>
          <tbody className="divide-y">
            {isLoading &&
              Array.from({ length: 5 }).map((_, i) => (
                <tr key={i}>
                  {Array.from({ length: 6 }).map((__, j) => (
                    <td key={j} className="px-4 py-3">
                      <div className="h-4 bg-gray-100 rounded animate-pulse w-24" />
                    </td>
                  ))}
                </tr>
              ))}

            {data?.bookings.map((b) => {
              const slot = b.slotId as Slot;
              const user = b.userId as User;
              return (
                <tr key={b._id} className="hover:bg-gray-50">
                  <td className="px-4 py-3">
                    <p className="font-medium">{user.name}</p>
                    <p className="text-gray-400 text-xs">{user.email}</p>
                  </td>
                  <td className="px-4 py-3">
                    {format(new Date(slot.date), "MMM d, yyyy")}
                  </td>
                  <td className="px-4 py-3">
                    {slot.startTime}–{slot.endTime}
                  </td>
                  <td className="px-4 py-3">
                    <Badge
                      color={
                        b.status === "CONFIRMED"
                          ? "green"
                          : b.status === "CANCELLED"
                          ? "red"
                          : "yellow"
                      }
                    >
                      {b.status}
                    </Badge>
                  </td>
                  <td className="px-4 py-3 text-gray-500">
                    {format(new Date(b.createdAt), "MMM d, HH:mm")}
                  </td>
                  <td className="px-4 py-3">
                    {b.status === "CONFIRMED" && (
                      <Button
                        size="sm"
                        variant="outline"
                        loading={cancelling}
                        onClick={() => cancelBooking(b._id)}
                        className="text-red-600 border-red-200"
                      >
                        Cancel
                      </Button>
                    )}
                  </td>
                </tr>
              );
            })}
          </tbody>
        </table>
      </div>

      {data && data.pagination.pages > 1 && (
        <div className="flex gap-2 mt-4 justify-end">
          <button
            disabled={page === 1}
            onClick={() => setPage((p) => p - 1)}
            className="px-3 py-1 rounded border disabled:opacity-40 text-sm"
          >
            ← Prev
          </button>
          <span className="px-3 py-1 text-sm text-gray-600">
            {page} / {data.pagination.pages}
          </span>
          <button
            disabled={page === data.pagination.pages}
            onClick={() => setPage((p) => p + 1)}
            className="px-3 py-1 rounded border disabled:opacity-40 text-sm"
          >
            Next →
          </button>
        </div>
      )}
    </div>
  );
}
```

---

## 15. Frontend — API Client + React Query Hooks

### `app/layout.tsx` (Provider setup)

```tsx
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import { Providers } from "./Providers";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: "SlotBook",
  description: "Appointment booking made simple",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

### `app/Providers.tsx`

```tsx
"use client";
import { QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { queryClient } from "@/lib/queryClient";
import { ToastProvider } from "@/components/ui/Toast";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      <ToastProvider>
        {children}
      </ToastProvider>
      {process.env.NODE_ENV === "development" && <ReactQueryDevtools />}
    </QueryClientProvider>
  );
}
```

### `middleware.ts` (Route protection)

```typescript
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

const PUBLIC_PATHS = ["/", "/login", "/register"];
const ADMIN_PATHS = ["/admin"];

export function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl;
  // Read from cookie (or localStorage — use cookie for SSR)
  const token = req.cookies.get("accessToken")?.value;
  const role = req.cookies.get("userRole")?.value;

  const isPublic = PUBLIC_PATHS.some((p) => pathname === p);
  if (!token && !isPublic) {
    return NextResponse.redirect(new URL("/login", req.url));
  }

  const isAdmin = ADMIN_PATHS.some((p) => pathname.startsWith(p));
  if (isAdmin && role !== "ADMIN") {
    return NextResponse.redirect(new URL("/dashboard", req.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico|api).*)"],
};
```

---

## 16. Frontend — Shared Components

### `components/ui/Button.tsx`

```tsx
import { ButtonHTMLAttributes, forwardRef } from "react";
import { cn } from "@/lib/utils";

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "outline" | "ghost";
  size?: "sm" | "md" | "lg";
  loading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ children, variant = "primary", size = "md", loading, className, disabled, ...props }, ref) => {
    const base = "inline-flex items-center justify-center font-medium rounded-lg transition-all focus:outline-none focus:ring-2 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed";
    const variants = {
      primary: "bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500",
      outline: "border border-gray-300 text-gray-700 hover:bg-gray-50 focus:ring-gray-400",
      ghost: "text-gray-600 hover:bg-gray-100 focus:ring-gray-400",
    };
    const sizes = {
      sm: "px-3 py-1.5 text-sm",
      md: "px-4 py-2 text-sm",
      lg: "px-6 py-3 text-base",
    };

    return (
      <button
        ref={ref}
        disabled={disabled || loading}
        className={cn(base, variants[variant], sizes[size], className)}
        {...props}
      >
        {loading && (
          <svg className="animate-spin -ml-1 mr-2 h-4 w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
            <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
          </svg>
        )}
        {children}
      </button>
    );
  }
);
Button.displayName = "Button";
```

### `components/ui/Input.tsx`

```tsx
import { InputHTMLAttributes, forwardRef } from "react";
import { cn } from "@/lib/utils";

interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, className, ...props }, ref) => (
    <div className="w-full">
      {label && (
        <label className="block text-sm font-medium text-gray-700 mb-1">
          {label}
        </label>
      )}
      <input
        ref={ref}
        className={cn(
          "w-full px-3 py-2 border rounded-lg text-sm outline-none transition",
          error
            ? "border-red-400 focus:ring-2 focus:ring-red-300"
            : "border-gray-300 focus:ring-2 focus:ring-blue-400 focus:border-blue-400",
          className
        )}
        {...props}
      />
      {error && <p className="mt-1 text-xs text-red-500">{error}</p>}
    </div>
  )
);
Input.displayName = "Input";
```

### `components/ui/Toast.tsx`

```tsx
"use client";
import { createContext, useContext, useState, useCallback, ReactNode } from "react";

type ToastType = "success" | "error" | "info";
interface Toast { id: number; message: string; type: ToastType }

const ToastCtx = createContext<{
  showToast: (message: string, type?: ToastType) => void;
}>({ showToast: () => {} });

export const useToast = () => useContext(ToastCtx);

export function ToastProvider({ children }: { children: ReactNode }) {
  const [toasts, setToasts] = useState<Toast[]>([]);
  let counter = 0;

  const showToast = useCallback((message: string, type: ToastType = "info") => {
    const id = ++counter;
    setToasts((t) => [...t, { id, message, type }]);
    setTimeout(() => setToasts((t) => t.filter((x) => x.id !== id)), 4000);
  }, []);

  const colors: Record<ToastType, string> = {
    success: "bg-green-600",
    error: "bg-red-600",
    info: "bg-blue-600",
  };

  return (
    <ToastCtx.Provider value={{ showToast }}>
      {children}
      <div className="fixed bottom-4 right-4 z-[9999] flex flex-col gap-2">
        {toasts.map((t) => (
          <div
            key={t.id}
            className={`${colors[t.type]} text-white px-4 py-3 rounded-lg shadow-lg text-sm max-w-xs animate-slide-in`}
          >
            {t.message}
          </div>
        ))}
      </div>
    </ToastCtx.Provider>
  );
}
```

### `components/ui/Skeleton.tsx`

```tsx
import { cn } from "@/lib/utils";

export function Skeleton({ className }: { className?: string }) {
  return (
    <div className={cn("animate-pulse bg-gray-200 rounded-lg", className)} />
  );
}
```

### `components/ui/Badge.tsx`

```tsx
import { cn } from "@/lib/utils";

const COLOR_MAP: Record<string, string> = {
  green: "bg-green-100 text-green-700",
  red: "bg-red-100 text-red-700",
  yellow: "bg-yellow-100 text-yellow-700",
  blue: "bg-blue-100 text-blue-700",
  gray: "bg-gray-100 text-gray-600",
};

export function Badge({
  children,
  color = "gray",
  className,
}: {
  children: React.ReactNode;
  color?: string;
  className?: string;
}) {
  return (
    <span
      className={cn(
        "inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium",
        COLOR_MAP[color] ?? COLOR_MAP.gray,
        className
      )}
    >
      {children}
    </span>
  );
}
```

### `components/admin/StatsCard.tsx`

```tsx
interface Props {
  label: string;
  value: number;
  icon: string;
  color?: "default" | "green" | "red" | "yellow";
}

const colorMap = {
  default: "bg-blue-50 text-blue-700",
  green: "bg-green-50 text-green-700",
  red: "bg-red-50 text-red-700",
  yellow: "bg-yellow-50 text-yellow-700",
};

export function StatsCard({ label, value, icon, color = "default" }: Props) {
  return (
    <div className={`rounded-2xl p-5 ${colorMap[color]}`}>
      <div className="text-3xl mb-2">{icon}</div>
      <p className="text-3xl font-bold">{value.toLocaleString()}</p>
      <p className="text-sm font-medium mt-1 opacity-80">{label}</p>
    </div>
  );
}
```

### `lib/utils.ts`

```typescript
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

---

## 17. Security Considerations

| Concern | Implementation |
|---|---|
| Password storage | `bcrypt` with cost factor 12 |
| JWT secret strength | Min 32-char random string, stored in env |
| Token expiry | Access: 15 min · Refresh: 7 days; refresh rotated on every use |
| Refresh token replay | Stored in DB; invalidated on logout; mismatch = full logout |
| Rate limiting | Auth endpoints: 10 req/15 min · Global: 100 req/15 min |
| Input validation | Zod on every route, including query params |
| MongoDB injection | Mongoose's typed schema prevents raw query injection |
| CORS | Whitelisted origins only via env var |
| Headers | `helmet` sets secure HTTP headers (no `X-Powered-By`, CSP, etc.) |
| RBAC | Middleware-enforced on every admin route |
| Double booking | MongoDB transaction + atomic `findOneAndUpdate` with condition |
| Password never returned | `select: false` on password field in Mongoose schema |

---

## 18. Scaling Strategy

```
Horizontal scaling:
  ├── Stateless JWT → multiple API servers behind load balancer
  ├── MongoDB Atlas → replica sets + sharding on userId/date
  └── Redis (optional):
        ├── Cache slot availability per date (TTL 30s)
        └── Distributed lock on slotId for atomic booking under extreme concurrency

Performance:
  ├── DB Indexes: (email), (date+startTime unique), (userId+status), (slotId+status)
  ├── Lean queries (.lean()) on all read paths
  ├── Pagination on all list endpoints (default 20, max 100)
  ├── React Query staleTime = 30s → avoids repeat network calls
  └── Debounced slot fetching in UI (300ms)

Future multi-tenancy:
  └── Add `organizationId` field to Slot + Booking → one deployment serves many clients
```

---

## 19. Deployment Guide

### Backend on Render / Railway

```bash
# 1. Build
cd apps/backend
npm install
npm run build   # tsc → dist/

# 2. Start command
node dist/index.js

# 3. Set all env vars in dashboard (see Section 2)
```

### Frontend on Vercel

```bash
cd apps/frontend

# 1. Link project
vercel link

# 2. Add env vars in Vercel dashboard
# NEXT_PUBLIC_API_URL=https://your-backend.onrender.com/api/v1

# 3. Deploy
vercel --prod
```

### MongoDB Atlas Setup

```
1. Create cluster → Network Access: allow 0.0.0.0/0 (or specific IPs)
2. Create DB user with readWrite on slot-booking DB
3. Copy connection string → set as MONGO_URI in backend env
4. Enable Performance Advisor → create suggested indexes
```

---

## 20. API Documentation (Postman-Ready)

> Base URL: `{{baseUrl}} = https://your-api.com/api/v1`
> Set `Authorization: Bearer {{accessToken}}` as collection-level header.

### Auth

| Method | Endpoint | Body | Auth | Description |
|---|---|---|---|---|
| POST | `/auth/register` | `{name, email, password}` | ❌ | Register user |
| POST | `/auth/login` | `{email, password}` | ❌ | Login, returns tokens |
| POST | `/auth/refresh` | `{refreshToken}` | ❌ | Rotate tokens |
| POST | `/auth/logout` | — | ✅ | Invalidate refresh token |

### Slots

| Method | Endpoint | Query | Auth | Description |
|---|---|---|---|---|
| GET | `/slots` | `?date=YYYY-MM-DD` OR `?week=YYYY-MM-DD` | ✅ | Get slots for date/week |
| GET | `/slots/:id` | — | ✅ | Get single slot |

### Bookings

| Method | Endpoint | Body | Auth | Description |
|---|---|---|---|---|
| POST | `/bookings` | `{slotId}` | ✅ | Create booking |
| GET | `/bookings` | `?page=1&limit=10` | ✅ | My bookings (paginated) |
| GET | `/bookings/:id` | — | ✅ | Single booking detail |
| PATCH | `/bookings/:id/cancel` | — | ✅ | Cancel booking |
| PATCH | `/bookings/:id/reschedule` | `{newSlotId}` | ✅ | Reschedule booking |

### Admin (role: ADMIN required)

| Method | Endpoint | Body / Query | Auth | Description |
|---|---|---|---|---|
| POST | `/admin/slots` | `{slots: [{date, startTime, endTime}]}` | ✅ ADMIN | Bulk create slots |
| PATCH | `/admin/slots/:id/disable` | — | ✅ ADMIN | Disable slot |
| PATCH | `/admin/slots/:id/enable` | — | ✅ ADMIN | Enable slot |
| GET | `/admin/bookings` | `?page=1&status=CONFIRMED` | ✅ ADMIN | All bookings |
| PATCH | `/admin/bookings/:id/cancel` | — | ✅ ADMIN | Admin cancel any booking |
| GET | `/admin/analytics` | — | ✅ ADMIN | Stats aggregate |

### Error Response Shape (all errors)

```json
{
  "success": false,
  "message": "This slot is no longer available"
}
```

### Success Response Shape

```json
{
  "success": true,
  "statusCode": 200,
  "data": { ... },
  "message": "Slots fetched"
}
```

---

## Edge Cases Covered

| Scenario | Handling |
|---|---|
| Two users book same slot simultaneously | MongoDB transaction + atomic `findOneAndUpdate` with `isBooked: false` condition |
| User books already-booked slot | 409 "This slot is no longer available" |
| Expired access token | 401 returned → frontend auto-retries with refresh token |
| Refresh token replay attack | Token stored in DB; mismatch → force logout |
| User cancels already-cancelled booking | 400 "Booking already cancelled" |
| Reschedule to same slot | 400 "New slot must be different" |
| Admin disables a booked slot | `isDisabled` flag set; existing booking unaffected |
| Past dates in slot picker | WeekCalendar disables past days visually + logically |
| Bulk slot creation with duplicates | `insertMany` with `ordered: false` + 11000 error caught → 409 |
| Admin cancels user booking | `isAdmin=true` bypasses ownership check |
| Network failure in frontend | Axios timeout (10s) + React Query retry once + toast notification |
| Invalid JWT format | `verifyAccessToken` catch → 401 "Invalid token" |
| XSS / injection in form fields | Zod trim + type enforcement; Mongoose typed queries |
| Unauthorized access to admin routes | `rbacMiddleware("ADMIN")` → 403 "Access denied" |
