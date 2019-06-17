---
description: This guide describes how to deploy a VerneMQ cluster on Kubernetes
---

# VerneMQ on Kubernetes

## Intro

Kubernetes \(K8s\) is possibly the most mature technology for deploying Docker containers at scale. While running a single Docker container is supposed to be easy, running a Kubernetes cluster definitely isn't. That's why we recommended to work with a certified Kubernetes partner such as [Amazon AWS EKS](https://aws.amazon.com/eks/), [Google Cloud GKE](https://cloud.google.com/kubernetes-engine/), [Microsoft Azure AKS](https://azure.microsoft.com/en-us/services/kubernetes-service/), or [DigitalOcean](https://www.digitalocean.com/products/kubernetes/).

If your applications already live in Docker containers and are deployed on Kubernetes it can be beneficial to also run VerneMQ on Kubernetes. This guide covers how to successfully deploy a VerneMQ cluster on Kubernetes. Multiple options exist to deploy a VerneMQ cluster at this point. This guide describes how to use the official Helm chart as well as the still experimental Kubernetes Operator.

For the sake of clarity, this guide defines the following terms:

* Kubernetes Node: A single virtual or physical machine in a Kubernetes cluster.
* Kubernetes Cluster: A group of nodes firewalled from the internet, that are the primary compute resources managed by Kubernetes.
* Edge router: A router that enforces the firewall policy for your cluster. This could be a gateway managed by a cloud provider or a physical piece of hardware.
* Cluster network: A set of links, logical or physical, that facilitate communication within a cluster according to the Kubernetes networking model.
* Service: A Kubernetes Service that identifies a set of pods using label selectors. Unless mentioned otherwise, Services are assumed to have virtual IPs only routable within the cluster network
* VerneMQ Cluster: A group of VerneMQ containers that are connected via the Erlang Distribution as well as the VerneMQ clustering mechanism.

{% hint style="info" %}
This guide assumes that you're familiar with Kubernetes
{% endhint %}

## Deploy VerneMQ with Helm

Helm calls itself the package manager for Kubernetes. In Helm a package is called a chart. VerneMQ comes with such a Helm chart simplifying the initial setup tremendously. If you don't have setup Helm yet, please navigate through their [quickstart guide](https://helm.sh/docs/using_helm/#quickstart-guide). 

Once Helm is properly setup just run the following command in your shell.

```text
helm install vernemq/vernemq
```

This will deploy a single node VerneMQ cluster. Have a look at the possible configuration [here](https://github.com/vernemq/docker-vernemq/tree/master/helm/vernemq#configuration).

## Deploy VerneMQ using the Kubernetes Operator

A Kubernetes Operator is a method of packaging, deploying and managing a Kubernetes application. The VerneMQ Operator is basically just a Pod with the task to deploy a VerneMQ cluster given a so called Custom Resource Definition \(CRD\). The [VerneMQ CRD](https://github.com/vernemq/vmq-operator/blob/master/docs/api.md#vernemqspec) aims that all required configuration can be made through the CRD and no further configuration should be required. The following command installs the operator along a two node VerneMQ cluster into the namespace `messaging` 

```text
curl -L https://codeload.github.com/vernemq/vmq-operator/zip/master --output repo.zip; \
unzip -j repo.zip '*/examples/only_vernemq/*' -d only_vernemq; \
kubectl apply -f only_vernemq
```

This will result in the following Pods:

```text
kubectl get pods --namespace messaging
NAME                                      READY   STATUS        RESTARTS   AGE
vernemq-k8s-0                             1/1     Running       0          53m
vernemq-k8s-1                             1/1     Running       0          4m14s
vernemq-k8s-deployment-59f5684549-s7jd4   1/1     Running       0          2d17h
vmq-operator-76f5f78f96-2jbwt             1/1     Running       0          4m28s
```

And the following cluster status:

```text
kubectl exec vernemq-k8s-0 vmq-admin cluster show --namespace messaging
+-----------------------------------------------------------------+-------+
|                              Node                               |Running|
+-----------------------------------------------------------------+-------+
|vmq@vernemq-k8s-0.vernemq-k8s-service.messaging.svc.cluster.local| true  |
|vmq@vernemq-k8s-1.vernemq-k8s-service.messaging.svc.cluster.local| true  |
+-----------------------------------------------------------------+-------+
```

{% hint style="info" %}
At this point you would like to further configure [authentication and authorization](../configuration/db-auth.md).  
The following port forwards may be useful at this point.

`kubectl port-forward svc/vernemq-k8s --namespace messaging 1883:1883`  
`kubectl port-forward svc/vernemq-k8s --namespace messaging 8888:8888`
{% endhint %}

## Load Balancing in Kubernetes

In a VerneMQ cluster it doesn't matter to which node a MQTT client connects, subscribes or publishes. A VerneMQ cluster looks like one big MQTT broker to the outside. While this is the main idea of VerneMQ it comes with a cost, namely the data replication/synchronization overhead when 'persistent' clients hop from one pod to the other. As a consequence, we recommend to intelligently choose how to load balance your MQTT clients.

Load balancing in Kubernetes is configured via the Service object. Multiple service types exist: 

#### ClusterIP

The [ClusterIP](https://kubernetes.io/docs/concepts/services-networking/#publishing-services-service-types) type is the default and only permits access from within the Kubernetes cluster. Other pods in the Kubernetes cluster can access VerneMQ via `ClusterIP:Port` . The underlying balancing strategy is based on the settings of kube-proxy. Also this type requires that one terminates TLS either in VerneMQ directly or via a different Pod e.g. HAproxy. 

#### NodePort

The [NodePort](https://kubernetes.io/docs/concepts/services-networking/#nodeport) type uses ClusterIP under the hood but allocates a Port on every Kubernetes node and routes incoming traffic from `NodeIP:NodePort` to the `ClusterIP:Port` . Like with ClusterIP this type requires that one terminates TLS either in VerneMQ directly or via a different Pod e.g. HAproxy.

#### Loadbalancer

The [Loadbalancer](https://kubernetes.io/docs/concepts/services-networking/#loadbalancer) type uses an external load balancer provided by the cloud provider. In fact this Service type only provides the glue code required to interact with the Loadbalancing services from different cloud providers. If you're running a bare-metal Kubernetes cluster you won't be able to use this Service type, unless you deploy a Kubernetes aware network loadbalancer yourself. Check out [MetalLB](https://metallb.universe.tf/), which provides a network loadbalancer for bare-metal Kubernetes clusters.

{% hint style="info" %}
Every Kubernetes node runs a **kube-proxy**. **kube-proxy** maps virtual IP addresses to services and creates the required routes in the system so that pods can communicate with each other. 

**kube-proxy** supports multiple modes of operation:  
- [userspace](https://kubernetes.io/docs/concepts/services-networking/#proxy-mode-userspace) since v1.0  
- [iptables](https://kubernetes.io/docs/concepts/services-networking/#proxy-mode-iptables) default since v1.2  
- [ipvs](https://kubernetes.io/docs/concepts/services-networking/#proxy-mode-ipvs) stable since v1.11, only available if the Kernel of the Kubernetes node supports it.

The performance and scalability characteristics of VerneMQ depend on the proxy-mode and the related configurations. This is especially true for load-balancing specific functionality such as session affinity. E.g. only ipvs supports an efficient way to provide session affinity via the source hashing strategy.
{% endhint %}

#### Ingress Controllers

Ingress controllers provide another way to do load balancing and TLS termination in a Kubernetes cluster. However the officially supported ingress controllers [nginx](https://git.k8s.io/ingress-nginx/README.md) and [GCE](https://git.k8s.io/ingress-gce/README.md) focus on balancing HTTP requests instead of plain TCP connections. Therefore their support for TLS termination is also limited to HTTPS.

Multiple third-party ingress controllers exist, however most of them focus on handling HTTP requests. One of the exceptions is [Voyager](https://github.com/appscode/voyager) by AppsCode an ingress controller based on HAProxy, which also efficiently terminates TLS.

### General recommendations for large scale deployments

1. Use an external loadbalancer provided by the cloud provider that is capable of terminating TLS and apply a load balancing strategy that provides session affinity e.g. via source hashing.
2. Terminate TLS outside VerneMQ.
3. Configure the Pod NodeAffinity correctly to ensure that only one VerneMQ pod is scheduled on any Kubernetes cluster node.
4. It's preferred to have a smaller number of Pods that are very powerful in terms of available CPU and RAM than the opposite.







