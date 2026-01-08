Below are **architecture diagrams suitable for direct inclusion in a GitHub README**, written and structured from a **paranoid, compliance-driven architect’s perspective**.

They use **Mermaid**, which is natively supported by GitHub and avoids external tooling, screenshots, or undocumented dependencies.

You can paste these **as-is** into your `README.md` or `/docs/architecture.md`.

---

## 1️⃣ High-Level System Architecture (Investor / Auditor View)

```mermaid
flowchart TB
    Internet((Internet))
    Cloudflare[Cloudflare\nDDoS • CDN • WAF]
    Domain[gyvenimo-kelias.lt\n~1000 subdomains]

    Internet --> Cloudflare --> Domain

    Domain --> ProxmoxVE[Proxmox VE 9.1\nPhysical Cluster]

    ProxmoxVE --> Frontend[Next.js SSR\nMaster Website]
    ProxmoxVE --> Tenants[Multi-Tenant Websites\n(LXC Containers)]
    ProxmoxVE --> K8s[Kubernetes Cluster\n(kubeadm)]
    ProxmoxVE --> CoreVMs[Core Service VMs]

    CoreVMs --> Databases[(PostgreSQL\nMongoDB)]
    CoreVMs --> CMS[1C-Bitrix CMS]
    CoreVMs --> CRM[Bitrix24 CRM]

    K8s --> APIs[Stateless APIs\nAI • Integrations]

    ProxmoxVE --> PBS[Proxmox Backup Server 4.1]

    PBS --> Backups[(Immutable Backups)]
```

### **Key Compliance Notes**

* No direct internet access to databases
* All inbound traffic filtered via Cloudflare
* Backup system is isolated and immutable

---

## 2️⃣ Proxmox VE Workload Isolation Model (Security-Critical)

```mermaid
flowchart LR
    subgraph ProxmoxVE[Proxmox VE Host]
        subgraph VM_Tier[Virtual Machines – Hard Security Boundary]
            DBVM[Database VM\nPostgreSQL/MongoDB]
            CMSVM[CMS VM\n1C-Bitrix]
            CRMVM[CRM VM\nBitrix24]
            K8sCP[K8s Control Plane VM]
            BlockchainVM[Blockchain Services VM]
        end

        subgraph LXC_Tier[LXC Containers – Scale Units]
            Site1[Church Site 1]
            Site2[Church Site 2]
            SiteN[Church Site N]
            API1[Django API]
            Worker1[Background Worker]
        end
    end
```

### **Architectural Rule**

> *Anything containing irreplaceable state or compliance risk runs in a VM.*

---

## 3️⃣ Multi-Tenant Website Architecture (Isolation by Design)

```mermaid
flowchart TB
    subgraph TenantA[Parish A]
        A_Web[LXC: Web]
        A_DB[(Tenant A DB)]
    end

    subgraph TenantB[Funeral Home B]
        B_Web[LXC: Web]
        B_DB[(Tenant B DB)]
    end

    subgraph SharedInfra[Shared but Controlled]
        Auth[Auth Service]
        CDN[Cloudflare]
        Logs[Centralized Logs]
    end

    A_Web --> A_DB
    B_Web --> B_DB

    A_Web --> Auth
    B_Web --> Auth

    Auth --> Logs
```

### **Compliance Guarantees**

* No cross-tenant database access
* No shared credentials
* Tenant data deletion is deterministic and auditable

---

## 4️⃣ Kubernetes Responsibility Boundary (No “K8s Everywhere”)

```mermaid
flowchart LR
    subgraph Kubernetes[Kubernetes Cluster]
        API[API Services]
        AI[AI / LLM Integrations]
        Events[Event Processing]
    end

    subgraph OutsideK8s[Outside Kubernetes]
        DB[(Databases)]
        CMS[CMS]
        Backups[PBS]
    end

    API --> DB
    AI --> API
```

### **Hard Rule**

> *Kubernetes does not own state.*

---

## 5️⃣ Backup & Disaster Recovery Architecture (Ransomware-Resistant)

```mermaid
flowchart TB
    Proxmox[Proxmox VE]
    PBS[Proxmox Backup Server]
    Storage[(Backup Storage)]
    Verification[Backup Verification]
    Restore[Controlled Restore]

    Proxmox --> PBS
    PBS --> Storage
    Storage --> Verification
    Verification --> Restore
```

### **Design Assumptions**

* Backups must survive:

  * Admin error
  * Malware
  * Insider threat
* Restore is **slower but guaranteed**

---

## 6️⃣ Scaling Model: 30 → 1000 Sites

```mermaid
flowchart LR
    Phase1[Phase 1\n~30 Sites]
    Phase2[Phase 2\n100–300 Sites]
    Phase3[Phase 3\n1000+ Sites]

    Phase1 --> Phase2 --> Phase3

    Phase1:::low
    Phase2:::med
    Phase3:::high

    classDef low fill:#e0f7fa
    classDef med fill:#fff3e0
    classDef high fill:#ffebee
```

### **Scaling Strategy**

* Horizontal first
* Automation before expansion
* No architectural changes between phases

---

## 7️⃣ Trust & Network Segmentation Model (Zero Trust)

```mermaid
flowchart TB
    Internet --> Cloudflare
    Cloudflare --> DMZ[DMZ Network]
    DMZ --> AppNet[Application Network]
    AppNet --> DataNet[Data Network]
    DataNet --> BackupNet[Backup Network]

    style DataNet fill:#ffe0e0
    style BackupNet fill:#ffcccc
```

### **Security Policy**

* Each network has explicit allow rules
* No lateral movement by default
* Backup network is write-only from Proxmox

---

## ✅ How to Organize These in the Repo

Recommended structure:

```text
/docs
  architecture.md   ← diagrams + explanations
  security.md
  compliance.md
  scaling.md
```

---

## Next Professional Steps (Recommended)

I can now:

1. **Add a threat model (STRIDE-based)**
2. **Create a tenant lifecycle diagram**
3. **Produce a Proxmox network VLAN design**
4. **Add an investor-friendly simplified diagram**
5. **Generate an ISO/VM/LXC build matrix**

State the next step explicitly.
