# 🛋️ Secure Open Format Architecture (SOFA) for Data Products

**SOFA** is a composable, Zero Trust-aligned architecture for building secure, tenant-owned, and portable Data Products using open formats and ephemeral compute. It blends fine-grained IAM, open storage and table formats, and modern orchestration to support auditability, portability, and compliance by design.

---

## 🎯 Purpose

- Enforce **Zero Trust** principles across all pipeline layers  
- Support **data portability** using open formats (Parquet, DuckDB)  
- Empower **tenant ownership** through isolated, S3-compatible buckets  
- Replace static infrastructure with **stateless, ephemeral components**

---

## 🧱 Core Architecture Stack

| **Layer**       | **Component**          | **Role**                                                                 |
|-----------------|------------------------|--------------------------------------------------------------------------|
| Secrets Mgmt    | Doppler                | Secure, centralized secret storage and injection                        |
| Storage         | Tigris Object Storage  | S3-compatible backend for tenant-isolated Parquet data                  |
| Lakehouse       | DuckLake + DuckDB      | Analytical engine with snapshot isolation and open format support       |
| Compute         | GitHub Codespaces      | Ephemeral dev environments with no local data exposure                  |
| Orchestration   | Tower.dev              | Python-native workflow automation and secrets rotation                  |
| Networking      | Tailscale              | Device-trusted, IP-scoped overlay network with tailnet enforcement      |
| Query Interface | MotherDuck             | Read-only SaaS access to DuckDB datasets                                |

---

## 🔐 Zero Trust Enforcement

| **Principle**           | **Enforcement**                                                                 |
|-------------------------|----------------------------------------------------------------------------------|
| No implicit trust       | No reliance on VPCs or internal IPs; all access via IAM                         |
| Per-request authN/Z     | Tigris IAM + Doppler-injected credentials + IP restrictions                     |
| Least privilege         | IAM-scoped keys per project, environment, and object prefix                     |
| Short-lived credentials | Tower.dev flows periodically rotate access keys and propagate to Doppler        |
| Secure transport        | All ingress/egress flows through Tailscale exit nodes                           |
| Auditable access        | Logs from Doppler, Tailscale, Tigris, Tower sent to central SIEM                |
| Data exfil prevention   | Codespaces restricts USB, clipboard, and local disk access                      |

---

## 🔓 Open Format Principles

- **Open File Formats**: All datasets stored as columnar Parquet  
- **Open Query Engine**: DuckDB for local/embedded; MotherDuck for hosted SQL  
- **Interoperable Storage**: Tigris object storage with S3-compatible APIs  
- **Tenant Isolation**: Each client has dedicated buckets and IAM policies  
- **Vendor Neutrality**: No proprietary formats; all interfaces are replaceable  

---

## 🔁 Secrets & Key Rotation Flow

1. **Generate** Tigris access key with IP and path restrictions  
2. **Store** in Doppler (`TIGRIS_KEY_ID`, `TIGRIS_KEY_SECRET`)  
3. **Inject** into runtime:  
   - **Codespaces** via `doppler run`  
   - **Tower.dev jobs** via `doppler secrets download`  
4. **Rotate** with Tower-scheduled task:  
   - Call `RotateAccessKey` via Tigris API  
   - Update Doppler using CLI/API  
5. **Restrict** access using IAM conditions like:  

```json
"Condition": {
  "IpAddress": { "aws:SourceIp": "198.51.100.42" }
}
```

---

## 🌐 Environment Access Model

| **Environment** | **Purpose**        | **Routing**            | **Secret Source** | **IAM Constraints**              |
|-----------------|--------------------|-------------------------|-------------------|----------------------------------|
| Codespaces      | Dev & analytics    | Tailscale exit node     | Doppler           | IP + path + action               |
| Tower.dev       | CI / orchestration | Tailscale or NAT        | Doppler           | IP + path + TTL-based rotation   |
| MotherDuck      | Query layer        | Direct static IP access | *(optional)*      | Read-only IAM with prefix scope  |

---

## ✅ Architecture Benefits

### 🔐 Security
- All access bound to identity, IP, path, and method  
- No hardcoded or shared credentials  
- Audit trail across all layers (secrets, storage, access)  

### 🧳 Portability
- Open formats (Parquet, DuckDB) across the entire lakehouse  
- S3-compatible storage for tenant-controlled exports  
- Tenant-owned, tenant-isolated, and vendor-neutral by default  

### ⚙️ Operational Simplicity
- Python-native DAGs in Tower.dev  
- Ephemeral orchestration and compute environments  
- No long-running services or persistent control planes  

---

## 🧩 Summary

**SOFA** combines Zero Trust with open data design to enable compliant, scalable, and future-proof pipelines. Every interaction is scoped, ephemeral, and observable — ensuring data stays owned, protected, and portable.

---

## 🔗 Integration with IndustryVault Architecture

SOFA serves as the foundational architecture for IndustryVault's data products, particularly in the mortgage servicing domain. It provides:

### Mortgage Data Product Benefits
- **Regulatory Compliance**: Built-in audit trails for mortgage servicing requirements
- **Data Portability**: Open formats enable easy migration and vendor independence
- **Tenant Isolation**: Secure multi-tenancy for different mortgage servicers
- **Real-time Processing**: Ephemeral compute enables rapid data processing for mortgage analytics

### Implementation Strategy
- **Phase 1**: Implement SOFA for new data products and pilot programs
- **Phase 2**: Migrate existing data pipelines to SOFA architecture
- **Phase 3**: Full SOFA adoption across all IndustryVault data products

### Competitive Advantage
- **Technology Leadership**: First-mover advantage in secure, open-format data architecture
- **Compliance Edge**: Zero Trust architecture addresses regulatory requirements proactively
- **Cost Efficiency**: Ephemeral compute reduces infrastructure costs
- **Vendor Independence**: Open formats reduce lock-in and increase flexibility

---

*This SOFA architecture document should be reviewed quarterly and updated based on implementation experience, security requirements, and technology evolution.* 