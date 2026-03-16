# QAVACH Phase 0–1 Walkthrough

## What Was Built

### Phase 0 — Infrastructure
- [.env](file:///home/snbhowmik/CODE/QAVACH/.env) — shared environment variables
- [docker-compose.yml](file:///home/snbhowmik/CODE/QAVACH/docker-compose.yml) — Redis + MinIO

### Phase 1A — GovSign API (port 8000)
PQC government signing microservice — **15 files**

| Component | Files |
|---|---|
| Entry point | [main.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/main.py) |
| Crypto | [signer.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/crypto/signer.py), [keys.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/crypto/keys.py), [kem.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/crypto/kem.py) |
| Database | [models.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/db/models.py), [cbom.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/db/cbom.py) |
| Routers | [sign.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/routers/sign.py), [keys.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/routers/keys.py), [departments.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/routers/departments.py), [sessions.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/routers/sessions.py), [cbom.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/routers/cbom.py) |
| Seeds | [seed_departments.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/seeds/seed_departments.py) |

### Phase 1B — Mock Issuer CA (port 8001)
Mock government document issuance — **8 files**

| Component | Files |
|---|---|
| Entry point | [main.py](file:///home/snbhowmik/CODE/QAVACH/services/mock-ca/main.py) |
| Issuers | [income_tax.py](file:///home/snbhowmik/CODE/QAVACH/services/mock-ca/issuers/income_tax.py) (SLH-DSA), [uidai.py](file:///home/snbhowmik/CODE/QAVACH/services/mock-ca/issuers/uidai.py) (ML-DSA-44), [revenue.py](file:///home/snbhowmik/CODE/QAVACH/services/mock-ca/issuers/revenue.py) (RSA-2048), [health.py](file:///home/snbhowmik/CODE/QAVACH/services/mock-ca/issuers/health.py) (RSA-2048) |
| Models | [models.py](file:///home/snbhowmik/CODE/QAVACH/services/mock-ca/models.py), [demo_citizens.py](file:///home/snbhowmik/CODE/QAVACH/services/mock-ca/demo_citizens.py) |

### Phase 1C — PQC Sidecar (port 8002)
Crypto sidecar for Flutter app — [sidecar.py](file:///home/snbhowmik/CODE/QAVACH/services/govsign/sidecar.py)

---

## Verification Results

> [!TIP]
> liboqs 0.15.0 renamed algorithm identifiers (SPHINCS+ → SLH_DSA_PURE_*) and removed `set_secret_key()`. Both were fixed during implementation.

### GovSign API ✅
- **Health**: `liboqs_version: 0.15.0`
- **Sign PQC** (UIDAI ML-DSA-44): Real 2,420-byte PQC signature generated
- **Sign classical** (REVENUE RSA-2048): `quantum_safe: false` correctly flagged
- **CBOM**: 9 departments (3 PQC, 2 hybrid, 4 classical), activity feed working
- **PGCA Session**: Session created with QR payload, polling endpoint working

### Mock CA ✅
- Priya Sharma credential issuance:
  - income_certificate — **SLH-DSA-SHAKE-128s** (quantum_safe: true)
  - aadhaar_attestation — **ML-DSA-44** (quantum_safe: true)
  - land_ownership — **RSA-2048** (quantum_safe: false)
  - health_record — **RSA-2048** (quantum_safe: false)

### PQC Sidecar ✅
- ML-DSA-44 keygen: public key 1,752 chars, private key 3,416 chars (base64)

---

## Running the Services

```bash
# Terminal 1: GovSign API
cd services/govsign
python3 seeds/seed_departments.py
python3 -m uvicorn main:app --port 8000

# Terminal 2: Mock CA
cd services/mock-ca
python3 -m uvicorn main:app --port 8001

# Terminal 3: PQC Sidecar
cd services/govsign
python3 -m uvicorn sidecar:app --port 8002

# Swagger docs
# GovSign:  http://localhost:8000/docs
# Mock CA:  http://localhost:8001/docs
# Sidecar:  http://localhost:8002/docs
```
