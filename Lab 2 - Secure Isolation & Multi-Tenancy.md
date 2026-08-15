# Secure Isolation & Multi-Tenancy

**Name:** Muhammad Asyraf kasyfi bin jafri  
**Course:** IKB42603 Cloud Computing  
**Lab:** Lab 2 — Secure Isolation and Multitenancy

## Objective

This lab demonstrates how a shared Kubernetes cluster can host multiple tenants while reducing cross-tenant risk. The implementation uses separate namespaces, resource quotas, network policies, RBAC, and secure data removal.

## Environment

- Kubernetes cluster: `csse-lab2` (kind)
- Tenants: `tenant-a` and `tenant-b`
- Workload: `web` pod and ClusterIP Service in each tenant
- Test image: `curlimages/curl`
- Persistent-data demonstration: Docker volume `csse-vol`

## Task 1 — Create isolated tenant workloads

Two namespaces were prepared: `tenant-a` and `tenant-b`. Each namespace contains its own `web` pod and a `web` ClusterIP Service. Although the service names are the same, namespaces keep their Kubernetes object names separate.

Commands used to confirm the workloads:

```bash
kubectl get pods,svc -n tenant-a
kubectl get pods,svc -n tenant-b
```

Observed results:

- `tenant-a`: `web-79d9f568b9-mhg1z` was Running and the `web` service had ClusterIP `10.96.8.177` on port `80/TCP`.
- `tenant-b`: `web-79d9f568b9-5mbnc` was Running and the `web` service had ClusterIP `10.96.204.36` on port `80/TCP`.

Evidence: 

<img width="581" height="124" alt="image" src="https://github.com/user-attachments/assets/33fe9d76-5625-4541-883e-98f4c2d022ce" />

<img width="577" height="131" alt="image" src="https://github.com/user-attachments/assets/5910f117-e4ea-4760-8604-7d369ba43bdc" />


## Task 2 — Demonstrate the default network behaviour

Before applying a NetworkPolicy, a temporary curl pod in `tenant-a` accessed the `tenant-b` service address:

```bash
kubectl -n tenant-a run probe --rm -it --image=curlimages/curl --restart=Never -- \
  curl -s -m 5 http://10.96.204.36 -o /dev/null -w 'HTTP %{http_code}\n'
```

The request returned `HTTP 200`. This shows that namespaces alone do **not** restrict pod-to-pod traffic; Kubernetes networking is permissive by default unless a supported network-policy implementation enforces rules.

Evidence: 

<img width="768" height="142" alt="image" src="https://github.com/user-attachments/assets/36275da7-02de-4362-9898-ab7cf1363b89" />



## Task 3 — Limit tenant resource consumption

A `ResourceQuota` named `tenant-a-quota` was applied in `tenant-a` and inspected with:

```bash
kubectl describe resourcequota tenant-a-quota -n tenant-a
```

The quota permits at most five pods, one CPU request, and `512Mi` of memory requests. At verification time, one pod was in use and no CPU or memory requests had been consumed.

| Resource | Used | Hard limit |
| --- | ---: | ---: |
| Pods | 1 | 5 |
| Requested CPU | 0 | 1 |
| Requested memory | 0 | 512Mi |

Resource quotas prevent one tenant from exhausting shared cluster capacity and affecting other tenants.

Evidence: 

<img width="509" height="164" alt="image" src="https://github.com/user-attachments/assets/19d7e300-0514-4ccb-bec5-3e54e6fe96b7" />



## Task 4 — Enforce default-deny ingress for tenant B

A namespace-wide NetworkPolicy was created in `tenant-b`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: tenant-b
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

Applied with:

```bash
kubectl apply -f -
```

The empty `podSelector` selects every pod in `tenant-b`. Because no ingress rules are present, inbound traffic to those selected pods is denied. Re-running the curl test from `tenant-a` returned `HTTP 000` and the probe terminated with an error, confirming that the previous access path was blocked.

Evidence: 

<img width="496" height="202" alt="image" src="https://github.com/user-attachments/assets/9c23c5e5-fbfe-4e44-99fe-ab40c2f1eb11" />

<img width="728" height="126" alt="image" src="https://github.com/user-attachments/assets/60f79033-f675-4100-a0ab-98950fca966d" />



## Task 5 — Verify least-privilege RBAC

The permissions of the configured service account (`$SA`) were checked with Kubernetes' authorization test:

```bash
kubectl auth can-i get secrets -n tenant-a --as=$SA
kubectl auth can-i get secrets -n tenant-b --as=$SA
```

The service account was allowed to get secrets in `tenant-a` (`yes`) and denied the same action in `tenant-b` (`no`). This verifies namespace-scoped RBAC: the identity has only the permissions assigned for its own tenant.

Evidence: 

<img width="479" height="50" alt="image" src="https://github.com/user-attachments/assets/507e6d64-1b29-4bb3-8fc1-ead9d2dd48ac" />
<img width="462" height="73" alt="image" src="https://github.com/user-attachments/assets/68b2b198-3364-4508-b46f-ee66010147d1" />




## Task 6 — Remove sensitive data securely

First, a temporary Alpine container wrote a record containing sensitive content into a mounted Docker volume, synchronized it, removed the file, searched the volume, and reported completion:

```bash
docker run --rm -v csse-vol:/data alpine sh -c \
  'echo SENSITIVE-PATIENT-RECORD > /data/phi.txt; sync; rm /data/phi.txt; \
   grep -a SENSITIVE /data/* 2>/dev/null; echo scan-done'
```

The container then created a second file and overwrote its contents with zero bytes before deleting it:

```bash
docker run --rm -v csse-vol:/data alpine sh -c \
  'echo SENSITIVE > /data/phi2.txt; sync; \
   dd if=/dev/zero of=/data/phi2.txt bs=1k count=1 conv=notrunc; rm /data/phi2.txt; \
   echo wiped'
```

The `dd` output confirms that 1,024 bytes were written and the command reported `wiped`. This is a demonstration of overwrite-before-delete; it should not be treated as a universal guarantee on copy-on-write filesystems, SSDs, managed storage, or snapshot-enabled volumes.

Evidence:

<img width="620" height="86" alt="image" src="https://github.com/user-attachments/assets/1abf2660-16c6-4de8-915b-1c34c4e1eda5" />


<img width="707" height="156" alt="image" src="https://github.com/user-attachments/assets/3b93a04c-f50f-4806-9424-b8c276b29f3c" />

 

## Final verification

The final checks showed that `tenant-b/default-deny-ingress` exists and that `tenant-a-quota` remains configured with its stated limits:

```bash
kubectl get networkpolicy -A
kubectl describe resourcequota tenant-a-quota -n tenant-a
```

Evidence: 

<img width="545" height="238" alt="image" src="https://github.com/user-attachments/assets/c16a404a-a304-4bf1-be2a-4abb2de4c7b2" />







## Cleanup and wrap-up

The lab resources were removed with:

```bash
kind delete cluster --name csse-lab2
docker volume rm csse-vol
```

The output confirms deletion of the kind cluster and Docker volume.

Evidence: 

<img width="363" height="95" alt="image" src="https://github.com/user-attachments/assets/15781623-5357-4118-8c68-ab08cc191891" />

## Short-answer question

### Question 1: Default Network Behavior & Risks
**Why can containers in different namespaces reach each other by default, and why is that dangerous in multi-tenant cloud?**

* **Reason:** By default, Kubernetes employs an open, flat networking approach that allows direct IP addresses to be used for communication across all pods across namespaces. Network borders are not provided by namespaces; they only offer administrative scopes and logical grouping.
* **Risk:** An attacker can freely undertake lateral movement, monitor internal networks, and get access to sensitive endpoints or services operating in Tenant B on the same physical cluster in a multi-tenant cloud by breaching a container in Tenant A.

---

### Question 2: Default-Deny Principle & Network Policy
**Explain the default-deny principle and how your Network Policy implements it.**

* **Default-Deny Principle:**All network communication should be inherently denied unless an explicit rule allows it, according to the Zero Trust security principle (least privilege access).
* **Implementation:** In `tenant-b`, the `default-deny-ingress` NetworkPolicy applies an empty `podSelector: {}` with `policyTypes: [Ingress]`. All inbound traffic that isn't specifically whitelisted is dropped, and all pods in that namespace are chosen.

---

### Question 3: Container vs. VM Isolation Strength
**How do virtual machines and containers differ in isolation strength? When would you add a VM boundary?**

* **Isolation Difference:** 
  * *Containers* share the host OS kernel and rely on Linux primitives (`namespaces`, `cgroups`), making them vulnerable to kernel-level privilege escalation or host compromise.
  * *Virtual Machines (VMs)* use a hypervisor to provide dedicated hardware virtualization and run independent guest kernels, offering significantly stronger boundary isolation.
* **When to add a VM boundary:** When running untrusted/third-party code, untrusted multitenant workloads, or highly sensitive applications subject to strict regulatory compliance (e.g., PCI-DSS, HIPAA).

---

### Question 4: Data Remanence & Cryptographic Erasure
**What is data remanence, and why is cryptographic erasure the preferred cloud solution?**

* **Data Remanence:** the remaining logical or physical data on underlying storage medium following the execution of normal file deletion commands (such as `rm`).
* **Why Cryptographic Erasure:** Physical destruction or overwrite wiping is not possible since cloud tenants do not have low-level block access or physical access to shared cloud storage disks. Cryptographic erasure makes the remaining data on shared storage permanently unrecoverable by encrypting data while it is at rest and destroying the particular decryption keys.

---

### Question 5: Isolation Dimension Mapping
**Which of the three isolation dimensions (compute, network, storage) did each task exercise?**

| Task | Primary Isolation Dimension |
| :--- | :--- |
| **Task 1: Two Tenants on One Cluster** | **Compute Isolation** (Logical separation via Kubernetes Namespaces) |
| **Task 2: Observe Default-Open Risk** | **Network Isolation** (Demonstrating missing network boundaries) |
| **Task 3: Contain Noisy Neighbour** | **Compute Isolation** (Resource allocation control via `ResourceQuota`) |
| **Task 4: Default-Deny Network Isolation** | **Network Isolation** (Traffic segmentation via `NetworkPolicy`) |
| **Task 5: Storage & Secret Isolation** | **Storage Isolation** (RBAC authorization protecting secrets) |
| **Task 6: Data Remanence & Secure Deletion** | **Storage Isolation** (Volume lifecycle, retention, and sanitization) |

## Conclusion

The lab successfully demonstrated practical isolation controls for a shared Kubernetes environment. The initial `HTTP 200` response established the default cross-tenant network risk, while the later `HTTP 000` result demonstrated mitigation using default-deny ingress. Quotas constrained resource use, RBAC limited secret access to the appropriate namespace, and the data-removal exercise reinforced that sensitive tenant data must be handled deliberately throughout its lifecycle.
