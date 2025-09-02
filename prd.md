
## **Product Requirements Document (PRD): Site CashFlow TrackerPro**

### **Goals and Background Context**
This PRD serves as the guiding document for the development of the **Site CashFlow TrackerPro**, a comprehensive cash management system. Its primary goal is to address the inefficiencies and inaccuracies of manual financial tracking on field-based projects. The application will enable real-time, accurate financial reporting, allowing for proactive financial oversight.

* **Goals**: Provide a reliable and accurate system for managing cash flow, with a focus on user adoption, data accuracy, and reporting efficiency.
* **Background Context**: Current manual processes on project sites lead to discrepancies and delayed financial reporting. The solution aims to streamline the tracking of expenses and receipts to provide a single source of truth for all financial activity.
* **Change Log**:
    | Date | Version | Description | Author |
    | :--- | :--- | :--- | :--- |
    | 2025-09-01 | 1.0 | Initial PRD creation based on Project Brief. | John (PM) |

---

## **Requirements**

### **Functional Requirements (FR)**
1.  The system shall allow administrators to create and manage **Project Managers** with an ID, name, contact, email, and status.
2.  The system shall allow administrators to create and manage **project Sites** with an ID, name, location, and associated Project Manager ID.
3.  The system shall support the creation and management of **Product/Expense Categories** with an ID, name, and unit.
4.  The system shall allow **Project Managers** to enter new **expense records**. Each expense entry must include a system-generated **Expense ID**, an associated **Site ID** (Foreign Key to the Sites table), the **Project Manager ID** (Foreign Key to the Project Managers table), the **Product/Expense Category ID** (Foreign Key to the Product/Expense Categories table), the **Expense Date**, the **Amount** of the expense, and a **Description** of the expense.
5.  The system shall allow **Project Managers** to upload and attach documents (images, PDFs) to an expense record.
6.  The system shall allow **Project Managers** to enter new **cash receipt records**, including a Receipt ID, Project Manager ID, Site ID, Date, Amount, Reference Number, and Remarks.
7.  The system shall provide **Supervisors/Administrators** with a real-time, read-only view of all cash flow data, including expenses and receipts, filtered by Project Manager and/or Site.

### **Non-Functional Requirements (NFR)**
1.  The application must be deployed on a **cloud platform** to ensure accessibility and support multiple users simultaneously for live operation.
2.  The system must maintain data integrity and consistency across all linked financial records.
3.  The application should have a high level of performance, with fast load times and efficient data entry.

---

## **User Interface Design Goals**

This section outlines the conceptual vision for the user experience and interface, serving as a guide for both design and development. The design will prioritize clarity, efficiency, and data-driven insights.

* **Overall UX Vision**: The core UX vision is to create a seamless experience for Project Managers in the field. The interface should be intuitive enough for a user with minimal technical experience to quickly and accurately input data. For Supervisors, the focus is on a clean dashboard that provides all necessary information at a glance without being cluttered.
* **Key Interaction Paradigms**: The design will prioritize the mobile experience for Project Managers, with streamlined forms and large tap targets. Data will be presented in a hierarchical fashion, allowing for a high-level overview with the ability to drill down into specific details.
* **Core Screens and Views**:
    * Login/Authentication Screen
    * Project Manager Dashboard
    * Expense & Receipt Entry Forms
    * Supervisor Dashboard

---

## **Technical Assumptions**

This section documents the initial technical decisions and assumptions that will serve as constraints for the Architect.

* **Repository Structure**: Monorepo. This keeps all related code in a single repository, making it easier to manage shared code and simplify dependency management.
* **Service Architecture**: Monolith. For an MVP, a monolithic architecture simplifies development, deployment, and testing.
* **Testing Requirements**: Full Testing Pyramid. This includes unit, integration, and End-to-End (E2E) tests to ensure high quality from the beginning.
* **Framework**: Next.js. A unified full-stack framework that handles both the front end and back end.
* **Platform**: Google Cloud Platform (GCP). The application will be deployed to a cloud platform for live operation.

---

## **Epic List**

* **Epic 1: Foundation & Core Infrastructure**: Establish the foundational project infrastructure, set up the database and core services, and create the basic authentication system.
* **Epic 2: Core Data Management**: Implement the core data models for Project Managers, Sites, and Product/Expense Categories.
* **Epic 3: Expense & Receipt Tracking**: Develop the primary user workflows for entering and managing expense and cash receipt records, including the ability to upload documents.
* **Epic 4: Supervisor Reporting & Analytics**: Build the read-only dashboard and reporting views for supervisors to monitor financial activity.

---

## **Epic Details**

### **Epic 1: Foundation & Core Infrastructure**

* **Epic Goal**: Establish the foundational project infrastructure, set up the database and core services, and create the basic authentication system to enable user logins.

    * **Story 1.1: Project Initialization & Database Setup**:
        * **As a** developer, **I want** to set up the project repository and database, **so that** we have a stable environment to build upon.
        * **Acceptance Criteria**:
            1.  The project repository is created with a basic file structure.
            2.  The database is set up and accessible.
            3.  A simple "health check" API endpoint is created to verify the application is running and can connect to the database.

    * **Story 1.2: User Authentication & Login**:
        * **As a** Project Manager, **I want** to securely log in to the application, **so that** I can access my project data.
        * **Acceptance Criteria**:
            1.  A user login page is implemented.
            2.  The backend API supports user authentication.
            3.  A secure token is generated upon successful login.

### **Epic 2: Core Data Management**

* **Epic Goal**: Implement the core data models for Project Managers, Sites, and Product/Expense Categories, including their creation, retrieval, and management.

    * **Story 2.1: Project Manager Management**:
        * **As an** administrator, **I want** to create, view, and update Project Manager details, **so that** I can track who is responsible for each site.
        * **Acceptance Criteria**:
            1.  The system can store Project Manager information.
            2.  An API endpoint allows for the creation of new Project Manager records.
            3.  API endpoints allow for the retrieval and updating of existing Project Manager records.

    * **Story 2.2: Site Management**:
        * **As an** administrator, **I want** to create, view, and update project site details, **so that** I can track financial activity by location.
        * **Acceptance Criteria**:
            1.  The system can store project site information.
            2.  An API endpoint allows for the creation of new site records, with a foreign key to the Project Manager ID.
            3.  API endpoints allow for the retrieval and updating of existing site records.

    * **Story 2.3: Product/Expense Category Management**:
        * **As an** administrator, **I want** to create, view, and update expense categories, **so that** Project Managers can properly categorize their expenses.
        * **Acceptance Criteria**:
            1.  The system can store Product/Expense Category information.
            2.  An API endpoint allows for the creation of new expense category records.
            3.  API endpoints allow for the retrieval and updating of existing expense category records.

### **Epic 3: Expense & Receipt Tracking**

* **Epic Goal**: Develop the primary user workflows for entering and managing expense and cash receipt records, including the ability to upload and attach documents.

    * **Story 3.1: Expense Entry Core Functionality**:
        * **As a** Project Manager, **I want** to enter a new expense record with all the required details, **so that** I can accurately track money spent on my site.
        * **Acceptance Criteria**:
            1.  A form is available for Project Managers to enter new expense records.
            2.  The form includes fields for all necessary data points.
            3.  All entries are properly validated before submission.

    * **Story 3.2: Bill/Receipt Attachment**:
        * **As a** Project Manager, **I want** to attach a document (image or PDF) to an expense record, **so that** I have a digital record of the bill.
        * **Acceptance Criteria**:
            1.  The expense entry form includes a field for uploading documents.
            2.  The system can accept and store image and PDF files.
            3.  The uploaded file is linked to the corresponding expense record.

    * **Story 3.3: Cash Receipt Entry**:
        * **As a** Project Manager, **I want** to record a new cash receipt, **so that** I can track all money received on my site.
        * **Acceptance Criteria**:
            1.  A separate form is available for Project Managers to enter cash receipt records.
            2.  The form includes fields for Receipt ID, Project Manager ID, Site ID, Date, Amount, Reference Number, and Remarks.
            3.  All entries are properly validated before submission.

### **Epic 4: Supervisor Reporting & Analytics**

* **Epic Goal**: Build the read-only dashboard and reporting views that allow supervisors to monitor cash flow and financial activity across all sites in real time.

    * **Story 4.1: Supervisor Dashboard**:
        * **As a** supervisor, **I want** a dashboard that provides a high-level overview of cash flow across all sites, **so that** I can quickly monitor financial health.
        * **Acceptance Criteria**:
            1.  The dashboard displays a summary of total expenses and receipts.
            2.  The data is presented in a read-only format.
            3.  The dashboard includes a filter to view data by Project Manager or Site.

    * **Story 4.2: Expense & Receipt Details View**:
        * **As a** supervisor, **I want** to see the detailed list of expenses and receipts for a specific site or Project Manager, **so that** I can review individual transactions.
        * **Acceptance Criteria**:
            1.  The dashboard includes links or buttons to navigate to a detailed view.
            2.  The detailed view displays a list of all expenses and receipts.
            3.  Each transaction in the list includes a link to the attached bill or receipt document.
