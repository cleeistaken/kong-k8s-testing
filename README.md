# Kong VKS Deployment

## Versions
* Kong AI Gateway 3.13
* vSphere Kubernetes 3.5 / VKr 1.34

## References
* [vSphere Supervisor Platform](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-9-0-and-later/9-0/vsphere-supervisor-installation-and-configuration.html)
* [Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)
* [Installing and Using VCF CLI v9.0](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-9-0-and-later/9-0/building-your-cloud-applications/getting-started-with-the-tools-for-building-applications/installing-and-using-vcf-cli-v9.html)
* [Install Kong Gateway in Konnect with Helm](https://developer.konghq.com/gateway/install/kubernetes/on-prem/)
* [Install Kong Gateway on-prem with Helm](https://developer.konghq.com/gateway/install/kubernetes/konnect/)


## Get Files 
```shell
git clone https://github.com/cleeistaken/kong-k8s-testing.git
```

## Deployment Procedure

* [Step 1. Configure Supervisor](1_CONFIGURE_SUPERVISOR.md)
* [Step 2. VKS Deployment](2_VKS_DEPLOYMENT.md)
* Kong Deployment
  * [Option 1: Step 3a. Kong Deployment - On-Prem](3A_KONG_DEPLOYMENT_ON_PREM.md)
  * [Option 2: Step 3b. Kong Deployment - Konnect](3B_KONG_DEPLOYMENT_KONNECT.md) 