# JD Edwards Server Manager Guide: Start, Stop, and Restart

This document outlines the standard procedures for managing the JD Edwards (JDE) Server Manager, including the local Agents, the central Console, and the correct sequence for system-wide operations.

---

## 1. Steps to Restart the Server Manager

### A. Restarting the Server Manager Agent (Target Servers)
The Agent must be restarted on the specific target server if local instances or logs stop responding.

**Windows:**
1. Press `Windows Key + R`, type `services.msc`, and press Enter.
2. Locate the service named **JD Edwards EnterpriseOne Server Manager Agent**.
3. Right-click the service and select **Restart**.

**Linux/UNIX:**
1. Open a terminal and navigate to the agent's bin directory (e.g., `cd /u01/Oracle/SCD/agent/bin`).
2. Run the stop script: `./stopAgent`
3. Verify the process has stopped, then run the start script: `./startAgent`

### B. Restarting the Server Manager Console (Web Interface)
**Modern Environments (WebLogic):**
1. Log in to your WebLogic Admin Console (usually `http://<admin_server>:7001/console`).
2. Navigate to **Environment** > **Servers** and click the **Control** tab.
3. Check the box next to your Server Manager Managed Server (often `SMC_Server`).
4. Click **Shutdown**. Wait for the status to change to "Shutdown".
5. Check the box again and click **Start**.

**Standalone/Older Environments:**
1. Navigate to the console's bin directory (e.g., `<JDE_HOME>/bin`).
2. **Windows:** Run `restartManagementConsole.bat`
3. **Linux/UNIX:** Run `./restartManagementConsole.sh`

---

## 2. Steps to Stop the Server Manager

### A. Stopping the Agent / Console Services
* **Agent (Windows):** Open `services.msc` > Right-click **JD Edwards EnterpriseOne Server Manager Agent** > **Stop**.
* **Agent (Linux):** Navigate to agent `bin` directory > Run `./stopAgent`.
* **Console (WebLogic):** Select the SMC Server in the Control tab > Click **Shutdown**.
* **Console (Standalone Linux):** Navigate to console `bin` directory > Run `./stopManagementConsole.sh`.

### B. Full JDE System Shutdown Sequence (Top-Down)
When stopping the actual JDE architecture managed *by* the Server Manager, follow this top-down order to prevent users from accessing a system that is shutting down:
1. Verify no critical reports (UBEs) are running and users are logged out.
2. **Stop Web Servers (HTML/JAS):** Closes the front door to users.
3. **Stop AIS & BSSV Servers:** Stops integrations and mobile apps.
4. **Stop Enterprise Server:** Shuts down the core logic and network kernels.
5. **Stop Database:** (Optional, only if doing a full system/hardware maintenance).

---

## 3. Steps to Start the Server Manager

### A. Starting the Agent / Console Services
* **Agent (Windows):** Open `services.msc` > Right-click **JD Edwards EnterpriseOne Server Manager Agent** > **Start**.
* **Agent (Linux):** Navigate to agent `bin` directory > Run `./startAgent`.
* **Console (WebLogic):** Select the SMC Server in the Control tab > Click **Start**.
* **Console (Standalone Linux):** Navigate to console `bin` directory > Run `./startManagementConsole.sh`.

### B. Full JDE System Startup Sequence (Bottom-Up)
When bringing the JDE architecture back online, follow this bottom-up dependency order:
1. **Start Database:** Ensure the database listener is active.
2. **Start Enterprise Server:** The JDE logic must be running first. Wait for processes (`jdenet_n` / `jdenet_k`) to fully initialize.
3. **Start AIS & BSSV Servers:** Connects integration layers to the logic.
4. **Start Web Servers (HTML/JAS):** Opens the front door for users to log in.