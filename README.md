# azure-log-analytics-poc
# Azure Log Analytics CORS Misconfiguration – Proof of Concept

## 🔍 Overview
This repository contains a **Proof of Concept (PoC)** for a **CORS misconfiguration** in the Azure Log Analytics API.

The API endpoint `https://api.loganalytics.azure.com/v1/workspaces/{workspaceId}/query` returns the header:
This allows **any website** to read responses from the API. Combined with the public `DEMO_KEY` (documented by Microsoft), an attacker can exfiltrate sensitive log data without authentication.

---

## 🚨 Vulnerability Details

| Item | Description |
|------|-------------|
| **Vulnerability Type** | Cross-Origin Resource Sharing (CORS) Misconfiguration |
| **Affected Endpoints** | `api.loganalytics.azure.com` <br> `api.loganalytics.io` (legacy) |
| **CORS Header** | `Access-Control-Allow-Origin: *` |
| **Authentication Required** | ❌ No – public `DEMO_KEY` works |
| **Exploitability** | Any website can fetch data cross‑origin |

---

## 📊 Data Exposed (Retrieved from Demo Workspace)

| Table | Data Leaked | Sensitivity |
|-------|-------------|-------------|
| **AppExceptions** | Full stack traces, source code file paths, method names, assembly versions | 🔴 **Critical** |
| **AppEvents** | Azure Subscription ID, Resource Group, AKS Cluster Name, Kubernetes API Host | 🔴 **Critical** |
| **AppDependencies** | Storage Account Names, Blob/Queue URLs, Container Names | 🔴 **High** |
| **AppTraces** | Function execution logs, invocation IDs, timestamps | 🟡 Medium |
| **ContainerLog_RST** | Kubernetes node errors and warnings | 🟡 Medium |
| **Usage** | Tenant ID, Resource URIs, data ingestion volumes | 🔴 **High** |

### Sample Leaked Data (Stack Trace)
```json
{
  "Method": "LoadGenerator.Function1+<Run>d__2.MoveNext",
  "Assembly": "LoadGenerator, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null",
  "OuterMessage": "Exception while executing function: Function2",
  "InnermostMessage": "An invalid request URI was provided. The request URI must either be an absolute URI or BaseAddress must be set."
}{
  "CLUSTER_RESOURCE_ID": "/subscriptions/ebb79bc0-aa86-44a7-8111-cabbe0c43993/resourceGroups/CH1-FabrikamRG/providers/Microsoft.ContainerService/managedClusters/CH1-GearamaAKS",
  "KUBERNETES_SERVICE_HOST": "gearamay37ha6p32ytzw-9en4utq9.hcp.eastus.azmk8s.io",
  "TenantId": "81a662b5-8541-481b-977d-5d956616ac5e"
}
