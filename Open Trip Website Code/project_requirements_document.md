# Project Requirements Document (PRD)

## 1. Project Overview

We’re building an **Open Trip Website** to connect independent travelers—backpackers, solo adventurers, and small groups—with trusted local trip organizers. The platform aggregates various open-trip packages, lets users compare routes, prices, and ratings, and provides a seamless booking and payment flow. It solves the problem of scattered information and unreliable providers by offering a centralized marketplace with verified reviews, transparent fees, and real-time communication.

The key objectives are:
- Enable organizers (Trip Leaders) to create and manage trip listings easily.
- Allow travelers (Participants) to find, filter, book, and pay for trips securely.
- Offer admins a comprehensive dashboard to oversee operations, commissions, and payouts.

**Success Criteria**
- 80% of new users complete a trip booking within their first session.
- 95% uptime and page load times under 2 seconds on desktop and mobile.
- Secure handling of payments via Stripe Connect and PayPal with automated notifications and reconciliation.

## 2. In-Scope vs. Out-of-Scope

### In-Scope (Version 1)
- User Roles:
  - Admin: full platform management and reporting.
  - Trip Leader: CRUD for trips, participant monitoring, payout requests.
  - Participant: search, filter, register, pay, view dashboard.
- Multi-language support (English & Bahasa Indonesia) via i18n.
- Notification system: email confirmations and in-app alerts for booking status, schedule changes, reminders.
- Payment integration: Stripe Connect (primary), PayPal (secondary), support for bank/m-banking via webhooks.
- Core modules: Trip management, search & filter, booking flow, payment processing, user dashboards, reviews & ratings.
- Google Maps integration for destination display.
- Calendar export (Google Calendar/iCal) for trip dates.
- Basic group chat or forum within each trip for organizer-participant communication.

### Out-of-Scope (Phase 2+)
- SMS notifications and WhatsApp integration.
- Tiered or dynamic pricing based on group size.
- AI-driven trip recommendations.
- Progressive Web App offline mode.
- Mobile-native apps (iOS/Android).
- Advanced social features (user follow, external social sharing).
- Multi-currency support beyond IDR and USD.

## 3. User Flow

When a newcomer visits the site, they land on a public homepage showcasing featured trips and a search bar. They can browse or filter trips by destination, date, price, and duration. Upon selecting a trip, they see detailed information: itinerary, organizer profile, reviews, and a map. To book, they sign up or log in, fill out a registration form, choose payment method (Stripe, PayPal, bank transfer), and complete checkout. An email confirmation and in-app notification appear, and the trip is added to their personal dashboard.

Trip Leaders register and verify their account via email. They access a dashboard where they create new trip listings by entering details—dates, price, quota, photos, itinerary—and submit for admin approval. Once live, they monitor bookings and can message participants in the trip’s chat. Admins log in to an admin panel to manage users, approve or suspend trips, view financial reports, and trigger payouts via Stripe Connect. All roles experience a consistent header/navigation with language switcher, notification bell, and user profile menu.

## 4. Core Features

- **Authentication & Authorization**
  - Email/password signup, JWT sessions, role-based access (Admin, Trip Leader, Participant).
- **Trip Management**
  - Full CRUD (Create/Edit/Delete/View) for trips.
  - Photo uploads, itinerary editor, Google Maps destination picker.
- **Search & Filtering**
  - Filter by date, destination, price range, duration, rating.
- **Booking & Payment**
  - Registration form tied to booking.
  - Stripe Connect and PayPal integration with webhooks for status updates.
  - Bank/m-banking option recorded manually or via Midtrans API.
- **User Dashboards**
  - Participant: upcoming trips, past trips, payment status.
  - Trip Leader: active trips, booking list, payout history.
  - Admin: platform overview, financial summary, user management.
- **Notifications**
  - Email (SendGrid/Mailgun) for confirmations, invoices, reminders.
  - In-app alerts for real-time updates.
- **Chat/Forum**
  - Per-trip group chat for organizers and participants.
- **Reviews & Ratings**
  - Post-trip feedback system with star ratings and comments.
- **Calendar Integration**
  - Export to Google Calendar or iCal.
- **Multi-language (i18n)**
  - English and Bahasa Indonesia with switcher.

## 5. Tech Stack & Tools

- Frontend:
  - Next.js (React) with TypeScript, Tailwind CSS for UI.
  - i18next for localization.
  - Axios or React Query for data fetching.
- Backend:
  - Node.js (Express) with TypeScript.
  - MongoDB (Atlas) for document storage.
  - REST API with JWT authentication.
- Payments:
  - Stripe Connect (primary) and PayPal SDK.
  - Webhooks to update booking/payment statuses.
- Notifications:
  - SendGrid or Mailgun for emails.
  - Custom in-app notifications stored in MongoDB.
- Maps & Calendar:
  - Google Maps JavaScript API.
  - Google Calendar API / iCal generation.
- Chat:
  - WebSocket (Socket.io) or real-time collections in MongoDB.
- DevOps & Tools:
  - Vercel for frontend hosting.
  - Heroku/AWS EC2 for backend, MongoDB Atlas.
  - GitHub Actions for CI/CD.
  - IDE integrations: VS Code with ESLint, Prettier, Tailwind IntelliSense.

## 6. Non-Functional Requirements

- **Performance**: Initial page load <2s on 3G mobile, Core Web Vitals LCP <2.5s.
- **Scalability**: Support 1,000 concurrent users initially; design modular monolith for future microservices.
- **Security**:
  - HTTPS everywhere, secure cookies, JWT with expiration.
  - Data encryption at rest (MongoDB) and in transit.
  - PCI DSS compliance via Stripe; OWASP Top 10 mitigation.
- **Usability**: Mobile-first, responsive layout, accessible (WCAG 2.1 AA).
- **Reliability**: 99.5% uptime, automatic retries for failed payment webhooks.
- **Maintainability**: 90% code coverage in unit/integration tests.

## 7. Constraints & Assumptions

- Stripe Connect and PayPal must be available in target regions.
- Google Maps API key and billing account are set up.
- Users will have modern browsers with JS enabled.
- SMS/WhatsApp notifications deferred to later phases.
- Bank/m-banking manual verification if Midtrans not integrated initially.
- Hosting costs and plan limits on MongoDB Atlas.

## 8. Known Issues & Potential Pitfalls

- **API Rate Limits**: Google Maps and Stripe enforce quotas—implement caching and exponential backoff.
- **Time Zones**: Trip dates require careful handling of user locale vs. UTC; use Moment.js or date-fns with IANA zones.
- **Payment Disputes**: Dispute handling and refunds need clear admin workflows.
- **Chat Scaling**: Real-time chat could hit performance issues; consider moving to a dedicated service (e.g., Firebase) if usage spikes.
- **Localization Gaps**: Ensure all static and dynamic text keys are covered in both languages to avoid missing translations.
- **Data Consistency**: Race conditions in booking when quota is low—use atomic operations or transactions in MongoDB.

---

This document serves as the definitive guide for the AI model to build the Open Trip Website. It provides clarity on scope, user journeys, features, and technical constraints, ensuring there’s no guesswork in subsequent technical designs and development tasks.