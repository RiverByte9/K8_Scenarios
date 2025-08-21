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

Master node वर default taint:

```bash
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


