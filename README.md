# ServerClient

## Setup
Deploy 3 Windows nodes on AKS.
1. Get names from all Windows Nodes:
```
kubectl get node --template "{{range .items}}{{.metadata.name}}{{\"\n\"}}{{end}}" --selector="kubernetes.io/os=windows"
```
2.	Add labels to nodes.
    *	`kubectl label nodes <node-1-name> testtype=onenode `
    *	`kubectl label nodes <node-2-name> <node-3-name> testtype=twonode`


## Deployment
This chart can deploy:
  * Server:
    * 2 services of type LoadBalancer with etp:local
    * 2 server deployments (`--set serverInstanceCount=2`)
    * Each server deployment with 2 pod replicas used by the service as backends (`--set deployment.server.serverReplicaCount=2`)
  * Client:
    * 2 client deployments (`--set clientInstanceCount=2`)
    * Each client deployment with 1 pod replica connecting to the respective target service (`--set deployment.client.clientReplicaCount=1`)

Important to note that there cannot be more client deployments than server deployments, because client `client-n` will try to connect to `service-n`. If you want multiple clients connecting to same Service, use `--set deployment.client.clientReplicaCount=2` instead.

## Deployment Configuration Examples

### Create test namespace and deploy with etp:local
```
helm install dsr --namespace test ./serverclient
```

### Deploy with etp:cluster to default namespace
```
helm install --set service.externalTrafficPolicy=Cluster non-dsr ./serverclient
```

### Deploy ClusterIP services:
```
helm install --set service.type=ClusterIP full-coral ./serverclient
```

### Deploy 2x etp:cluster, 3x etp:local
```
helm install --set service.externalTrafficPolicy=Cluster non-dsr ./serverclient
helm install --set clientInstanceCount=3 --set serverInstanceCount=3 dsr ./serverclient
```

### Deploy servers on one cluster, clients on another cluster
This only works if service type is set to LoadBalancer (used by default):
```
# Deploy servers onto cluster 1
helm install --kubeconfig <path_to_cluster1_config> --set clientInstanceCount=0 must-use-same-name ./serverclient
# Deploy clients only onto cluster 2
helm install --kubeconfig <path_to_cluster2_config> --set serverInstanceCount=0 must-use-same-name ./serverclient
```

### Wait for deployment to complete
`helm install --wait full-coral ./serverclient`

### To uninstall:
`helm uninstall full-coral`


