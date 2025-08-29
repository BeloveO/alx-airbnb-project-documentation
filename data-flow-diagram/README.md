# Data Flow Diagram (DFD)

This Data Flow Diagram (DFD) provides a high-level visual map of how information moves through the Airbnb clone backend system. It illustrates where data comes from, how it is processed, where it is stored, and where it goes.

## Key Components:

*   **Actors/Entities (Rectangles & Stick Figures):** These are sources or destinations of data outside our system.
    *   **Primary Actors:** `Guest`, `Host`, `Admin`
    *   **External Systems:** `Payment Gateway`, `Notification Service`

*   **Processes (Circles):** These are the core functions that transform data. Each is numbered for reference (e.g., `1.0 Manage User Accounts`, `4.0 Manage Bookings`).

*   **Data Stores (Cylinders):** This is where the system's information is permanently stored. They represent the database tables (e.g., `Users DB`, `Properties DB`).

*   **Data Flows (Labeled Arrows):** These show the path and type of data moving between entities, processes, and data stores (e.g., `Booking Request`, `Search Results`).

## Example Data Flow: Booking a Property

To understand the diagram, follow a simple flow:

1.  **Input:** A `Guest` sends a **`Booking Request`** to the **`4.0 Manage Bookings`** process.
2.  **Processing & Storage:** The process checks the **`Bookings DB`** for availability, then sends a **`Payment Request`** to the **`5.0 Process Payments`** process.
3.  **External Interaction:** The `Process Payments` process sends **`Payment Details`** to the external **`Payment Gateway`**.
4.  **Output:** After the gateway confirms, a **`Payment Confirmation`** flows back. The `Manage Bookings` process updates the **`Bookings DB`** and sends a **`Booking Confirmation`** back to the `Guest`.
5.  **Side-Effect:** A **`Notification Trigger`** is sent to the **`7.0 Send Notifications`** process, which then sends data to the external `Notification Service`.
