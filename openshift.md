# OpenShift Interview Preparation

A structured guide to OpenShift interview questions, ordered from fundamentals to
production troubleshooting. Source notes: [`openshift_questions`](./openshift_questions).

Aimed at DevOps / Kubernetes / OpenShift (SRE, Platform, Senior DevOps) interviews.

---

## Table of Contents

**Fundamentals**
1. [OpenShift Fundamentals](#1-openshift-fundamentals)
2. [OpenShift Architecture](#2-openshift-architecture)
3. [Projects and Namespaces](#3-projects-and-namespaces)

**Platform Features**
4. [Routes](#4-routes)
5. [Security Context Constraints (SCC)](#5-security-context-constraints-scc)
6. [Operators](#6-operators)
7. [ImageStreams](#7-imagestreams)
8. [Builds](#8-builds)

**Infrastructure**
9. [Networking](#9-networking)
10. [NetworkPolicy](#10-networkpolicy)
11. [Authentication and RBAC](#11-authentication-and-rbac)
12. [Storage](#12-storage)
13. [Monitoring](#13-monitoring)

**Interview Practice**
14. [Troubleshooting Scenarios](#14-troubleshooting-scenarios)
15. [Advanced Question Bank](#15-advanced-question-bank)
16. [Top 10 Priority Topics](#16-top-10-priority-topics)

---

## 1. OpenShift Fundamentals

### Q1. What is OpenShift?

OpenShift is Red Hat's enterprise Kubernetes platform. It provides Kubernetes plus
additional capabilities:

- Integrated authentication / RBAC
- Routes for exposing applications
- Operators and OperatorHub
- ImageStreams
- Security Context Constraints (SCC)
- Integrated monitoring and logging
- Developer tooling
- Enterprise lifecycle and support

> **Interview answer:** "OpenShift is essentially an enterprise Kubernetes platform from
> Red Hat. It extends Kubernetes with features such as Routes, SCCs, Operators,
> ImageStreams and integrated platform services, while still using Kubernetes underneath."

### Q2. Kubernetes vs OpenShift

| Kubernetes | OpenShift |
| --- | --- |
| Container orchestration platform | Enterprise Kubernetes platform |
| `kubectl` | `oc` + `kubectl` |
| Ingress | Route + Ingress |
| Pod Security Admission | SCC |
| No built-in enterprise registry by default | Integrated registry capability |
| Basic Kubernetes installation | Opinionated enterprise platform |
| Operators available | Strong Operator / OLM integration |
| More DIY | More integrated |

> **Important:** OpenShift is *not* a replacement for Kubernetes. OpenShift is *built on*
> Kubernetes.

### Q3. What is `oc`?

`oc` is the OpenShift CLI. It also performs most Kubernetes operations because it
incorporates `kubectl` functionality.

```bash
oc get pods
oc get nodes
oc get projects
oc get routes
oc describe pod <pod>
oc logs <pod>
oc exec -it <pod> -- bash
```

---

## 2. OpenShift Architecture

### Q4. Explain OpenShift architecture

```
                    Users
                      |
                 oc / Web Console
                      |
                      v
              +---------------+
              | API Server    |
              +---------------+
                      |
          +-----------+-----------+
          |                       |
          v                       v
   Control Plane              Worker Nodes
   ------------               ------------
   API Server                 kubelet
   etcd                       CRI-O
   Scheduler                  kube-proxy / CNI
   Controllers                    |
                                  v
                                Pods
```

OpenShift additionally provides: OAuth, Routes, Ingress Controller, Operators, OLM,
Image Registry, SCC, and Monitoring.

### Q5. What runs on an OpenShift control plane?

Core Kubernetes components plus OpenShift-specific controllers and services.

```
oc command
   |
   v
API Server
   |
   +---- etcd
   |
   +---- Scheduler
   |
   +---- Controllers
```

| Component | Role |
| --- | --- |
| API Server | Entry point for all cluster operations |
| etcd | Cluster state store |
| Scheduler | Pod placement decisions |
| Controller Manager | Reconciliation loops |

---

## 3. Projects and Namespaces

### Q6. What is a Project?

An OpenShift Project is an enhanced Kubernetes namespace.

```bash
oc new-project payment
```

It provides isolation around applications, resources, users, RBAC, quotas, and network
policies.

> **Interview answer:** "A project provides a logical isolation boundary for workloads and
> users in OpenShift and is implemented on top of Kubernetes namespaces with additional
> OpenShift capabilities."

### Q7. Project vs Namespace

A namespace is a Kubernetes concept; a project is an OpenShift concept built around a
namespace.

```
OpenShift Project
       |
       v
Kubernetes Namespace
       |
       +---- Pods
       +---- Services
       +---- Routes
       +---- Secrets
       +---- ConfigMaps
```

---

## 4. Routes

> Very important for OpenShift interviews.

### Q8. What is an OpenShift Route?

A Route exposes a service outside the cluster.

```
Internet
   |
   v
https://myapp.example.com
   |
   v
OpenShift Router
   |
   v
Service
   |
   v
Pods
```

```bash
oc expose service myapp
oc get routes
```

### Q9. Route vs Kubernetes Ingress

```
Kubernetes:            OpenShift:
Client                 Client
  |                      |
  v                      v
Ingress                Route
  |                      |
  v                      v
Service                Service
```

Routes provide OpenShift-specific functionality: TLS termination, edge termination,
passthrough, re-encryption, and traffic configuration.

### Q10. Route TLS termination types

**Edge** — TLS terminates at the router.

```
Client --HTTPS--> Router (decrypts) --HTTP--> Service
```

**Passthrough** — Router does not decrypt TLS.

```
Client --HTTPS--> Router --HTTPS--> Pod
```

**Re-encrypt** — Router decrypts, then opens a new TLS connection to the backend.

```
Client --HTTPS--> Router (decrypt + re-encrypt) --HTTPS--> Pod
```

| Type | Router decrypts? | Backend connection |
| --- | --- | --- |
| Edge | Yes | HTTP |
| Passthrough | No | HTTPS (end-to-end) |
| Re-encrypt | Yes | HTTPS (new session) |

---

## 5. Security Context Constraints (SCC)

> Extremely important — the most common OpenShift-specific interview topic.

### Q11. What is an SCC?

SCC = Security Context Constraints. It controls what permissions a pod/container is
allowed to have:

- Run as root?
- Run as non-root?
- Use privileged mode?
- Host networking?
- Host PID?
- Allowed capabilities?
- SELinux context?
- Allowed volume types?

### Q12. Why does a workload work in Kubernetes but fail in OpenShift?

Very common interview scenario. Suppose the container expects:

```yaml
securityContext:
  runAsUser: 0
```

OpenShift may reject this because the workload is not permitted by the applicable SCC.
Typical errors:

```
container has runAsNonRoot and image will run as root
```

...or other permission / security-context errors.

### Q13. Common SCCs

```
restricted
restricted-v2
anyuid
privileged
hostnetwork
nonroot
```

```bash
oc get scc
oc describe scc restricted-v2
```

### Q14. Granting a service account access to an SCC

```bash
oc adm policy add-scc-to-user anyuid \
  -z my-service-account \
  -n my-project
```

Do not casually grant `privileged` or `anyuid`.

> **Strong interview answer:** "I would first determine why the workload requires the
> elevated permission. I would prefer modifying the container to run securely as a
> non-root user rather than granting a broader SCC. If an elevated SCC is genuinely
> required, I would bind it only to the specific service account."

---

## 6. Operators

### Q15. What is an Operator?

Software that automates management of an application using Kubernetes APIs. It watches
Kubernetes resources and continuously reconciles toward the desired state.

```
Kafka
 |
 v
Kafka Operator
 |
 +---- Create Kafka cluster
 +---- Configure brokers
 +---- Create users
 +---- Manage topics
 +---- Perform upgrades
 +---- Recover / reconcile
```

### Q16. What is Operator Lifecycle Manager (OLM)?

OLM manages Operators in OpenShift:

- Installing operators
- Upgrading operators
- Managing operator versions
- Managing dependencies
- Controlling operator installation scope

### Q17. What is OperatorHub?

A catalog of Operators available for OpenShift.

```
OperatorHub
    |
    +---- Kafka
    +---- PostgreSQL
    +---- Elasticsearch
    +---- etc.
```

---

## 7. ImageStreams

### Q18. What is an ImageStream?

An OpenShift abstraction for managing container images. Instead of hardcoding
`registry/app:v1` everywhere, OpenShift tracks image updates through:

```
ImageStream
     |
     v
ImageStreamTag
     |
     v
Deployment
```

```bash
oc get imagestream
```

### Q19. What happens when an ImageStreamTag changes?

A deployment or build can be triggered, depending on configuration.

```
New image pushed
       |
       v
ImageStream updated
       |
       v
Image change trigger
       |
       v
New deployment
       |
       v
New Pods
```

---

## 8. Builds

### Q20. What is a BuildConfig?

A BuildConfig defines how OpenShift should build an application / container image.

```
Git repository
      |
      v
BuildConfig
      |
      v
Build
      |
      v
Container Image
      |
      v
ImageStream
```

```bash
oc get bc
oc describe bc <name>
```

---

## 9. Networking

### Q21. Explain OpenShift networking

External traffic path:

```
External Client
      |
      v
Load Balancer
      |
      v
Ingress Controller
      |
      v
Route
      |
      v
Service
      |
      v
Pod
```

Inside the cluster:

```
Pod
 |
 | CNI
 v
Cluster Network
 |
 v
Service Network
```

Modern OpenShift deployments commonly use **OVN-Kubernetes** as the network plugin.

### Q22. What is OVN-Kubernetes?

An OpenShift networking implementation based on OVN, Open vSwitch, OpenFlow, Geneve
tunneling, and the Kubernetes networking APIs.

Provides: pod networking, service networking, network policies, routing, and network
isolation.

---

## 10. NetworkPolicy

### Q23. Does OpenShift support NetworkPolicy?

Yes.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

This denies ingress traffic to the pods selected by the policy.

### Q24. SCC vs NetworkPolicy

Very common interview comparison.

| | SCC | NetworkPolicy |
| --- | --- | --- |
| Answers | "What can the container do?" | "Who can talk to the pod?" |
| Examples | Run as root, privileged mode, host resources | Frontend → Backend allowed, Frontend → DB denied |
| Scope | Pod security capabilities | Pod-to-pod traffic |

```
Frontend ---> Backend
Backend  ---> Database
Frontend -X-> Database
```

---

## 11. Authentication and RBAC

### Q25. How does authentication work in OpenShift?

OpenShift runs an OAuth server.

```
User
 |
 v
OpenShift OAuth
 |
 v
Identity Provider
 |
 v
Authentication
 |
 v
RBAC authorization
 |
 v
API Server
```

Identity providers: LDAP, GitHub, HTPasswd, OpenID Connect, Active Directory
integrations.

### Q26. Authentication vs Authorization

- **Authentication:** Who are you?
- **Authorization:** What are you allowed to do?

```
User
 |
 | Authentication
 v
Authenticated
 |
 | RBAC
 v
Can user create deployment?
```

### Q27. How do you check permissions?

```bash
oc auth can-i get pods
oc auth can-i get pods --as=user1
```

Very useful troubleshooting command.

---

## 12. Storage

### Q32. How does storage work in OpenShift?

Same fundamental Kubernetes model:

```
Pod
 |
 v
PVC
 |
 v
PV
 |
 v
StorageClass
 |
 v
Storage backend
```

Dynamic provisioning:

```
PVC
 |
 v
StorageClass
 |
 v
CSI Driver
 |
 v
Cloud Disk
```

### Q33. What is CSI?

Container Storage Interface — allows Kubernetes/OpenShift to communicate with storage
systems.

```
OpenShift
   |
   v
CSI Driver
   |
   v
AWS EBS / Azure Disk / Ceph / etc.
```

---

## 13. Monitoring

### Q34. How does monitoring work in OpenShift?

OpenShift ships an integrated monitoring stack: Prometheus, Alertmanager, Thanos, and
Grafana / console visualization.

```
Pod
 |
 v
Metrics endpoint
 |
 v
Prometheus
 |
 +---- Alert rules
 |
 v
Alertmanager
```

### Q35. How would you monitor an OpenShift cluster?

| Layer | What to monitor |
| --- | --- |
| Cluster | CPU, memory, disk, API server, etcd, node health |
| Workload | Pod restarts, CPU/memory, request rate, latency, errors |
| Kubernetes | Scheduling failures, PVC failures, API latency, controller health |
| Application | HTTP 5xx, Kafka lag, DB connections, queue depth |

---

## 14. Troubleshooting Scenarios

> Particularly important for senior DevOps interviews.

### Q28. Pod stuck in `CrashLoopBackOff`

Go systematically:

```bash
oc get pod
oc describe pod <pod>
oc logs <pod>
oc logs <pod> --previous
```

Then check:

1. Application logs
2. Exit code
3. Environment variables
4. Secrets
5. ConfigMaps
6. Resource limits
7. Probes
8. Volume mounts
9. SCC / security context
10. Dependencies

### Q29. Pod stuck in `Pending`

```bash
oc describe pod <pod>   # look at Events
```

Potential reasons: insufficient CPU, insufficient memory, node selector mismatch,
taints/tolerations, PVC not bound, affinity rules, ResourceQuota.

### Q30. Pod stuck in `ContainerCreating`

```bash
oc describe pod <pod>
```

Typical causes: image pull, PVC mounting, CNI networking, secret/ConfigMap,
volume attachment, SCC.

### Q31. Route returns 503

Start from outside and move inward:

```
Client
 ↓
Route
 ↓
Router
 ↓
Service
 ↓
Endpoints
 ↓
Pods
```

```bash
oc get route
oc describe route <route>

oc get svc
oc describe svc <service>

oc get endpoints <service>

oc get pods -o wide
```

Test the service directly:

```bash
oc port-forward svc/myservice 8080:8080
```

If the service has no endpoints, the selector does not match the pod labels:

```
Service selector
       |
       X
       |
   Pod labels
```

```bash
oc get svc myservice -o yaml
oc get pods --show-labels
```

---

## 15. Advanced Question Bank

### Architecture

1. Explain OpenShift architecture.
2. OpenShift vs Kubernetes?
3. What happens when you run `oc apply -f deployment.yaml`?
4. What happens when you create a Route?
5. Explain OpenShift ingress architecture.
6. How does the OpenShift API server work?
7. How does etcd participate in OpenShift?

### Security

8. What is SCC?
9. SCC vs Pod Security Admission?
10. Why does a container fail in OpenShift but work in vanilla Kubernetes?
11. How do you allow a container to run as root?
12. Why shouldn't you give the `privileged` SCC to everyone?
13. Explain OpenShift RBAC.
14. Authentication vs authorization.
15. How do you troubleshoot permission denied errors?

### Networking

16. Explain OpenShift networking.
17. What is OVN-Kubernetes?
18. Route vs Ingress?
19. Edge vs passthrough vs re-encrypt?
20. How does traffic reach a pod from the Internet?
21. How do NetworkPolicies work?
22. Service vs Route?
23. How would you troubleshoot a 503 Route?

### Storage

24. Explain PVC / PV / StorageClass.
25. What happens if a PVC is stuck `Pending`?
26. What is CSI?
27. How do you troubleshoot a pod unable to mount a volume?

### Operators

28. What is an Operator?
29. What is OLM?
30. What is OperatorHub?
31. Explain Operator reconciliation.
32. How would you troubleshoot an Operator?
33. What is a CRD?
34. CRD vs CR?
35. What happens when you create a custom resource?

### Troubleshooting

36. Pod stuck `Pending`.
37. Pod stuck `CrashLoopBackOff`.
38. Pod stuck `ContainerCreating`.
39. Route returning 503.
40. Node `NotReady`.
41. `ImagePullBackOff`.
42. PVC `Pending`.
43. API server unavailable.
44. etcd unhealthy.
45. Operator stuck progressing.

---

## 16. Top 10 Priority Topics

With a Kubernetes / AWS / DevOps background, don't over-invest in memorizing `oc`
commands. Focus on explaining the underlying mechanism.

1. Kubernetes vs OpenShift
2. OpenShift architecture
3. SCC — and why containers fail because of SCC
4. Route vs Ingress
5. Route TLS termination
6. OVN-Kubernetes networking
7. Operators + OLM + reconciliation
8. OpenShift authentication + RBAC
9. Troubleshooting Route → Service → Endpoint → Pod
10. What happens internally when `oc apply` is executed

If you can explain those from packet/request flow plus Kubernetes internals, you'll sound
considerably more senior than someone who only knows `oc` commands.

---

## Quick Command Reference

| Task | Command |
| --- | --- |
| List pods | `oc get pods` |
| List projects | `oc get projects` |
| Create project | `oc new-project <name>` |
| List routes | `oc get routes` |
| Expose a service | `oc expose service <svc>` |
| Pod logs (previous crash) | `oc logs <pod> --previous` |
| Shell into pod | `oc exec -it <pod> -- bash` |
| List SCCs | `oc get scc` |
| Inspect an SCC | `oc describe scc restricted-v2` |
| Bind SCC to SA | `oc adm policy add-scc-to-user anyuid -z <sa> -n <ns>` |
| Check permission | `oc auth can-i get pods --as=<user>` |
| Service endpoints | `oc get endpoints <service>` |
| Pod labels | `oc get pods --show-labels` |
| Port-forward a service | `oc port-forward svc/<svc> 8080:8080` |
| List BuildConfigs | `oc get bc` |
| List ImageStreams | `oc get imagestream` |
