# JD Edwards (JDE) CNC: Beginner's Guide to User Creation

Creating a user in JD Edwards (JDE) involves a specific sequence of steps across different applications. As a CNC (Configurable Network Computing) administrator, you use these tools to build the user's profile, secure their login, and give them the correct permissions.

---

## 1. P0092: User Profiles
**Purpose:** This is the very first step. You use `P0092` to create the actual user profile and define their personal display and regional preferences.

* **Name of application:** User Profiles. You access it by typing `P0092` into the Fast Path.
* **Click on Add button:** Opens the blank form to create a brand new user ID.
* **Address number:** Every employee usually has an Address Book number in JDE. Linking it here connects the login ID to the physical employee's record. You can leave it blank if they don't have an Address Book record yet.
* **Who's who search:** If an Address Book number is attached, this lets you search the contact details (like email or phone) or find out who their approver/manager is.
* **Batch job queue:** JDE runs reports in the background using "Job Queues". If left blank, the system assigns the default queue. As a CNC, you might assign them to a less-used queue so their heavy reports don't slow down the system for others.
* **Printer setup:** Determines the default printer where the user's physical reports and documents will print after running.
* **Date format:** Dictates how dates look on their screen (e.g., `MDY` for Month/Day/Year or `DME` for Day/Month/Year).
* **Decimal format character:** Determines if a period (`.`) or comma (`,`) is used for decimals, which changes based on geography (e.g., `1,000.50` vs. `1.000,50`).
* **Localization country code:** If the user is in a specific country (like France or Brazil), entering the country code here unlocks country-specific menus, languages, and tax features.
* **Universal time:** Sets the user's specific Time Zone.
* **Time format:** Chooses between a standard 12-hour clock (AM/PM) or a 24-hour military clock.
* **Daylight saving rules:** Tells the system whether to automatically adjust the user's time forward or backward for Daylight Saving Time.

---

## 2. P98OWSEC: User Security
**Purpose:** Once the profile is created, you use `P98OWSEC` to handle their sign-in security.

* **Setting the Password:** This is where you create their initial login password. *Note: In standard or older JDE setups, this password can only be a maximum of 10 characters.*
* **System User Mapping:** You map the JDE user to a database "System User." This is a critical CNC step because it gives the user permission to actually read and write data to the underlying database tables.
* **Enable/Disable:** You can temporarily disable an account here (e.g., if the user goes on leave or leaves the company) or enable a locked account.

---

## 3. P98LPSEC: Long Password Security
**Purpose:** This is the modern upgrade to `P98OWSEC`.

* **Why it exists:** Because 10-character passwords are no longer considered secure by modern IT standards, companies can activate the "Long Password" feature in JDE.
* **How it works:** Once Long Passwords are enabled globally, the old `P98OWSEC` application gets locked, and CNC administrators are forced to use `P98LPSEC` instead.
* **Key Feature:** It does the exact same job as `P98OWSEC` (passwords, enabling/disabling, system users) but allows for passwords up to **40 characters** and enforces stricter password policies (like requiring special characters, numbers, and uppercase letters).

---

## 4. P95921: Role Relationships
**Purpose:** Now that the user exists and has a password, they need permissions to see menus and do their job. You use `P95921` to assign "Roles."

* **Assigning Roles:** A role is a pre-packaged bundle of permissions (e.g., `AP_CLERK`, `HR_MANAGER`). You attach these roles to the user ID using this application.
* **Multiple Roles & Sequencing:** A single user can have multiple roles. `P95921` lets you sequence them to decide which role's rules take priority if there is a conflict.
* **Effective & Expiration Dates:** You can set a role to expire automatically. For example, if a user is temporarily covering for a manager, you can assign them the manager role but set it to expire exactly 30 days from today.