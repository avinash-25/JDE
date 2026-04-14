# JD Edwards CNC Basics: Core Concepts

## Overview
JDE CNC (Configurable Network Computing) is special because it separates the software code from the physical servers and databases. It doesn't hardcode where things live. Instead, it uses a flexible mapping system. Here are the four core pieces that make this work.

---

## 1. Data Source (The "Address Book")
A Data Source tells the JD Edwards system exactly where to find a specific database. It does not contain data itself; it is just a pointer.

* **What it does:** It acts like a saved address in a GPS. It holds the server name, database name, and the login details needed to connect.
* **Simple Example:** A Data Source named `Business Data - PROD` tells the system, "Go to the Oracle database on Server A, look in the PROD folder, and use these passwords to get in."
* **Why it is useful:** If you buy a new physical server, you don't have to rewrite JDE code. You just update the Data Source to point to the new server address.

## 2. OCM - Object Configuration Manager (The "Traffic Cop")
OCM is the heart of JDE's flexible design. It is a giant rulebook that routes users to the correct Data Source.

* **How it works:** When a user clicks a button to open a table or run a report, JDE asks OCM where to go. OCM looks at three things to make a decision:
    1.  **Environment:** Which workspace is the user in? (e.g., Production or Test)
    2.  **User/Role:** Who is asking? (e.g., A regular user or a system admin)
    3.  **Object:** What are they looking for? (e.g., A specific table like F0101)
* **The Result:** Based on those three things, OCM points the user to the correct **Data Source**.
* **Why it is useful:** OCM lets you safely test things. You can copy your Production environment, name it "Test," and use OCM to make sure "Test" points to dummy data instead of your real business data.

## 3. DTA - Business Data (The "Everyday Work")
DTA stands for Data. In CNC terms, it specifically means **Business Data**. This is a specific database schema (pointed to by a Data Source) where the company's real work is saved.

* **What is stored here:** All daily transactions. This includes things like customer orders, employee records, paychecks, and inventory counts.
* **Common Names:** You will usually see it named after the environment it belongs to, like `PRODDTA` (Production Data) or `CRPDTA` (Test Data).

## 4. CTL - Control Tables (The "Rules of the Game")
CTL stands for Control. This is a separate database schema that holds the rules and setup settings for the JDE software.

* **What is stored here:** System configurations. For example, what the next automatic invoice number should be, drop-down menu lists (User Defined Codes), and how the user's navigation menus are set up.
* **Common Names:** Like DTA, these are named after the environment, such as `PRODCTL` or `CRPCTL`.
* **Why separate DTA and CTL?** Sometimes you want to test new software rules (CTL) without messing up your real transaction data (DTA). Keeping them in separate buckets allows the CNC administrator to mix and match them safely using OCM.

---

## Summary: The Restaurant Analogy
If JD Edwards were a large restaurant:
* **Data Source:** The physical address of the pantry or the office.
* **OCM:** The waiter who takes your order and knows exactly which kitchen to send the ticket to.
* **DTA (Business Data):** The actual food and ingredients that are consumed and restocked every day.
* **CTL (Control Tables):** The recipe book and menus that dictate *how* the food is made and *what* you are allowed to order.