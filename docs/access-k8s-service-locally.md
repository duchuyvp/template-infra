## Access Kubernetes Service Locally

This document describes how to access a Kubernetes service from a machine in LAN.

First, I create a simple deployment and service in Kubernetes.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin-deployment
  namespace: default
spec:
  selector:
    matchLabels:
      app: httpbin
  replicas: 2
  template:
    metadata:
      labels:
        app: httpbin
    spec:
      containers:
        - name: httpbin
          image: kennethreitz/httpbin:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: default
spec:
  selector:
    app: httpbin
  ports:
    - protocol: TCP
      port: 80
```

To access a Kubernetes service from a machine in LAN, there are several options. I use service `web-service` as described above to demonstrate example.

### 1. Port Forwarding (Recommended)

```bash
kubectl port-forward -n default svc/web-service 8080:80
```

Now, you can access the service at `http://localhost:8080`.

#### Limitation

- You need to keep the terminal open to keep the port forwarding active.
- You can only access the service from the machine where the port forwarding is running.

### 2. Using LoadBalancer service

If you have a load balancer service or using cloud provider, you can access the service using the external IP address of the load balancer.

Edit the service to change the type to `LoadBalancer`.

```bash
kubectl patch svc -n default web-service -p '{"spec": {"type": "LoadBalancer"}}'
```

Get the external IP address assigned to service.

```bash
kubectl get svc -n default web-service
```

```
NAME          TYPE              CLUSTER-IP      EXTERNAL-IP     PORT(S)         AGE
web-service   LoadBalancer      10.43.116.84    192.168.1.101   80:32430/TCP    43s
```

Now, you can access to service `web-service` by `http://<EXTERNAL-IP>:80` from everywhere in LAN.

If EXTERNAL-IP of service is `<pending>`, that means you have no load balancer service support or no IP left from the pool. You can use NodePort service instead.

#### Limitation

- You need to have a load balancer service or using cloud provider. (You can use MetalLB to create a load balancer service in on-premise environment)
- EXTERNAL-IP of service may change after restart or re-deploy service.
- service type LoadBalancer also expose a NodePort, both NodePort and IP address are limited.

### 3. Using NodePort service

Edit the service to change the type to `NodePort`.

```bash
kubectl patch svc -n default web-service -p '{"spec": {"type": "NodePort"}}'
```

Get the NodePort assigned to service.

```bash
kubectl get svc -n default web-service
```

```
NAME          TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
web-service   NodePort   10.43.116.84   <none>        80:32430/TCP   8m35s
```

Now, you can access to service `web-service` by `http://<node-ip>:32430` from everywhere in LAN.

#### Limitation

- You need to know the IP address of the node where the service is running.
- NodePort is limited

### 4. Using Ingress (Recommended)

Ingress is an API object that manages external access to the services in a cluster, typically HTTP.

Create an Ingress for `web-service`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: web.hoaem.vn
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

Now, you can access to service `web-service` by `http://web.hoaem.vn` from everywhere in the world.

This way required you have a domain and may not work for all domain because some specific domain require TLS certificate and auto redirect to `https` (.dev, etc.), but currently, I use free TLS certificate by Cloudflare to secure the connection so you can feel free without TLS.
Many providers do not provide free TLS certificate or simply you don't trust Cloudflare, you can generate and add your own TLS certificate to Ingress (I'm using Let's Encrypt).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  rules:
    - host: web.hoaem.vn
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
  tls:
    - hosts:
        - web.hoaem.vn
      secretName: web-hoaem-vn-tls
```

#### Limitation

- You need to have a domain.
- Cloudflare restricts to use their TLS certificate and only free for *.hoaem.vn, hoaem.vn. All other common names will be charged.
- Service is publicly accessible.
