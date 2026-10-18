# NB6013CEM — Penetration Testing Coursework Laboratory

An isolated, multi-target containerized penetration testing lab designed for security assessment, vulnerability discovery, controlled exploitation, and formal reporting.

---

## Overview

This repository contains a full Docker-based environment providing three distinct web application targets with isolated backend networks, dedicated MySQL databases, and synthetic seed datasets. The environment is configured strictly for authorized coursework exercises and security testing.

### Target Systems

| Target Application | Default URL | Tech Stack | Description |
| :--- | :--- | :--- | :--- |
| **OWASP Juice Shop** | `http://127.0.0.1:3000` | Node.js / Angular / SQLite / Nginx | Industry-standard insecure web application configured in assessment mode (quiet mode with hidden challenges/hints). |
| **Northstar Vehicle Care** | `http://127.0.0.1:8081` | Node.js (Express + TS) / Vite / MySQL 8.4 | Custom vehicle service and maintenance portal featuring customer management, document storage, and booking workflows. |
| **SecureBank** | `http://127.0.0.1:8082` | Node.js (Express + TS) / Vite / MySQL 8.4 | Custom simulated retail banking portal featuring account management, inter-account transfers, and support operations. |

---

## Lab Architecture & Containment

The lab enforces strict containment principles to ensure testing remains safe and localized:

- **Localhost Only**: Ingress reverse proxies are explicitly bound to `127.0.0.1`.
- **Network Segmentation**: Each target application runs within dedicated internal Docker bridge networks (`juice_net`, `vehicle_net`, `bank_net`) with separate edge ingress networks (`juice_edge`, `vehicle_edge`, `bank_edge`).
- **Internal Database Isolation**: Database services (`vehicle_db`, `bank_db`) and internal API services do not publish host ports; they are accessible only via the reverse proxy / internal network bridges.
- **Controlled Seed Data**: All accounts, transactions, vehicle records, and personal identifiers are entirely fictional and synthetic.

---

## Prerequisites

Ensure the following tools are installed on your host machine:

- **Docker Engine** / **Docker Desktop** (with Docker Compose v2 support)
- **Node.js 20+** (required for verification and integration test suites)
- **Bash** shell (for utility scripts)
- **curl** / **wget** / **lsof**

---

## Quick Start

### 1. Configure Environment

Copy the example environment configuration file:

```bash
cp .env.example .env
```

If any default host ports (`3000`, `8081`, `8082`) conflict with existing services on your machine, configure custom ports in `.env`:

```env
# Optional host port overrides
JUICE_PORT=3000
VEHICLE_PORT=8081
BANK_PORT=8082
```

### 2. Deploy the Laboratory

Run the deployment script to build containers, wait for health checks to pass, and verify isolation:

```bash
./scripts/deploy.sh
```

Once running, access the web targets in your browser:

- Juice Shop: [http://127.0.0.1:3000](http://127.0.0.1:3000)
- Northstar Vehicle Care: [http://127.0.0.1:8081](http://127.0.0.1:8081)
- SecureBank: [http://127.0.0.1:8082](http://127.0.0.1:8082)

---

## Synthetic Credentials

The following synthetic user accounts are seeded into the custom applications for testing:

| Application | Email / Username | Default Password | Role |
| :--- | :--- | :--- | :--- |
| **Vehicle Care** | `maya@vehicle.test` | `Welcome123!` | Customer |
| **Vehicle Care** | `leo@vehicle.test` | `Welcome123!` | Customer |
| **SecureBank** | `elena@securebank.test` | `Welcome123!` | Customer |
| **SecureBank** | `marcus@securebank.test` | `Welcome123!` | Customer |

*Note: Self-registration is also enabled on both custom web applications.*

---

## Lab Management & Scripts

Helper scripts located in `scripts/` simplify environment lifecycle operations:

- **`./scripts/deploy.sh`**: Checks prerequisite ports, starts all Compose services in detached mode, and verifies lab readiness.
- **`./scripts/verify.sh`**: Validates health check status across all 7 containers, confirms localhost network bindings, verifies lack of source map leaks in production bundles, and executes smoke tests.
  - Run read-only verification: `./scripts/verify.sh --read-only`
- **`./scripts/reset.sh`**: Shuts down containers, wipes the MySQL named volumes (`coursework-lab_vehicle_data`, `coursework-lab_bank_data`), rebuilds, and re-seeds deterministic initial data.
  - Skip interactive confirmation: `./scripts/reset.sh --yes`
- **`./scripts/seed.sh`**: Ensures services are up and executes seed validation.

To manually shut down the lab:

```bash
docker compose down
```

---

## Testing & OpenAPI Specifications

- **Integration Smoke Tests**: `tests/integration/smoke.mjs` validates authentication, account fetching, vehicle creation, document handling, and unauthorized endpoint enforcement.
- **API Specifications**: `tests/apisec-capture-*.json` provides captured OpenAPI 3.0 specifications for target endpoints.

Run smoke tests directly with:

```bash
node tests/integration/smoke.mjs
```

---

## Repository Structure

```text
.
├── apps/
│   ├── securebank/           # SecureBank application (backend, frontend, DB schema)
│   └── vehicle-portal/       # Northstar Vehicle Care (backend, frontend, DB schema)
├── docs/
│   ├── assessment-brief.md   # Coursework assessment brief and grading criteria
│   └── student-setup.md      # Detailed student onboarding instructions
├── infra/
│   └── nginx/                # Ingress configuration and quiet mode overrides for Juice Shop
├── scripts/
│   ├── deploy.sh             # Main deployment automation script
│   ├── lab-ports.sh          # Port extraction helper
│   ├── reset.sh              # Data reset and re-seed script
│   ├── seed.sh               # Database seeding runner
│   └── verify.sh             # Comprehensive health and security validation script
├── tests/
│   ├── apisec-capture-*.json # Captured OpenAPI v3 definitions
│   └── integration/          # Integration and smoke tests
├── docker-compose.yml        # Multi-service container definitions & network segmentation
├── .env.example              # Template environment variables
└── README.md                 # Project documentation
```

---

## Rules of Engagement & Ethics

This lab environment is created strictly for academic and educational penetration testing coursework.

1. **Scope Boundaries**: Only test the three local services mapped to `127.0.0.1`.
2. **Prohibited Targets**: Do **not** target the host operating system, Docker daemon/socket, peer student lab instances, or any external Internet services.
3. **Synthetic Data**: Use only generated or synthetic test accounts. Never input real personal, financial, or production credentials.
4. **Non-Destructive Testing**: Avoid permanent denial-of-service payloads, brute-force floods that destabilize containers, or host escape attempts.
5. **Data Restoration**: If testing alters or corrupts synthetic data, use `./scripts/reset.sh --yes` to restore the lab to its deterministic baseline.
