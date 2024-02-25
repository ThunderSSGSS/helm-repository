# **CHART minio-cluster** 
This chart helps you to deploy and maintain a minio standalone or cluster instances.<br>


## **Cluster Requirements**
To use all features that this charts provides, your K8s cluster must have:
* [Cert-Manager](https://cert-manager.io/);
* [Metric Server](https://kubernetes-sigs.github.io/metrics-server/);
* [Nginx Ingress Controller](https://github.com/kubernetes/ingress-nginx).<br>


## **Values**

### Default Values:
```yaml
# Resources names prefix, all resources names will starts with <name>-<name_suffix>
name: test
name_suffix: minio

imagePullPolicy: IfNotPresent # all images pull policy
imageTag: RELEASE.2023-10-16T04-13-43Z # tag of minio official image

ingress:
  enable: false # when true creates a ingress
  # additionalAnnotations: # Set additional nginx controller annotations
  #   nginx.ingress.kubernetes.io/proxy-body-size: 30m 
  host: host.com
  consoleHost: console.host.com
  enable_tls: false # only when true | enables tls and requests a certificate from clusterIssuer
  clusterIssuer: letsencrypt-prod
  # tlsSecretName: secret-name # set certificate secret name
  # cache configurations
  cache:
    enable: false # if enable will create a new ingress with static cache configurations
    pathRegex: /.+\.(css|js|jpg|jpeg|png|gif|ico)
    configuration: |
      expires 1y;
      proxy_cache_bypass $http_x_purge;
      proxy_cache static-cache;
      proxy_cache_valid 200              168h;
      proxy_cache_valid 301              4h;
      proxy_cache_valid 302              1h;
      proxy_cache_valid 400 401 403 404  30s;
      proxy_cache_valid 500 501 502 503  30s;
      proxy_cache_valid 429              10s;
      proxy_cache_use_stale error timeout invalid_header updating http_429 http_500 http_502 http_503 http_504;
      add_header X-Proxy-Cache $upstream_cache_status;

# CREDENTIALS
root_user: root
root_user_pwd: password

# DATABASES
buckets:
- name: static # name of the buckets | for best practice use "_" to separe words
  is_public: false
  #admin_user: admin # database admin user | the user will be created if doesn't exist
  #admin_user_pwd: password # must be setted if admin_user is setted


# RESOURCES
resources:
  enable: false
  requests:
    cpu: 50m
    memory: 100Mi
  limits:
    cpu: 200m
    memory: 1500Mi

# CLUSTER
cluster:
  # please check https://min.io/product/erasure-code-calculator to calculate the number of nodes you want
  nodes: 1

# STORAGE
storage:
  enable: false
  storageClassName: gp2
  capacity: 10Gi
```
*Note: All features on a comment is disabled, discomment if you want to enables*
<br>

## **Examples**
### Example1
We want to deploy a standalone minio instance with one private bucket and 5GB of persistent storage:<br>
<br>
**The standalone database values**:
```yaml
name: standalone

root_user: standalone_user
root_user_pwd: hjh13yj14g32j4hgj3h

buckets:
- name: tests
  admin_user: tests_admin
  admin_user_pwd: asdasdasdaasdasdasdasd

cluster:
  nodes: 1

storage:
  enable: true
  storageClassName: storage
  capacity: 5Gi
```

### Example2
We want to deploy a minio cluster with:<br>
* 4 instances;
* 2 buckets, "static_files" and "private", the "static_files" bucket needs public read;
* 16GB of cluster storage capacity.<br>

**The cluster values:**
```yaml
name: example-cluster

root_user: cluster_user
root_user_pwd: hjh13yj14g32j4hgj3h

buckets:
- name: static_files
  is_public: true
  admin_user: static_files_admin
  admin_user_pwd: asdasdasdaasdasdasdasd
- name: private
  admin_user: private_files_admin
  admin_user_pwd: asdasdasdaasdasdasdasd

cluster:
  nodes: 4

storage:
  enable: true
  storageClassName: storage
  capacity: 8Gi
```
*NB: Use the command "helm template" to see all configurations generated*


## **Upgrade**
When you make any changes on the values, the helm chart creates a configuration job that will apply all "Values.buckets" configurations on your instances without restart. If you remove a bucket on "Values.buckets", the bucket wont be delected on instances, so you need to delete the bucket and admin user manually.

### **Things that cannot be changed on values**
* Values.name;
* Values.name_suffix;
* Values.imageTag.

### **Attention**
* Some changes will restart the minio instances;
* If you change the root username or password, the minio instances will be restarted.


## DevInfos:
- James Artur (Thunder), a DevOps and infrastructure enthusiastics.