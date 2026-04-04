Author: Kehinde Oduyeye
# Offboarding-User-Termination-Automation

Device Timeline Automation is a threat-enrichment automation script designed for Security Operations Centers (SOC).
It ingests IP addresses and URLs from endpoint/network event logs, enriches them through multiple threat-intel sources, and produces a structured, high-fidelity output for triage, hunting, and incident investigation.

Blog post that goes into more details, can be found here: [https://www.kehinde-oduyeye.dev/#/blog/1764701331243](https://www.kehinde-oduyeye.dev/#/blog/1764701331243)

## Architecture Overview
The workflow is triggered via HTTP request or interactive Teams Adaptive Card input. It then:

Validates the target user
Retrieves secrets securely from Key Vault
Executes identity containment:
Password reset
Session revocation
Block sign-in enforcement
Queries and processes all registered user devices
Applies conditional response actions:
Isolate Windows endpoints
Lock iOS devices (Lost Mode)
Logs results into Azure Table Storage
Sends a summarized report to Microsoft Teams
Repeats the process for privileged (admin) account if detected

## Features
This Logic App automates end-to-end user offboarding and containment by integrating Microsoft Graph, Defender for Endpoint, Azure Table Storage, Key Vault, and Microsoft Teams into a unified workflow.

It performs:

✔ Secure secret retrieval via Azure Key Vault (no hardcoded credentials)

✔ User validation and profile lookup via Microsoft Graph

✔ Automated password reset with strong randomized password generation

✔ Session revocation and presence clearing

✔ Conditional block sign-in via group membership

✔ Managed device discovery and response actions

✔ Windows device isolation via Defender for Endpoint

✔ iOS device lock (Lost Mode) and forced sync via Intune

✔ Device activity filtering (last sync within 30 days)

✔ Admin account detection and parallel containment workflow

✔ Full workflow telemetry logging to Azure Table Storage

✔ Adaptive Card-driven interaction and status reporting via Microsoft Teams

✔ Error handling and fallback HTTP logging for failed actions

## Repository Structure

├── REACT-UserTerminationWorkflow.json   # ARM template (Logic App definition)
├── README.md                           # Documentation


## Requirements
Azure Subscription with the following services:

Azure Logic Apps
Azure Key Vault
Azure Table Storage
Microsoft Graph API access
Microsoft Defender for Endpoint API
Microsoft Teams connector
Office 365 Users & Groups connectors

## Configuration
1. Deploy ARM Template
   Deploy the Logic App using:
   * Azure Portal
   * Azure CLI
   * ARM deployment pipeline
   
2. Configure Connections
Ensure the following API connections are configured:

| Service           | Purpose                        |
| ----------------- | ------------------------------ |
| Key Vault         | Secret management              |
| Office 365 Users  | User profile lookup            |
| Office 365 Groups | Block sign-in group membership |
| Microsoft Teams   | Notifications & approvals      |
| Azure Tables      | Logging & audit storage        |


3. Secrets (Stored in Key Vault)

  * Graph API client secret
  * Defender API credentials
  * Isolation credentials

⚠️ All secrets must be stored securely—never hardcoded.

4. Required Input

"emailInput"

This is collected via:

HTTP POST body
OR Teams Adaptive Card input

## Example Workflow Execution
Input

"emailInput: user@company.com"

Actions Performed
* User profile retrieved
* Password reset with generated secure password
* Active sessions revoked
* User added to block sign-in group
* Devices retrieved and evaluated:
  * Active Windows devices → isolated
  * Active iOS devices → locked (Lost Mode)
  * Stale devices → logged only
* Admin account (if exists) processed with same controls

## Example Output (Summary)
| Action                    | Status  |
| ------------------------- | ------- |
| Password Reset            | Success |
| Revoke Sessions           | Success |
| Block Sign-In             | Success |
| Windows Devices Processed | 3       |
| iOS Devices Processed     | 1       |
| Total Devices             | 5       |


## Teams Notification Example
* Workflow triggered confirmation
* User not found alert (if applicable)
* Admin account detection notification
* Final workflow summary with:
  * Identity actions status
  * Device processing counts
  * Link to detailed device report

## Data Logging

Use Azure Table Storage Tables or Dataverse to store the following data.
* OffboardingStatusTable
  * Tracks workflow execution status per user

* DeviceDetailsQueue
  * Stores per-device action results:
    * Device name
    * OS
    * Last sync time
    * Action taken
    * Outcome

## Error Handling

Each critical stage includes fallback HTTP logging for:

* Password reset failures
* Session revocation failures
* Block sign-in failures

Captured data includes:

* Workflow name
* Affected user
* Error message
* Timestamp
* Run ID

## Screenshots

<img width="443" height="741" alt="image" src="https://github.com/user-attachments/assets/2845ff2e-8773-4669-88d4-8feea0f8f977" />
<img width="715" height="605" alt="image" src="https://github.com/user-attachments/assets/9f33ccf4-8489-4d41-a3b9-e30200e1529d" />
<img width="640" height="675" alt="image" src="https://github.com/user-attachments/assets/aec69247-3bd6-49ef-83ac-739389539b0f" />
<img width="644" height="547" alt="image" src="https://github.com/user-attachments/assets/107ddb01-8698-4d6b-9c49-42f61d16b5fa" />
<img width="344" height="618" alt="image" src="https://github.com/user-attachments/assets/11c9c395-b613-428b-ae87-4534eafc8b43" />


## What I Learned While Building This
Building this workflow provided deep insight into how enterprise-grade identity and endpoint response automation can be orchestrated using Azure Logic Apps. I learned how to securely integrate multiple Microsoft services—Graph API, Defender for Endpoint, and Intune—while maintaining proper secret hygiene through Key Vault.

One of the most impactful lessons was designing conditional logic for device handling. Not all endpoints should be treated equally, and filtering based on recent activity (e.g., last sync within 30 days) significantly improves response accuracy while avoiding unnecessary disruption.

Generating secure passwords dynamically within Logic Apps highlighted both the flexibility and limitations of workflow expressions, reinforcing the importance of randomness and complexity in automated identity actions.

I also gained experience in building resilient workflows with structured error handling and fallback logging. This is critical in SOC environments, where failed automation without visibility can be more dangerous than no automation at all.

Finally, integrating Microsoft Teams Adaptive Cards demonstrated how powerful human-in-the-loop automation can be—allowing analysts to trigger, monitor, and validate actions directly within their collaboration platform, dramatically improving operational efficiency and response time.

## Contributions
Contributions are welcome!
