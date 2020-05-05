# New Relic Kubernetes Integration Config

## Base-integration installation

### Install kube-state-metrics
`curl -L -o kube-state-metrics-1.7.2.zip https://github.com/kubernetes/kube-state-metrics/archive/v1.7.2.zip && unzip kube-state-metrics-1.7.2.zip && kubectl apply -f kube-state-metrics-1.7.2/kubernetes`

### Download the Integration Config File
`curl -O https://download.newrelic.com/infrastructure_agent/integrations/kubernetes/newrelic-infrastructure-k8s-latest.yaml`

### Gather the Cluster Name (if installed using `kubeadm`)
`kubectl -n kube-system get configmap kubeadm-config -o yaml | grep clusterName`

### Update the Integration Config File

`vim newrelic-infrastructure-k8s-latest.yaml`
```yaml
env:
  - name: "NRIA_LICENSE_KEY"
    value: "<KEY>"
  - name: "CLUSTER_NAME"
    value: "<NAME>"
  - name: "NRIA_VERBOSE"
    value: 0
```

### Verify `kube-state-metrics` is installed
`kubectl get pods --all-namespaces | grep kube-state-metrics`

### Create the daemon set and verify
`kubectl create -f newrelic-infrastructure-k8s-latest.yaml`

`kubectl get daemonsets`
```
NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
newrelic-infra   2         2         2       2            2           <none>          90s
```

## Control Plane Metrics

### Validate Master Node Label
Master node is required to be labeled with with `node-role.kubernetes.io/master=""` or `kubernetes.io/role=master`

Validate: 

`kubectl get nodes --show-labels | grep master`

Add Label:

`kubectl label nodes NODE_NAME kubernetes.io/role=master`

### Validate Control Plane Components Labels

Validate:

`kubectl get pods --all-namespaces --show-labels | grep -E 'apiserver|etcd|scheduler|controller-manager'`

Add Labels as Necessary:

`kubectl label pods POD_NAME LABEL=VALUE -n NAMESPACE`


API SERVER: `kubectl label pods POD_NAME k8s-app=kube-apiserver -n NAMESPACE`

ETCD: `kubectl label pods POD_NAME k8s-app=etcd-manager-main -n NAMESPACE`

SCHEDULER: `kubectl label pods POD_NAME k8s-app=kube-scheduler -n NAMESPACE`

CONTROLLER MANAGER: `kubectl label pods POD_NAME k8s-app=kube-controller-manager -n NAMESPACE`

### Validate ETCD Setup

Installation of CF SSL tools to create certificate (if required)
```
sudo curl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -o /usr/local/bin/cfssl
sudo curl https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -o /usr/local/bin/cfssljson
sudo chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson
```

Create `csr` and `pem` files:
```json
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
    "hosts": [
        "localhost"
    ],
    "CN": "newrelic-infra.pod.cluster.local",
    "key": {
        "algo": "ecdsa",
        "size": 256
    }
}
EOF
```

Use the CA of ETCD to sign your newly created CSR
```
kubectl cp $(kubectl get pods -l k8s-app=etcd-manager-main -n kube-system -o jsonpath="{.items[0].metadata.name}"):/etc/kubernetes/pki/etcd-manager/etcd-clients-ca.crt ./cacert -n kube-system
kubectl cp $(kubectl get pods -l k8s-app=etcd-manager-main -n kube-system -o jsonpath="{.items[0].metadata.name}"):/etc/kubernetes/pki/etcd-manager/etcd-clients-ca.key ./cacert.key -n kube-system
```

## Kubernetes Events

### Download the YAML config and edit the Cluster Name and License Key


`curl -O https://download.newrelic.com/infrastructure_agent/integrations/kubernetes/nri-kube-events-latest.yaml`

```
clusterName: "YOUR_CLUSTER_NAME"
[...]
	- name: "NRIA_LICENSE_KEY"
	value: "YOUR_LICENSE_KEY" 
```

### Deploy the integration into the cluster
`kubectl apply -f nri-kube-events-latest.yaml`

**NOTE YOU MAY NEED TO UPDATE THE DEPLOYMENTS API VERSION**
```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: nr-kube-events
  namespace: default
  labels:
    app: nr-kube-events
spec: ...
```

Update to: `apiVersion: apps/v1`

## Kubernetes Logs

### Clone the Logging Integration GitHub repo
```
git clone https://github.com/newrelic/kubernetes-logging.git
cd kubernetes-logging/
```

### Add your license key
`vim new-relic-fluent-plugin.yml`

```yaml
      containers:
      - name: newrelic-logging
        env:
          - name: ENDPOINT
            value : "https://log-api.newrelic.com/log/v1"
          - name: SOURCE
            value: "kubernetes"
          - name: LICENSE_KEY
            value: <LICENSE_KEY>
          - name: BUFFER_SIZE
            value: "256000"
          - name: MAX_RECORDS
            value: "500"
          - name: LOG_LEVEL
            value: "info"
          - name: PATH
            value: "/var/log/containers/*.log"
```

### Deploy the integration into the cluster
`kubectl apply -f .`

**NOTE YOU MAY NEED TO UPDATE THE DAEMONSET API VERSION**
```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata: ...
```

Update to: `apiVersion: apps/v1`
