# 🖥️ JDE CNC — Deployment Server & Core Concepts

---

## 1. Deployment Server

### What is it?
A **dedicated Windows server** that acts as the **central hub** of JDE installation, packaging, and distribution. It is the **first server installed** in any JDE setup.

> Think of it as the **"post office"** of JDE — everything passes through it before reaching other servers.

### What Does It Do?

| Task                   | Description                                      |
| ---------------------- | ------------------------------------------------ |
| **Stores DEP9.2.0**    | Holds the master copy of all JDE objects         |
| **Package Building**   | Compiles and builds packages from source objects |
| **Package Deployment** | Pushes packages to Enterprise Server & clients   |
| **Patch Management**   | Receives and applies ESUs/ASUs from Oracle       |
| **Hosts Planner**      | Stores JDE Plans (installation metadata)         |
| **Software Updates**   | Manages JDE Tools and Application updates        |

### Key Facts
- Runs on **Windows OS only**
- Contains the **DEP 9.2.0 path code** (master source)
- Without it, **no package builds or updates** can happen
- Communicates with Enterprise Server, Web Server, and client machines

---

## 2. JDE Plan

### What is it?
A **blueprint/configuration file** created before JDE is installed or upgraded. It defines exactly *what* to install and *where*.

> Like an **architect's plan** before constructing a building.

### What a Plan Defines

| Item                   | Example                                 |
| ---------------------- | --------------------------------------- |
| Path Codes to create   | DV920, PY920, PD920                     |
| Environments to create | DV920, PY920, PD920, PS920              |
| Data Sources           | Which DB to connect to each environment |
| Packages               | Which packages to build                 |
| Languages              | English, local language packs           |

### Types of Plans

| Plan                  | When Used                                       |
| --------------------- | ----------------------------------------------- |
| **Installation Plan** | Fresh JDE install from scratch                  |
| **Upgrade Plan**      | Moving from old version to new (e.g. 9.1 → 9.2) |
| **Update Plan**       | Applying patches (ESU/ASU) to existing system   |

> Plans are managed on the **Deployment Server** via the Planner application (P9600).

---

## 3. Path Code

### What is it?
A **pointer to the physical location** where JDE application objects (forms, reports, code, business functions) are stored on the server.

> Like a **shelf label in a library** — tells JDE exactly where to find the right set of objects.

### Standard Path Codes in JDE 9.2

| Path Code    | Name        | Purpose                                |
| ------------ | ----------- | -------------------------------------- |
| **DEP9.2.0** | Deployment  | Master source on Deployment Server     |
| **DV920**    | Development | Developers build & modify objects here |
| **PY920**    | Prototype   | Testing & UAT objects                  |
| **PD920**    | Production  | Live objects used by end users         |
| **PS920**    | Pristine    | Oracle's original, unmodified baseline |

### Key Points
- Each path code = a **physical folder** on the server
- Developers work in **DV920**, users run from **PD920**
- **DEP9.2.0** is the master — all package builds pull from here
- Applying an Oracle patch (ESU) = updating **DEP9.2.0** first

---

## 4. Environment

### What is it?
A **complete working space** in JDE that combines:

```
Environment  =  Path Code  +  Database  +  User Access Settings
```

> Like a **floor in an office building** — same building, but different people, different data, different rules.

### Standard Environments in JDE 9.2

| Environment | Path Code | Database      | Used By       | Purpose                   |
| ----------- | --------- | ------------- | ------------- | ------------------------- |
| **DV920**   | DV920     | Dev DB        | Developers    | Build & configure         |
| **PY920**   | PY920     | Test DB       | Key Users, QA | Testing & UAT             |
| **PS920**   | PS920     | Pristine DB   | CNC Admins    | Oracle baseline reference |
| **PD920**   | PD920     | Production DB | All End Users | Live business operations  |

### What is DEP 9.2.0?

**DEP 9.2.0** is the **master path code** that lives on the Deployment Server.

- It is **NOT** an environment users log into
- It holds the **original source objects** for all environments
- Oracle patches (ESU/ASU) are applied here **first**
- All **package builds** pull objects from DEP 9.2.0
- If DEP 9.2.0 is corrupted → **no packages can be built** → critical issue

```
Oracle ESU/ASU Patch
        │
        ▼
   DEP 9.2.0  (Deployment Server)
        │
   Package Build
        │
        ▼
DV920 / PY920 / PD920  (Enterprise Server)
```

---

## 5. DB → PY → PS → PD (Promotion Path)

This is the **standard flow** that code and configuration follow — from development all the way to live production.

### The Four Stages

```
  DB (DV920)  →  PY (PY920)  →  PS (PS920)  →  PD (PD920)
   Build           Test          Compare          Go Live
```

### Stage-by-Stage Breakdown

#### 🔧 DB — Development (DV920)
- Developers **build, configure, and customize** here
- Uses **dummy/test data** — not real business data
- Unstable — frequent changes happening
- No real business impact if something breaks

#### 🧪 PY — Prototype / QA (PY920)
- **Key Users and Testers** validate the work done in DV
- **UAT (User Acceptance Testing)** happens here
- May use a **copy of production data** for realistic testing
- Only approved, tested objects move forward

#### 📋 PS — Pristine (PS920)
- Oracle's **clean, unmodified** JDE baseline
- **Never customized** — always kept vanilla
- Used to check: *"Is this a JDE bug or our customization issue?"*
- CNC Admins use it as a **reference** during troubleshooting and upgrades

#### 🏭 PD — Production (PD920)
- The **live system** with real business data
- All end users work here daily
- **No direct changes** ever made here — always promoted from PY
- Most critical environment — any issue = direct business impact
- Strictly access-controlled and regularly backed up

### Complete Flow Diagram

```
  DEVELOPER          KEY USER / QA        CNC ADMIN         END USER
      │                    │                  │                 │
   DB (DV920)              │                  │                 │
   Build & Config          │                  │                 │
      │                    │                  │                 │
      └── Package ──► PY (PY920)              │                 │
                      Test & UAT              │                 │
                          │                  │                 │
                          └── Approved ──► PS (PS920)          │
                                         Compare               │
                                             │                 │
                                             └── Deploy ──► PD (PD920)
                                                             Go Live! ──► │
```

### Why This Flow Matters
- Ensures **no untested code** reaches production
- PS acts as a **safety net** to identify bugs vs. custom issues
- Protects business data and operations at all times

---

## 🧠 Quick Revision

```
Deployment Server  =  Central hub for installs, packages & updates (Windows only)

JDE Plan           =  Blueprint for what to install and where
                      Types: Installation | Upgrade | Update

Path Code          =  Physical location of JDE objects on server
                      DEP9.2.0 (master) → DV920 → PY920 → PD920 → PS920

Environment        =  Path Code + Database + User Access
                      DV=Dev | PY=Test | PS=Pristine | PD=Production

Promotion Flow:    DB (Build) → PY (Test) → PS (Compare) → PD (Go Live)
```

---
*Short notes on JDE CNC — Deployment Server, Plan, Path Code, Environment & Promotion Flow*