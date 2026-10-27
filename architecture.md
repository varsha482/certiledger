# CertiLedger – Technical Design Document

## 1. Overview
CertiLedger issues academic certificates as PDFs, hashes them with SHA-512, stores them on IPFS, and registers the hash and CID on Polygon. Verification recomputes the hash and compares it with the on-chain record.

## 2. System Architecture
```mermaid
flowchart LR
    subgraph Client
        A[Admin Dashboard]
        S[Student Portal]
        V[Public Verification Portal]
    end
    subgraph Backend[FastAPI Backend]
        API[REST API + JWT/RBAC]
        GEN[PDF + QR Generator]
        HASH[SHA-512 Service]
        BC[Blockchain Service]
        IP[IPFS Service]
        AUD[Audit + Notifications]
    end
    DB[(PostgreSQL)]
    IPFS[(IPFS / Pinning Service)]
    POLY[[Polygon Smart Contract]]

    A --> API
    S --> API
    V --> API
    API --> GEN
    API --> HASH
    API --> BC
    API --> IP
    API --> AUD
    API --> DB
    IP --> IPFS
    BC --> POLY
```

## 3. Certificate Issuance Flow
```mermaid
sequenceDiagram
    actor Admin
    participant API as FastAPI
    participant DB as PostgreSQL
    participant IPFS
    participant Chain as Polygon Contract
    Admin->>API: Upload results / request issuance
    API->>DB: Validate student and results
    API->>API: Generate certificate PDF
    API->>API: Compute SHA-512 hash
    API->>IPFS: Upload PDF
    IPFS-->>API: CID
    API->>Chain: registerCertificate(certId, hash, CID)
    Chain-->>API: Transaction hash
    API->>API: Generate QR code (verify URL)
    API->>DB: Store record, tx hash, status = ISSUED
    API-->>Admin: Certificate issued
```

## 4. Verification Flow
```mermaid
flowchart TD
    A[Scan QR or enter Certificate ID] --> B[Fetch on-chain record]
    B --> C{Record exists?}
    C -- No --> INV[Invalid]
    C -- Yes --> D{Revoked?}
    D -- Yes --> REV[Revoked]
    D -- No --> E[Fetch PDF from IPFS using CID]
    E --> F[Recompute SHA-512]
    F --> G{Hash equals on-chain hash?}
    G -- Yes --> OK[Verified]
    G -- No --> INV
```
Optionally the verifier uploads a PDF; its hash is compared instead of the IPFS copy.

## 5. Data Model
```mermaid
erDiagram
    USERS ||--o{ AUDIT_LOGS : performs
    STUDENTS ||--o{ RESULTS : has
    STUDENTS ||--o{ CERTIFICATES : receives
    TEMPLATES ||--o{ CERTIFICATES : formats
    CERTIFICATES ||--o{ VERIFICATION_LOGS : checked_in
    USERS {
        uuid id PK
        string email
        string password_hash
        string role
    }
    STUDENTS {
        uuid id PK
        string roll_no
        string name
        string program
    }
    RESULTS {
        uuid id PK
        uuid student_id FK
        int semester
        float cgpa
    }
    CERTIFICATES {
        uuid id PK
        string certificate_id
        uuid student_id FK
        string sha512_hash
        string ipfs_cid
        string tx_hash
        string status
        datetime issued_at
    }
    TEMPLATES {
        uuid id PK
        string name
        json layout
    }
    VERIFICATION_LOGS {
        uuid id PK
        string certificate_id
        string result
        datetime checked_at
    }
    AUDIT_LOGS {
        uuid id PK
        uuid user_id FK
        string action
        datetime at
    }
```

## 6. Smart Contract Design
| Function | Access | Purpose |
|---|---|---|
| `registerCertificate(certId, sha512Hash, cid)` | Issuer only | Store hash and CID; reject duplicates |
| `revokeCertificate(certId)` | Issuer only | Mark certificate revoked |
| `getCertificate(certId)` | Public (view) | Return hash, CID, issuer, timestamp, revoked flag |

Events: `CertificateRegistered`, `CertificateRevoked`. Because SHA-512 is 64 bytes, store it as `bytes` or two `bytes32` values.

## 7. REST API (summary)
| Method | Endpoint | Role |
|---|---|---|
| POST | `/api/auth/login` | Public |
| POST | `/api/students`, `/api/results/upload` | Admin |
| POST | `/api/certificates/issue` | Admin |
| POST | `/api/certificates/{id}/revoke` | Admin |
| GET | `/api/certificates/me` | Student |
| GET | `/api/verify/{certificateId}` | Public/Verifier |

## 8. Access Control
| Capability | Admin | Student | Verifier |
|---|---|---|---|
| Manage students/results | Yes | No | No |
| Issue/revoke certificates | Yes | No | No |
| View/download own certificates | No | Yes | No |
| Verify certificates | Yes | Yes | Yes |

## 9. Security
JWT authentication, RBAC, SHA-512 integrity, digital signatures on issued records, encrypted wallet key storage, audit logging, input validation, and an approval step for data entry.

## 10. Risks and Mitigation
| Risk | Mitigation |
|---|---|
| Blockchain transaction delays | Use Polygon; retry with status tracking |
| Loss of wallet keys | Secure key management, encrypted backups |
| Unauthorized access | JWT and RBAC |
| Network outages | Database backups, retry mechanisms |
| IPFS node availability | Reliable pinning services |
| Revocation management | On-chain revocation flag plus status checks |
| Human data entry errors | Validation rules, approval workflow |
| Scalability | Modular architecture, cloud deployment |
