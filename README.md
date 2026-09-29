# Hospitality Management System

## Project Report

### Abstract

The Hospitality Management System is a web application for managing the daily operations of a hotel. It provides hotel owners and authorized staff with tools to manage rooms, reservations, guests, check-in and check-out, payments, food orders, invoices, and operational reports. A separate super-administrator login is provided for platform-level access.

The application has a React single-page frontend, an Express REST API, and Supabase as its active database service. Cloudinary is used for hotel-logo uploads. Invoice PDFs are generated on the server with Puppeteer and Handlebars templates.

### Objectives

- Centralize hotel room, reservation, guest, and staff records.
- Track room availability and reservation status.
- Record room and food-service payments, including partial payments and refunds.
- Support kitchen order tickets (KOT) and food-order history.
- Generate invoices and provide cashier and payment reports.
- Give staff access according to permissions assigned by the hotel owner.

### Scope and Architecture

```text
Hotel owner / staff / super administrator
								 |
								 v
			React + Vite single-page app
								 |
					HTTP JSON requests
								 |
								 v
			 Express REST API (Node.js)
					|              |
					v              v
 Supabase database   Cloudinary images
					|
					v
 Invoice data -> Handlebars templates -> Puppeteer PDF
```

The frontend uses React Router for navigation, Axios/fetch for HTTP requests, and local storage for the authentication token and staff permissions. The API uses Supabase's JavaScript client to query PostgreSQL tables. No SQL migration/schema files are included in this repository; the entity descriptions below are inferred from the queries in the application code.

### Technology Stack

| Area | Technologies |
|---|---|
| Frontend | React 18, Vite, React Router, Axios, Tailwind CSS, regular CSS |
| UI and charts | Framer Motion, Headless UI, Lucide React, React Icons, Recharts, Swiper |
| Forms and validation | Formik, React Hook Form, Yup |
| Backend | Node.js, Express 4, JSON Web Tokens |
| Database | Supabase JavaScript client / PostgreSQL |
| Password hashing | bcryptjs |
| File upload | Multer and Cloudinary |
| Invoice generation | Handlebars, Puppeteer, HTML invoice templates |
| Date handling | Moment Timezone, date-fns, Day.js (depending on module) |

## Repository Structure

```text
backend/
	src/
		app.js                  Express app, middleware and route registration
		config/                 Supabase client configuration
		controllers/            Business logic for API features
		middleware/             Authentication, access and input validation
		routes/                 HTTP endpoint definitions
		utils/                  Cloudinary, storage and invoice helpers
	config/database.js        Legacy, commented MySQL/Sequelize configuration
	controllers/              Legacy controller outside the active src tree
frontend/
	src/
		App.jsx                 Public, protected and super-admin routes
		components/             Reusable forms, dialogs, navigation and UI
		hooks/                  Shared React hooks
		layouts/                Public site layout
		pages/                  Public site and hotel-management screens
		styles/                 Feature-specific stylesheets
		utils/                  API constants, invoice helpers and utilities
	public/                   Logos and menu images
README.md                   Project documentation
```

The backend entry point is `backend/src/app.js`; the frontend entry point is `frontend/src/main.jsx`. The files directly under `backend/controllers/` and `backend/config/` are separate from the server's active `backend/src/` application.

## Installation and Local Setup

### Prerequisites

- Node.js and npm (use a Node.js version compatible with the installed Puppeteer release).
- A Supabase project with the tables used by the application.
- Cloudinary credentials for hotel-logo uploads.

### Backend

```bash
cd backend
npm install
```

Create `backend/.env` and provide the values below. Keep real credentials private and do not commit the file.

```dotenv
PORT=5000
JWT_SECRET=replace-with-a-long-random-secret
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-supabase-anon-key
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-cloudinary-api-key
CLOUDINARY_API_SECRET=your-cloudinary-api-secret
```

Start the API:

```bash
npm run dev
```

`npm start` runs the same Express application without nodemon. The server listens on `process.env.PORT`; the code does not define a fallback port.

### Frontend

In a second terminal:

```bash
cd frontend
npm install
```

Create `frontend/.env`:

```dotenv
VITE_API_URL=http://localhost:5000
```

Start the Vite development server:

```bash
npm run dev
```

Vite is configured for port `3000`. The backend CORS allowlist currently includes `http://localhost:3000`; update that allowlist when the frontend is hosted at another origin. The frontend API calls generally append paths beginning with `/api`, so `VITE_API_URL` should normally be the backend origin without a trailing `/api`.

### Available Frontend Commands

| Command | Purpose |
|---|---|
| `npm run dev` | Start Vite development server |
| `npm run build` | Create the production frontend build in `dist/` |
| `npm run preview` | Preview the built frontend locally |
| `npm run lint` | Run ESLint on JavaScript and JSX files |
| `npm run format` | Format frontend source files with Prettier |
| `npm run analyze` | Run a Vite build using the analyze mode |

The backend package provides `npm run start` and `npm run dev`. It does not define a test command. No test/spec files were found in the repository when this documentation was prepared.

## Configuration

### Backend Configuration

| File | Responsibility |
|---|---|
| `backend/src/app.js` | Loads dotenv, creates Express app, enables CORS and JSON parsing, registers API route groups, exposes `/api/health`, and starts the server. |
| `backend/src/config/db.js` | Creates and exports the Supabase client using `SUPABASE_URL` and `SUPABASE_ANON_KEY`. Most backend modules import this client directly. |
| `backend/src/config/supabase.js` | Creates and exports the same Supabase client as a named `supabase` property; used by food-payment code. |
| `backend/src/middleware/auth.js` | Verifies bearer JWTs using `JWT_SECRET` and attaches claims to `req.user`. Staff claims are mapped to their creator's `user_id` for existing tenant-scoped handlers. |
| `backend/src/middleware/isSuperAdmin.js` | Verifies the super-admin role in a JWT and checks that the administrator still exists in Supabase. |
| `backend/src/middleware/verifyBookingAccess.js` | Checks whether at least one room in a booking belongs to the authenticated hotel before allowing booking-specific actions. |
| `backend/src/middleware/validateHotelDetails.js` | Validates hotel address requirements, Indian GST-number format and six-digit PIN code format. |
| `backend/src/utils/cloudinary.js` | Configures Cloudinary and accepts a single image in the `logo` field, with a 5 MB limit. |
| `backend/src/middleware/fileUpload.js` | Alternate Multer/base64 upload helper; the active user routes use the Cloudinary handler instead. |

`backend/src/app.js` currently permits CORS requests from `http://localhost:3000`. It accepts JSON request bodies through Express. API route groups are mounted under `/api` as documented below.

### Frontend Configuration

| File | Responsibility |
|---|---|
| `frontend/vite.config.js` | Enables the React plugin, defines `@` path aliases, sets the dev port to 3000, configures polling-based file watching, and defines production chunks/minification. |
| `frontend/tailwind.config.js` | Scans the HTML and source files for Tailwind classes and defines project colors, fonts and container spacing. |
| `frontend/postcss.config.js` | Runs Tailwind CSS and Autoprefixer. |
| `frontend/eslint.config.js` | Enables recommended JavaScript, React Hooks and React Refresh rules. |
| `frontend/.prettierrc` | Configures semicolons, single quotes, two-space indentation, ES5 trailing commas and the Tailwind Prettier plugin. |
| `frontend/index.html` | Vite HTML shell containing the React mount element and `src/main.jsx` entry script. |
| `frontend/vercel.json` | Rewrites incoming paths to `index.html` for client-side React Router navigation. |
| `frontend/src/utils/constants.js` | Defines a shared API URL and endpoint constants. Its fallback currently differs in shape from the `/api`-prefixed convention used by many screens. |
| `.gitignore` | Excludes dependencies, environment files, build output, logs, editor files and generated files. |

The root-level `backend/config/database.js` contains commented Sequelize/MySQL setup and is not used by the active Express application. Supabase is the active database integration.

## Authentication and Access Control

- Hotel-owner registration and login use email/password. Passwords are hashed with bcryptjs; successful login returns a signed JWT that expires after 24 hours.
- Staff login uses a phone identifier and password. The JWT expires after seven days and contains `staff_id` and `created_by` claims.
- The shared authentication middleware requires `Authorization: Bearer <token>`. It maps a staff token's `created_by` claim to `req.user.user_id` so existing controllers can scope records to the hotel.
- Staff permissions are returned by the staff APIs and are checked by the frontend's protected route component. The frontend stores authentication and permission state in local storage.
- Super-admin login returns a role-bearing JWT that expires after 24 hours. The `isSuperAdmin` middleware also checks the database record for routes that use it.
- Most room, booking, customer, menu, report and food-order routes require authentication. Booking-specific routes additionally use `verifyBookingAccess` where indicated.

## Frontend Functionality

### Public Pages

| Path | Screen / purpose |
|---|---|
| `/` | Marketing home page with product features, testimonials and FAQ sections. |
| `/about` | About/story, team and values content. |
| `/features` | Feature overview and comparison content. |
| `/pricing` | Pricing information. |
| `/contact` | Contact page and form. |
| `/login` | Public login page. |
| `/register` | Placeholder page; registration is not yet presented as a finished frontend flow. |
| `*` | Not-found page. |

### Hotel and Staff Pages

| Path | Screen / purpose |
|---|---|
| `/dashboard` | Hotel overview, room availability and today's check-in/check-out information. |
| `/bookings` | Booking list and booking actions, including payment, check-in/out and invoice-related actions. |
| `/bookings/new` | Create a booking and enter guest, room, date and payment details. |
| `/edit-booking/:bookingId` | Edit guest, room, date, rate and booking information. |
| `/booking/view/:bookingId` | View booking, guest, room and food-order details. |
| `/rooms` | Room inventory, room types, prices, status and history. |
| `/customers` | Customer directory and customer detail access. |
| `/settings` | Staff account and permission management. |
| `/menu` | Food menu management. |
| `/bookings/:bookingId/invoice` | Invoice preview/printing. |
| `/booking/:bookingId/order-food` | Food ordering, order status, item cancellation and KOT-related operations. |
| `/food-payment-report` | Food payment summary, trends and transaction reporting. |
| `/cashier-report` | Room/booking payment summary, daily transactions and trends. |
| `/view-profile`, `/edit-profile` | Owner profile views and edits; staff are redirected away from these owner-only routes. |
| `/staff-profile` | Staff profile page. |

### Super-Administrator Pages

| Path | Screen / purpose |
|---|---|
| `/super-admin/login` | Super-administrator sign-in. |
| `/super-admin/dashboard` | Super-administrator dashboard, guarded in the frontend by a super-admin token. |

Protected hotel pages are wrapped in the application layout and can be restricted by a permission key. The public marketing pages use a separate root layout with shared header and footer.

## Backend API Routes

All paths below are relative to the API origin. For example, `GET /api/rooms` is the complete rooms endpoint. `Auth` means the route requires the JWT middleware. `Booking access` means the route also checks ownership through `verifyBookingAccess`.

### Health Check

| Method | Path | Access | Purpose |
|---|---|---|---|
| `GET` | `/api/health` | Public | Queries Supabase and returns API/database health status. |

### Users and Hotel Profile (`/api/users`)

| Method | Path | Access | Purpose |
|---|---|---|---|
| `POST` | `/api/users/register` | Public | Registers an owner, creates hotel details and optionally uploads a logo. |
| `POST` | `/api/users/login` | Public | Authenticates an owner and returns a JWT. |
| `GET` | `/api/users/profile` | Auth | Gets the authenticated owner's profile. |
| `GET` | `/api/users/` | Auth + super admin | Lists owner accounts for platform administration. |
| `PUT` | `/api/users/profile` | Auth | Updates owner profile fields. |
| `PUT` | `/api/users/change-password` | Auth | Verifies the current password and stores a new hash. |
| `GET` | `/api/users/hotel-details` | Auth | Gets the current hotel's business details. |
| `PUT` | `/api/users/hotel-details` | Auth | Creates or updates hotel address, GST and logo information. |

### Super Administrator (`/api/super-admin`)

| Method | Path | Access | Purpose |
|---|---|---|---|
| `POST` | `/api/super-admin/login` | Public | Authenticates a super administrator and returns a role-bearing JWT. |

### Rooms (`/api/rooms`)

All routes in this group require authentication.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/rooms/types` | Lists available room types with base price and count. |
| `GET` | `/api/rooms/available` | Finds rooms by room type and optional check-in/check-out dates and requested quantity. |
| `POST` | `/api/rooms` | Creates a room for the authenticated hotel. |
| `GET` | `/api/rooms` | Lists the hotel's rooms with associated booking information. |
| `PUT` | `/api/rooms/:room_id` | Updates room details and status. |
| `DELETE` | `/api/rooms/:room_id` | Deletes a room if it has no active/upcoming booking. |
| `GET` | `/api/rooms/:room_id/history` | Gets bookings associated with a room. |

### Bookings (`/api/bookings`)

Every route requires authentication. Routes marked `Booking access` additionally verify that the booking is associated with a room belonging to the authenticated hotel.

| Method | Path | Access | Purpose |
|---|---|---|---|
| `POST` | `/api/bookings` | Auth | Validates guest/date/room input and creates a booking with room and guest records. |
| `GET` | `/api/bookings` | Auth | Lists bookings belonging to the authenticated hotel's rooms. |
| `GET` | `/api/bookings/:booking_id/invoice/details` | Auth + Booking access | Gets hotel, guest, room and booking data for an invoice. |
| `GET` | `/api/bookings/:booking_id/invoice/print-data` | Auth + Booking access | Gets invoice data and, when present, related food-bill data. |
| `GET` | `/api/bookings/:booking_id` | Auth + Booking access | Gets complete booking details, guests and food orders. |
| `PUT` | `/api/bookings/:booking_id/checkin` | Auth + Booking access | Marks an upcoming booking checked in and occupies its rooms. |
| `PUT` | `/api/bookings/:booking_id/checkout` | Auth + Booking access | Checks out a checked-in booking, subject to outstanding food dues, and releases its rooms. |
| `PUT` | `/api/bookings/:booking_id/cancel` | Auth + Booking access | Cancels an eligible booking and may record a refund. |
| `PUT` | `/api/bookings/:booking_id/payment` | Auth + Booking access | Changes booking payment status. |
| `POST` | `/api/bookings/:booking_id/payment` | Auth + Booking access | Records a payment transaction and updates the booking's paid amount/status. |
| `GET` | `/api/bookings/:booking_id/bill` | Auth + Booking access | Gets booking billing data, rooms and guests. |
| `GET` | `/api/bookings/:booking_id/invoice/download` | Auth + Booking access | Generates and downloads the room invoice PDF, with a food bill when available. |
| `GET` | `/api/bookings/:booking_id/details` | Auth + Booking access | Gets detailed booking information through the booking-details controller. |
| `GET` | `/api/bookings/available-rooms` | Auth | Intended to find rooms during booking edits using query parameters. |
| `PUT` | `/api/bookings/:booking_id` | Auth + Booking access | Updates guest, dates, room assignments, rates and booking/payment data. |

### Customers (`/api/customers`)

All routes require authentication.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/customers` | Lists customers associated with the hotel's bookings. |
| `GET` | `/api/customers/current` | Lists customers with checked-in bookings. |
| `GET` | `/api/customers/past` | Lists customers with checked-out bookings. |
| `GET` | `/api/customers/search?query=...` | Searches customer name, phone and email. |
| `GET` | `/api/customers/:customer_id` | Gets a customer's details, booking history and calculated visit/spend/night totals. |
| `PUT` | `/api/customers/:customer_id` | Updates customer fields. |
| `PUT` | `/api/customers/:customer_id/additional-guests` | Replaces additional guest records for an active booking. |

### Dashboard and Cashier Reports

| Method | Path | Access | Purpose |
|---|---|---|---|
| `GET` | `/api/dashboard/stats` | Auth | Returns room counts, current guests and today's check-ins/check-outs. |
| `GET` | `/api/cashier/summary?start_date=...&end_date=...` | Auth | Summarizes booking totals, collections, refunds and payment methods for a date range. |
| `GET` | `/api/cashier/daily-transactions?date=...` | Auth | Lists transactions for one date; a date range may also be supplied. |
| `GET` | `/api/cashier/payment-trends?start_date=...&end_date=...` | Auth | Groups collections and refunds by day and payment method. |

### Staff (`/api/staff`)

| Method | Path | Access | Purpose |
|---|---|---|---|
| `POST` | `/api/staff/login` | Public | Authenticates an active staff member by phone and password. |
| `GET` | `/api/staff` | Auth | Lists staff created by the authenticated hotel owner. |
| `POST` | `/api/staff` | Auth | Creates a staff account with a hashed password. |
| `PUT` | `/api/staff/:id` | Auth | Updates staff profile, password, permissions or active status, subject to creator ownership. |
| `PUT` | `/api/staff/:id/permissions` | Auth | Updates only a staff member's permissions. |
| `PUT` | `/api/staff/:id/status` | Auth | Enables or disables a staff account. |
| `DELETE` | `/api/staff/:id` | Auth | Deletes a staff account owned by the authenticated creator. |
| `GET` | `/api/staff/profile` | Auth | Gets the current staff member's profile. |
| `GET` | `/api/staff/profile-with-hotel` | Auth | Gets the staff profile and associated hotel/owner information. |

### Food Menu (`/api/menu`)

All routes require authentication.

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/menu` | Creates a menu item for the current hotel. |
| `GET` | `/api/menu` | Lists the hotel's menu items. |
| `GET` | `/api/menu/:id` | Gets one menu item. |
| `PUT` | `/api/menu/:id` | Updates a menu item owned by the hotel. |
| `PATCH` | `/api/menu/:id/toggle-status` | Changes a menu item's active status. |
| `DELETE` | `/api/menu/:id` | Deletes a menu item owned by the hotel. |

### Food Orders (`/api/food-orders`)

All routes require authentication. New food orders are intended for checked-in bookings. Food payments are recorded through the separate food-payment routes.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/food-orders/check/:booking_id` | Checks whether the booking has a food order. |
| `GET` | `/api/food-orders/booking/:booking_id/details` | Gets order details and remaining item quantities. |
| `GET` | `/api/food-orders/history/:booking_id` | Gets KOT history for a booking, with times formatted for IST. |
| `POST` | `/api/food-orders/create` | Creates an order, stores its items, calculates the GST-inclusive total and records an initial KOT snapshot. |
| `POST` | `/api/food-orders/:orderId/print-kot` | Returns current order/KOT data for printing. |
| `PUT` | `/api/food-orders/:orderId` | Adds, updates or removes order items and recalculates the total/due amount. |
| `PATCH` | `/api/food-orders/:orderId/status` | Changes status to `pending`, `preparing`, `delivered` or `cancelled`. |
| `DELETE` | `/api/food-orders/:orderId` | Cancels an order and removes its item rows. |
| `PATCH` | `/api/food-orders/:orderId/items/:itemId/cancel` | Voids all or part of an item's quantity and recalculates the order total. |

### Food Payments (`/api/food-payments`)

All routes require authentication. Report date ranges use the `Asia/Kolkata` timezone in the controller logic.

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/food-payments/record` | Records a food-order payment and updates paid/due/payment status. |
| `GET` | `/api/food-payments/food-order/:food_order_id` | Lists transactions for one food order. |
| `GET` | `/api/food-payments/booking/:booking_id` | Lists food payment transactions associated with a booking. |
| `GET` | `/api/food-payments/summary?start_date=...&end_date=...` | Summarizes food orders, collections, refunds and payment methods. |
| `GET` | `/api/food-payments/trends?start_date=...&end_date=...` | Groups food collections and refunds by day and payment method. |
| `GET` | `/api/food-payments/transactions?date=...` | Lists food payment transactions for one date or a date range. |

## Backend Controller Responsibilities

| Controller | Main functions and responsibility |
|---|---|
| `userController.js` | `register`, `login`, `getProfile`, `updateProfile`, `changePassword`, `getAllUsers`, `getUserDetails`, `getHotelDetails`, `updateHotelDetails`. Owner accounts, credentials, profile, hotel information and super-admin user listing. |
| `superAdminController.js` | `login`. Validates platform administrator credentials and issues a JWT with the `super_admin` role. |
| `staffController.js` | `getAllStaff`, `createStaff`, `updateStaff`, `updatePermissions`, `updateStatus`, `deleteStaff`, `staffLogin`, `getStaffProfile`, `getStaffProfileWithHotel`. Staff lifecycle, authentication and hotel-scoped ownership checks. |
| `roomController.js` | `getRoomTypes`, `getAvailableRooms`, `addRoom`, `getRooms`, `updateRoom`, `deleteRoom`, `getRoomHistory`. Room inventory and availability operations. |
| `bookingController.js` | `createBooking`, `getBookings`, `checkinBooking`, `checkoutBooking`, `updatePaymentStatus`, `addPayment`, `getBookingForBill`, `getInvoiceDetails`, `getInvoiceDataForPrint`, `downloadInvoice`. Helpers: `convertUTCToISTTime`, `getFormattedDepartureDate`, `getTestBookingId`, `validateRoomAvailability`, `validateIdProof`, `validatePhoneNumber`, `validatePinCode`, `validateEmail`. |
| `updateBookingController.js` | `updateBooking`, `getAvailableRooms`. Updates booking information and room assignments, and provides an edit-time availability query. |
| `getBookingDetailsController.js` | `getBookingDetails`. Combines booking, customer, guest, room and food-order information for a detail view. |
| `cancelBookingController.js` | `cancelBooking`. Cancels eligible bookings, optionally records a refund, and releases assigned rooms. |
| `customerController.js` | `getAllCustomers`, `getCurrentCustomers`, `getPastCustomers`, `getCustomerDetails`, `updateCustomer`, `searchCustomers`. Customer directory, history, search and statistics. |
| `additionalGuestController.js` | `updateAdditionalGuests`. Replaces additional guest records for a checked-in booking. |
| `dashboardController.js` | `getDashboardStats`. Calculates room status totals, today's arrivals/departures and current guest count. |
| `cashierController.js` | `getSummary`, `getDailyTransactions`, `getPaymentTrends`. Room/booking payment reports, refunds, payment modes and date-based trends. |
| `foodController.js` | `createMenuItem`, `getMenuItems`, `getMenuItem`, `updateMenuItem`, `deleteMenuItem`, `toggleMenuItemStatus`. Hotel-scoped menu CRUD and availability. |
| `foodOrderController.js` | `createOrder`, `updateOrder`, `getOrderDetails`, `cancelOrder`, `checkOrderExists`, `printKOT`, `getKOTHistory`, `cancelItem`. Food ordering, item adjustments, KOT history and voids. |
| `foodPaymentController.js` | `recordFoodPayment`, `getFoodOrderTransactions`, `getBookingFoodPaymentTransactions`, `getFoodPaymentSummary`, `getFoodPaymentTrends`, `getFoodPaymentTransactions`. Food payment recording and reporting. |
| `backend/controllers/roomController.js` (legacy) | `getAllRooms`, `getRoomTypes`, `getAvailableRooms`, `getRoomById`, `updateRoomStatus`, `createRoom`. This separate controller is not registered by the active `backend/src/app.js` server. |

`backend/src/controllers/getAllUsers.js` contains a `getAllUsers` function but is not registered by the active routes and does not export a handler in its current file. The active application imports `backend/src/controllers/roomController.js` for room APIs.

## Business Rules and Main Workflows

### Booking Lifecycle

1. The owner/staff submits primary guest details, optional additional guests, selected rooms, stay dates and payment information.
2. The API validates required guest fields, phone/email/identity proof/PIN formats, date order and room availability.
3. The API creates a customer, booking, room assignments and guest records, then updates room status.
4. A booking can be checked in and checked out. Check-in marks rooms occupied; check-out releases them.
5. Check-out is blocked when food-order dues remain. Booking cancellation may create a refund transaction and return rooms to available status.

The code uses booking states such as `Upcoming`, `Checked-in`, `Checked-out`, `Booked` and `Cancelled`; payment states include `PAID`, `PARTIAL`, `UNPAID` and refund-related values. The exact capitalization is not fully uniform across modules.

### Food Ordering and Payments

Food orders are associated with bookings and menu items. The order controller stores price snapshots for ordered items, calculates a 5% GST-inclusive total, and keeps KOT history snapshots. Item cancellation is represented by a `voided_quantity` rather than reducing the original quantity. Food payments are separate transactions and update the order's paid amount, due amount and payment status.

### Invoice and Time Handling

Room invoices are available only for non-cancelled bookings whose payment status is `PAID`. Invoice PDFs are rendered from Handlebars templates with Puppeteer. A food bill can be appended when food-order data exists. The invoice utility calculates 5% total GST, split into 2.5% CGST and 2.5% SGST; for a GST-inclusive total it derives the pre-tax base by dividing by `1.05`. Report controllers convert the requested IST date boundaries to UTC for database queries and format returned timestamps for `Asia/Kolkata`.

## Data Entities Referenced by the Code

The application queries the following Supabase tables. The relationships listed are inferred from joins and inserts in the source; they are not a substitute for the database's authoritative schema.

| Entity/table | Role inferred from usage |
|---|---|
| `users` | Hotel owner/admin account and login/profile data. |
| `hotel_details` | Hotel address, logo, PIN and GST information. |
| `super_admins` | Platform-level administrator credentials and identity. |
| `staff_users` | Hotel staff accounts, permissions and active status. |
| `rooms` | Hotel room inventory, type, price, capacity and status. |
| `customers` | Primary customer information associated with bookings. |
| `bookings` | Stay dates, nights, amounts, status and payment summary. |
| `booking_rooms` | Many-to-many room assignments and room-specific rate information. |
| `booking_guests` | Primary and additional guest details attached to a booking. |
| `additional_guests` | Used by a separate additional-guest update handler. |
| `payment_transactions` | Room/booking payments and refunds. |
| `menu_items` | Hotel food menu and item prices/status. |
| `food_orders` | Food order totals, booking link, status and paid/due amounts. |
| `food_order_items` | Item quantities, price snapshots and void/cancellation metadata. |
| `food_order_history` | KOT snapshots and order-change history. |
| `food_payment_transactions` | Food-order payment transaction history. |

## Data Structures and Algorithms (DSA)

There is no separate DSA/algorithm package in the repository. The application uses standard JavaScript collections and aggregation patterns inside business workflows:

| Technique | Where it is used | Complexity of the in-memory portion |
|---|---|---|
| Hash set (`Set`) for membership and deduplication | Removes duplicate booking IDs for dashboard/booking data and tracks booked room IDs during availability filtering. | Expected `O(n)` construction/lookups; filtering `R` rooms against `B` booked IDs is expected `O(R + B)`. Database query cost is separate. |
| Hash map (`Map`) for grouping | Groups available rooms by type and groups customer records/bookings while building customer-centric results. | Expected `O(n)` for one pass over `n` input records; additional space `O(k)` for `k` groups. |
| Object/dictionary buckets | Cashier and food-payment trends accumulate totals into a date bucket and payment-mode bucket. | `O(T + D)` accumulation for `T` transactions over `D` dates, plus sorting dates where required, `O(D log D)`. |
| Array `reduce` aggregation | Totals for payments, refunds, nights, customer spend and order line amounts. | `O(n)` over `n` values. |
| Array filtering/search | Filters room status, booking status, remaining quantities, and in-memory grouped results. | Usually `O(n)` per pass; repeated passes are separate linear scans. |
| Date arithmetic and timezone conversion | Calculates stay nights, report boundaries and IST display timestamps. | Constant work per date/transaction; processing a list of `n` timestamps is `O(n)`. |

### Room Availability Logic (Conceptual)

1. Query rooms belonging to the requested hotel/type.
2. Query bookings that overlap the requested stay window according to the controller's filters.
3. Place booked room IDs into a `Set` for fast membership checks.
4. Keep rooms whose IDs are not in that set and return the candidates.

The actual database overlap predicates and active-status filters are defined in the relevant controllers. Availability is therefore implemented as a combination of Supabase filtering and JavaScript collection processing, not as a standalone scheduling algorithm.

## Utilities and Generated Documents

- `backend/src/utils/invoiceGenerator.js` provides `calculateGST`, `generateFoodBillPDF` and `generateInvoicePDF`; it registers Handlebars formatting helpers, compiles room and food bill templates, and uses Puppeteer to return PDF bytes.
- `backend/src/utils/templates/invoice.html` is the room invoice template.
- `backend/src/utils/templates/foodBill.html` is the food bill template.
- `backend/src/utils/cloudinary.js` uploads hotel logos and adds the Cloudinary URL to the request before registration/profile handlers run.
- `backend/src/utils/storage.js` contains a Supabase Storage upload helper for hotel logos; it is separate from the Cloudinary flow.
- `frontend/src/utils/invoiceUtils.js` contains browser-side invoice printing and PDF download helpers.
- `frontend/src/utils/animations.js`, `frontend/src/utils/cn.js` and shared hooks support UI animation/class composition and reusable behavior.

## Implementation Notes and Limitations

- The active server only registers routes from `backend/src/routes/`. Files in the separate top-level `backend/controllers/` directory are not part of that server flow.
- The `GET /api/bookings/available-rooms` route is declared after `GET /api/bookings/:booking_id`. Because the parameter route can match the one-segment path first, this endpoint may be intercepted as a booking ID; route order should be corrected before relying on it.
- `frontend/src/hooks/useFoodOrder.js` calls `/api/food-orders/order...` paths that are not declared in `foodOrderRoutes.js`; the active routes use `/create`, `/:orderId`, and `/booking/:booking_id/details` patterns. Verify this hook before using it as the source of the supported API contract.
- The booking UI includes a `DELETE /api/bookings/:bookingId` request, while the backend exposes cancellation as `PUT /api/bookings/:booking_id/cancel`; these are not the same endpoint.
- Most screens build API URLs as `${VITE_API_URL}/api/...`, while `frontend/src/utils/constants.js` has a fallback that already ends in `/api`. Keep the configured base URL convention consistent with the calling module.
- The super-admin dashboard route is guarded in the frontend, but the backend's `/api/super-admin` route group currently exposes login only. Other super-admin APIs are not defined in that route file.
- Staff permission enforcement is visible in frontend routing. Backend route middleware generally verifies authentication and ownership, but does not apply a common permission-key check to every controller.
- `backend/src/middleware/fileUpload.js` and `backend/src/utils/storage.js` are present as alternate upload utilities; the current registration and hotel-details routes use the Cloudinary upload handler.
- Data writes spanning bookings, rooms, guests and payments are performed as multiple Supabase client requests. The source comments refer to transactions in places, but no explicit database transaction/RPC wrapper is shown in this repository.

## Future Improvements

- Add database migrations/schema documentation and seed data for repeatable setup.
- Add API and component tests, especially for room-date overlap, booking cancellation/refunds, partial payments and staff ownership.
- Normalize status values and centralize route/request types to reduce frontend/backend contract drift.
- Reorder static booking routes before `/:booking_id` and align frontend calls with the registered API endpoints.
- Add explicit database transactions or server-side RPC functions for multi-table booking changes.
- Add production CORS origins, request validation and deployment-specific environment instructions.

## Conclusion

The project combines hotel operations, guest management, food service and cashier reporting in one web application. Its modular Express routes/controllers and React pages cover the core day-to-day workflows, while Supabase, Cloudinary and Puppeteer provide data, image storage and invoice-generation services. The implementation notes above distinguish currently wired features from legacy or mismatched code paths so that the project can be evaluated and extended accurately.
