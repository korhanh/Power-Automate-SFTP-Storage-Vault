# Zetaleap SFTP / FTP Storage Vault: Power Automate & Cloud Integration Gateway

> **Fast, Serverless, and Secure SFTP / FTP RESTful API Gateway**  
> *Transform legacy SFTP and FTP servers into clean REST APIs for Microsoft Power Automate, Power Apps, and Cloud Workflow Automation.*

---

## 📌 Overview & Purpose

Integrating external SFTP or FTP servers into **Microsoft Power Automate** and **Power Apps** has historically been a significant pain point for enterprise developers and automation consultants:

* **On-Premises Data Gateway (OPDG) Overhead:** Standard SFTP connectors frequently require setting up and maintaining dedicated Windows-based On-Premises Data Gateways.
* **Flow Timeouts & Payload Limits:** Directory listings and file downloads on legacy servers often exceed Power Automate's HTTP/connector timeout boundaries (120 seconds).
* **No Built-in Browser Inspection:** Troubleshooting requires third-party desktop utilities (such as FileZilla) because flows provide no visual way to browse directories or verify files before automation runs.
* **Complex Network & Security Setup:** Opening firewall ports and managing keys across corporate networks is prone to configuration drift.

**Zetaleap SFTP / FTP Storage Vault** solves these challenges by functioning as an edge-accelerated, serverless TCP bridge. It translates remote SFTP (Port 22) and FTP (Port 21) protocols into lightweight **RESTful JSON APIs**. 

With this gateway, Power Automate flows can list, stream, and download files using **a single standard HTTP GET action**—without any gateway installation or connection timeouts.

---

## 🏛️ Architecture Workflow

```mermaid
flowchart LR
    A[Remote SFTP / FTP Server] <-->|Serverless TCP Socket| B[Zetaleap Edge Gateway]
    B <-->|RESTful JSON / Binary Stream| C[Microsoft Power Automate]
    C -->|Store Files| D[SharePoint / OneDrive]
    C -->|Trigger Action| E[Power Apps / Dynamics 365]
    B <-->|Automated Cron Sync| F[(Zetaleap Storage Vault)]
```

---

## ✨ Key Features

* **⚡ Native RESTful Gateway:** Access remote SFTP (Password or SSH Key) and FTP endpoints over clean, standardized HTTPS endpoints.
* **🧭 Browser-Based Live Remote Explorer:** Inspect, navigate, and download files directly from your browser via native serverless TCP sockets without installing FTP desktop software.
* **⏱️ Automated Scheduled Background Sync (Cron):** Ingest remote files into **Zetaleap Storage Vault** on customizable recurring schedules (Hourly, Daily, Custom Cron) with optional Move/Cut-Paste mode.
* **🔑 Self-Service Personal Access Tokens (PAT):** Generate secure Bearer API tokens directly from your user profile with instant activation.
* **🗑️ Complete Endpoint Lifecycle Management:** Add, configure, test, and safely delete custom connection profiles directly from the web interface.
* **🛡️ Enterprise Role-Based Access Control:** Protect sensitive automated background cron tasks with administrator-only permissions.

---

## 🚀 Power Automate Integration Guide

### Scenario 1: List Remote SFTP Files in Power Automate

You no longer need specialized or premium SFTP connectors. Simply add a standard **HTTP** action to your Cloud Flow:

1. **Add Action:** `HTTP`
2. **Method:** `GET`
3. **URI:** 
   ```http
   https://zetaleap.com/api/sftp/rebex?path=/
   ```
4. **Headers:**
   ```http
   Authorization: Bearer zl_pat_3f9a7c8e2b1d40a5bc8e9102
   ```

#### Sample Response Payload:
```json
{
  "success": true,
  "target_path": "/",
  "files": [
    {
      "filename": "orders_2026_10.csv",
      "size": 154200,
      "is_dir": false,
      "modify_time": "2026-10-04T18:00:00Z"
    },
    {
      "filename": "readme.txt",
      "size": 405,
      "is_dir": false,
      "modify_time": "2026-10-01T12:00:00Z"
    },
    {
      "filename": "inbox",
      "size": 0,
      "is_dir": true,
      "modify_time": "2026-10-01T12:00:00Z"
    }
  ]
}
```

5. **Parse JSON Action Schema:**
```json
{
  "type": "object",
  "properties": {
    "success": { "type": "boolean" },
    "target_path": { "type": "string" },
    "files": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "filename": { "type": "string" },
          "size": { "type": "integer" },
          "is_dir": { "type": "boolean" },
          "modify_time": { "type": "string" }
        }
      }
    }
  }
}
```

---

### Scenario 2: Download File & Save to SharePoint

Once files are listed, download and store them into a SharePoint document library:

1. **Add Action:** `Apply to each` using `body('Parse_JSON')?['files']`
2. **Add Condition:** `item()?['is_dir'] is equal to false`
3. **Add Action inside Condition:** `HTTP (Download Remote File)`
   * **Method:** `GET`
   * **URI:** 
     ```http
     https://zetaleap.com/api/sftp/rebex/download?file=@{item()?['filename']}
     ```
   * **Headers:**
     ```http
     Authorization: Bearer zl_pat_3f9a7c8e2b1d40a5bc8e9102
     ```
4. **Add Action:** `SharePoint - Create file`
   * **Site Address:** `https://zetaleap.sharepoint.com/sites/Automation`
   * **Folder Path:** `/Shared Documents/SFTP Ingestion`
   * **File Name:** `@{item()?['filename']}`
   * **File Content:** `body('HTTP_Download')`

---

### Scenario 3: On-Demand File Access via Power Apps

1. The user clicks a button in **Power Apps** (e.g., *"Fetch Vendor Invoice"*).
2. Power Apps passes the invoice number to a Power Automate flow.
3. The flow queries the Zetaleap gateway:
   ```http
   GET https://zetaleap.com/api/sftp/rebex/download?file=/invoices/INV-9021.pdf
   ```
4. The flow returns the file link or email attachment directly to the user in seconds.

---

## 🛠️ Step-by-Step Setup

1. **Create an Account & Sign In:**  
   Navigate to [zetaleap.com/login](https://zetaleap.com/login) and log in.

2. **Generate a Personal Access Token (API Key):**  
   Go to **My Profile** > **Personal Access Tokens** > click **Generate Token**. Copy your new Bearer token.

3. **Register your SFTP/FTP Connection:**  
   Open **SFTP / FTP Storage Vault** > **➕ New Connection**.  
   * **Protocol:** `SFTP` (Port 22) or `FTP` (Port 21)
   * **Host & Port:** Target server host/IP
   * **Authentication:** Password or SSH Private Key (PEM)
   * **Default Root Path:** Target initial directory (e.g., `/` or `/inbox`)

4. **Verify Live Explorer:**  
   Switch to **Live Explorer** to browse remote directories in real time directly from your browser.

5. **Copy Ready-to-Use Snippets:**  
   Under **API & cURL Docs**, select your token to view pre-populated cURL commands and copy them directly into your Power Automate flows.

---

## 📡 API Reference Summary

| Action | HTTP Method | Endpoint URI | Description |
| :--- | :---: | :--- | :--- |
| **List Directory** | `GET` | `/api/sftp/rebex?path=/` | Returns JSON list of files and subdirectories. |
| **Download File** | `GET` | `/api/sftp/rebex/download?file=readme.txt` | Streams the binary file content directly. |
| **List Saved Connections**| `GET` | `/api/sftp/companies` | Returns all configured SFTP/FTP profiles. |
| **Delete Connection** | `DELETE` | `/api/sftp/companies/my_conn` | Deletes a custom connection profile. |
| **List Storage Vault** | `GET` | `/api/sftp/r2/files` | Lists archived files in Zetaleap Storage Vault. |
| **Download from Vault** | `GET` | `/api/sftp/r2/download?key=orders.csv` | Downloads an ingested file from storage. |

---

## 📋 Requirements

* Microsoft Power Automate subscription (Cloud Flows)
* Basic familiarity with HTTP actions in Power Automate or Power Apps
* A valid Zetaleap account and Personal Access Token (PAT)

---

## 🤝 Contributions & Feedback

Contributions, feedback, and feature suggestions are welcome! Feel free to open an issue or submit a pull request on GitHub.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 👨‍💻 Author & Credits

Created and maintained by **[korhanh](https://github.com/korhanh)**.  
*Power Platform, Cloud Integration & Automation Specialist*

⭐ **If you find this gateway useful in your Power Automate workflows, please consider starring this repository on GitHub!**
