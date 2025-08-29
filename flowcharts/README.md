# Flowchart of Booking and Payment process

This flowchart provides a detailed, step-by-step visualization of the entire property booking process, from initial search to final confirmation. It merges a comprehensive user journey with a clear, structured layout.

## Flowchart Structure:

The diagram is organized into three **swimlanes** to show who is responsible for each action:

1.  **Guest:** Represents actions performed by the user (e.g., searching, clicking "Reserve," submitting payment).
2.  **Airbnb Clone Backend System:** Represents all automated logic, calculations, and database interactions handled by your server.
3.  **Payment Gateway:** Represents the external, third-party service (like Stripe) that securely processes the financial transaction.

## The Step-by-Step Workflow:

The process can be broken down into four main phases:

1.  **Initiation and Validation (Steps 1-5):**
    *   A `Guest` finds a property and clicks "Reserve."
    *   The `System` first checks if the user is logged in. If not, it prompts them to do so.
    *   Next, the `System` checks the **Bookings DB** to ensure the property is available for the requested dates. If it's not available, the process ends in failure.

2.  **Pricing and Payment (Steps 6-8):**
    *   If available, the `System` calculates the total price by querying the **Properties DB**.
    *   The `Guest` is shown the total and submits their payment details.
    *   The `System` securely forwards this information to the external **Payment Gateway** to process the transaction.

3.  **Confirmation and Record Keeping (Step 9):**
    *   If the payment is successful, the `System` performs the critical final actions:
        *   Creates a "confirmed" record in the **Bookings DB**.
        *   Creates a transaction record in the `Payments DB`.
        *   Updates the property's availability to prevent double-booking.

4.  **Notification (Step 10):**
    *   Finally, the `System` triggers confirmation notifications to be sent to both the `Guest` and the `Host`, completing the process successfully.
