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

### 3. Push images

```bash
docker tag <image-id> jcr.lunalovegood.dev/docker/<image-name>:<tag>
docker push jcr.lunalovegood.dev/docker/<image-name>:<tag>
```

### Additional Information

The container registry is hosted at `jcr.lunalovegood.dev` also supports for `helm` and `oci` repo. You can figure out by yourself.
