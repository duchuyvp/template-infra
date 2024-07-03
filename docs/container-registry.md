## This page describes how to pull and push to the self-hosted container registry

### 1. Credentials

You can use either the username and password or the token to authenticate with the container registry.

Contact Huy to get the credentials.

```bash
docker login jcr.lunalovegood.dev -u <username> -p <password or token>
```

### 2. Pull images

```bash
docker pull jcr.lunalovegood.dev/docker/<image-name>:<tag>
```

#### Pull image in Kubernetes
  
Specifying `imagePullSecrets` on a Pod (or Deployment, StatefulSet, or any workload)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hoaem-fe-prod
spec:
  containers:
  - name: hoaem-fe-prod
    image: jcr.lunalovegood.dev/docker/hoaem-fe:latest
  
  imagePullSecrets:
  - name: jcr-image-pull-secret
```

You can see I use secret `jcr-image-pull-secret` and may wonder where this secret comes from. It is (*They are*) created when I deploy private registry and you can use it to pull images from the registry on every current and future namespaces.

### 3. Push images

```bash
docker tag <image-id> jcr.lunalovegood.dev/docker/<image-name>:<tag>
docker push jcr.lunalovegood.dev/docker/<image-name>:<tag>
```

### Additional Information

The container registry is hosted at `jcr.lunalovegood.dev` also supports for `helm` and `oci` repo. You can figure out by yourself.
