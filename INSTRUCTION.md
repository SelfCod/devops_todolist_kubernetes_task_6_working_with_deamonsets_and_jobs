# Django Todo llist testing instructions
## 1. Applying daemonset.yml and cronjob.yml manifests:
```bash
kubectl apply -f .infrastructure/{daemonset.yml,cronjob.yml}
```


## 2. Validate the solution
Here's how to view logs for both your `DaemonSet` and `CronJob` after applying the manifest:

### For the **DaemonSet** (`mydaemonset`):
Since DaemonSet creates one Pod per node, use:
```bash
# View logs from ALL DaemonSet Pods (all nodes)
kubectl logs -n mateapp -l key=busybox --all-containers=true --tail=10

# View logs from a specific DaemonSet Pod
kubectl logs -n mateapp $(kubectl get pods -n mateapp -l key=busybox -o jsonpath='{.items[0].metadata.name}')
```

### For the **CronJob** (`myjob`):
CronJobs create individual Job Pods, so you need to:
```bash
# 1. First find the latest Job
LATEST_JOB=$(kubectl get jobs -n mateapp -o jsonpath='{.items[?(@.metadata.ownerReferences[0].name=="myjob")].metadata.name}' | awk '{print $1}')

# 2. Then view its logs
kubectl logs -n mateapp $(kubectl get pods -n mateapp -l job-name=$LATEST_JOB -o jsonpath='{.items[0].metadata.name}')
```

### One-liner for both:
```bash
# DaemonSet logs
echo "=== DaemonSet Logs ===" && kubectl logs -n mateapp -l key=busybox --all-containers=true --tail=5
