# Kong Deployment
## Versions
* Kong AI Gateway 3.13
* vSphere Kubernetes 3.5 / VKr 1.34

## References
* https://developer.konghq.com/gateway/install/kubernetes/konnect/

## Requirements
### CLI Tools
* kubectl cli v1.34: https://dl.k8s.io/release/v1.34.0/bin/linux/amd64/kubectl
* vcf cli v9.0.1: https://packages.broadcom.com/artifactory/vcf-distro/vcf-cli/linux/amd64/v9.0.1/
* helm cli v3.19: https://get.helm.sh/helm-v3.19.0-linux-amd64.tar.gz

### Get files
```shell
git clone https://github.com/cleeistaken/kong-k8s-testing.git
```

### Required vSphere Steps
These steps require access to vCenter and typically handled by an infrastructure administrator.

1. In WCP, create a supervisor namespace (e.g. 'kong')
2. Add storage policy to supervisor namespace
   * vsan-esa-default-policy-raid5
3. Add VM classes to supervisor namespace
   * best-effort-medium
   * best-effort-2xlarge

**Note**: We are using the previous storage policies and VM classes as an example. If different policies and classes are desired, update the sample vks.yaml file with the required values.


## Deployment Procedure

### 1. Set environment variables
To facilitate the creation of multiple deployments in different environments we create and use variables throughout this sample deployment procedure.
```bash
# Update with correct values
export SUPERVISOR_IP="<supervisor_ip>"
export SUPERVISOR_USERNAME="<username>"
export SUPERVISOR_NAMESPACE_NAME="<supervisor_namespace>"

# These variables will be used to create things
export SUPERVISOR_CONTEXT="<supervisor_context>"
export CLUSTER_CONTEXT="<cluster_context>"
export CLUSTER_NAME="<vks_cluster_name>"
export CLUSTER_NAMESPACE_NAME="vks_cluster_namespace"

# Kong specific variables
export KONG_PAT="<kong_personal_access_token>"
```
<details>
<summary>Example</summary>

```bash
# Example
export SUPERVISOR_IP="192.168.1.1"          
export SUPERVISOR_USERNAME="user@vsphere.local"
export SUPERVISOR_NAMESPACE_NAME="kong"
export SUPERVISOR_CONTEXT="kong-ctx"
export CLUSTER_NAME="kong-vks"
export CLUSTER_CONTEXT="kong-vks-ctx"
export CLUSTER_NAMESPACE_NAME="kong-vks-ns"
export KONG_PAT="kpat_abc123..."
```
</details>
<br>
<br>


### 2. Clean kubectl and vcf configs
This step is not required but helps avoid issues related to stale contexts or collisions between environments using the same context names.
```shell
rm ~/.kube/config
rm -rf ~/.config/vcf/
```
<br>
<br>


### 3. Create supervisor context 
```bash
vcf context create "$SUPERVISOR_CONTEXT" \
  --endpoint "$SUPERVISOR_IP" \
  --username "$SUPERVISOR_USERNAME" \
  --insecure-skip-tls-verify
```

<details>
<summary>Expected output</summary>

```bash
[i] Some initialization of the CLI is required.
[i] Lets set things up for you.  This will just take a few seconds.

[i] 
[i] Initialization done!
[i] ==
[i] Auth type vSphere SSO detected. Proceeding for authentication...
Provide Password: 

Logged in successfully.

You have access to the following contexts:
   kong-ctx
   kong-ctx:kong

If the namespace context you wish to use is not in this list, you may need to
refresh the context again, or contact your cluster administrator.

To change context, use `vcf context use <context_name>`
[ok] successfully created context: kong-ctx
[ok] successfully created context: kong-ctx:kong
```
</details>
<br>
<br>


### 4. Set supervisor context
```bash
vcf context use "$SUPERVISOR_CONTEXT":"$SUPERVISOR_NAMESPACE_NAME"
```

<details>
<summary>Expected output</summary>

```shell
[ok] Token is still active. Skipped the token refresh for context "kong-ctx:kong"
[i] Successfully activated context 'kong-ctx:kong' (Type: kubernetes) 
[i] Fetching recommended plugins for active context 'kong-ctx:kong'...
[i] No image repository override information was found
[ok] All recommended plugins are already installed and up-to-date. 
```
</details>
<br>
<br>


### 5. Create VKS cluster
In this step we create a VKS cluster as defined in vks.yaml. 
```bash
sed "s/cluster-vks/$CLUSTER_NAME/" vks.yaml | kubectl apply -f -
```

<details>
<summary>Expected output</summary>

```shell
cluster.cluster.x-k8s.io/kong-vks created
```
</details>
<br>
<br>


### 6. Wait for VKS cluster creation
Wait until output shows "Available: True" 
```bash
kubectl get cluster "$CLUSTER_NAME" --watch
```

<details>
<summary>Expected output</summary>

```shell
NAME         CLUSTERCLASS             AVAILABLE   CP DESIRED   CP AVAILABLE   CP UP-TO-DATE   W DESIRED   W AVAILABLE   W UP-TO-DATE   PHASE         AGE   VERSION
kong-vks   builtin-generic-v3.5.0   False       1            0              1               4           0             4              Provisioned   67s   v1.34.1+vmware.1
[...]
kong-vks   builtin-generic-v3.5.0   False       1            1              1               4           3             4              Provisioned   3m18s   v1.34.1+vmware.1
kong-vks   builtin-generic-v3.5.0   True        1            1              1               4           3             4              Provisioned   3m18s   v1.34.1+vmware.1
```
</details>
<br>
<br>


### 7. Connect to VKS cluster
```shell
vcf context create "$CLUSTER_CONTEXT" \
  --endpoint "$SUPERVISOR_IP" \
  --username "$SUPERVISOR_USERNAME" \
  --workload-cluster-namespace="$SUPERVISOR_NAMESPACE_NAME" \
  --workload-cluster-name="$CLUSTER_NAME" \
  --insecure-skip-tls-verify 
```

<details>
<summary>Expected output</summary>

```shell
[i] Logging in to Kubernetes cluster (kong-vks) (kong)
[i] Successfully logged in to Kubernetes cluster 10.138.216.200

You have access to the following contexts:
    vks
    vks:kong-vks

If the namespace context you wish to use is not in this list, you may need to
refresh the context again, or contact your cluster administrator.
 
To change context, use `vcf context use <context_name>`
[ok] successfully created context: vks
[ok] successfully created context: vks:kong-vks
```
</details>
<br>
<br>


### 8. Set VKS Cluster Context
```bash
vcf context use "$CLUSTER_CONTEXT":"$CLUSTER_NAME"
```

<details>
<summary>Expected output</summary>

```shell
[ok] Token is still active. Skipped the token refresh for context "vks:kong-vks"
[i] Successfully activated context 'vks:kong-vks' (Type: kubernetes) 
[i] Fetching recommended plugins for active context
```
</details>
<details>
<summary>Test Command: Get nodes</summary>

```shell
# Get Nodes
kubectl get nodes -o wide

# Expected output: 
NAME                                       STATUS   ROLES           AGE   VERSION            INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
kong-vks-node-pool-1-8jzgf-bhrz2-vmnw7   Ready    <none>          36m   v1.34.1+vmware.1   172.26.0.5    <none>        Ubuntu 24.04.3 LTS   6.8.0-86-generic   containerd://2.1.4+vmware.3-fips
kong-vks-node-pool-2-zd45h-59jlx-mrr64   Ready    <none>          36m   v1.34.1+vmware.1   172.26.0.6    <none>        Ubuntu 24.04.3 LTS   6.8.0-86-generic   containerd://2.1.4+vmware.3-fips
kong-vks-node-pool-3-v8j28-bjm5t-w2h2q   Ready    <none>          36m   v1.34.1+vmware.1   172.26.0.4    <none>        Ubuntu 24.04.3 LTS   6.8.0-86-generic   containerd://2.1.4+vmware.3-fips
kong-vks-node-pool-4-btqp5-z4ts9-sx9bd   Ready    <none>          36m   v1.34.1+vmware.1   172.26.0.7    <none>        Ubuntu 24.04.3 LTS   6.8.0-86-generic   containerd://2.1.4+vmware.3-fips
kong-vks-trdx9-gmtbm                     Ready    control-plane   38m   v1.34.1+vmware.1   172.26.0.3    <none>        Ubuntu 24.04.3 LTS   6.8.0-86-generic   containerd://2.1.4+vmware.3-fips

```
</details>
<br>
<br>


### 9. Create namespace
Create a namespace on the VKS cluster
```bash
kubectl create namespace $CLUSTER_NAMESPACE_NAME
```

<details>
<summary>Expected output</summary>

```bash
namespace/kong-ns created
```
</details>
<br>
<br>


### 10. Update context to use namespace
```bash
kubectl config set-context --current --namespace=$CLUSTER_NAMESPACE_NAME
```

<details>
<summary>Expected output</summary>

```bash
Context "kong-vks-ctx:kong-vks" modified.
```
</details>
<br>
<br>


### 11. (Optional) Create Secret with Docker.io Credentials
May be required if the deployment hits errors about the site hitting image pull limits.
```bash
# Create secret with Docker login credentials in Kubernetes
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=<docker_username> \
  --docker-password=<docker_password> \
  --docker-email=<docker_email> 
  --namespace=$CLUSTER_NAMESPACE_NAME

# Automatically use credentials for all pods in namespace 
kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "regcred"}]}'
```
<br>
<br>


### 12. Install Kong Helm Charts
```bash
helm repo add kong https://charts.konghq.com
helm repo update kong
```
<details>
<summary>Expected output</summary>

```bash
helm repo update kong
"kong" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kong" chart repository
Update Complete. ⎈Happy Helming!⎈

```
</details>

<details>
<summary>Test command: Get Kong operation versions</summary>

```bash
helm search repo kong/kong-operator  --versions

NAME              	CHART VERSION	APP VERSION  	DESCRIPTION         
kong/kong-operator	1.0.2        	2.0          	Deploy Kong Operator
kong/kong-operator	1.0.1        	2.0          	Deploy Kong Operator
kong/kong-operator	1.0.0        	2.0.0        	Deploy Kong Operator
kong/kong-operator	0.0.7        	2.0.0-alpha.5	Deploy Kong Operator
kong/kong-operator	0.0.6        	2.0.0-alpha.5	Deploy Kong Operator
kong/kong-operator	0.0.5        	2.0.0-alpha.4	Deploy Kong Operator
kong/kong-operator	0.0.4        	2.0.0-alpha.3	Deploy Kong Operator
kong/kong-operator	0.0.3        	2.0.0-alpha.2	Deploy Kong Operator
kong/kong-operator	0.0.2        	2.0.0-alpha.2	Deploy Kong Operator
kong/kong-operator	0.0.1        	2.0.0-alpha.0	Deploy Kong Operator
```
</details>
<br>
<br>


### 13. Install Kong operator
```bash
helm upgrade \
  --install kong-operator kong/kong-operator \
  --namespace kong-system \
  --create-namespace \
  --set image.tag=2.0.6 \
  --set kubernetes-configuration-crds.enabled=true \
  --set env.ENABLE_CONTROLLER_KONNECT=true
```

<details>
<summary>Expected output</summary>

```bash
Release "kong-operator" does not exist. Installing it now.
NAME: kong-operator
LAST DEPLOYED: Thu Jan 15 18:58:43 2026
NAMESPACE: kong-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
kong-operator-kong-operator has been installed. Check its status by running:

  kubectl --namespace kong-system get pods

For more details, please refer to the following documents:

* https://developer.konghq.com/operator/dataplanes/get-started/kic/create-gateway/
* https://developer.konghq.com/operator/dataplanes/konnectextension/
```
</details>

<details>
<summary>Test command: Wait for Kong operator ready</summary>

```bash
kubectl wait deployment/kong-operator-kong-operator-controller-manager \
  --namespace kong-system  \
  --for=condition=Available=true \
  --timeout=120s 

# Expected output
deployment.apps/kong-operator-kong-operator-controller-manager condition met
```
</details>

<details>
<summary>Test command: Get pods</summary>

```bash
# Get pods
kubectl --namespace kong-system get pods

# Expected output
NAME                                                              READY   STATUS    RESTARTS   AGE
kong-operator-kong-operator-controller-manager-5584999c9b-gz247   1/1     Running   0          4m49s
```
</details>

<details>
<summary>Test command: Describe Kong operator</summary>

```bash
kubectl describe pod \
  --namespace kong-system \
  $(kubectl get pod -n kong-system -o json | jq -r '.items[].metadata | select(.name | startswith("kong-operator"))' | jq -r '.name')

# Expected output
Name:             kong-operator-kong-operator-controller-manager-5584999c9b-gz247
Namespace:        kong-system
Priority:         0
Service Account:  controller-manager
Node:             kong-vks-node-pool-3-6knqv-rzvl7-7w4vb/172.26.0.5
Start Time:       Thu, 15 Jan 2026 18:58:46 +0000
Labels:           app=kong-operator-kong-operator
                  app.kubernetes.io/component=ko
[...]  
```
</details>

<details>
<summary>Test command: Get Kong operator logs</summary>

```bash
 
kubectl logs \
  --namespace kong-system 
  --follow $(kubectl get pod -n kong-system -o json | jq -r '.items[].metadata | select(.name | startswith("kong-operator"))' | jq -r '.name') 

# Expected output
{"level":"info","ts":"2026-01-15T18:58:50Z","logger":"setup","msg":"starting controller manager","release":"2.0.6-amd64","repo":"https://github.com/Kong/kong-operator.git","commit":"b6054456f7b71f5eeceb5af89002a288c1c41d7b"}
{"level":"info","ts":"2026-01-15T18:58:50Z","logger":"setup","msg":"leader election enabled","namespace":"kong-system"}
{"level":"info","ts":"2026-01-15T18:58:50Z","msg":"Setting up index","index":"*v2beta1.ControlPlane[dataplane]"}
{"level":"info","ts":"2026-01-15T18:58:50Z","msg":"Setting up index","index":"*v2beta1.ControlPlane[KonnectExtension]"}
{"level":"info","ts":"2026-01-15T18:58:50Z","msg":"Setting up index","index":"*v1beta1.DataPlane[KonnectExtension]"}
{"level":"info","ts":"2026-01-15T18:58:50Z","msg":"Setting up index","index":"*v1beta1.DataPlane[DataPlaneOnOwnerGateway]"}
[...]
```
</details>
<br>
<br>

### 14. Create 'kong' namespace
```bash
kubectl create namespace kong
```
<details>
<summary>Expected output</summary>

```bash
namespace/kong created
```
</details>
<br>
<br>

### 15. Create 'konnect-pat' secret
```bash
kubectl create secret generic konnect-pat \
 --namespace kong \
 --from-literal=token=$(echo $KONG_PAT)
kubectl label secret konnect-pat konghq.com/credential=konnect --namespace kong 
kubectl label secret konnect-pat konghq.com/secret=true --namespace kong
```
<details>
<summary>Expected output</summary>

```bash
secret/konnect-pat created
secret/konnect-pat labeled
secret/konnect-pat labeled
```
</details>

<details>
<summary>Test command: Get Kong Personal Access Token</summary>

```bash
kubectl get secret konnect-pat -n kong -o json | jq -r '.data.token' | base64 -d 
kpat_abc123[...]
```
</details>
<br>
<br>


### 16. Create KonnectGatewayControlPlane
```bash
echo '
kind: KonnectGatewayControlPlane
apiVersion: konnect.konghq.com/v1alpha2
metadata:
  name: gateway-control-plane
  namespace: kong
spec:
  createControlPlaneRequest:
    name: gateway-control-plane
  konnect:
    authRef:
      name: konnect-api-auth
' | kubectl apply -f -
```
<details>
<summary>Expected output</summary>

```bash
konnectgatewaycontrolplane.konnect.konghq.com/gateway-control-plane created
```
</details>

<details>
<summary>Test command: Verify KonnectGatewayControlPlane is reconciled</summary>

```bash
kubectl get -n kong konnectgatewaycontrolplane gateway-control-plane \
  -o=jsonpath='{.status.conditions[?(@.type=="Programmed")]}' | jq
  
# Expected output
{
  "observedGeneration": 1,
  "reason": "Programmed",
  "status": "True",
  "type": "Programmed"
}
```
</details>
<br>
<br>

### 17. Create DataPlane
```bash
echo '
apiVersion: gateway-operator.konghq.com/v1beta1
kind: DataPlane
metadata:
  name: dataplane-example
  namespace: kong
spec:
  extensions:
  - kind: KonnectExtension
    name: my-konnect-config
    group: konnect.konghq.com
  deployment:
    podTemplateSpec:
      spec:
        containers:
        - name: proxy
          image: kong/kong-gateway:3.13
' | kubectl apply -f -
```
<details>
<summary>Expected output</summary>

```bash
dataplane.gateway-operator.konghq.com/dataplane-example created
```
</details>

<details>
<summary>Test command: Verify KonnectGatewayControlPlane is reconciled</summary>

```bash
kubectl get -n kong dataplane dataplane-example \
  -o=jsonpath='{.status.conditions[?(@.type=="Ready")]}' | jq
  
# Expected output
{
  "observedGeneration": 1,
  "reason": "Ready",
  "status": "True",
  "type": "Ready"
}
```
</details>
<br>
<br>

### 18. Create Service
```bash
echo '
kind: KongService
apiVersion: configuration.konghq.com/v1alpha1
metadata:
  name: service
  namespace: kong
spec:
  name: service
  host: httpbin.konghq.com
  controlPlaneRef:
    type: konnectNamespacedRef
    konnectNamespacedRef:
      name: gateway-control-plane
' | kubectl apply -f -
```
<details>
<summary>Expected output</summary>

```bash
kongservice.configuration.konghq.com/service created
```
</details>
<br>
<br>

### 18. Create Route
```bash
echo '
kind: KongRoute
apiVersion: configuration.konghq.com/v1alpha1
metadata:
  name: route-with-service
  namespace: kong
spec:
  name: route-with-service
  protocols:
  - http
  paths:
  - "/"
  serviceRef:
    type: namespacedRef
    namespacedRef:
      name: service
' | kubectl apply -f -
```
<details>
<summary>Expected output</summary>

```bash
kongroute.configuration.konghq.com/route-with-service created
```
</details>
<br>
<br>

### 19. Send test traffic
```bash
NAME=$(kubectl get -o yaml -n kong service | yq '.items[].metadata.name | select(contains("dataplane-ingress"))' | tr -d '"')
export PROXY_IP=$(kubectl get svc -n kong $NAME -o jsonpath='{range .status.loadBalancer.ingress[0]}{@.ip}{@.hostname}{end}')
echo "Proxy IP: $PROXY_IP"
```



## Cleanup Procedure

```shell
# Remove kong operator
helm uninstall kong-operator --namespace kong-system

# Delete kong-system namespace
kubectl delete namespace kong-system

#Delete the namespace
kubectl delete namespace $CLUSTER_NAMESPACE_NAME

# Switch to supervisor context 
vcf context use $SUPERVISOR_CONTEXT:$SUPERVISOR_NAMESPACE_NAME

# Delete VKS cluster as defined in vks.yaml
kubectl delete -f vks.yaml
```


## Troubleshooting

### Useful Commands
```shell
# List all
kubectl get all

# Get detailed pod information
kubectl get pods -o wide

# Get container logs
kubectl logs -f <container name>

# Get all services
kubectl get svc

# Expose a service
kubectl expose service <service_name>  --type=LoadBalancer --name=<service_name>-external

# Get the external IP
kubectl get svc <service_name>-external

# Get TKR releases
kubectl get tkr -l '!kubernetes.vmware.com/kubernetesrelease'

# Get TKE releaase specs
# e.g. kubectl get tkr 'v1.34.1---vmware.1-vkr.4' -o yaml
kubectl get tkr TKR_NAME -o yaml  
```

