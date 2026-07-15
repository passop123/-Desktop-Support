# Log & Dump Upload Process

Purpose: describe how to securely collect, package, encrypt and upload logs and memory dumps for incident analysis.

1. Prepare evidence package
- Run the official collection script (`docs/scripts/collect-incident.ps1`) as Administrator. The script will collect systeminfo, drivers, recent events and minidump files and create a zip package.

2. Naming convention
- Use: {INCIDENT-<ticket>}_{HOSTNAME}_{YYYYMMDD_HHMM}.zip
- Example: INCIDENT-1234_WORKSTATION01_20260715_1430.zip

3. Encryption and transport
- If using company SFTP or ITSM attachment, prefer SFTP upload to /incoming/{ticket}/ and notify L2 with upload path.
- If file size > 200MB, request temporary secure cloud upload (pre-signed link) from L2 and record expiry time.
- All zip files MUST be encrypted. Supported methods:
  - ZIP with password (AES-256) and password passed via secure channel (phone or secure chat), or
  - PGP encryption to the analysis team's public key.

4. Sensitive data & retention
- Dumps and logs may contain sensitive data. Remove PII if possible and document what was removed.
- Retention: keep evidence in secured storage for 90 days by default, then archive per company policy.

5. Upload checklist (include in ITSM ticket)
- Evidence package name & location
- Upload method (SFTP/Presigned URL/ITSM attachment)
- Encryption method and password delivery channel
- Files included (systeminfo, driverquery, minidump list, event logs)
- Contact person for retrieval

6. Notes
- Do NOT email unencrypted dumps. If unsure, contact L2/InfoSec before transfer.
