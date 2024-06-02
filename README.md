# Basic image-registry

A basic insecure home lab deployment of image-registry

Don't use this in production or any public server!

> This deployment was tested in k3s only. You may need to adapt one thing or two if you are deploying this image-registry in a different cluster.

## Deployment

All required resources are contained in a single YAML file, which creates:

1. A `image-registry` namespace, where all resources will be deployed
1. The image-registry deployment
1. A service exposing the port 5000
1. A PersistentVolumeClaim referencing the local-path storage class

## Configuration

After the image registry is deployed, make the following configuration:

### In the k3s cluster

Create the file `/etc/rancher/k3s/registries/yaml` with the following content:

```yaml
mirrors:
  "homelab:5000":
    endpoint:
      - "http://homelab:5000"
  "homelab.local:5000":
    endpoint:
      - "http://homelab.local:5000"
configs:
  "homelab:5000":
    tls:
      insecure_skip_verify: true

    "homelab.local:5000":
    tls:
      insecure_skip_verify: true
```

Change the hostname and ports accordingly.

Optional: Enable k3s logs with `sudo k3s server --log=/etc/rancher/k3s/log.txt`

Restart k3s with `sudo systemctl restart k3s`.

> A sample file is provided in this repo

### In the client machines

Setup Docker CLI to be able to accept the insecure image registry.

Create the file `/etc/docker/daemon.json` (for Linux or adapt it to your OS, Docker version and so on):

```json
{
  "insecure-registries": [
    "homelab:30729",
    "homelab:5000"
  ]
}
```

Fix the hostname accordingly.

> A sample file is provided in this repo

## Testing

If all went well, create a local image, tag it accordingly and try to push it to the image registry:

```bash
docker build -t myapp .
docker tag myapp homelab:5000/myapp
docker push homelab:5000/myapp
```

