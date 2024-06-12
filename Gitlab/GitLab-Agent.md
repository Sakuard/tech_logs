### GitOps: Build and Deploy on K8S via Gitlab-CI with Gitlab-Agnet
![GitOps-Flow](https://hackmd.io/_uploads/HJNG9VBrC.png)
> Flow
1. Commit a new udpate to a repository.
2. Trigger Gitlab-ci.
3. In ```.gitlab-ci.yml```, stage: build
Build image and push it to registry.
4. In ```.gitlab-ci.yml```,stage: deploy
Trigger Gitlab-Agent.
5. Gitlab-Agent will verify it operation permission for the repository.
6. Trigger the operatino on the registered K8S cluster.
7. Execute the commands written in ```.gitlab-ci.yml``` on K8S cluster.

> Requirement
- Gitlab Account
- Access to a K8S cluster

#### Block 1
1. Create a gitlab repo for Gitlab-Agent
2. Create an Agent
3. Create a K8S cluster
4. Install and register connection between K8S cluster and Gitlab-Agnet
5. Set up project repo's accessibility

#### Block 2
1. Create a gitlab project repo
2. Build a service
3. Build a .gitlab-ci.yml

### Step by Step

1. Create 2 repo
One for Gitlab-Agnet([ref](https://docs.gitlab.com/ee/user/clusters/agent/ci_cd_workflow.html)), another for Project
```bash=
# path of gitlab-agent's repo
.gitlab/agents/<agnet-name>/config.yaml
```
2. Build and test docker image locally, and push it on to Gitlab-Registry manually
3. Build a .gitlab-ci.yml with automatically image-op([ref](https://docs.gitlab.com/ee/user/packages/container_registry/build_and_push_images.html))
```yaml=
stages:
  - build

variables:
  GIT_REPO: "<path/to/gitlab-repo>" #<對應Gitlab repo的路徑>
  IMAGE_NAME: "go-server"
  IMAGE_TAG: "0.0.1"

build:
  image: docker
  stage: build
  services:
    - docker:dind
  script:
    # docker login內的變數is set default by Gitlab.
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - if
        docker manifest inspect $CI_REGISTRY/$GIT_REPO/$IMAGE_NAME:$IMAGE_TAG > /dev/null 2>&1;
        then echo "Image tag already exists, failing the job";
        exit 1;
      fi
    - docker build -t $CI_REGISTRY/$GIT_REPO/$IMAGE_NAME:$IMAGE_TAG .
    - docker push $CI_REGISTRY/$GIT_REPO/$IMAGE_NAME:$IMAGE_TAG
```
4. Set up permision for K8S to access image pulling of Gitlab-Registry
```bash=
# create a yaml file, which includes secret config
kubectl create secret docker-registry <secret-name> --docker-server=registry.gitlab.com --docker-username="user" --docker-password="pass" --dry-run=client -o yaml > secret.yaml
# apply this secret.yaml to k8s
kubectl apply -f secret.yaml
```
5. Create K8S's yaml and apply on K8S to check if its works
```yaml=
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-dep
  namespace: default
spec:
  replicas: 1
  revisionHistoryLimit: 1
  selector:
    matchLabels:
      app: go-serv
  template:
    metadata:
      labels:
        app: go-serv
    spec:
      containers:
      - image: registry.gitlab.com/<path/to/repo>/<image-name>:<tag>
        name: go-server
        imagePullPolicy: Always
        resources:
          limits:
            cpu: 100m
          requests:
            cpu: 50m
        ports:
          - containerPort: 3000
      - image: nginx:latest
        name: nginx
        resources:
          limits:
            cpu: 100m
          requests:
            cpu: 50m
        volumeMounts:
        - mountPath: /var/www/html
          name: shared-files
        - mountPath: /etc/nginx/nginx.conf
          name: go-serv-nginx
          subPath: nginx.conf
      volumes:
      - emptyDir: {}
        name: shared-files
      - configMap:
          name: go-serv-nginx
        name: go-serv-nginx
```
6. Add stage:deploy([ref](https://docs.gitlab.com/ee/user/clusters/agent/ci_cd_workflow.html))
```yaml=
stages:
  - build
  - deploy

variables:
  GIT_REPO: "<path/to/gitlab-repo>" #<對應Gitlab repo的路徑>
  IMAGE_NAME: "go-server"
  IMAGE_TAG: "0.0.1"

build:
  image: docker
  stage: build
  services:
    - docker:dind
  script:
    # docker login內的變數is set default by Gitlab.
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY/$GIT_REPO/$IMAGE_NAME:$IMAGE_TAG .
    - docker push $CI_REGISTRY/$GIT_REPO/$IMAGE_NAME:$IMAGE_TAG
    
# CI/CD deploy config
deploy:
  image:
    name: bitnami/kubectl:latest
    entrypoint: ['']
  script:
    - kubectl config get-contexts
    - kubectl config use-context path/to/agent/project:agent-name
    - kubectl get pods
```
7. Set up Gitlab-Agent's repo permission
```yaml=
# Project repo's .gitlab-ci.yml
deploy:
  image:
    name: bitnami/kubectl:latest
    entrypoint: ['']
  script:
    - kubectl config get-contexts
    - kubectl config use-context path/to/agent/project:agent-name
    - kubectl get pods
```
```yaml=
# Gitlab-Agent's repo permission
# .gitlab/agents/<agent-name>/config.yaml
ci_access:
  groups:
    - id: path/to/group/subgroup
```

#### Ref:
- Youtube
1. [How to Build and Deploy and app on K8S by GitLab CI/CD pipeline](https://youtu.be/fwtxi_BRmt0?si=mzOx3satl8CPMhzV)
- GitLab Document
1. [Using GitLab CI/CD with a K8S cluster](https://docs.gitlab.com/ee/user/clusters/agent/ci_cd_workflow.html)
2. [Build and push container images to the container registry](https://docs.gitlab.com/ee/user/packages/container_registry/build_and_push_images.html)
