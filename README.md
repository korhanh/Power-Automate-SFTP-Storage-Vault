# Zetaleap SFTP / FTP Storage Vault: Power Automate & Cloud Integration Gateway

> **Fast, Serverless, and Secure SFTP / FTP RESTful API Gateway**  
> *Transform legacy SFTP and FTP servers into clean REST APIs for Microsoft Power Automate, Power Apps, and Cloud Workflow Automation.*

> [!NOTE]
> **Hosted Service Documentation & Integration Reference:**  
> This guide provides architectural documentation, REST API references, and practical Power Automate flow examples for the hosted **Zetaleap SFTP / FTP Cloud Gateway** service. It is not an open-source self-hosted codebase. The accompanying MIT License applies to the documentation, code snippets, and integration examples provided herein.

---

## 📌 Overview & Purpose

Integrating external SFTP or legacy FTP servers into **Microsoft Power Automate** and **Power Apps** can present significant operational and architectural hurdles:

* **Firewall & Gateway Complexity:** Connecting to remote or internal servers often involves strict IP whitelisting, non-standard ports, or managing Windows-based On-Premises Data Gateways (OPDG) when servers are behind corporate networks.
* **Flow Timeouts & Large Payloads:** Legacy servers with high latency or deep directory trees can exceed Power Automate's 120-second connector timeout boundary.
* **No Built-in Browser Inspection:** Troubleshooting flow failures often requires third-party desktop tools (such as FileZilla) because cloud flows provide no visual directory explorer prior to execution.
* **Protocol Inconsistencies:** Legacy FTP endpoints frequently present handshake inconsistencies and certificate verification issues when connected from cloud environments.

**Zetaleap SFTP / FTP Storage Vault** acts as a lightweight serverless TCP bridge. It translates remote SFTP (Port 22, SSH) and legacy FTP (Port 21) protocols into predictable, structured **RESTful JSON APIs**.

With this gateway, Power Automate flows can list, stream, and download files using standard **HTTP REST actions**, significantly reducing handshake latency and connection management overhead.

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

## 🛡️ Security, Privacy & Data Handling

Enterprise automation requires clear visibility into how credentials and data are handled:

* **Streaming Architecture (Privacy-by-Design):** For live directory listings and on-demand file downloads, the gateway operates strictly as an in-memory streaming proxy. File payloads are transferred in flight and are **never written to disk or cached** on gateway servers.
* **Transport Encryption:** All traffic between Power Automate and the Zetaleap Gateway is enforced over **HTTPS (TLS 1.3)**. For remote connections, SFTP operates over **SSH (Port 22)**.
* **Dedicated Vault Storage:** Only when you explicitly configure **Automated Sync Schedules (Recurring Cron Tasks)** are files stored in encrypted cloud storage (AES-256 at rest), fully partitioned per account.
* **Credential Protection:** Remote server passwords and SSH private keys (PEM) are stored with industry-standard AES-256 encryption at rest.
* **Plain FTP Security Advisory:**
  > [!WARNING]
  > Plain FTP (Port 21) transmits credentials and file payloads in unencrypted cleartext between the gateway and the remote server. While supported for legacy equipment compatibility, **SFTP (Port 22, SSH)** is strongly recommended for all production workflows.

---

## ✨ Key Features

* **⚡ Native RESTful Gateway:** Access remote SFTP (Password or SSH Key) and FTP endpoints over standardized HTTPS endpoints.
* **🧭 Browser-Based Live Remote Explorer:** Inspect, navigate, and download files directly from your browser via serverless TCP sockets without installing desktop FTP software.
* **⏱️ Automated Scheduled Background Sync (Cron):** Ingest remote files into **Zetaleap Storage Vault** on customizable recurring schedules (Hourly, Daily, Custom Cron) with optional Move/Cut-Paste mode.
* **🔑 Self-Service Personal Access Tokens (PAT):** Generate secure Bearer API tokens directly from your user profile with instant activation.
* **🗑️ Complete Endpoint Lifecycle Management:** Add, configure, test, and safely delete custom connection profiles directly from the web interface.
* **🛡️ Enterprise Role-Based Access Control:** Protect sensitive automated background cron tasks with administrator-only permissions.

> [!IMPORTANT]
> ### 🔒 Administrative Authorization Required for Automated Sync Schedules
>
> While all registered users with an active Personal Access Token (API Key) can configure remote connections, browse files in **Live Explorer**, and integrate on-demand file transfers directly within **Microsoft Power Automate**, setting up **Automated Sync Schedules (Recurring Cron Tasks)** requires elevated privileges:
>
> * **Role-Based Protection:** Creating, modifying, or triggering scheduled synchronization tasks requires system administrator privileges or explicit **SFTP Cron** authorization granted by a system administrator in the Admin Portal.
> * **Resource Protection:** This policy safeguards network bandwidth and cloud storage resources against accidental high-frequency polling or unauthorized loop executions.
> * **Requesting Schedule Access:** Non-admin users attempting to configure automated sync schedules will be presented with a Security Alert prompt. To request cron schedule permissions for your account, contact the system administration team at [support_team@zetaleap.com](mailto:support_team@zetaleap.com).

---

## 🚀 Power Automate Integration Guide

> [!TIP]
> **Prerequisites & Best Practices:**
> * **Licensing Note:** The `HTTP` action in Power Automate requires a **Power Automate Premium** or per-user/process license.
> * **Connection Identifiers:** In the examples below, `rebex` refers to the public demo SFTP profile. In your own flows, replace `rebex` with your custom connection slug/ID registered in the portal (e.g., `https://zetaleap.com/api/sftp/{your_connection_id}`).
> * **Token Security:** Do not hardcode Personal Access Tokens (PAT) in plain text within production flows. Store your token in a **Power Automate Solution Environment Variable** or **Azure Key Vault**, and turn on **Settings > Secure Inputs** and **Secure Outputs** on HTTP actions to prevent token exposure in flow run history logs.

### Scenario 1: List Remote SFTP Files in Power Automate

Add a standard **HTTP** action to your Cloud Flow:

1. **Add Action:** `HTTP`
2. **Method:** `GET`
3. **URI:** 
   ```http
   https://zetaleap.com/api/sftp/{connection_slug}?path=/
   ```
4. **Headers:**
   ```http
   Authorization: Bearer YOUR_API_TOKEN
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
   * **URI (URL-Encoded):** 
     ```http
     https://zetaleap.com/api/sftp/{connection_slug}/download?file=@{encodeUriComponent(items('Apply_to_each')?['filename'])}
     ```
     *(Note: If downloading from a subfolder, specify the full relative path, e.g. `?file=@{encodeUriComponent(concat('/inbox/', items('Apply_to_each')?['filename']))}`)*
   * **Headers:**
     ```http
     Authorization: Bearer YOUR_API_TOKEN
     ```
4. **Add Action:** `SharePoint - Create file`
   * **Site Address:** `https://yourtenant.sharepoint.com/sites/Automation`
   * **Folder Path:** `/Shared Documents/SFTP Ingestion`
   * **File Name:** `@{items('Apply_to_each')?['filename']}`
   * **File Content:** `body('HTTP_Download')`

---

### Scenario 3: On-Demand File Access via Power Apps

1. The user clicks a button in **Power Apps** (e.g., *"Fetch Vendor Invoice"*).
2. Power Apps passes the invoice number or filename to a Power Automate flow.
3. The flow queries the Zetaleap gateway:
   ```http
   GET https://zetaleap.com/api/sftp/{connection_slug}/download?file=@{encodeUriComponent(triggerBody()?['text'])}
   ```
4. The flow returns the file binary or email attachment directly to the user in seconds.

---

## 🛠️ Step-by-Step Setup

1. **Create an Account & Sign In:**  
   Navigate to [zetaleap.com/login](https://zetaleap.com/login) and log in.

2. **Generate a Personal Access Token (API Key):**  
   Go to **My Profile** > **Personal Access Tokens** > click **Generate Token**. Copy your new Bearer token and store it securely.

3. **Register your SFTP/FTP Connection:**  
   Open **SFTP / FTP Storage Vault** > **➕ New Connection**.  
   * **Protocol:** `SFTP` (Port 22, SSH) or `FTP` (Port 21)
   * **Host & Port:** Target server host/IP
   * **Authentication:** Password or SSH Private Key (PEM)
   * **Default Root Path:** Target initial directory (e.g., `/` or `/inbox`)

4. **Verify Live Explorer:**  
   Switch to **Live Explorer** to browse remote directories in real time directly from your browser.

5. **Copy Ready-to-Use Snippets:**  
   Under **API & cURL Docs**, select your connection and token to view pre-populated cURL commands and copy them directly into your Power Automate flows.

---

## 📡 API Reference Summary

| Action | HTTP Method | Endpoint URI | Description |
| :--- | :---: | :--- | :--- |
| **List Directory** | `GET` | `/api/sftp/{connection_slug}?path=/` | Returns JSON list of files and subdirectories. |
| **Download File** | `GET` | `/api/sftp/{connection_slug}/download?file={path}` | Streams the binary file content directly. |
| **List Saved Connections**| `GET` | `/api/sftp/companies` | Returns all configured SFTP/FTP profiles. |
| **Delete Connection** | `DELETE` | `/api/sftp/companies/{connection_slug}` | Deletes a custom connection profile. |
| **List Storage Vault** | `GET` | `/api/sftp/r2/files` | Lists archived files in Zetaleap Storage Vault. |
| **Download from Vault** | `GET` | `/api/sftp/r2/download?key={key}` | Downloads an ingested file from storage. |

---

## 🛑 API Status & Error Codes

| Status Code | Reason | Resolution |
| :---: | :--- | :--- |
| `200 OK` | Success | Request processed and payload returned. |
| `400 Bad Request` | Missing / Invalid Parameters | Verify `path` or `file` parameters and ensure values are URL-encoded. |
| `401 Unauthorized` | Invalid / Missing Token | Ensure Bearer PAT token is valid and active in your user profile. |
| `403 Forbidden` | Insufficient Permissions | User account lacks administrative privilege (e.g., Cron Sync setup). |
| `404 Not Found` | Remote File / Path Not Found | Verify the target directory or filename exists on the remote SFTP/FTP server. |
| `429 Too Many Requests` | Rate Limit Exceeded | Flow polling frequency is too high. Introduce a delay or utilize Scheduled Sync. |
| `502 Bad Gateway` | Remote Host Connection Failure | Remote server unreachable, authentication rejected, or port blocked by firewall. |
| `504 Gateway Timeout` | Remote Host Unresponsive | Remote host timed out during TCP handshake or socket transmission. |

---

## 📋 Requirements

* Microsoft Power Automate subscription (Cloud Flows with Premium HTTP access)
* Basic familiarity with HTTP actions and JSON parsing in Power Automate or Power Apps
* A valid Zetaleap account and Personal Access Token (PAT)

---

## 🤝 Contributions & Feedback

Contributions, feedback, and feature suggestions are welcome! Feel free to open an issue or submit a pull request on GitHub.

---

## 📄 License

This documentation and integration guide is licensed under the [MIT License](LICENSE).

---

## 👨‍💻 Author & Credits

Created and maintained by **[korhanh](https://github.com/korhanh)**.  
*Power Platform, Cloud Integration & Automation Specialist*

⭐ **If you find this gateway useful in your Power Automate workflows, please consider starring this repository on GitHub!**
