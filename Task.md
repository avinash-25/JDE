# Topic to be completed

## 1. Architecture & Server Roles

- **Deployment Server:** The master repository for JDE software and code.
- **Enterprise Server:** Where the "logic" runs and the Kernels live.
- **HTML / Web Server:** The interface users interact with.
- **Specialized Servers:**
   * AIS Server (Adaptive Intelligence Inventory - for mobile/orchard).
   * BSSV Server (Business Services - for web service integrations).
* **Server Manager:** The management console used to monitor and configure all the above.

## 2. Software Management & Updates

* **JDE Plan:** The blueprint for installation or upgrades.
* **ESU (Electronic Software Update):** Minor fixes or single "bug" patches.
* **ASU (Application Software Update):** Larger functional updates.
* **Tools Release (TR):** The foundation/technical layer updates (the "engine" of JDE).

## 3. The Foundation: Environments & Path Codes

* **Path Codes:** The physical location of central objects (folders/specs).
* **Environments:** The logical definition (e.g., DV920, PY920, PD920).
* **The Promotion Path:** The lifecycle of code (Development → Prototype → Production).
* **DEP920:** The specific Deployment environment used for administrative tasks.

## 4. Middleware & Configuration (The "CNC Core")

* **OCM (Object Configuration Manager):** The "traffic cop" that tells JDE where to find data or run logic.
* **Data Sources:** Definitions of where databases reside (Server Map vs. System Map).
* **Security Server:** Handles user authentication and permissions.
* **Kernels:** Individual processes on the Enterprise server that handle specific tasks (Security, Workflow, etc.).

## 5. Database & Object Storage

* **Central Objects:** The actual code/specs of JDE objects (Report, Application, etc.).
* **Control Tables (CTL):** Tables containing Next Numbers, UDCs, and Task Views.
* **Data Tables (DTA):** Business data (Address Book, Ledger, etc.).
* **Object Storage:** How JDE distinguishes between code and data.

## 6. Troubleshooting & Maintenance

* **Logs:** (jdedebug.log, jde.log) Essential for diagnosing kernel or JAS failures.