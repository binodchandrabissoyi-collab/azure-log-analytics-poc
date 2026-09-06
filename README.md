# Azure Log Analytics CORS Misconfiguration – Critical Chain (P1)

## 🔍 Overview
This repository contains a **Proof of Concept (PoC)** for a **critical vulnerability chain** in the Azure Log Analytics API:

1. **CORS misconfiguration** – `Access-Control-Allow-Origin: *`
2. **Unauthenticated data access** – public `DEMO_KEY` works without login.
3. **Bearer token extraction** – logs contain valid Kubernetes service account tokens.
4. **Full cluster takeover** – the extracted token grants administrative access to an AKS cluster.

---

## 🚨 Vulnerability Details

| Item | Description |
|------|-------------|
| **Vulnerability Type** | CORS Misconfiguration + Sensitive Data Leakage + Token Exposure |
| **Affected Endpoints** | `api.loganalytics.azure.com`<br>`api.loganalytics.io` (legacy) |
| **CORS Header** | `Access-Control-Allow-Origin: *` |
| **Authentication Required** | ❌ No – public `DEMO_KEY` works |
| **Exploitability** | Any website can fetch logs cross‑origin |
| **Impact** | **Critical (P1)** – Full compromise of AKS cluster |

---

## 🔥 Critical Chain – How It Works

1. **CORS Misconfiguration** – Any website can read responses from the Log Analytics API.
2. **Public `DEMO_KEY`** – Allows unauthenticated access to the demo workspace.
3. **Logs Contain Bearer Tokens** – The `AppEvents` table contains Kubernetes API requests with `Authorization: Bearer <token>`.
4. **Token Abuse** – The extracted token can be used to authenticate to the AKS cluster and perform administrative actions (e.g., patch namespaces, read secrets).

---

## 📊 Data Exposed (Retrieved from Demo Workspace)

| Table | Data Leaked | Sensitivity |
|-------|-------------|-------------|
| **AppExceptions** | Full stack traces, source code file paths, method names, assembly versions | 🔴 **Critical** |
| **AppEvents** | Azure Subscription ID, Resource Group, AKS Cluster Name, Kubernetes API Host, **Bearer Tokens** | 🔴 **Critical** |
| **AppDependencies** | Storage Account Names, Blob/Queue URLs, Container Names | 🔴 **High** |
| **AppTraces** | Function execution logs, invocation IDs, timestamps | 🟡 Medium |
| **ContainerLog_RST** | Kubernetes node errors and warnings | 🟡 Medium |
| **Usage** | Tenant ID, Resource URIs, data ingestion volumes | 🔴 **High** |

### Sample Extracted Bearer Token (from `AppEvents`)
```json
"Authorization": "Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6ImtmUUVCcDd6Z0NHT2wzUk45aTFNSUpyU1pROXRKcWYzcm5mNUVQbjVxeWMifQ.eyJhdWQiOlsiaHR0cHM6Ly9nZWFyYW1heTM3aGE2cDMyeXR6dy05ZW40dXRxOS5oY3AuZWFzdHVzLmF6bWs4cy5pbyIsIlwiZ2VhcmFtYXkzN2hhNnAzMnl0enctOWVuNHV0cTkuaGNwLmVhc3R1cy5hem1rOHMuaW9cIiJdLCJleHAiOjE4MjAxNTA1NTIsImlhdCI6MTc4ODYxNDU1MiwiaXNzIjoiaHR0cHM6Ly9nZWFyYW1heTM3aGE2cDMyeXR6dy05ZW40dXRxOS5oY3AuZWFzdHVzLmF6bWs4cy5pbyIsImp0aSI6ImY3Mzk4ZGM2LTgzNTMtNDI4NC1iMjhkLTJjN2Q5NGYxNzAyNSIsImt1YmVybmV0ZXMuaW8iOnsibmFtZXNwYWNlIjoia3ViZS1zeXN0ZW0iLCJub2RlIjp7Im5hbWUiOiJha3Mtbm9kZXBvb2wxLTk0MTY0NDUzLXZtc3MwMDAwMHkiLCJ1aWQiOiIwZDU3M2JhYS02YWU5LTRkYzMtOTBhNC1iOTEwZDE1MjVhNzAifSwicG9kIjp7Im5hbWUiOiJhcHAtbW9uaXRvcmluZy13ZWJob29rLTViYjc2ODQ0OWYtazh0cnIiLCJ1aWQiOiJmNWJlNTBkZi01ODc5LTQyOGEtYmFkYy04MTc5Y2JlNjBkOTgifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImFwcC1tb25pdG9yaW5nLXdlYmhvb2siLCJ1aWQiOiJiNDM5Mzc2YS1hNDQwLTQ4YTctOTg4ZC1kZWIxYjUxYmEwNTAifSwid2FybmFmdGVyIjoxNzg4NjE4MTU5fSwibmJmIjoxNzg4NjE0NTUyLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YXBwLW1vbml0b3Jpbmctd2ViaG9vayJ9.OmY9l2E791T1E2kVg1LitOBsAYvLKIl0OaHAkUzsnq8DZUZyJKjTVN7ai66c7hfjb14qggfr6DoM1l5ZBANXNMB7lpzQjttIKI5RJicbIPKjArS4aSJda7EzO0ISbxkW_3yCuxQca6ok9YFybCz6h2f7vR_zSUsWSMB7pBeFYzs9M37Nh89yx_s_gNOYShogGQkcyP8LOpoERgiJDswKTYIMB9DmUa7Encxzkv-GswifXq4BdY0690k8xc4Q0OsDj42KRNi2tmZbinAWLdsK388aL3aY4EChQxXntUZcsM3u8k926TRE6eNjBvf_65dMYVIU_BvYz7oeyMEdNXtNfW5L971KMZ4f4ZOoYcZEZqpHUNLfT7hA1CxIcdupBwgSsw6an0LI9pBTNA_9jxSuk-G-sD1H8C_qHhOSti2af_92DL3ihao27akoEwxCFPgSUQdx1eMierVhzbGsW-KNqiuqjvw-87p3koiIMhnxWKf5FPn5ZdcSZPpuBdEz9B3vk743GNMKhO7_c4Vfw7VGGMmBG7RoGQxKpL15mwTBtKuLOhHyYk343iwjdiWODZMMHX3s36vRnBwsTH2Y0MmQawUaPtkRwLrc0w7EtX_bzTTvLZXiWFA7fTd-Ip91j1x49TRSk2EIHLyVjC2BNUQZA3pypfEgDu-Gk1W6qSuC2o8"
