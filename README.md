# 🐳 IBM webMethods MWS & MSR Docker Compose Setup

This Docker Compose configuration sets up a local development environment with the following services:

- **MWS (My webMethods Server)**
- **MSR (Microservices Runtime)**
- **PostgreSQL database (`wmdb`)**
- **DCC-based DB initialization (`db-init`)**

---

## 🚀 Getting Started

### 1. Clone the repository (or prepare your directory layout):

```bash
git clone <your-repo>
cd <your-repo>
```

### 2. Create Required Folder Structure

```bash
mkdir -p \
  mws/volumes/{apps,configs,data,libs,logs,patches} \
  msr/{license,server} \
  dcc/{logs,init}
```

You may also place initialization files in `dcc/init` if needed.

### 3. Start the environment

```bash
docker-compose up -d
```

---

## 🧩 Service Breakdown

### 🔹 `mws-app`

> My webMethods Server container

- **Image**: `mws:10.15`
- **Ports**: `8585` exposed for web access
- **Volumes**:
  - Mounts app, config, and data folders from `./mws/volumes`
  - Receives DCC init signaling via `./dcc/init`
- **Environment Variables**:
  - Database credentials and connection string
  - Server instance and volume base paths
- **User**: Runs as UID/GID `1724`
- **Depends on**: `db-init`

### 🔹 `msr`

> Integration Server / Microservices Runtime

- **Image**: `msr:10.15`
- **Ports**:
  - `5555`: HTTP port
  - `9999`: Admin or diagnostics port
- **Volumes**:
  - Licenses and server state under `./msr`
  - Shared init signaling via `./dcc/init`
- **Depends on**: `db-init`

### 🔹 `wmdb`

> PostgreSQL database container

- **Image**: `postgres:latest`
- **Ports**: `5432` for database access
- **Credentials**:
  - User: `wm`
  - Password: `wm`
  - DB: `wm`
- **Volume**:
  - `db_data` for persistent storage

### 🔹 `db-init`

> Database configuration via IBM webMethods's DCC tool

- **Image**: `dcc:10.15`
- **Entry point**: Overridden to run custom init command
- **Command**:
  - Initializes the MWS DB schema using `dbConfigurator.sh`
  - Signals readiness by touching `/tmp/ready`
- **Healthcheck**:
  - Waits for `/tmp/ready` before allowing dependent services to start

---

## 🗂️ Volumes

| Volume         | Purpose                        |
|----------------|--------------------------------|
| `db_data`      | PostgreSQL persistent data     |
| `./mws/volumes`| MWS runtime volumes            |
| `./msr/server` | IS/MSR server runtime data     |
| `./msr/license`| License files for MSR          |
| `./dcc/logs`   | DCC log output                 |
| `./dcc/init`   | Shared signaling and init data |

---

## 🌐 Networks

| Network    | Description              |
|------------|--------------------------|
| `localdev` | Shared network for all services |

---

## 🧹 Stopping & Cleanup

To stop the services:
```bash
docker-compose down
```

To stop and clean everything including volumes:
```bash
docker-compose down --volumes --remove-orphans
```

---

## 🧪 Troubleshooting

- Check logs with:
  ```bash
  docker-compose logs -f
  ```
- If `mws-app` or `msr` won't start, check whether `db-init` completed successfully (look for `/tmp/ready` or logs in `dcc/logs`).

---

## 📌 Notes

- Replace image names (`mws:10.15`, `msr:10.15`, `dcc:10.15`) with your actual image names or tags as required.
- Ensure your license files and server directories exist where expected.

---
