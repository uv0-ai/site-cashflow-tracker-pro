The primary objective is to create an application for **cash management** with a focus on tracking expenses and receipts. The system should allow for the management of Project Managers and Sites, as well as detailed expense entries.



Key functional requirements include:

*   **Cash Management Core:** The system should track money that is "credited" and expenses that are "debited".

*   **User and Site Management:** Ability to manage Project Managers and project Sites. This includes creating and updating details for each Project Manager (ID, name, contact, email, status) and Site (ID, name, location, associated Project Manager ID, status).

*   **Product Expense Entry:** This is a crucial feature, allowing for the detailed recording of expenses. Each expense entry must include an Expense ID, the associated Site ID (Foreign Key), Project Manager ID (Foreign Key), Product ID (Foreign Key), Expense Date, Amount, and Description.

*   **Bill Attachment:** The system must support attaching relevant documents like PDF or image files (e.g., bill copies) to expense entries.

*   **Product/Expense Category Management:** Products or expense categories should be definable, with fields for Product ID, Product Name, Unit, and Status.

*   **Cash Receipt Entry (Implied from ERD):** The system's broader cash management aspect also includes recording cash receipts, with details like Receipt ID, associated Project Manager ID (FK), Site ID (FK), Receipt Date, Amount, Reference Number, and Remarks.

*   **Deployment:** While the application can initially run locally for testing, for live operation with multiple supervisors, it will need to be **hosted on a cloud platform**.
