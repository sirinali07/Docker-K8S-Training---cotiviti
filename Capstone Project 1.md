#  Kubernetes Capstone Project: Deploy and Auto-Scale a Web Application

> **Hands-on lab:** deploy a Python web server on Kubernetes, expose it with a Service, measure it with Metrics Server, and let the Horizontal Pod Autoscaler add Pods when demand grows.

| 🎯 Level | ⏱️ Suggested duration | 👥 Mode |
| :---: | :---: | :---: |
| **Beginner** | **60 minutes** | **Individual or pairs** |

### 📑 Contents

| # | Section | Audience |
| :-: | :-- | :-- |
| 1 | [🏢 Business scenario](#business-scenario) | Everyone |
| 2 | [🧰 Prerequisites](#prerequisites) | Everyone |
| 3 | [🏗️ Architecture](#architecture) | Everyone |
| 4 | [📝 Participant assignment](#participant-assignment) | Participants |
| 5 | [🛠️ Trainer reference solution](#trainer-reference-solution) | Trainer |
| 6 | [⏱️ Trainer session plan](#trainer-session-plan) | Trainer |
| 7 | [❓ Questions for participants](#questions-for-participants) | Participants (answer guide: Trainer) |
| 8 | [🏆 Evaluation rubric](#evaluation-rubric) | Trainer |
| 9 | [✅ Final verification checklist](#final-verification-checklist) | Trainer |
| 10 | [🧹 Cleanup](#cleanup) | Everyone |

---

<a id="business-scenario"></a>

## 🏢 1. Business scenario

> Imagine your company has launched a simple website where customers can view product information. Initially, two application instances are enough. During a promotional campaign, however, the number of visitors may increase.
>
> The company wants the application to remain available and handle increased demand without an administrator manually creating additional Pods.

**You are the Kubernetes administrator responsible for deploying the application.**

### 🎯 Your objectives

1. 🐍 Deploy a **Python HTTP Server** inside Kubernetes.
2. 📦 Run **two application replicas** using a **Deployment**.
3. 🔀 Expose the application through a **Service**.
4. 📏 Configure **CPU and memory requests and limits**.
5. 📊 Verify resource usage using **Metrics Server**.
6. ⚖️ Configure **Horizontal Pod Autoscaler (HPA)** to scale between **two and five replicas** based on CPU utilization.
7. 🚀 **Generate traffic** and observe whether Kubernetes increases the number of Pods.

---

<a id="prerequisites"></a>

## 🧰 2. Prerequisites

- [ ] A working Kubernetes cluster, such as **Minikube** or a training cluster.
- [ ] `kubectl` configured for the cluster.
- [ ] **Metrics Server** installed and working.
- [ ] Basic understanding of Pods, Deployments, Services, resource requests/limits, and HPA.

---

<a id="architecture"></a>

## 🏗️ 3. Architecture

```mermaid
flowchart TB
    user(["👤 <b>User / Browser</b>"])
    svc["🔀 <b>Service</b><br/>web-service : 80<br/><i>NodePort 30080</i>"]

    subgraph dep["📦 Deployment: web-app"]
        pod1["🐍 <b>Pod</b><br/>python:3.12-alpine<br/>port 8000"]
        pod2["🐍 <b>Pod</b><br/>python:3.12-alpine<br/>port 8000"]
    end

    ms["📊 <b>Metrics Server</b><br/>CPU and memory usage"]
    hpa["⚖️ <b>HPA</b><br/>target CPU 50%<br/>min 2 · max 5"]

    user -->|"HTTP request"| svc
    svc -->|"label app: web-app<br/>targetPort 8000"| pod1
    svc --> pod2
    pod1 -.->|"usage"| ms
    pod2 -.-> ms
    ms -->|"resource metrics"| hpa
    hpa ==>|"scales replicas<br/>2 ⇄ 5"| dep

    classDef client fill:#E8F0FE,stroke:#326CE5,color:#0B2A6F,stroke-width:2px
    classDef service fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px
    classDef pod fill:#FEF3C7,stroke:#D97706,color:#78350F,stroke-width:2px
    classDef metric fill:#FFE4E6,stroke:#E11D48,color:#881337,stroke-width:2px
    classDef scaler fill:#EDE9FE,stroke:#7C3AED,color:#3B0764,stroke-width:2px
    class user client
    class svc service
    class pod1,pod2 pod
    class ms metric
    class hpa scaler
    style dep fill:#FFFBEB,stroke:#D97706,stroke-width:2px,stroke-dasharray:6 4,color:#78350F

    linkStyle 0,1,2 stroke:#16A34A,stroke-width:2px
    linkStyle 3,4,5 stroke:#E11D48,stroke-width:2px
    linkStyle 6 stroke:#7C3AED,stroke-width:3px
```

**Legend:** 🟢 request traffic · 🔴 metrics flow · 🟣 scaling decisions

| Component | Role |
| :-- | :-- |
| 🔀 **Service** | Provides a stable endpoint and forwards traffic to matching Pods. |
| 📦 **Deployment** | Maintains the desired replica count. |
| 📊 **Metrics Server** | Provides resource metrics. |
| ⚖️ **HPA** | Uses those metrics to decide whether to change the replica count. |

### 🔄 How the pieces work together

```mermaid
sequenceDiagram
    autonumber
    actor U as 👤 User / Load generator
    participant S as 🔀 Service
    participant P as 🐍 Pods (web-app)
    participant M as 📊 Metrics Server
    participant H as ⚖️ HPA
    participant D as 📦 Deployment

    U->>S: HTTP requests (port 80)
    S->>P: Forward to matching Pods (port 8000)
    Note over P: Python's built-in HTTP server does little CPU work,<br/>so plain requests may not push CPU above the target

    loop Continuously
        M->>P: Collect CPU and memory usage
        H->>M: Read current CPU utilization
        Note over H: Compare utilization with the 50% target<br/>(measured against the 100m CPU request)
    end

    alt CPU utilization above target
        H->>D: Scale up (maximum 5 replicas)
        D->>P: Create additional Pods
    else CPU utilization below target
        H->>D: Scale down (minimum 2 replicas)
        D->>P: Remove surplus Pods
    end
```

> 💡 HPA calculates CPU utilization **relative to the requested CPU**, which is why CPU requests must be configured for autoscaling to work.

---

<a id="participant-assignment"></a>

## 📝 4. Participant assignment

> [!NOTE]
> Initially, participants should receive the requirements below, **not** the completed YAML files. Ask them to create the files independently before checking the reference solution.

```mermaid
flowchart LR
    t1["🧩 <b>Task 1</b><br/>Create a<br/>Deployment"] --> t2["🔀 <b>Task 2</b><br/>Create a<br/>Service"] --> t3["📊 <b>Task 3</b><br/>Check resource<br/>metrics"] --> t4["⚖️ <b>Task 4</b><br/>Configure<br/>HPA"] --> t5["🚀 <b>Task 5</b><br/>Generate<br/>traffic"]

    classDef task fill:#E8F0FE,stroke:#326CE5,color:#0B2A6F,stroke-width:2px
    class t1,t2,t3,t4,t5 task
    linkStyle 0,1,2,3 stroke:#326CE5,stroke-width:2px
```

### 🧩 Task 1 — Create a Deployment

Create a Deployment named `web-app` with these requirements:

| Setting | Requirement |
| :-- | :-- |
| Container image | `python:3.12-alpine` |
| Command | Run Python's built-in HTTP server on port `8000` |
| Replicas | `2` |
| CPU request | `100m` |
| CPU limit | `200m` |
| Memory request | `64Mi` |
| Memory limit | `128Mi` |

✅ **Verify** the Deployment and Pods.

### 🔀 Task 2 — Create a Service

Create a Service named `web-service` with these requirements:

| Setting | Requirement |
| :-- | :-- |
| Type | `NodePort` |
| Service port | `80` |
| Target port | `8000` |
| NodePort | `30080` |
| Pod selector | Label `app: web-app` |

✅ **Verify** the Service and its endpoints.

### 📊 Task 3 — Check resource metrics

- Run `kubectl top nodes`.
- Run `kubectl top pods`.
- Confirm CPU and memory usage is reported.
- Identify the resource requests and limits in the Deployment.

### ⚖️ Task 4 — Configure HPA

Configure HPA for Deployment `web-app`:

| Setting | Requirement |
| :-- | :-- |
| CPU utilization target | `50%` |
| Minimum replicas | `2` |
| Maximum replicas | `5` |

✅ **Verify** HPA status and events.

### 🚀 Task 5 — Generate traffic

- Generate repeated HTTP requests to `web-service`.
- Watch HPA metrics and Pod count.
- Record whether the replica count changes.
- If scaling does not happen, inspect CPU utilization, metrics, and HPA events.

### 📬 Submission requirements

Submit:

- [ ] `web-app.yaml`
- [ ] `service.yaml`
- [ ] HPA creation command **or** HPA YAML
- [ ] Output or screenshots from `kubectl get pods`, `kubectl get svc`, `kubectl top pods`, and `kubectl get hpa`
- [ ] A short explanation of how the Deployment, Service, resource settings, Metrics Server, and HPA work together

---

<a id="trainer-reference-solution"></a>
