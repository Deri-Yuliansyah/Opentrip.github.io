# App Flow Document for Open Trip Website

## Onboarding and Sign-In/Sign-Up
When a new visitor arrives, they land on a public homepage that displays featured open trips and a language selector for Bahasa Indonesia and English. The header offers options to sign up or sign in with a simple click. A new user clicks the sign-up link which opens a registration page where they provide their name, email, phone number, and password, and then submit the form. An email is sent via the email service to confirm their address and they must click the link in that message to activate their account. If they forget their password, they select the “Forgot Password” link which prompts them to enter their email address, receives a reset link, and then sets a new password. Once signed up and verified, the user signs in by entering their email and password or by clicking a social login button if enabled, and on a successful login, they are redirected to the appropriate dashboard. Signing out is done by clicking the profile avatar in the header and selecting the sign-out option, which returns the user to the homepage.

## Main Dashboard or Home Page
After signing in, participants land on a home screen that highlights a search bar at the top, featured upcoming trips, and filter options for destination, date, price, and duration. In the header, a notification bell displays in-app alerts and an icon shows unread email notifications. In the profile menu, users can switch between Bahasa Indonesia and English. The main navigation invokes different sections depending on the role. Participants see Trip Search, My Bookings, and My Profile. Trip Leaders see My Trips, Create Trip, Chat Groups, and Profile. Admins see Dashboard, Manage Users, Manage Trips, Reports, and Profile. From any of these sections, a click on the navigation menu or header elements takes the user into the detailed workflows of trip creation, booking, user management, or reporting.

## Detailed Feature Flows and Page Transitions
### Participant Trip Search and Booking Flow
A participant enters search criteria on the home screen and clicks “Search.” They are taken to a results page where they scroll through trips and apply filters. Tapping on a trip thumbnail opens the trip detail page which shows itinerary, organizer profile, map with Google Maps API, reviews, pricing, and available slots. To join, the participant clicks “Book Now,” fills out a registration form for themselves or group members, selects payment method—Stripe, PayPal, or manual bank transfer via m-banking—and confirms. The system displays a payment page with secure gateway integration. After successful payment, a webhook updates the booking status to confirmed, an email invoice is sent and an in-app notification appears. The participant is then redirected to a chat page for that trip, where they can upload identification documents and discuss itinerary details.

### Trip Leader Trip Management Flow
A Trip Leader signs in and clicks “Create Trip” in the sidebar. A form loads where they enter title, description, dates, destination coordinates via map picker, quota, price, upload photos, and build a daily itinerary. Upon submission, the trip appears immediately in their “My Trips” list and on the public trip catalogue. From “My Trips,” the leader can click on any listing to edit details or to delete the trip. When participants book, the leader views the booking list on the trip detail page and can open the group chat to message all participants. Before departure, the leader can export the attendee list for printing or use the calendar integration button to push dates to Google Calendar or iCal. After the trip ends, the leader views participant ratings and feedback and can respond if necessary.

### Admin Panel Flow
An Admin logs in and arrives at the Admin Dashboard which shows platform metrics such as total bookings, revenue, pending payouts, and new user sign-ups. In the “Manage Users” section, the admin searches or filters users and can view, suspend, or restore any account. In “Manage Trips,” the admin reviews all trip listings and has the power to remove or flag any trip. In “Reports,” the admin generates financial summaries which calculate commission earnings and administrative fees. The admin clicks a “Process Payouts” button that triggers Stripe Connect payout API calls. Notifications are sent via email and in-app alerts to Trip Leaders when their funds are disbursed.

## Settings and Account Management
Users access account settings by clicking their avatar in the header and selecting “Settings.” On the Profile page, they can update their name, phone number, avatar, language preference, and password. In the Notifications tab, they choose to enable or disable email confirmations and in-app reminders. Participants see a Payments section where they can add or remove saved payment methods and view past transactions. Trip Leaders and Admins have an additional Billing page where they set up payout account details for Stripe Connect and review commission rates. After saving any changes, the user clicks “Back to Dashboard” in the side menu to return to the main flow.

## Error States and Alternate Paths
If a user enters invalid credentials on sign-in, an error banner displays, and they are prompted to retry or reset their password. During registration, missing required fields trigger inline validation messages next to each field. If network connectivity is lost mid–booking or payment, the system shows an offline error page with a retry button which attempts to reconnect. If the payment gateway returns failure, the user sees a payment failed page with options to retry payment or choose another method. A participant who attempts to book a trip that has reached full capacity sees a “Sold Out” overlay and is offered to join a waitlist. Trip Leaders who try to access admin-only routes are redirected to an unauthorized error page. In all cases, the error pages provide clear instructions for recovery or to contact support.

## Conclusion and Overall App Journey
From the moment a traveler discovers the website, they can explore open trips, sign up, and complete a booking with confidence using secure payment options. Participants receive timely email confirmations and in-app notifications about schedule changes and reminders. Trip Leaders enjoy a straightforward interface for creating and managing trip listings and can engage participants through integrated group chats and calendar exports. Administrators maintain oversight through a powerful dashboard that handles user management, trip monitoring, financial reporting, and payouts. Together, these flows ensure that every user—from a first-time solo backpacker to a seasoned trip organizer—reaches their goal of planning, booking, managing, and reviewing open trips in one seamless platform.

```plaintext
             +-------------+        +---------+        +-----------+
             | Landing Page| -----> | Sign Up | -----> | Email OTP |
             +-------------+        +---------+        +-----------+
                    |                     |                  |
                    v                     v                  v
               +---------+             +------+           +---------+
               | Sign In | <--------   | Reset|           | Main    |
               +---------+             |Password|         |Dashboard|
                    |                  +------+           +---------+
                    v                     |                  |
          +------------------+            v                  v
          | Select User Role |        +----------+       +------+ 
          +------------------+        | New Trip |<---- | Trips|
                    |                 +----------+       +------+ 
                    v                      |               |
             +------------+                v               v
             | Participant| ---> Trip List ---> Trip Detail---> Booking Flow 
             +------------+                                    |
                    |                                         v
                    v                                    Payment Success
          +------------------+                               |
          | Group Chat &     | <------------------------------+
          | Notifications    |
          +------------------+
