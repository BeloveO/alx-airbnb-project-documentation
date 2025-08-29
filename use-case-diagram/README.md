# Use Case Diagram of the Features and Functionalities of the Backend system

## 1. Diagram
**This diagram visualizes the primary interactions within the Airbnb Clone backend system.**

<img width="1213" height="861" alt="Untitled Diagram drawio-8" src="https://github.com/user-attachments/assets/bd07eee8-8813-4367-94cd-ab16fd302ac0" />

### Actors

Actors are the entities that interact with the system.

*   **Primary Actors:**
    *   **Guest:** A user who searches for, books, and reviews properties.
    *   **Host:** A user who lists and manages properties and their bookings.
    *   **Admin:** A privileged user who oversees the entire platform.
*   **Secondary (System) Actors:**
    *   **Payment Gateway:** An external service (like Stripe or PayPal) that processes financial transactions.
    *   **Email Service:** An external service (like SendGrid) that sends notifications.

## 2. Use Cases (System Functionalities)

**Common User Use Cases (for Guests & Hosts):**
*   **Register Account:** Create a new user profile.
*   **Log In:** Authenticate and start a session.
*   **Manage Profile:** Update personal information.

**Guest-Specific Use Cases:**
*   **Search for Properties:** Find available listings using filters.
*   **Book Property:** Initiate and confirm a reservation for a property.
*   **Manage My Bookings:** View booking history, status, and cancel bookings.
*   **Write Review:** Leave feedback for a property after a completed stay.

**Host-Specific Use Cases:**
*   **Manage Listings:** Create, edit, delete, and add photos to property listings.
*   **Manage Property Bookings:** View and manage incoming booking requests (confirm/decline).
*   **Respond to Review:** Write a public reply to a guest's review.

**Admin-Specific Use Cases:**
*   **Manage Users:** Oversee and manage all user accounts.
*   **Manage All Listings:** Moderate and manage all properties on the platform.
*   **Oversee Bookings:** View and manage all bookings system-wide.

#### Key Relationships

*   **Association (Solid Line):** Shows that an actor interacts with a use case (e.g., `Guest` -> `Book Property`).
*   **`<<include>>` (Dashed Arrow):** Represents functionality that is *required* by another use case.
    *   `Book Property` **includes** `Make Payment` (You cannot book without paying).
    *   Several actions (like `Register Account` and `Book Property`) **include** `Send Notification` to alert the user.
*   **`<<extend>>` (Dashed Arrow):** Represents *optional* functionality that can extend a use case.
    *   `Write Review` **extends** `Manage My Bookings` (A guest can optionally write a review after their booking is completed).
