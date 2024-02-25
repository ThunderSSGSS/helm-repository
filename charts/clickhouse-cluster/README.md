# **CHART Clickhouse-cluster** 
This chart helps you to deploy and maintain a Clickhouse standalone or cluster instances.<br>

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
name_suffix: clickhouse

ingress:
  create: false # when true creates a ingress
  host: host.com
  bodySize: 6m
  enable_tls: false # only when true | enables tls and requests a certificate from clusterIssuer
  clusterIssuer: letsencrypt-prod
  # tlsSecretName: secret-name # set certificate secret name

imagePullPolicy: IfNotPresent # all images pull policy
version: 23.10.3.5 # tag of clickhouse official alpine image

# CREDENTIALS
root_user: root
root_user_pwd: password

# DATABASES
databases:
- name: test_db # name of the database | for best practice use "_" to separe words 
  enable_backup: false # if true, enable backup and create cronjob
  # backup_suspend: false # if true, suspend backup cronjob
  backup_schedule: "0 2 * * 0" # used when enable_backup is true
  backup_s3_bucket_path: "test_db_backups" # used when enable_backup is true
  # admin_user: admin # database admin user | the user will be created if doesn't exist
  # admin_user_pwd: password # must be setted if admin_user is setted 

# RESOURCES LIMITS
resources: # instances resource limits
  enable: false
  requests:
    cpu: 50m
    memory: 100Mi
  limits:
    cpu: 200m
    memory: 1500Mi


# CLUSTER MODE
cluster_mode:
  enable: false
  cluster_key: S0tLS0tLS0tLS0tLS0tLS0tLS0tLS0t # cluster encription key
  readReplicas: 1 # minumum read replicas
  autoscaler: # you must enable resources limits to enable autoscaler
    enable: false
    targetCPUUtilizationPercentage: 80
    maxReadReplicas: 2
  schedule:
    enable: false
    # dbInstancesNodeSelector: # node selector for database instances
      # node_label: value


# STORAGE
storage:
  enable: false
  storageClassName: gp2
  capacity: 10Gi

# SERVICE
service:
  add_node_port: false # if true, exposes master instance on node port
  nodePort: 31950

# BACKUPS
backups: # this will be used when one or more databases have enable_backup=true
  aws_access_key_id_base64: b2xh
  aws_secret_access_key_base64: b2xh
  aws_region: "eu-west-1"
  aws_s3_bucket_name: "backups-example"
```
*Note: All features on a comment is disabled, discomment if you want to enables*
<br>

## **Examples**
### Example1
We want to deploy a standalone Clickhouse instance with one database and 2GB of persistent storage:<br>
<br>
**The standalone database values**:
```yaml
name: standalone

root_user: standalone_user
root_user_pwd: hjh13yj14g32j4hgj3h

databases:
- name: standalone_database

storage:
  enable: true
  storageClassName: storage
  capacity: 2Gi
```

### Example2
We want to deploy a Clickhouse cluster with:<br>
* 2 read replicas;
* 2 databases, "prod" and "staging", the "prod" database needs weekly backup;
* 5Gi of persistent storage.<br>

**The cluster values:**
```yaml
name: example-cluster

root_user: cluster_user
root_user_pwd: hjh13yj14g32j4hgj3h

databases:
- name: prod
  enable_backup: true
  backup_schedule: "0 2 * * 0"
  backup_s3_bucket_path: "prod_backups"
  admin_user: prod_admin
  admin_user_pwd: jhj4h6jhj54h5h4h5jhsd
- name: staging
  admin_user: staging_admin
  admin_user_pwd: ju3h434b3u4bububu34d

cluster_mode:
  enable: true
  cluster_key: ashdjashdjashdjhjashdjashdjashdjas
  readReplicas: 2

storage:
  enable: true
  storageClassName: storage
  capacity: 5Gi

backups:
  aws_access_key_id_base64: ZXhhbXBsZSBLRVkgS0VZIEtFWSBLRVkK
  aws_secret_access_key_base64: YXNqZGhhc2pkaGFzZ2dhc2dkGF==
  aws_region: "eu-west-1"
  aws_s3_bucket_name: "backup-bucket"
```
*NB: Use the command "helm template" to see all configurations generated*


## **Upgrade**
When you make any changes on the values, the helm chart creates a configuration job that will apply all "Values.databases" configurations on your instances without restart. If you remove a database on "Values.databases", the database wont be delected on instances, so you need to delete the database and admin user manually.

### **Things that cannot be changed on values**
* Values.name;
* Values.name_suffix;
* Values.version.

### **Attention**
* Some changes will restart the database instances;
* If you change the root username or password, the database instances will be restarted.

## DevInfos:
- James Artur (Thunder), a DevOps and infrastructure enthusiastics.