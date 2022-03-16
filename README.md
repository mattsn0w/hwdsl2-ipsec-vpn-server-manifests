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

### Verify things are running
```
ubuntu@vm1:~/manifests/hwdsl2-ipsec-vpn-server$ kubectl get all,eips,secrets -n hwdsl2-ipsec-vpn
NAME                                 READY   STATUS    RESTARTS   AGE
pod/ipsec-vpn-home-965cdf896-fpd8r   1/1     Running   0          3h24m

NAME                                  TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                        AGE
service/ipsec-vpn-home-layer2lb-svc   LoadBalancer   10.43.74.245   172.16.1.90   500:31186/UDP,4500:31080/UDP   18h

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ipsec-vpn-home   1/1     1            1           3h24m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/ipsec-vpn-home-965cdf896   1         1         1       3h24m

NAME                                           CIDR          USAGE   TOTAL
eip.network.kubesphere.io/ipsec-vpn-home-eip   172.16.1.90   1       1

NAME                                        TYPE                                  DATA   AGE
secret/default-token-2trq4                  kubernetes.io/service-account-token   3      18h
secret/ipsec-vpn-psk-user-password-secret   Opaque                                3      3h40m
ubuntu@vm1:~/manifests/hwdsl2-ipsec-vpn-server$
```
