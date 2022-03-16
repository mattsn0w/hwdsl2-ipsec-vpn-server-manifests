# hwdsl2/ipsec-vpn-server on Kubernetes

## Objective
* An IPSec VPN for iOS/MacOS devices that I can use while out and about.  
* Run IaC on Kubernetes at home, or work. 
* Learn and test Longhorn for Kubernetes native storage and OpenELB for on-premise load balancer.


## Requirments
* [A self-hosted k3s cluster](https://github.com/mattsn0w/k3s-home) (_do you even home lab, br0?_)
* [OpenELB](https://openelb.github.io/docs/getting-started/installation/install-openelb-on-k3s/) installed (_for Layer-2, too lazy for BGP, HA goodness!_)
* [Longhorn](https://longhorn.io/docs/1.2.3/deploy/install/install-with-kubectl/) installed (_for k8s cloud-native storage_)
* 

## Tweaks
* Update `secrets_template.yaml` with the desired PSK, Username, and Password. These details will be needed for client configuration. Rename to secrets.yaml, and preferably store this encrypted at rest.  
* Update `deployment.yaml` with your internal DNS server or comment out the key:value env setting for this.
* Update `eip.yaml` with the internal IP that you will use for your loadbalancer.  

## Apply the manifests to your cluster
The order of these manifests does matter as we are creating this in a particular namespace, and there are special environment variables created in the pod containers for call backs to through the service load balancer.

```
for YAML in namespace.yaml pvc.yaml secrets.yaml eip.yaml service.yaml deployment.yaml; do
    kubectl apply -f $YAML
done
```
