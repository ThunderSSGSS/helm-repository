# Chart Base-app 
This chart helps you to deploy stateless apps on k8s.

### Values
The default values:
```yaml
name: app-name

ingress:
  create: false
  host: host.com
  paths:
    - path: /path/
      pathType: Prefix

deployment:
  replicas: 1
  image: image_name
  tag: tag
  pullPolicy: IfNotPresent
  
  # ports:
  # - name: http
  #   port: 80
  
  # envs:
  #  - name: ENV_NAME1
  #    value: value

  #secrets:
  # - name: ENV_NAME2
  #   value: base64_value

service:
  create: false
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
```

## DevInfos:
- Name: James Artur (Thunder);
- A DevOps and infrastructure enthusiastics.