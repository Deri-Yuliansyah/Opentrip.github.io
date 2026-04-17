# Backend Structure Document

This document explains the backend setup for the Open Trip Website in everyday language. It covers how the system is built, where it runs, how data is managed, and how everything works together securely and reliably.

## 1. Backend Architecture

- We use a **modular monolith** approach with Node.js and Express. This means all parts live in one codebase, but they are organized into clear modules (authentication, trips, bookings, payments, chat, notifications) for easier maintenance.
- Each module follows a **layered structure**:
  1. **Controllers** handle incoming HTTP requests.
  2. **Services** contain business logic (e.g., calculating fees, checking quotas).
  3. **Repositories** deal with database operations (CRUD on MongoDB).
  4. **Middlewares** manage authentication (JWT), error handling, logging, and internationalization (i18n).
- **Socket.io** powers real-time group chat and in-app notifications in a separate channel but within the same server process.
- We follow **RESTful design** for most operations, with clear routes and HTTP methods (GET, POST, PUT, DELETE). Webhook endpoints listen for payment status updates from Stripe and PayPal.

### How This Design Helps
- **Scalability**: Modules are loosely coupled, so we can extract one into a microservice later if needed (for example, chat or notifications).
- **Maintainability**: Clear separation of concerns makes it easier to update one part without breaking others.
- **Performance**: Express is lightweight and non-blocking. Socket.io handles real-time with minimal overhead.

## 2. Database Management

- We use **MongoDB Atlas** (NoSQL document store) for all main data:
  - User profiles, trips, bookings, payments, reviews, notifications, chat messages.
- Atlas gives us automatic **backups**, **replica sets** for high availability, and **sharding** options if data grows large.
- We define **indexes** on common query fields (e.g., trip dates, user email, booking status) to speed up searches.
- Data access is done through a thin repository layer, keeping database-specific code in one place and making it simple to swap or upgrade the database later.

## 3. Database Schema (NoSQL Collections)

Below is a human-readable overview of our main collections and their fields. References between collections use unique IDs.

### Users Collection
- _id: unique user ID
- name, email, phoneNumber
- passwordHash (encrypted password)
- role ("admin", "leader", "participant")
- languagePreference ("en", "id")
- createdAt, updatedAt

### Trips Collection
- _id: unique trip ID
- title, description
- destinationName, coordinates (latitude, longitude)
- startDate, endDate, durationDays
- pricePerPerson, participantQuota
- images (array of URLs)
- itinerary (array of day-by-day activities)
- leaderId (reference to Users)
- status ("draft", "published", "cancelled")
- createdAt, updatedAt

### Bookings Collection
- _id: unique booking ID
- tripId (reference to Trips)
- userId (reference to Users)
- numberOfParticipants
- totalPrice
- status ("pending", "confirmed", "cancelled")
- createdAt, updatedAt

### Payments Collection
- _id: unique payment ID
- bookingId (reference to Bookings)
- method ("stripe", "paypal", "bank")
- amount, currency
- status ("pending", "paid", "failed")
- transactionDate
- paymentGatewayId (gateway-provided reference)
- createdAt, updatedAt

### Reviews Collection
- _id: unique review ID
- tripId (reference to Trips)
- userId (reference to Users)
- rating (1–5 stars)
- comment
- createdAt

### Notifications Collection
- _id: unique notification ID
- userId (reference to Users)
- type ("email", "in-app")
- category ("booking", "reminder", "update")
- payload (detailed message data)
- read (boolean)
- createdAt

### ChatMessages Collection
- _id: message ID
- tripId (reference to Trips)
- senderId (reference to Users)
- messageText
- attachments (optional URLs)
- createdAt

## 4. API Design and Endpoints

We expose a **versioned REST API** (`/api/v1`) using Express routes. Key endpoints include:

### Authentication & Users
- POST `/api/v1/auth/register`: create new user
- POST `/api/v1/auth/login`: return JWT token
- POST `/api/v1/auth/forgot-password`: send reset link
- POST `/api/v1/auth/reset-password`: update password
- GET `/api/v1/users/me`: fetch current user profile
- PUT `/api/v1/users/me`: update profile settings

### Trips
- GET `/api/v1/trips`: list all published trips with filters (destination, date, price)
- GET `/api/v1/trips/:id`: get details of a single trip
- POST `/api/v1/trips`: (leader) create a new trip
- PUT `/api/v1/trips/:id`: (leader) edit trip details
- DELETE `/api/v1/trips/:id`: (leader) remove a trip

### Bookings & Payments
- POST `/api/v1/bookings`: participant registers and initiates payment
- GET `/api/v1/bookings/me`: list user’s bookings
- GET `/api/v1/bookings/:id`: booking details
- POST `/api/v1/webhooks/stripe`: receive Stripe events
- POST `/api/v1/webhooks/paypal`: receive PayPal events

### Reviews
- POST `/api/v1/reviews`: submit a post-trip review
- GET `/api/v1/trips/:id/reviews`: list reviews for a trip

### Notifications
- GET `/api/v1/notifications`: list in-app notifications
- PUT `/api/v1/notifications/:id/read`: mark as read

### Chat (WebSocket)
- namespace: `/chat`
- join room: `socket.emit('join', { tripId, userId })`
- send message: `socket.emit('message', { tripId, text })`
- receive messages: `socket.on('message', ...)`

## 5. Hosting Solutions

- **Backend** runs on **Heroku** (easy setup, built-in scaling) with Node.js dynos. We can switch to **AWS EC2/ECS** if we need more control.
- **Database** on **MongoDB Atlas**, a managed cloud database with built-in high availability and automatic backups.
- **Frontend** (Next.js) is hosted on **Vercel**, which provides edge caching and a global CDN for fast page loads.
- Environment variables (API keys, DB URIs, secrets) are stored securely in each platform’s dashboard.

## 6. Infrastructure Components

- **Load Balancer**: Heroku’s routing layer automatically balances requests across dynos. On AWS, we would use an Application Load Balancer.
- **CDN**: Vercel CDN for static assets (images, CSS, JS).
- **Caching**: In-memory caching with **Redis** (optional) for frequently accessed data like popular trips or rate limits.
- **Message Broker**: Socket.io handles real-time communication without a separate message queue at this stage.
- **CI/CD Pipeline**: **GitHub Actions** run tests, linting, and deploy backend and frontend on merge to `main`.
- **API Gateway**: Express middleware enforces rate limiting, CORS, and centralized logging.

## 7. Security Measures

- **HTTPS** enforced on all endpoints with TLS certificates (managed by Heroku and Vercel).
- **JWT** tokens for stateless authentication, stored in secure HTTP-only cookies or local storage.
- **Password Hashing**: bcrypt with salt to protect user passwords.
- **Role-Based Access Control**: only admins and leaders can access protected routes.
- **Input Validation & Sanitization**: via libraries like `express-validator` to prevent injection attacks.
- **Helmet** and **cors** middlewares** to set safe HTTP headers and control resources sharing.
- **Payment Security**: raw card data never touches our servers; Stripe and PayPal handle PCI compliance.
- **Encryption**: Sensitive data (passwords, tokens) is encrypted in transit (TLS) and at rest (Atlas encryption).

## 8. Monitoring and Maintenance

- **Error Tracking**: Sentry captures exceptions and performance issues in real time.
- **Logging**: Winston (or similar) logs important events and requests to stdout, aggregated by Heroku logs or an external service.
- **Performance Metrics**: Heroku Metrics or third-party (Datadog, New Relic) to monitor response times and resource usage.
- **Health Checks**: a simple `/health` endpoint returns status of the app and database connection.
- **Database Backups**: automated daily snapshots from MongoDB Atlas with point-in-time restores.
- **Dependencies**: regular audits and updates (npm audit) to avoid known vulnerabilities.

## 9. Conclusion and Overall Backend Summary

This backend is a modular, Express-based monolith using MongoDB Atlas for flexible data storage. It supports three user roles, multi-language content, secure payment flows via Stripe and PayPal, and real-time chat with Socket.io. Hosting on Heroku and Vercel ensures reliability and fast performance. Built-in security measures, monitoring tools, and a clear CI/CD pipeline keep the system safe, stable, and easy to maintain. Together, these components meet the project goals: a scalable, user-friendly platform for independent travelers and trip organizers.