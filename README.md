# Eriknet Kubernetes getting started

## Prerequisites
- eriknet kubernetes config file
- docker registry account

## Before you begin
In the steps below:
- make sure to replace abcd with a simple name of your own 
- replace YOUR_DOCKER_ACCOUNT with your docker account id

## Create a Docker container

Dockerfile
```
FROM nginx:latest
RUN echo "Hello abcd" > /usr/share/nginx/html/test.txt
```

create the docker image
```
docker build . -t YOUR_DOCKER_ACCOUNT/nginx-abcd
```

push to the docker register
```
docker push YOUR_DOCKER_ACCOUNT/nginx-abcd
```



## Kubernetes

### Create a namespace
```
kubectl --kubeconfig eriknet create namespace abcd
```

### Create a Kubernetes deployment and service
nginx-abcd.yaml
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-abcd
spec:
  selector:
    matchLabels:
      app: nginx-abcd
  template:
    metadata:
      labels:
        app: nginx-abcd
    spec:
      containers:
      - name: echo1
        image: YOUR_DOCKER_ACCOUNT/nginx-abcd
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-abcd
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx-abcd
```

#### apply the yaml
```
kubectl --kubeconfig eriknet -n abcd apply -f nginx-abcd.yml
```

#### check if everythings is OK
```
kubectl --kubeconfig eriknet -n abcd get deployment
kubectl --kubeconfig eriknet -n abcd get pods
kubectl --kubeconfig eriknet -n abcd get services
```

### Create the ingress


#### ingress yaml
ningx-abcd-ingress.yml
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-xzy
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - abcd.eriknet.net
    secretName: abcd-tls
  rules:
  - host: abcd.eriknet.net
    http:
      paths:
      - backend:
          serviceName: nginx-abcd
          servicePort: 80
```

#### apply the ingress
```
kubectl --kubeconfig eriknet -n abcd apply -f nginx-abcd-ingress.yml
```


## Check the result

point your browser to: 

https://abcd.eriknet.net/test.txt
