# 🛡️ Azure Encryption-at-Rest Compliance Engine

<img width="1568" height="536" alt="image" src="https://github.com/user-attachments/assets/75659a6f-0a25-45e1-b43e-1d38b5b30e51" />

An automated GRC validation tool that scans Azure environments to verify data-at-rest encryption across Storage Accounts, Managed Disks, and Key Vaults — producing audit-ready compliance evidence for SOC 2, NIST 800-53, and ISO 27001.

Instead of manually clicking through hundreds of resources in the Azure Portal to confirm encryption is enabled and properly configured, this tool programmatically queries Azure Resource Manager, evaluates each resource against a strict compliance baseline, and generates structured reports with specific remediation recommendations for anything that falls short.

---

## ✨ Features

- **Automated Evidence Collection:** Queries Azure Resource Manager (ARM) to collect encryption states across an entire subscription.
- **Multi-Service Scanning:** Evaluates Azure Storage Accounts, Compute Managed Disks, and Azure Key Vaults.
- **Risk Classification:** Categorizes resources by compliance risk — immediately flagging unencrypted assets or publicly exposed storage containers as HIGH risk.
- **Dual-Format Reporting:** Generates JSON reports for SIEM integration and CSV summaries for auditors.
- **Secretless Execution:** Authenticates via Azure Managed Identities or local CLI session — no hardcoded credentials.
- **Actionable Remediation:** Provides specific, per-resource recommendations for fixing non-compliant configurations.

---

## 🔒 Security

- **Least Privilege:** Runs strictly on read-only API endpoints (`Reader` or `Security Reader` roles) with no ability to modify infrastructure.
- **Secret Management:** Uses `DefaultAzureCredential` exclusively — no environment variables, hardcoded keys, or service principal secrets.
- **Control Plane Isolation:** Operates solely on the Azure control plane. The tool cannot read actual data contents (blob data, disk sectors, vault secrets) — only configuration metadata.
- **Input Validation:** Command-line arguments are strictly typed and validated before execution.
- **Secure Communication:** All API calls to Azure Resource Manager are enforced over TLS 1.2+ with certificate validation.

---

## 🛠️ Technologies Used

- **Language:** Python 3.14+
- **Cloud Platform:** Microsoft Azure
- **Azure SDKs:** `azure-mgmt-compute`, `azure-mgmt-storage`, `azure-mgmt-keyvault`, `azure-identity`
- **Authentication:** Azure Entra ID, Managed Identities, DefaultAzureCredential
- **Output Formats:** JSON, CSV

---

## 🏗️ System Architecture

```text
+-------------------+       +-----------------------+       +-------------------+
|                   |       |                       |       |                   |
|  User / Auditor   +------>+  Compliance Engine    +------>+  Azure ARM API    |
|  (CLI Execution)  |       |  (Python Orchestrator)|       |  (Control Plane)  |
|                   |       |                       |       |                   |
+--------^----------+       +-----------+-----------+       +-------------------+
         |                              |                             |
         |      +-----------------------v-----------------------+     |
         +------+ JSON & CSV Audit Evidence (Timestamped Files) |<----+
                +-----------------------------------------------+
```

- **Interface:** CLI-driven, compatible with automated pipelines.
- **Core Engine:** Python orchestrator handling authentication, API queries, and state evaluation.
- **External Services:** Queries Azure Resource Manager (`management.azure.com`) over TLS 1.2+.

---

## 🚀 How It Works

1. **Authentication:** The engine requests a temporary, scoped OAuth token from Azure AD using the host's Managed Identity or CLI session.
2. **Resource Discovery:** Read-only API calls are dispatched to Azure Resource Manager to catalog all active storage accounts, managed disks, and key vaults in the subscription.
3. **State Evaluation:** Returned metadata (e.g. `allowBlobPublicAccess`, `encryption.services.blob.enabled`) is evaluated against predefined compliance baselines.
4. **Report Generation:** Findings are aggregated into a timestamped JSON report and a CSV summary, saved locally as audit evidence.

<img width="1284" height="704" alt="image" src="https://github.com/user-attachments/assets/1f8152d9-8123-4f32-86d0-09d7844a5256" />

<img width="1302" height="670" alt="image" src="https://github.com/user-attachments/assets/8046a65f-f785-467e-b7b6-083ac5c2cac8" />

<img width="1568" height="293" alt="image" src="https://github.com/user-attachments/assets/1d933e4e-3a47-4e05-b0d4-0b606001c9b6" />

*The tool detected public/anonymous blob access enabled on `grcappsadh2s4j` and flagged it as NON_COMPLIANT — the setting will be remediated to Disabled.*



---

## 💡 Key Design Decisions

- **Stateless Execution:** The engine queries live, point-in-time Azure state rather than caching locally — auditors always see the real, current configuration.
- **Dual-Format Output:** JSON for programmatic integration (e.g. shipping to Sentinel), CSV for non-technical auditors who need a spreadsheet.
- **Management SDKs Only:** Using `azure-mgmt-*` SDKs instead of data-plane SDKs guarantees the tool cannot accidentally access sensitive data (PII/PHI) stored within the resources it audits.

---

## 📈 Scalability

- **Stateless Design:** No local state between runs, making it straightforward to containerize and run across many subscriptions via CI/CD pipelines.
- **Serverless Ready:** The core logic is decoupled from the execution layer and can be wrapped into an Azure Function for scheduled, event-driven auditing.

---

## 📂 Project Structure

- `azure_encryption_validator.py` — Core execution script and validation engine.
- `azure_encryption_compliance_report_*.json` — Generated machine-readable evidence files.
- `azure_encryption_compliance_summary_*.csv` — Generated human-readable audit summaries.
- `AZURE_ENCRYPTION_VALIDATOR_GUIDE.md` — Technical usage documentation and user guide.

---

## 🚀 Getting Started

### Prerequisites
- Python 3.11+
- Azure CLI (`az`)

### Installation
1. Clone the repository:
```bash
   git clone <YOUR_REPO_URL>
   cd <YOUR_REPO_DIRECTORY>
```
2. Set up a virtual environment:
```bash
   python3 -m venv venv
   source venv/bin/activate
```
3. Install dependencies:
```bash
   pip install azure-identity azure-mgmt-compute azure-mgmt-storage azure-mgmt-keyvault
```

### Usage
1. Authenticate to Azure:
```bash
   az login
```
2. Run the validator:
```bash
   python3 azure_encryption_validator.py --subscription-id <YOUR_SUBSCRIPTION_ID> --output-format both
```
3. Check the working directory for the generated `.json` and `.csv` files containing your compliance findings.

---

## 🔮 Future Improvements

- **Multi-Cloud Support:** Extend to AWS (boto3) and GCP for unified hybrid-cloud compliance reporting.
- **Automated Remediation:** An opt-in `--auto-remediate` flag to automatically apply standard encryption policies to non-compliant resources.
- **Serverless Integration:** Package as an Azure Function triggered by Event Grid for real-time compliance alerting on resource creation.

---

## 🎓 Skills Demonstrated

- Cloud security architecture (Azure Resource Manager, Managed Identities)
- GRC automation and encryption-at-rest validation
- Secure software development (secretless design, control-plane isolation)
- Azure SDK integration and API data parsing
- Stateless application design for audit-trail generation
