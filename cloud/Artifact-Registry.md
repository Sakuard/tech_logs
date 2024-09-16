
### pull image
```bash=

# gcloud auth
gcloud auth application-default print-access-token | helm registry login -u oauth2accesstoken \
--password-stdin https://< region >-docker.pkg.dev

# pull image
helm pull oci://< image-registry-path >/< image-name> --version < version >
tar -xvf < image-name >-< version >.tgz

# use chart with value
helm diff upgrade agent-gateway . \
  -f ../values.yaml \
  -n ingress-nginx \
  --set controller.ingressClassResource.name=agent-gateway \
  --set controller.ingressClass=agent-gateway
```
