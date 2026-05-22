# TodoApp Deployment Instructions

## 1. Deploy the application

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/nodeport.yml
```

Verify pods are running:

```bash
kubectl get pods -n mateapp
kubectl get deployment -n mateapp
kubectl get hpa -n mateapp
```

---

## 2. Accessing the app

**Via port-forward:**

```bash
kubectl port-forward service/todoapp-clusterip 8080:80 -n mateapp
```

Open in browser: `http://localhost:8080`

**Via NodePort:**

Open in browser: `http://localhost:30080`

---

## 3. Resource requests and limits

| | CPU | Memory |
|---|---|---|
| requests | 100m | 128Mi |
| limits | 200m | 256Mi |

- **requests** define the minimum guaranteed resources per pod. 100m CPU and 128Mi memory is enough for a Django app in idle state.
- **limits** are set to 2x requests to allow bursts without risking node resource exhaustion.
- HPA uses requests as a baseline to calculate utilization percentage, so accurate requests are important for correct autoscaling.

---

## 4. HPA configuration

- **minReplicas: 2** — ensures high availability at all times, even under no load.
- **maxReplicas: 5** — limits resource usage, enough to handle traffic spikes for a small app.
- **CPU threshold: 50%** — triggers scaling before the pod becomes overloaded.
- **Memory threshold: 50%** — memory leaks or heavy requests will trigger scale-out early.

---

## 5. RollingUpdate strategy

RollingUpdate replaces pods one by one without downtime.

Default values used:
- **maxSurge: 1** — allows 1 extra pod during update (3 pods max during rollout)
- **maxUnavailable: 1** — allows 1 pod to be unavailable during update

This ensures the app stays available during deployments while keeping resource usage controlled.
