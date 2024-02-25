# **CHART network** 
This chart helps you to configure network policies on namespace.<br>

## **Cluster Requirements**
To use all features that this charts provides, your K8s cluster must have:
* [Container Network Interface (CNI)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/);
* [Metric Server](https://kubernetes-sigs.github.io/metrics-server/).<br>


## **Values**

### Default Values:
```yaml
name: network

allowInternetAccess: true
allowedNamespacesIngress: ["ingress-nginx", "monitoring", "kube-system"]
# allowedNamespaceEgress: ["database"]
```
*Note: All features on a comment is disabled, discomment if you want to enables*
<br>

## DevInfos:
- James Artur (Thunder), a DevOps and infrastructure enthusiastics.