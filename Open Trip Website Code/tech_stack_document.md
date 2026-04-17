# Tech Stack Document

## 1. Frontend Technologies

To build a fast, user-friendly, and SEO-friendly interface for travelers and organizers, we chose the following technologies:

- **Next.js (React-based Meta-Framework)**
  - Provides Server-Side Rendering (SSR) and Static Site Generation (SSG) for fast page loads and better SEO.
  - Includes built-in routing, image optimization, and API routes for light server‐side logic.
- **React with TypeScript**
  - Enforces type safety to catch errors at compile time and improve developer productivity.
  - Allows building reusable UI components for trip listings, forms, dashboards, and chat.
- **Tailwind CSS**
  - Utility-first styling for rapid, consistent, and responsive design.
  - Ensures mobile-first layouts and a cohesive look and feel across the site.
- **i18next**
  - Manages English and Bahasa Indonesia translations seamlessly.
  - Simplifies language switching in the UI.
- **Data Fetching & State**
  - **React Query** or **Axios** for fetching bookings, trip data, and real-time updates.
  - Client-side caching and background refetching for a snappy user experience.
- **Real-Time Chat**
  - **Socket.io-client** to connect to the backend WebSocket server for group chats in each trip.
- **Developer Tools**
  - **ESLint** and **Prettier** for code consistency.
  - **Tailwind IntelliSense** in VS Code to speed up styling.

These choices combine fast initial loads (SSG) with dynamic updates (SSR and client-side fetching), delivering a smooth, cross-device user experience.

## 2. Backend Technologies

The backend handles data storage, business logic, authentication, real-time messaging, and integrations:

- **Node.js (Express) with TypeScript**
  - Lightweight, event-driven server framework ideal for both REST endpoints and real-time features.
  - TypeScript support brings type safety to controllers, services, and middleware.
- **MongoDB Atlas**
  - Flexible, document-based storage for users, trips, bookings, payments, and reviews.
  - Atlas provides managed hosting, automated backups, and horizontal scaling when needed.
- **REST API & Authentication**
  - JWT-based authentication with role-based access control (Admin, Trip Leader, Participant).
  - Routes for trip CRUD, booking flows, user management, reviews, and notifications.
- **Real-Time Chat**
  - **Socket.io** on the server to power per-trip group chats and instant in-app notifications.
- **Payment Processing**
  - **Stripe Connect** for commission splitting, marketplace payouts, and secure PCI-compliant flows.
  - **PayPal SDK** as an alternative payment method.
  - Webhooks to keep booking/payment statuses in sync and trigger email/in-app notifications.
- **Email Service**
  - **SendGrid** or **Mailgun** to send booking confirmations, invoices, password resets, and reminders.
- **Calendar Export**
  - **Google Calendar API** and **iCal** generation so users can add trip dates to personal calendars.

Together, these backend components ensure reliable data handling, real-time communication, and secure payment workflows.

## 3. Infrastructure and Deployment

Our infrastructure decisions focus on reliability, scalability, and streamlined developer workflows:

- **Version Control**
  - **Git** hosted on **GitHub** for source code management and pull-request collaboration.
- **CI/CD Pipeline**
  - **GitHub Actions** to automatically run linting, tests, and deployments on every merge to main.
- **Frontend Hosting**
  - **Vercel** for zero-config deployments of the Next.js app, edge caching, and global CDN.
- **Backend Hosting**
  - **Heroku** or **AWS EC2/ECS** for hosting the Node.js server and Socket.io service.
  - Environment variables managed securely via platform dashboards.
- **Database Hosting**
  - **MongoDB Atlas** clusters with automated scaling and high availability.
- **Monitoring & Logging**
  - Application performance and error tracking via services like **Sentry** or **LogRocket** (optional).

This setup allows rapid iteration, automatic rollbacks on failure, and scaling to handle peaks in bookings and real-time chat usage.

## 4. Third-Party Integrations

We integrate specialized services to handle payments, notifications, maps, and calendars:

- **Payment Gateways**
  - **Stripe Connect** for marketplace commissions and direct payouts to organizers.
  - **PayPal** as an alternative gateway for users without cards.
  - Future support for local bank/m-banking via webhooks or Midtrans.
- **Mapping & Location**
  - **Google Maps JavaScript API** to display trip destinations, itinerary stops, and map pickers in the admin panel.
- **Calendar**
  - **Google Calendar API** and iCal file generation to let users add trip schedules to personal calendars.
- **Email**
  - **SendGrid** or **Mailgun** for transactional emails (confirmations, reminders, invoices).
- **Optional Messaging**
  - Future integration with **Twilio SMS** or **WhatsApp Business API** for urgent alerts.

By leveraging these services, we offload complex features to trusted providers, speeding development and improving reliability.

## 5. Security and Performance Considerations

### Security Measures

- Enforce **HTTPS** across frontend and backend with TLS certificates.
- Use **hashed & salted passwords** (bcrypt) and secure **JWT** tokens with expiration.
- Protect against common web vulnerabilities (OWASP Top 10) via input validation, rate limiting, and CORS policies.
- Leverage Stripe’s PCI-compliant checkout flows to avoid handling raw card data.
- Encrypt sensitive data at rest in MongoDB Atlas and in transit over the network.

### Performance Optimizations

- **Server-Side Rendering (SSR) & Static Generation (SSG)** in Next.js to speed up initial loads and improve SEO.
- Client-side caching with React Query to reduce redundant API calls.
- **Turbopack** (or Webpack) code splitting and tree shaking to minimize JavaScript bundle size.
- HTTP caching headers on static assets (images, CSS, scripts) served by Vercel.
- Load-tested to ensure 95% of pages render in under 2 seconds, even under moderate concurrency.
- **Socket.io** namespaces and rooms to isolate chat traffic per trip and reduce overhead.
- **Atomic operations** in MongoDB when updating quotas to prevent overselling slots.

## 6. Conclusion and Overall Tech Stack Summary

Our chosen stack aligns perfectly with the goals of the Open Trip platform:

- **Next.js + React + TypeScript + Tailwind CSS** deliver a fast, SEO-friendly, and responsive user interface that works in two languages.
- **Node.js (Express) + TypeScript + MongoDB Atlas** provide a scalable, type-safe backend for handling users, trips, bookings, payments, and real-time chat.
- **Stripe Connect** and **PayPal** integration ensure secure and transparent commission and payout flows.
- **Vercel**, **Heroku/AWS**, and **GitHub Actions** enable easy, reliable deployments and continuous delivery.
- **SendGrid/Mailgun**, **Google Maps API**, and **Google Calendar API** add polished, core features without reinventing the wheel.

Unique aspects of this stack include real-time group chat via Socket.io, multi-region performance optimizations in Next.js, and a marketplace payout model powered by Stripe Connect. Together, these technologies create a seamless, scalable, and secure platform for travelers and trip organizers alike.