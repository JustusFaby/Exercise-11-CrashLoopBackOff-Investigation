# Exercise 11 – CrashLoopBackOff Investigation

## Objective

Investigate and identify the root cause of a Kubernetes pod entering a CrashLoopBackOff state.

---

## Scenario

A pod named `payment-service` was failing and continuously restarting.

### Observed Status

```bash
kubectl get pods
```

Output:

```text
payment-service   CrashLoopBackOff
```

### Application Logs

```bash
kubectl logs <payment-pod>
```

Output:

```text
Connecting to DB...
```

The application attempted to connect to a PostgreSQL database and exited when the connection failed.

---

## Investigation Tasks

The following areas were investigated:

### 1. DNS Investigation

Checked whether the database service name could be resolved inside the cluster.

Commands Used:

```bash
kubectl get svc

kubectl run debug --image=busybox -it --rm -- sh

nslookup postgres-service
```

Result:

* Service name resolved successfully.
* DNS functionality was verified.

**Status:** PASS

---

### 2. Secret Investigation

Verified whether application failure was caused by missing or invalid secrets.

Commands Used:

```bash
kubectl get secrets

kubectl describe secret postgres-secret
```

Result:

* Secret existed and contained the required keys.
* No authentication or secret-related errors were observed.

**Status:** PASS

---

### 3. Database Investigation

Verified whether the PostgreSQL database was available.

Commands Used:

```bash
kubectl get deployments

kubectl get pods
```

Result:

* PostgreSQL deployment was unavailable.
* No running database pod existed behind the service.

**Status:** FAIL

---

## Root Cause Analysis

The `payment-service` application attempted to connect to PostgreSQL on port `5432`.

Since the PostgreSQL deployment was unavailable, the connection failed and the application exited with an error.

Kubernetes continuously restarted the failed container, resulting in repeated crashes and a CrashLoopBackOff condition.

---

## Solution

Recreated the PostgreSQL deployment.

Command:

```bash
kubectl apply -f postgres-deployment.yaml
```

Verification:

```bash
kubectl get pods
```

Output:

```text
postgres           Running
payment-service    Running
```

The application successfully connected to the database and recovered.

---

## Commands Demonstrated

```bash
kubectl get pods

kubectl logs <payment-pod>

kubectl describe pod <payment-pod>

kubectl get svc

kubectl run debug --image=busybox -it --rm -- sh

nslookup postgres-service

kubectl get deployments

kubectl apply -f postgres-deployment.yaml

kubectl get pods -w
```

---

## Project Structure

```text
exercise11/
│
├── payment.yaml
├── postgres-deployment.yaml
├── postgres-service.yaml
├── secret.yaml
└── README.md
```

---

## Demo Video

Add your demonstration video link below:

**Demo Video:**
[PASTE_DEMO_VIDEO_LINK_HERE]

---

## Key Learnings

* Understanding CrashLoopBackOff behavior.
* Using logs and events for troubleshooting.
* Investigating Kubernetes DNS issues.
* Verifying Secrets and Service configurations.
* Identifying database availability problems.
* Performing root cause analysis and recovery.
