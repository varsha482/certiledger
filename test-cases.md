# CertiLedger – Test Cases

Sample data referenced below is in `sample-data/`.

| ID | Area | Scenario | Input | Expected Result |
|---|---|---|---|---|
| TC-01 | Auth | Valid admin login | admin@certiledger.test / valid password | 200, JWT returned |
| TC-02 | Auth | Wrong password | valid email, bad password | 401 |
| TC-03 | RBAC | Student calls issue endpoint | Student JWT | 403 |
| TC-04 | RBAC | Verifier calls results upload | Verifier JWT | 403 |
| TC-05 | Results | Upload valid results CSV | `results.csv` | Rows stored, summary returned |
| TC-06 | Results | Upload CSV with missing roll number | malformed row | Row rejected with validation error |
| TC-07 | Hash | Hash same file twice | `CL-2026-0001.txt` | Identical 128-hex-character SHA-512 |
| TC-08 | Hash | Change one byte | modified file | Different hash |
| TC-09 | Issue | Issue certificate for valid student | roll `CS2022001` | Status ISSUED, CID and tx hash stored, QR generated |
| TC-10 | Issue | Duplicate certificate ID | existing ID | Rejected (contract and API) |
| TC-11 | Contract | Register by non-issuer account | other wallet | Transaction reverts |
| TC-12 | Contract | `getCertificate` unknown ID | random ID | Empty record |
| TC-13 | Verify | Untampered certificate | `CL-2026-0001` | Verified |
| TC-14 | Verify | Tampered file | `CL-2026-0003` (stored hash mismatch) | Invalid |
| TC-15 | Verify | Revoked certificate | `CL-2026-0002` | Revoked |
| TC-16 | Verify | Unknown certificate ID | `CL-2026-9999` | Invalid (not found) |
| TC-17 | Verify | Scan QR code | QR from issued certificate | Opens portal with correct result |
| TC-18 | Revoke | Admin revokes certificate | `CL-2026-0001` | On-chain flag set; later verify returns Revoked |
| TC-19 | Audit | Issue and revoke actions | any | Audit log entries created |
| TC-20 | Resilience | IPFS unavailable during issue | simulated outage | Retry, no partial record marked ISSUED |
| TC-21 | Resilience | RPC timeout during register | simulated timeout | Status PENDING, retried, no duplicate registration |
| TC-22 | Student | Student views own certificates | Student JWT | Only own certificates listed |
