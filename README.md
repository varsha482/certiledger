# CertiLedger – Blockchain-Based Certificate Verification System

CertiLedger lets educational institutions issue academic certificates whose **SHA-512 hash** and **IPFS CID** are registered on the **Polygon** blockchain. Every certificate carries a QR code linking to a public verification portal, where employers and institutions get an instant result: **Verified**, **Invalid**, or **Revoked**.

## Features
- Role-based access (Admin, Student, Verifier) with JWT authentication
- Student, faculty, academic and result management
- Certificate template builder and PDF generation
- SHA-512 hashing, IPFS storage, Polygon registration
- QR code and Certificate-ID verification, public verification portal
- Revocation, audit logs, notifications, reports and blockchain dashboard

## Tech Stack
| Layer | Technology |
|---|---|
| Frontend | React, Vite, Tailwind CSS, React Router, Framer Motion, Lucide React |
| Backend | FastAPI (Python), JWT, REST |
| Database | PostgreSQL |
| Blockchain | Polygon, Solidity |
| Storage | IPFS (via pinning service) |
| Security | SHA-512, digital signatures, RBAC, QR verification |

## Repository Structure
```
certiledger/
├── frontend/        React app
├── backend/         FastAPI app
├── contracts/       Solidity smart contract + deployment scripts
├── sample-data/     Demo students, results, certificates
├── tests/           Automated tests
└── docs/            Architecture and test-case documents
```

## Prerequisites
- Node.js 20+ and npm
- Python 3.11+
- PostgreSQL 15+
- A Polygon Amoy testnet wallet funded with test MATIC (from a public faucet)
- An IPFS pinning account (e.g. Pinata) or a local IPFS node

## Setup

### 1. Clone and configure
```bash
git clone https://github.com/<your-username>/certiledger.git
cd certiledger
cp .env.example backend/.env      # then edit values
cp .env.example frontend/.env     # keep only the VITE_ variables
```

### 2. Database
```bash
createdb certiledger
# Set DATABASE_URL in backend/.env, then run migrations (e.g. Alembic)
cd backend && alembic upgrade head
```

### 3. Smart contract
```bash
cd contracts
npm install
npx hardhat run scripts/deploy.js --network amoy
# Copy the printed address into CONTRACT_ADDRESS in backend/.env
```

### 4. Backend
```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```
API docs: http://localhost:8000/docs

### 5. Frontend
```bash
cd frontend
npm install
npm run dev
```
App: http://localhost:5173

### 6. Load sample data
Import the files in `sample-data/` (see `sample-data/README.md`) via the admin dashboard or a seed script.

## Environment Variables
| Variable | Used by | Description |
|---|---|---|
| `DATABASE_URL` | backend | PostgreSQL connection string |
| `JWT_SECRET` | backend | Secret for signing JWTs |
| `JWT_ALGORITHM` | backend | JWT algorithm (default HS256) |
| `JWT_EXPIRY_MINUTES` | backend | Token lifetime |
| `POLYGON_RPC_URL` | backend, contracts | RPC endpoint |
| `CHAIN_ID` | backend, contracts | 80002 for Amoy |
| `PRIVATE_KEY` | backend, contracts | Issuer wallet key (keep secret) |
| `CONTRACT_ADDRESS` | backend | Deployed contract address |
| `IPFS_API_URL` | backend | IPFS/pinning API endpoint |
| `IPFS_PINNING_KEY` | backend | Pinning API key |
| `IPFS_GATEWAY_URL` | backend | Gateway for fetching certificates |
| `VITE_API_BASE_URL` | frontend | Backend API URL |
| `VITE_VERIFY_BASE_URL` | frontend/backend | Base URL embedded in QR codes |

## Testing
```bash
cd backend && pytest ../tests
cd contracts && npx hardhat test
```
Test cases are listed in [docs/test-cases.md](docs/test-cases.md).

## Documentation
- [Technical Design Document](docs/architecture.md)
- [Test cases](docs/test-cases.md)

## Security Notes
Never commit `.env` files or private keys. Use a dedicated issuer wallet, encrypted key backups, and a reliable IPFS pinning service.

## License
MIT. See [LICENSE](LICENSE).
