# **Backend Requirements Specification: AirBnB Clone Project**

## **Introduction & Scope**

This document outlines the detailed technical and functional requirements for the core backend services of the vacation rental platform: **User Authentication**, **Property Management**, and the **Booking System**. These specifications serve as a blueprint for development, testing, and documentation, ensuring a secure, scalable, and reliable system.

## **System-Wide Technical Requirements**

*   **Architecture:** A microservices or modular monolith architecture is recommended to ensure scalability and separation of concerns.
*   **API Style:** All communication will be via a RESTful API.
*   **Data Format:** JSON will be the exclusive data format for all API requests and responses.
*   **Authentication:** JWT (JSON Web Tokens) will be used for securing endpoints. The token will be passed in the `Authorization` header as a Bearer token (`Authorization: Bearer <token>`).
*   **Database:** A relational database (e.g., PostgreSQL) is required to ensure data integrity through ACID transactions, especially for the booking system.
*   **Security:**
    *   All communication must be encrypted via HTTPS/SSL.
    *   Passwords must be securely hashed using a strong, one-way algorithm (e.g., bcrypt).
    *   Standard security headers (e.g., CORS, HSTS) must be implemented.

---

### **1. User Authentication & Management**

This service is responsible for user identity, authentication, authorization, and profile management.

#### **1.1. Functional Requirements**
| ID | Feature Description | Priority |
| :--- | :--- | :--- |
| **FR-AUTH-01** | **User Registration:** A new user (Guest or Host) must be able to create an account using their first name, last name, email, and a password. | High |
| **FR-AUTH-02** | **Email Verification:** A verification email with a secure, single-use token must be sent upon registration. Account functionality will be limited until the email is verified. | High |
| **FR-AUTH-03** | **User Login:** A registered user must be able to log in with their email and password to receive an access token (JWT) and a refresh token. | High |
| **FR-AUTH-04** | **Password Reset:** A user must be able to request a password reset via email, receiving a secure, time-limited link to set a new password. | Medium |
| **FR-AUTH-05** | **Profile Management:** An authenticated user must be able to view and update their own profile information. | High |
| **FR-AUTH-06** | **Role-Based Access Control (RBAC):** The system must distinguish between `guest`, `host`, and `admin` roles and enforce access controls based on these roles for all relevant endpoints. | High |

#### **1.2. Technical Specifications & API Endpoints**

| Endpoint | Method | Description | Auth Required | Request Body (Example) | Response (Success `201`/`200`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `/auth/register` | POST | Register a new user. | No | `{ "firstName": "John", "lastName": "Doe", "email": "user@mail.com", "password": "Pass123!", "role": "guest" }` | `201 Created`: `{ "message": "User created. Verification email sent.", "user": { "id": "...", "email": "..." } }` |
| `/auth/login` | POST | Authenticate a user. | No | `{ "email": "user@mail.com", "password": "Pass123!" }` | `200 OK`: `{ "user": { "id": "...", "role": "guest" }, "accessToken": "jwt...", "refreshToken": "jwt..." }` |
| `/auth/verify-email` | POST | Verify email with a token. | No | `{ "token": "verification_token" }` | `200 OK`: `{ "message": "Email verified successfully." }` |
| `/auth/forgot-password` | POST | Request a password reset. | No | `{ "email": "user@mail.com" }` | `200 OK`: `{ "message": "Password reset email sent." }` |
| `/auth/reset-password` | POST | Reset password with a token. | No | `{ "token": "reset_token", "newPassword": "NewPass123!" }` | `200 OK`: `{ "message": "Password reset successfully." }` |
| `/users/me` | GET | Get current user's profile. | Yes (Bearer JWT) | N/A | `200 OK`: `{ "id": "...", "email": "...", "firstName": "John", "lastName": "Doe" }` |
| `/users/me` | PUT | Update current user's profile. | Yes (Bearer JWT) | `{ "firstName": "Jane" }` | `200 OK`: `{ "message": "Profile updated", "user": { ... } }` |

#### **1.3. Validation Rules**
*   **Email:** Required, must be a valid email format, and must be unique in the database.
*   **Password:** Required on registration. Minimum 8 characters, containing at least one uppercase letter, one lowercase letter, one number, and one special character.
*   **Name Fields:** `firstName` and `lastName` are required, max 50 characters.
*   **JWT Token:** Must be validated for signature, expiration, and issuer on every authenticated request via middleware.

#### **1.4. Performance & Security Criteria**
*   **Latency:** Login and registration API responses should be **< 500ms** under normal load.
*   **Security:** Implement rate limiting on login endpoints (e.g., max 5 failed attempts per minute per IP) to prevent brute-force attacks.
*   **Uptime:** The authentication service must maintain 99.9% uptime.

---

### **2. Property Listings Management**

This service handles the creation, retrieval, updating, and deletion of property listings.

#### **2.1. Functional Requirements**
| ID | Feature Description | Priority |
| :--- | :--- | :--- |
| **FR-PROP-01** | **Create Listing:** A user with the `host` role must be able to create a new property listing with detailed information. | High |
| **FR-PROP-02** | **Read Listings:** Any user can view and filter a list of published properties. A host can view all their own listings (published and unpublished). | High |
| **FR-PROP-03** | **Update Listing:** A host must be able to update their own listings. An admin can update any listing. | High |
| **FR-PROP-04** | **Delete Listing:** A host can soft-delete their own listing (marking it as inactive). An admin can permanently delete any listing. | Medium |
| **FR-PROP-05** | **Photo Management:** A host must be able to upload, reorder, and delete photos for their listings. | High |
| **FR-PROP-06** | **Search & Filter:** Users must be able to search and filter listings by location, price range, number of guests, and other key amenities. | High |

#### **2.2. Core Technical Requirements**
*   **Ownership Authorization:** All `PUT`, `POST`, and `DELETE` endpoints related to a specific property must verify that the authenticated user is the owner of the property or has `admin` privileges.
*   **Image Handling:** The service will not store image blobs. It will handle URLs to images stored in a cloud object storage service (e.g., AWS S3, Google Cloud Storage) for scalability.
*   **Database Indexing:** Key filterable columns (e.g., `city`, `pricePerNight`, `maxGuests`) must be indexed in the database to ensure fast query performance.

#### **2.3. Technical Specifications & API Endpoints**

| Endpoint | Method | Description | Auth Required | Request Body / Query Params | Response (Success `200`/`201`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `/properties` | GET | Get a paginated list of published properties. | No | `?city=Malibu&minPrice=200&page=1&limit=10` | `200 OK`: `{ "data": [...], "pagination": { "currentPage": 1, ... } }` |
| `/properties/my-properties` | GET | Get all properties for the current host. | Yes (Host/Admin) | N/A | `200 OK`: `[ {property}, ... ]` |
| `/properties/{id}` | GET | Get a specific property by ID. | No | URL Param: `id` | `200 OK`: `{ "id": "prop456", "title": "Beach Villa", ... }` |
| `/properties` | POST | Create a new property listing. | Yes (Host) | `{ "title": "Beach Villa", "pricePerNight": 150, "address": { ... }, ... }` | `201 Created`: `{ "message": "Property created", "property": { ... } }` |
| `/properties/{id}` | PUT | Update a property listing. | Yes (Owner/Admin) | `{ "pricePerNight": 160 }` | `200 OK`: `{ "message": "Property updated", "property": { ... } }` |
| `/properties/{id}` | DELETE | Soft-delete a property. | Yes (Owner/Admin) | URL Param: `id` | `200 OK`: `{ "message": "Property deleted" }` |
| `/properties/{id}/photos` | POST | Upload a photo for a property. | Yes (Owner/Admin) | `multipart/form-data` (image file) | `201 Created`: `{ "photoUrl": "https://.../image.jpg" }` |

#### **2.4. Validation Rules**
*   **Title:** Required, 5-100 characters.
*   **Description:** Required, 50-1000 characters.
*   **pricePerNight:** Required, must be a positive number.
*   **maxGuests:** Required, must be a positive integer.
*   **Address:** Must be a structured object with required fields (e.g., street, city, country).
*   **File Uploads:** Must accept only `image/jpeg`, `image/png`, `image/webp`. Max file size 5MB.

#### **2.5. Performance Criteria**
*   **Latency:** Fetching a filtered list of properties (`GET /properties`) should respond in **< 1000ms**. Fetching a single property should be **< 300ms**.
*   **Throughput:** The system must support concurrent image uploads without significant performance degradation.

---

### **3. Booking System**

This service is the core transactional component, managing property reservations.

#### **3.1. Functional Requirements**
| ID | Feature Description | Priority |
| :--- | :--- | :--- |
| **FR-BOOK-01** | **Check Availability:** Users must be able to check if a property is available for a given date range without creating a booking. | High |
| **FR-BOOK-02** | **Create Booking:** An authenticated `guest` must be able to create a booking for an available property. | High |
| **FR-BOOK-03** | **View Bookings:** A user can view their own bookings. A host can view bookings for their properties. An admin can view all bookings. | High |
| **FR-BOOK-04** | **Booking Status:** The system must track booking statuses: `pending` (awaiting payment), `confirmed`, `cancelled`, `completed`. | High |
| **FR-BOOK-05** | **Cancel Booking:** A guest can cancel their own booking, with refunds processed according to the property's cancellation policy. | Medium |
| **FR-BOOK-06** | **Prevent Double-Booking:** Creating a booking must atomically block the dates for that property, making them unavailable for subsequent requests. | Critical |

#### **3.2. Core Technical Requirements**
*   **Atomic Transactions:** The process of checking availability and inserting a booking record must be executed within a single, atomic database transaction (e.g., `SERIALIZABLE` isolation level) to prevent race conditions.
*   **Concurrency Control:** The system must be thread-safe and handle high concurrency for booking attempts on the same property and dates. This can be achieved with database-level locking or optimistic concurrency control.
*   **Idempotency:** The booking creation endpoint should support an idempotency key to allow clients to safely retry requests without creating duplicate bookings in case of network failures.

#### **3.3. Technical Specifications & API Endpoints**

| Endpoint | Method | Description | Auth Required | Request Body / Query Params | Response (Success `200`/`201`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `/properties/{id}/availability` | GET | Check availability for a property. | No | `?startDate=2024-10-20&endDate=2024-10-25` | `200 OK`: `{ "isAvailable": true, "totalPrice": 1250.00 }` |
| `/bookings` | POST | Create a new booking. | Yes (Guest) | `{ "propertyId": "...", "startDate": "...", "endDate": "...", "numberOfGuests": 2 }` | `201 Created`: `{ "message": "Booking confirmed", "booking": { ... } }` |
| `/bookings/my-bookings` | GET | Get current user's bookings. | Yes (User) | `?status=upcoming` | `200 OK`: `[ {booking}, ... ]` |
| `/properties/{id}/bookings` | GET | Get bookings for a host's property. | Yes (Owner/Admin) | N/A | `200 OK`: `[ {booking}, ... ]` |
| `/bookings/{id}` | GET | Get a specific booking by ID. | Yes (User/Owner)* | URL Param: `id` | `200 OK`: `{ "id": "book789", "status": "confirmed", ... }` |
| `/bookings/{id}/cancel` | PATCH | Cancel a booking. | Yes (User/Owner)* | N/A | `200 OK`: `{ "message": "Booking cancelled", "refundAmount": 150.00 }` |

*\*Users can only access their own bookings. Hosts can only access bookings for their properties.*

#### **3.4. Validation Rules & Business Logic**
*   **Date Validation:**
    *   `startDate` must be before `endDate`. `startDate` cannot be in the past.
    *   The system must enforce property-specific rules like minimum/maximum stay duration.
*   **Availability Check:** The `POST /bookings` endpoint must perform a final, transactional check to ensure the dates have not been taken since the initial availability lookup. If unavailable, it must return a `409 Conflict` error.
*   **Price Calculation:** The backend must authoritatively calculate the total price: `(numberOfNights * pricePerNight) + fees`.
*   **Cancellation Policy:** Logic for calculating refund amounts must be encapsulated on the backend based on the property's policy and the time of cancellation.

#### **3.5. Performance Criteria**
*   **Latency:** The availability check (`GET`) should be **< 400ms**. The booking creation endpoint (`POST`) must respond in **< 1500ms**, including the database transaction.
*   **Reliability:** The booking system must guarantee zero double-bookings, even under high load (e.g., 100+ concurrent requests for the same property).
