## Opening Shell to Pod
`kubectl exec -it POD_NAME -- /bin/bash`

## Installing Ping for DNS Testing on Pod
`apt-get update && apt-get install iputils-ping -y`

## kubectl Commands

### List all pods and their IP addresses
`kubectl get pods --all-namespaces -o wide`

### List all daemonsets
`kubectl get daemonsets --all-namespaces`

### Describe a particular daemonset
`kubectl describe daemonset DAEMONSET_NAME`

### Collect K8s logs from a specific pod
`kubectl logs POD_NAME -n NAMESPACE_NAME`

### List all deployments
`kubectl get deployments --all-namespaces`

### Restart a deployment
`kubectl -n NAMESPACE_NAME rollout restart deployment DEPLOYMENT_NAME`

## Querying Control Plane Endpoints

### API Server
`kubectl exec -ti POD_NAME -- wget -O - localhost:8080/metrics`

### ETCD
`kubectl exec -ti POD_NAME -- wget -O - localhost:4001/metrics`

### Scheduler
`kubectl exec -ti POD_NAME -- wget -O - localhost:10251/metrics`

### Controller Manager
`kubectl exec -ti POD_NAME -- wget -O - localhost:10252/metrics`

### List all pods in use by New Relic
`kubectl exec -ti $(kubectl get pods --all-namespaces --field-selector spec.nodeName=$(kubectl get nodes -l node-role.kubernetes.io/master="" -o jsonpath="{.items[0].metadata.name}") -l name=newrelic-infra -o jsonpath="{.items[0].metadata.name}") -- wget -O - localhost:10251/metrics`
