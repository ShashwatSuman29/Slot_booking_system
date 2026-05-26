# CONTEXT AND RULE:

You are a senior software engineer and product manager who is responsible for building a production grade web application for booking slot system which should be performance optimized , maintainable , scalable.

# OBJECTIVE:

Design and implement a full-stack Slot Booking Web Application that allows users to:

1-View available time slots

2-Book appointments

3-Manage bookings which could be create , cancel & rescheduled

4-Prevent double booking conflicts

5-Provide role-based access RBAC (Admin , User)

# TECH STACK:

## Frontend:

* Next.js
* TypeScript
* Tailwind CSS
* React Query / SWR (server state management)
* Zustand (optional client state)

## Backend:

* Node.js + Express OR Next.js API Routes
* RESTful APIs
* JWT Authentication
* Zod / Joi validation

## Database:

* MongoDB (Mongoose ODM)

## Infra:

* Vercel (Frontend)
* Cloudinary (optional for media)
* Redis (optional for caching & locking slots)

# UI REQUIREMENTS:

## Pages:

* Landing Page
* Authentication (Login/Register)
* Dashboard
* Slot Selection Page
* Booking Confirmation Page
* Admin Panel

## UI Components:

* Calendar view for slot selection
* Time-slot cards (available / booked / disabled)
* Booking modal (confirm/cancel)
* Toast notifications
* Loading skeletons
* Error states UI
* Admin analytics cards

# LAYOUT REQUIREMENTS:

## Dashboard:

* Sidebar navigation (Bookings, Profile, Settings)

### Main content area:

* Available slots grid
* Upcoming bookings section

## Slot Page:

* Weekly calendar view (top)
* Time slots grid (center)
* Booking CTA button (bottom fixed or modal trigger)

## Admin Panel:

* Stats cards (total bookings, cancellations)
* User booking table
* Slot management interface

# WEB APPLICATION WORKFLOW:

## USER FLOW:

* User registers/logs in
* Lands on dashboard
* Selects date → views slots
* Chooses slot → clicks "Book"
* System validates availability
* Booking confirmed
* Entry stored in DB

## ADMIN FLOW:

* Admin logs in
* Views all bookings
* Can create/disable slots
* Can cancel user bookings
* Views analytics dashboard

# CORE FEATURES:

* Authentication (JWT)
* Role-based access control
* Slot conflict prevention
* Real-time slot availability update
* Booking CRUD operations
* Admin control panel
* Email notifications (optional)

# ERROR HANDLING REQUIREMENTS:

* Slot already booked → "this slot is not available"
* Unauthorized access → redirect to login
* Invalid token → logout user
* Server error → fallback UI with retry
* Network failure → offline error toast
* Form validation errors → inline field errors

# PERFORMANCE REQUIREMENTS:

* Debounced slot fetching
* Pagination for admin tables
* Indexed DB fields (date, userId, slotId)
* Optimistic UI updates
* Caching for slot availability
* Lazy loading pages/components

# DATA PROCESSING REQUIREMENTS:

* Data normalization before DB storage
* Data sanitization for all user inputs
* Efficient aggregation pipelines for analytics
* Batch processing support for admin operations
* Real-time slot availability synchronization
* Consistent date/timezone handling
* Structured logging for booking events
* Background job processing for notifications/cache refresh
* Scalable data querying strategy
* High concurrency transaction handling

# DATABASE DESIGN:

## Users

* id
* name
* email
* password
* role (USER / ADMIN)

## Slots

* id
* date
* time
* isBooked

## Bookings

* id
* userId
* slotId
* status
* createdAt

# OUTPUT REQUIREMENTS:

The final output must include:

1-Full frontend code (Next.js)

2-Backend APIs (auth + booking logic)

3-Database schema

4-Folder structure

5-Deployment guide

6-Environment variables setup

7-API documentation (Postman-ready)

# DOCUMENTATION REQUIREMENTS:

1-Setup instructions

2-Architecture explanation

3-API endpoints list

4-DB schema explanation

5-Security considerations

6-Scaling strategy

# SECURITY REQUIREMENTS:

-Password hashing (bcrypt)

-JWT expiration + refresh strategy

-Rate limiting on APIs

-Input validation on all routes

-Role-based route protection

# FINAL NOTE:

System must be:

1-Modular

2-Production-ready

3-Scalable

4-Cleanly structured

5-Easy to extend (multi-service future support)
