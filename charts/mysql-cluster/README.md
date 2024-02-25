# **CHART mysql-cluster** 
This chart helps you to deploy and maintain a mysql standalone or cluster instances.<br>

## **Cluster Requirements**
To use all features that this charts provides, your K8s cluster must have:
* [Metric Server](https://kubernetes-sigs.github.io/metrics-server/).<br>

## **Values**

### Default Values:
```yaml
# Resources names prefix, all resources names will starts with <name>-<name_suffix>
name: test
name_suffix: mysql

imagePullPolicy: IfNotPresent # all images pull policy
version: 8.0.33 # tag of mysql official image

# CREDENTIALS
root_user: root # the root user | we recommend dont change
root_user_pwd: password

# this option will change authentication method for all databases admin users
authentication_method: caching_sha2_password # mysql_native_password

databases:
- name: test # name of the database | for best practice use "_" to separe words 
  enable_backup: false # if true, enable backup and create cronjob
  # backup_suspend: false # if true, suspend backup cronjob
  backup_schedule: "0 2 * * 0" # used when enable_backup is true
  backup_s3_bucket_path: "test-db-backups" # used when enable_backup is true
  # admin_user: admin # database admin user | the user will be created if doesn't exist
  # admin_user_pwd: password # must be setted if admin_user is setted 

# CONNECTIONS
max_connections: 150

# RESOURCES LIMITS
resources:
  enable: false
  dbInstances: # mysql instances resource limits
    requests:
      cpu: 50m
      memory: 100Mi
    limits:
      cpu: 200m
      memory: 1500Mi
  proxysql: # proxysql instances resource limits
    requests:
      cpu: 50m
      memory: 100Mi
    limits:
      cpu: 200m
      memory: 1500Mi


# CLUSTER MODE
cluster_mode:
  enable: false
  cluster_key: S0tLS0tLS0tLS0tLS0tLS0tLfgfhfhf # cluster user password | used on instance to instance authentication
  readReplicas: 1 # minumum mysql read replicas
  proxysqlReplicas: 1 # minumum proxysql instances
  proxysqlTag: 2.5.2 # tag of proxysql official image | we recommend use the default value
  autoscaler: # you must enable resources limits to enable autoscaler
    enable: false
    targetCPUUtilizationPercentage: 80
    maxReadReplicas: 2
    maxProxysqlReplicas: 2
  schedule:
    enable: false
    #proxysqlNodeSelector: # node selector for proxysql
      #node_label: value
    #dbInstancesNodeSelector: # node selector for database instances
      #node_label: value


# STORAGE
storage:
  enable: false
  storageClassName: gp2
  capacity: 10Gi

# SERVICE
service:
  add_node_port: false # if true, exposes master instance on node port
  nodePort: 31951


# PROMETHEUS METRICS
metrics: # you need to enable cluster_mode to use this feature
  enable: false # if true, exposes read replicas metrics to prometheus operator
  exporterTag: v0.14.0 # tag of mysql exporter official image | we recommend use the default value
  scrapeInterval: 15s
  additionalLabels: {} # labels to add on podmonitor


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
We want to deploy a standalone mysql instance with one database and 2GB of persistent storage:<br>
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
We want to deploy a mysql cluster with:<br>
* 2 read replicas;
* 2 databases, "prod" and "staging", the "prod" database needs weekly backup;
* 5Gi of persistent storage.<br>

**The cluster values:**
```yaml
name: cluster

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
  proxysqlReplicas: 1

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