# Hands-on: DaemonSet Scheduling
##Key Topics Covered:
1. Cluster Setup

```bash
kubectl get nodes
```

Output:

```
NAME                         STATUS   ROLES           AGE     VERSION
demo-cluster-control-plane   Ready    control-plane   10m     v1.33.1
demo-cluster-worker          Ready    <none>          9m      v1.33.1
demo-cluster-worker2         Ready    <none>          9m      v1.33.1
```

Default Taint on Master Node


```bash
kind create cluster --name demo-cluster --config kind-config.yaml
kubectl describe node demo-cluster-control-plane | grep Taints
```

Output:

```
node-role.kubernetes.io/control-plane:NoSchedule
```

---

Shows how to check nodes and identify master node taints
Explains the default node-role.kubernetes.io/control-plane:NoSchedule taint



2. Three Practical Scenarios:

#Scenario 1: DaemonSet without toleration

Result: Pods only run on worker nodes

```
kubectl apply -f ds-no-toleration.yaml
kubectl get pods -o wide

```


#Scenario 2: DaemonSet with toleration only

Result: Pods mostly on worker nodes, master possible but not guaranteed

```
kubectl apply -f ds-toleration-only.yaml
kubectl get pods -o wide
```

#Scenario 3: DaemonSet with toleration + nodeSelector

Result: Pod guaranteed to run on master node
```

kubectl apply -f ds-toleration-nodeselector.yaml
kubectl get pods -o wide
```


# Kubernetes Toleration and NodeSelector Scenarios

### Pod Scheduling Behavior on Different Node Types


**Scenario 1: ds-no-toleration**
- Toleration: ❌
- NodeSelector: ❌  
- Master Node: ⬜ Pod NOT scheduled
- Worker1: 🟩 Pod running
- Worker2: 🟩 Pod running
- Notes: Master node has NoSchedule taint; Pod blocked

**Scenario 2: ds-toleration-only**
- Toleration: ✅
- NodeSelector: ❌
- Master Node: 🟦 Pod scheduled (tolerated)  
- Worker1: 🟩 Pod running
- Worker2: 🟩 Pod running
- Notes: Toleration allows Pod on master, but placement depends on scheduler

**Scenario 3: ds-toleration-nodeselector**  
- Toleration: ✅
- NodeSelector: ✅
- Master Node: 🟦 Pod scheduled (guaranteed)
- Worker1: ⬜ Pod NOT scheduled  
- Worker2: ⬜ Pod NOT scheduled
- Notes: NodeSelector forces Pod to master node

## Detailed Explanations

### Scenario 1: ds-no-toleration
```yaml
# No toleration, no nodeSelector
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-no-toleration
spec:
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example
        image: nginx
```
**Result:** Pods only run on worker nodes because master has `NoSchedule` taint.

### Scenario 2: ds-toleration-only
```yaml
# With toleration, no nodeSelector
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-toleration-only
spec:
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"
      containers:
      - name: example
        image: nginx
```
**Result:** Pods can run on all nodes including master, but scheduler decides final placement.

### Scenario 3: ds-toleration-nodeselector
```yaml
# With toleration and nodeSelector
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-toleration-nodeselector
spec:
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"
      nodeSelector:
        node-role.kubernetes.io/control-plane: ""
      containers:
      - name: example
        image: nginx
```
**Result:** Pod is guaranteed to run ONLY on master node due to nodeSelector constraint.

## Key Concepts

### Taints and Tolerations
- **Taint**: Applied to nodes to repel pods
- **Toleration**: Applied to pods to allow scheduling on tainted nodes
- **Effect Types**:
  - `NoSchedule`: New pods won't be scheduled
  - `PreferNoSchedule`: Scheduler tries to avoid scheduling
  - `NoExecute`: Existing pods will be evicted

### NodeSelector
- **Purpose**: Constrains pods to run on specific nodes
- **Mechanism**: Matches node labels with pod requirements
- **Behavior**: Pod won't be scheduled if no nodes match the selector

## Common Master Node Taints

```bash
# Check current taints
kubectl describe nodes | grep Taints

# Common taints on master nodes:
node-role.kubernetes.io/control-plane:NoSchedule  # Kubernetes 1.24+
node-role.kubernetes.io/master:NoSchedule         # Older versions
```

## Testing Commands

```bash
# Create test DaemonSets
kubectl apply -f ds-no-toleration.yaml
kubectl apply -f ds-toleration-only.yaml
kubectl apply -f ds-toleration-nodeselector.yaml

# Check pod placement
kubectl get pods -o wide

# Check DaemonSet status
kubectl get ds
```


