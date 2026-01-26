# Kong Operator On-Prem Deployment
## Versions
* Kong Gateway 3.9
* vSphere Kubernetes 3.5 / VKr 1.34

## References
* https://developer.konghq.com/operator/dataplanes/get-started/kic/install/
* https://cert-manager.io/docs/installation/

## Requirements
### CLI Tools
* kubectl cli v1.34: https://dl.k8s.io/release/v1.34.0/bin/linux/amd64/kubectl
* vcf cli v9.0.1: https://packages.broadcom.com/artifactory/vcf-distro/vcf-cli/linux/amd64/v9.0.1/
* helm cli v3.19: https://get.helm.sh/helm-v3.19.0-linux-amd64.tar.gz

## Deployment Procedure Step 1 - Operator

### 1. Create namespace 'kong'
Create a 'kong' namespace.
```bash
kubectl create namespace kong
```
<details>
<summary>Expected output</summary>

```bash
namespace/kong created
```
</details>
<details>
<summary>Test command: List Namespaces</summary>

```bash
# List certificates
kubectl get namespaces

# Expected output
NAME                                 STATUS   AGE
default                              Active   26m
kong                                 Active   14m # <---- kong namespace
kong-vks-ns                          Active   22m
kube-node-lease                      Active   26m
kube-public                          Active   26m
kube-system                          Active   26m
secretgen-controller                 Active   25m
tkg-system                           Active   25m
velero-vsphere-plugin-backupdriver   Active   25m
vmware-system-antrea                 Active   25m
vmware-system-auth                   Active   25m
vmware-system-cloud-provider         Active   25m
vmware-system-csi                    Active   25m
vmware-system-tkg                    Active   25m
```
</details>
<br>
<br>

### 2. Install Kong Helm Charts
Kong provides a Helm chart for deploying Kong Gateway. Add the charts.konghq.com repository and run helm repo update to ensure that you have the latest version of the chart.
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
<summary>Test command: Get Kong operator versions</summary>

```bash
helm search repo kong/kong-operator  --versions


# Expected Output
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


### 3. Install Cert Manager (Optional)
If you want cert-manager to issue and rotate the admission and conversion webhook certificates, install cert-manager to your cluster and enable cert-manager integration by passing the ```--set global.webhooks.options.certManager.enabled=true``` argument.

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.19.2/cert-manager.yaml
```
<details>
<summary>Expected output</summary>

```bash
namespace/cert-manager created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
serviceaccount/cert-manager-cainjector created
serviceaccount/cert-manager created
serviceaccount/cert-manager-webhook created
clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrole.rbac.authorization.k8s.io/cert-manager-cluster-view created
clusterrole.rbac.authorization.k8s.io/cert-manager-view created
clusterrole.rbac.authorization.k8s.io/cert-manager-edit created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrole.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
role.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
role.rbac.authorization.k8s.io/cert-manager:leaderelection created
role.rbac.authorization.k8s.io/cert-manager-tokenrequest created
role.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
rolebinding.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager-tokenrequest created
rolebinding.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
service/cert-manager-cainjector created
service/cert-manager created
service/cert-manager-webhook created
deployment.apps/cert-manager-cainjector created
deployment.apps/cert-manager created
deployment.apps/cert-manager-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
```
</details>
<br>
<br>


### 4. Install Kong Operator
Install Kong Operator using Helm.

**Note**. We use cert-manager to issue and rotate certificates. If you do not enable this, the chart will generate and inject self-signed certificates automatically. We recommend enabling cert-manager to manage the lifecycle of these certificates.

```bash
helm upgrade --install kong-operator kong/kong-operator \
  --namespace kong-system \
  --create-namespace \
  --set image.tag=2.0.5 \
  --set global.webhooks.options.certManager.enabled=true
```
<details>
<summary>Expected output</summary>

```bash
Release "kong-operator" does not exist. Installing it now.
I0123 18:20:54.166115  107846 warnings.go:110] "Warning: spec.privateKey.rotationPolicy: In cert-manager >= v1.18.0, the default value changed from `Never` to `Always`."
I0123 18:20:54.199135  107846 warnings.go:110] "Warning: spec.privateKey.rotationPolicy: In cert-manager >= v1.18.0, the default value changed from `Never` to `Always`."
NAME: kong-operator
LAST DEPLOYED: Fri Jan 23 18:20:49 2026
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
<summary>Test command: Get kong-system pods versions</summary>

```bash
kubectl --namespace kong-system get pods

# Expected Output
NAME                                                              READY   STATUS    RESTARTS   AGE
kong-operator-kong-operator-controller-manager-77c669f477-stfmk   1/1     Running   0          66s

```

</details>
<br>
<br>


### 5. Add License (Optional)
This tutorial doesn’t require a license, but you can add one using KongLicense. This assumes that your license is available in ./license.json.
```bash
echo "
apiVersion: configuration.konghq.com/v1alpha1
kind: KongLicense
metadata:
 name: kong-license
rawLicenseString: '$(cat ./license.json)'
" | kubectl apply -f -
```
<details>
<summary>Expected output</summary>

```bash
konglicense.configuration.konghq.com/kong-license created
```
</details>


### 6. Wait for Kong Operator Ready
```bash
kubectl wait deployment/kong-operator-kong-operator-controller-manager \
  --namespace kong-system  \
  --for=condition=Available=true \
  --timeout=120s
```

<details>
<summary>Expected output</summary>

```bash
deployment.apps/kong-operator-kong-operator-controller-manager condition met
```
</details>
<br>
<br>

## Deployment Procedure Step 2 - Gateway

### 1. Create Gateway Configuration
```bash
echo 'kind: GatewayConfiguration
apiVersion: gateway-operator.konghq.com/v2beta1
metadata:
  name: kong
  namespace: kong
spec:
  dataPlaneOptions:
    deployment:
      podTemplateSpec:
        spec:
          containers:
          - name: proxy
            image: kong:3.9.1' | kubectl apply -f -
```
<details>
<summary>Expected output</summary>

```bash
gatewayconfiguration.gateway-operator.konghq.com/kong created
```
</details>
<br>
<br>

### 2. Create GatewayClass
To use the Gateway API resources to configure your Routes, you need to create a GatewayClass instance and create a Gateway resource that listens on the ports that you need.
```bash
echo '
kind: GatewayClass
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: kong
  namespace: kong
spec:
  controllerName: konghq.com/gateway-operator
  parametersRef:
    group: gateway-operator.konghq.com
    kind: GatewayConfiguration
    name: kong
    namespace: kong
---
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: kong
  namespace: kong
spec:
  gatewayClassName: kong
  listeners:
  - name: http
    protocol: HTTP
    port: 80' | kubectl apply -f -
```
<details>
<summary>Expected output</summary>

```bash
gatewayclass.gateway.networking.k8s.io/kong created
gateway.gateway.networking.k8s.io/kong created
```
</details>
<details>
<summary>Test command: List Gateways</summary>

```bash
# Get gateway 'kong'
kubectl get gateway kong \
 --namespace kong \
 --output wide
 
# Expected output
NAME   CLASS   ADDRESS          PROGRAMMED   AGE
kong   kong    10.138.216.219   True         6m21s
```
</details>
<br>
<br>

### 3. Check Programmed Status
If the **Gateway** has **Programmed** condition set to **True**, you can visit Konnect and see your configuration being synced by the self-managed Control Plane. 

You can verify the **Gateway** was reconciled successfully by checking its **Programmed** condition.

```bash
kubectl get -n kong gateway kong \
  -o=jsonpath='{.status.conditions[?(@.type=="Programmed")]}' | jq
  ```

<details>
<summary>Expected output</summary>

```json
{
  "lastTransitionTime": "2026-01-23T20:47:20Z",
  "message": "",
  "observedGeneration": 1,
  "reason": "Programmed",
  "status": "True",
  "type": "Programmed"
}
```
</details>
<br>
<br>

## Deployment Procedure Step 3 - Create a Route

### 1. Configure an Echo Service
In order to route a request using Kong Gateway we need a Service running in our cluster. Install an echo Service using the following command:

```bash
kubectl apply -f https://developer.konghq.com/manifests/kic/echo-service.yaml \
  --namespace kong
```

<details>
<summary>Expected output</summary>

```bash
service/echo created
deployment.apps/echo created
```
</details>
<br>
<br>

### 2. Create a Route
Create an HTTPRoute to send any requests that start with /echo to the echo Service.
```bash
echo '
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: echo
  namespace: kong
spec:
  parentRefs:
    - group: gateway.networking.k8s.io
      kind: Gateway
      name: kong
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /echo
      backendRefs:
        - name: echo
          port: 1027
' | kubectl apply -f -
```
<details>
<summary>Expected output</summary>

```bash
httproute.gateway.networking.k8s.io/echo created
```
</details>
<br>
<br>

### 3. Test the Configuration
Run kubectl get gateway kong -n default to get the IP address for the gateway and set that as the value for the variable PROXY_IP.
```bash
export PROXY_IP=$(kubectl get gateway kong -n kong -o jsonpath='{.status.addresses[0].value}')
curl "$PROXY_IP/echo" --no-progress-meter --fail-with-body 
```

<details>
<summary>Expected output</summary>

```bash
Welcome, you are connected to node kong-vks-node-pool-1-nl7vd-rmjzn-bdj97.
Running on Pod echo-79d4b9b8bc-68nkx.
In namespace kong.
With IP address 192.168.1.3.
```
</details>
<br>
<br>


## Deployment Procedure Step 4 - Proxy HTTP Traffic

### 1. Configure an Echo Service
```bash
kubectl apply -f  https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/standard-install.yaml
```
