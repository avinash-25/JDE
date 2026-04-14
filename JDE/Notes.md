## Server Manager

Think of the **JD Edwards (JDE) Server Manager** as the central command center or "dashboard" for a massive software ecosystem.

When a company runs JD Edwards EnterpriseOne, the software isn't just sitting on one computer. It is usually spread across many different servers—some handle web traffic, some process complex business logic, and others manage the database.

Instead of a system administrator logging into each of these machines individually to start programs, check memory, or read error logs, they use the Server Manager to control everything from one web browser.

Here is a breakdown of how it works and why it is used.

### The Three Main Components
To understand Server Manager, you need to know its three basic parts:

1. **The Console (The Brain):** This is the web-based user interface. It’s the application you actually log into to see your entire JDE landscape. It stores all the configuration data for your servers.
2.  **The Agents (The Minions):** An agent is a small, lightweight program installed on *every single physical or virtual machine* that hosts a JDE component. The agent's only job is to communicate with the Console. It takes orders from the Console (e.g., "Restart the server") and sends data back (e.g., "CPU usage is at 80%").
3.  **The Managed Instances (The Workers):** These are the actual JD Edwards server components doing the heavy lifting. Examples include:
    * **Enterprise Servers:** Where the core business logic (C/C++ business functions) runs.
    * **HTML Web Servers (JAS Servers):** The Java-based middleware that translates the JDE logic into the web pages the end-users see.
    * **Database Servers:** Where the data lives.

### What does Server Manager actually do?
If you are managing the backend infrastructure, Server Manager gives you several "superpowers":

* **Centralized Configuration:** Historically, configuring JDE meant hunting down text files (like `jde.ini` or `jas.ini`) on different servers and editing them manually. Server Manager gives you a clean web interface to change these settings safely, tracking who made the change and when.
* **Lifecycle Management (Start/Stop):** You can start, stop, or restart any server instance across your entire network with the click of a button. You can also use it to install (provision) new server instances.
* **Real-Time Monitoring:** It allows you to monitor the health of your system. You can see how much memory a Java Application Server is consuming, check database connection pools, or see exactly how many users are currently logged into the system.
* **Log Viewing:** If an application crashes or a user reports an error, you don't need to remotely connect to the physical server to read the logs. You can search, view, and download the log files directly through the Server Manager interface.

### Summary
In short, JDE Server Manager turns a chaotic network of separate servers into a single, manageable system. It is the mandatory tool used by CNCs (Configurable Network Computing administrators—the JD Edwards term for system admins) to keep the ERP system running smoothly.


## 1. jde.ini (The Backend Engine)

**What it is:** The master settings file for the JD Edwards backend. You will find it on the Enterprise Server, Deployment Server, and Development (Fat) Clients.

**How it works (Your daily tasks):**
* **Database Connection:** Tells the Enterprise Server exactly where your database is located and how to log into it.
* **Network Traffic:** Configures "JDENET", which is the specific way JDE servers talk to each other.
* **Process Management:** Controls how many "kernels" (background processes) spin up to handle security, business logic, and batch jobs (UBEs).
* **Troubleshooting:** If a background report or C-code fails, this is the file where you turn on "Debug Logging" to find out why.

---

## 2. jas.ini (The Web Frontend)

**What it is:**
The master settings file for the JD Edwards web interface. "JAS" stands for Java Application Server. You will find it on your Web Servers (like Oracle WebLogic or IBM WebSphere).

**How it works (Your daily tasks):**
* **Bridge to the Backend:** It points the web server to the Enterprise Server so web clicks actually do something.
* **Session Management:** Controls how long a user can be idle before the system logs them out (timeout settings).
* **Screen Controls:** Sets limits on user interface actions, such as the maximum number of rows a user is allowed to export to Excel at one time.
* **Web Troubleshooting:** Used to capture logs if users are getting red error pop-ups on their screens.

---

### Quick Comparison

| Feature             | `jde.ini`                                  | `jas.ini`                          |
| :------------------ | :----------------------------------------- | :--------------------------------- |
| **Location**        | Enterprise Server & Fat Clients            | Web Server (HTML tier)             |
| **Focus**           | Backend heavy lifting & Database           | Frontend browser & User experience |
| **Technology**      | C / C++                                    | Java                               |
| **Common CNC Task** | Turning on debug logs for a failed report. | Changing user idle timeout limits. |


- Kernal
- Logs
- Security Server
- Enterprise Server
- JDEPLAN
- DEP920
- Path code
- Environment
- OCM
- DV - PY - PD
- Data Source
- Fat Client
- Central objects
- control table
- web server
- Security server
-