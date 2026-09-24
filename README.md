# K9s + Kubernetes Context Cheatsheet

> Quick reference for day-to-day Kubernetes debugging and production work.

---

## 🧠 Mental Model

```text
Kubernetes
│
├── Context
│   ├── Cluster
│   ├── User / Credentials
│   └── Default Namespace
│
└── Resources
    ├── Pods
    ├── Deployments
    ├── Services
    ├── Jobs
    └── Ingresses
```

### Remember

```text
Context   = Where am I operating?
Namespace = Which area of that cluster am I looking at?
```

---

# 1. Kubernetes Contexts

## See all contexts

```bash
kubectl config get-contexts
```

Example:

```text
CURRENT   NAME              CLUSTER
*         dev-cluster       dev
          staging-cluster   staging
          prod-cluster      prod
```

`*` = current context.

---

## See current context

```bash
kubectl config current-context
```

---

## Switch context

```bash
kubectl config use-context <context-name>
```

Example:

```bash
kubectl config use-context staging-cluster
```

Verify:

```bash
kubectl config current-context
```

---

## Inspect current context

```bash
kubectl config view --minify
```

Useful when you want to know exactly what the current context points to.

---

# 2. K9s Context Management

Start k9s:

```bash
k9s
```

Open context selector:

```text
:ctx
```

Then:

```text
↑ / ↓    Navigate
Enter    Select context
Esc      Go back
```

### Typical workflow

```text
:ctx
  ↓
Select cluster
  ↓
:ns
  ↓
Select namespace
  ↓
:po
```

---

# 3. Namespaces

## List namespaces

K9s:

```text
:ns
```

Kubectl:

```bash
kubectl get namespaces
```

or:

```bash
kubectl get ns
```

---

## Change namespace in k9s

```text
:ns
```

Select namespace → `Enter`

---

## Kubectl: specify namespace

```bash
kubectl get pods -n <namespace>
```

Example:

```bash
kubectl get pods -n payments
```

---

# 4. Essential K9s Resources

| K9s Command | Resource    |
| ----------- | ----------- |
| `:po`       | Pods        |
| `:deploy`   | Deployments |
| `:rs`       | ReplicaSets |
| `:svc`      | Services    |
| `:ns`       | Namespaces  |
| `:nodes`    | Nodes       |
| `:jobs`     | Jobs        |
| `:cronjobs` | CronJobs    |
| `:ing`      | Ingresses   |
| `:cm`       | ConfigMaps  |
| `:secrets`  | Secrets     |
| `:events`   | Events      |

---

# 5. Essential K9s Actions

Select a resource and use:

| Key            | Action           |
| -------------- | ---------------- |
| `Enter`        | Inspect resource |
| `l`            | View logs        |
| `d`            | Describe         |
| `s`            | Shell into pod   |
| `e`            | Edit resource    |
| `Ctrl+d`       | Delete resource  |
| `Esc`          | Go back          |
| `/`            | Filter           |
| `f`            | Follow logs      |
| `q` / `Ctrl+c` | Quit             |

> Exact key bindings can vary slightly by k9s version/configuration. `?` inside k9s shows the current keybindings.

---

# 6. Pods

Go to pods:

```text
:po
```

Useful workflow:

```text
Select Pod
    │
    ├── l → Logs
    ├── d → Describe
    ├── s → Shell
    └── Ctrl+d → Delete
```

---

# 7. Logs

Select pod:

```text
l
```

Follow live logs:

```text
f
```

For a pod with multiple containers, use the container selection/navigation available in k9s.

### Kubectl equivalent

```bash
kubectl logs <pod>
```

Follow:

```bash
kubectl logs -f <pod>
```

Specific container:

```bash
kubectl logs <pod> -c <container>
```

Previous crashed container:

```bash
kubectl logs <pod> --previous
```

---

# 8. Describe Resources

Select resource:

```text
d
```

This is useful for debugging:

* Scheduling problems
* Image pull failures
* Readiness/liveness failures
* Volume mount problems
* Container states
* Kubernetes events

Kubectl equivalent:

```bash
kubectl describe pod <pod-name>
```

---

# 9. Shell Into a Pod

Select pod:

```text
s
```

Then investigate:

```bash
env
pwd
ls
```

Test connectivity:

```bash
curl localhost:8080
```

or:

```bash
curl http://<service-name>:<port>
```

Kubectl equivalent:

```bash
kubectl exec -it <pod> -- /bin/sh
```

Some images use bash:

```bash
kubectl exec -it <pod> -- /bin/bash
```

---

# 10. Kubernetes Events

K9s:

```text
:events
```

Kubectl:

```bash
kubectl get events
```

For a namespace:

```bash
kubectl get events -n <namespace>
```

Events are especially useful for:

```text
FailedScheduling
FailedMount
ImagePullBackOff
ErrImagePull
BackOff
Unhealthy
Killing
```

---

# 11. Deployments

K9s:

```text
:deploy
```

Check:

```text
READY
UP-TO-DATE
AVAILABLE
```

Remember the relationship:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
```

If a deployment is unhealthy:

```text
:deploy
    ↓
select deployment
    ↓
d
    ↓
inspect ReplicaSet
    ↓
:po
    ↓
inspect pods
```

---

# 12. Services

K9s:

```text
:svc
```

Useful for checking:

* Service exists
* Port configuration
* Target port
* Selector
* Endpoints

Kubectl:

```bash
kubectl get svc
```

Detailed:

```bash
kubectl describe svc <service>
```

---

# 13. Filtering

Inside a resource list:

```text
/
```

Then type the filter.

Example:

```text
/api
```

Useful when you have hundreds of pods/resources.

---

# 14. Production Debugging Flow

When someone says:

> "The API is failing."

Use this flow:

```text
1. Check Context
       ↓
   :ctx

2. Check Namespace
       ↓
   :ns

3. Check Deployment
       ↓
   :deploy

4. Check Pods
       ↓
   :po

5. Check Pod Status
       ↓
   READY / RESTARTS / AGE

6. Check Logs
       ↓
   l

7. Follow Logs
       ↓
   f

8. Describe Pod
       ↓
   d

9. Check Events
       ↓
   :events

10. Shell into Pod if required
       ↓
   s

11. Check Service
       ↓
   :svc
```

---

# 15. Pre-Production Safety Check ⚠️

Before doing anything destructive:

```text
:ctx
```

Confirm:

```text
Cluster = correct?
```

Then:

```text
:ns
```

Confirm:

```text
Namespace = correct?
```

Only then consider:

```text
Ctrl+d
```

or editing resources.

### Especially be careful with:

```text
Ctrl+d
e
```

because these can modify/delete resources.

---

# 16. Useful Kubectl Equivalents

| K9s       | Kubectl                   |
| --------- | ------------------------- |
| `:po`     | `kubectl get pods`        |
| `:deploy` | `kubectl get deployments` |
| `:svc`    | `kubectl get services`    |
| `:ns`     | `kubectl get namespaces`  |
| `:nodes`  | `kubectl get nodes`       |
| `:events` | `kubectl get events`      |
| `l`       | `kubectl logs`            |
| `d`       | `kubectl describe`        |
| `s`       | `kubectl exec -it`        |
| `Ctrl+d`  | `kubectl delete`          |
| `e`       | `kubectl edit`            |

---

# 17. Must-Know Kubectl Commands

### Current context

```bash
kubectl config current-context
```

### All contexts

```bash
kubectl config get-contexts
```

### Switch context

```bash
kubectl config use-context <context>
```

### Current context configuration

```bash
kubectl config view --minify
```

### Pods

```bash
kubectl get pods
```

### Pods across all namespaces

```bash
kubectl get pods -A
```

### Deployment

```bash
kubectl get deployments
```

### Services

```bash
kubectl get svc
```

### Detailed pod information

```bash
kubectl describe pod <pod>
```

### Logs

```bash
kubectl logs <pod>
```

### Follow logs

```bash
kubectl logs -f <pod>
```

### Previous container logs

```bash
kubectl logs <pod> --previous
```

### Execute command in pod

```bash
kubectl exec -it <pod> -- /bin/sh
```

### Events

```bash
kubectl get events
```

---

# 18. The 15 Things to Memorize First

If you don't want to memorize everything, start with these:

```text
:ctx       → Context
:ns        → Namespace
:po        → Pods
:deploy    → Deployments
:svc       → Services
:events    → Events

Enter      → Inspect
l          → Logs
f          → Follow logs
d          → Describe
s          → Shell
e          → Edit
Ctrl+d     → Delete
/          → Filter
Esc        → Back
```

And these kubectl commands:

```bash
kubectl config current-context
kubectl config get-contexts
kubectl config use-context <context>

kubectl get pods
kubectl get pods -A
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs -f <pod>
kubectl exec -it <pod> -- /bin/sh
kubectl get events
```

---

# 🎯 Mental Model for Production Debugging

Don't randomly jump between resources.

Think in this order:

```text
WHERE?
  ↓
Context
  ↓
Namespace

WHAT?
  ↓
Deployment
  ↓
ReplicaSet
  ↓
Pod

WHY?
  ↓
Pod Status
  ↓
Logs
  ↓
Describe
  ↓
Events

CAN I REACH IT?
  ↓
Service
  ↓
Ingress
  ↓
Network / Dependencies
```

This mental model is more valuable than memorizing every k9s command.
