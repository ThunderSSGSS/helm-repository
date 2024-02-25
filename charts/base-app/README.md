# **CHART base-app** 
This chart helps you to deploy stateless apps on k8s, you can enables ingress and
defines network policies for internal or external services. <br>

## **Cluster Requirements**
To use all features that this charts provides, your K8s cluster must have:
* [Cert-Manager](https://cert-manager.io/);
* [Container Network Interface (CNI)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/);
* [Metric Server](https://kubernetes-sigs.github.io/metrics-server/);
* [Nginx Ingress Controller](https://github.com/kubernetes/ingress-nginx).<br>

## **Values**

### Default Values:
```yaml
name: app-name

ingress:
  create: false # when true creates a ingress
  # additionalAnnotations: # Set additional nginx controller annotations
  #   nginx.ingress.kubernetes.io/proxy-body-size: 30m 
  host: host.com
  # hosts: ["host.com"]
  # redirectHosts: ["www.host.com"]
  enable_tls: false # only when true | enables tls and requests a certificate from clusterIssuer
  clusterIssuer: letsencrypt-prod
  # tlsSecretName: secret-name # set certificate secret name
  paths:
  - path: /
    pathType: Prefix # values: Exact, Prefix
    servicePort: 80
  # error page
  errorPage:
    enable: false
    path: /static/project/error/index.html
    host: tech-minio.cdn
    port: 9000
    protocol: HTTP # HTTP, HTTPS, GRPC, GRPCS and FCGI
  # cache configurations
  cache:
    enable: false # if enable will create a new ingress with static cache configurations
    servicePort: 80
    pathRegex: /.+\.(css|js|jpg|jpeg|png|gif|ico)
    configuration: |
      expires 7d;
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


deployment:
  replicas: 1

  image: image_name
  tag: image_tag
  pullPolicy: Always
  # command: ['/bin/bash', 'hello.sh'] # same as entrypoint
  # args: ['--arg=true'] # same as command
  # imagePullSecret: example-registry

  # securityContext:
  #   privileged: true

  # nodeSelector:
  #   KEY: VALUE

  # annotations: # add more pod annotations
  #   linkerd.io/inject: enabled

  # ports:
  # - name: http # UNIQUE
  #   protocol: TCP # values: TCP, UDP
  #   containerPort: 8080
  #   servicePort: 80
  #   nodePort: 5833 # OPTIONAL if set will enabled node port

  # resources:
  #   requests:
  #     cpu: 50m
  #     memory: 100Mi
  #   limits:
  #     cpu: 100m
  #     memory: 250Mi

  # envs: # will be stored on a secret
  #   KEY: VALUE

  # dotenv: # wil be stored on dotenv file
  #   path: /app
  #   file: ".env"
  #   content: |
  #     asdasdasdasdasdasda

  # services: # will use same configs, volumes, security context and more
  # - name: name
  #   image: image
  #   tag: tag
  #
  #   command: ['/bin/bash', 'hello.sh']
  #   args: ['--arg=true']
  #
  #   securityContext:
  #     privileged: true

  # volumes:
  # - name: data # UNIQUE
  #   storageClassName: gp2
  #   mountPath: /data # UNIQUE
  #   capacity: 1Gi

  # hostPathVolumes:
  # - name: data3 # UNIQUE
  #   type: DirectoryOrCreate # File # Directory
  #   hostPath: /data/host
  #   mountPath: /data/container


horizontalAutoscaler:
  create: false # when true, creates HorizontalPodAutoscaler | only set to true if you define .Values.deployment.resources
  maxReplicas: 2 # must be more than .Values.deployment.replicas
  targetCPUUtilizationPercentage: 80


# upstreams: # creates network policies to access other pods
# - label: app # pod selector label
#   label_value: app2 # pod selector label value
#   containerPort: 8080
#   namespace: default # OPTIONAL | default: This namespace
#   protocol: TCP # OPTIONAL | values: TCP, UDP | default: TCP
#   paths: # OPTIONAL | only when http
#   - path: /
#     pathType: Prefix # values: Prefix, Exact
#     methods: ["GET","HEAD","POST","PUT","DELETE","CONNECT","OPTIONS","TRACE"] # you can use "[]" to enables all methods      


# external_upstreams: # creates services and network policies external services
# - host: external.aws.amazon.com
#   protocol: TCP # OPTIONAL | values: TCP, UDP | default: TCP 
#   ips: ["192.168.100.100"]
#   port: 5003
#   paths: # OPTIONAL | only when http
#   - path: /metrics/mega
#     pathType: Exact # values: Prefix, Exact
#     methods: ["GET"] # you can use "[]" to enables all methods         
```
*Note: All features on a comment is disabled, discomment if you want to enables*
<br>

### **Root Values**
| Name | Type |  Description  |
|:----:|:----:|---------------|
| name | string | The name of the app, used to create all k8s resources |
| ingress | block | Contain all configurations to generate ingress and to enables tls |
| deployment | block | Contain all configurations to create app configmap, secrets, deployments, services and volumes |
| horizontalAutoscaler | block | Contain configurations to create autoscaler |
| upstreams | list | Contain configurations to create network policies to access other pods |
| external_upstreams | list | Contain configurations to create network policies to access external hosts |<br>

## **Examples**
### Example1
We want to deploy a SpringBoot api:<br>
![title](./example1.png)
<br>
**The api values**:
```yaml
name: api

ingress:
  create: true
  host: api.domain.com
  enable_tls: true
  clusterIssuer: letsencrypt-prod
  paths:
  - path: /
    pathType: Prefix
    servicePort: 80

deployment:
  replicas: 1
  image: api-image
  tag: v1
  pullPolicy: Always

  ports:
  - name: http
    protocol: TCP
    containerPort: 8080
    servicePort: 80
  
  resources:
    requests:
      cpu: 50m
      memory: 100Mi
    limits:
      cpu: 100m
      memory: 250Mi

  envs: # will be stored on a secret
    DATABASE_URL: jdbc:postgresql://rds.12345.aws.com/api_database
    DATABASE_USER: user
    DATABASE_PASSWORD: password
    CACHE_HOST: cache-db-svc
    CACHE_PORT: 6379
  
  dotenv: # will be stored on a secret
    path: /app
    file: application.yaml
    content: |
      server:
        port: 8080
      spring:
        cache:
          redis:
            port: ${CACHE_PORT}
            host: ${CACHE_HOST}
        datasource:
          url: ${DATABASE_URL}
          username: ${DATABASE_USER}
          password: ${DATABASE_PASSWORD}

horizontalAutoscaler:
  create: true
  maxReplicas: 3
  targetCPUUtilizationPercentage: 80

upstreams:
- label: app
  label_value: cache-db
  containerPort: 6379
  protocol: TCP

external_upstreams:
- host: rds.12345.aws.com
  protocol: TCP
  ips: ["192.168.1.100"]
  port: 5432
```
*NB: Use the command "helm template" to see all configurations generated*


## DevInfos:
- James Artur (Thunder), a DevOps and infrastructure enthusiastics.