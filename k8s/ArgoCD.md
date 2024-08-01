> ### :question: Why ArgoCD
#### :point_right: ArgoCD handle CD with a UI friendly interface
<!-- ![Argo-UI](https://hackmd.io/_uploads/BJu1swG8R.png) -->
![Argo-UI](https://github.com/Sakuard/k8s/blob/main/src/argo/Argo-UI.png)

### :point_right: GitLab-Agent vs ArgoCD
<!-- ![Agent-Argo](https://hackmd.io/_uploads/SJG3GdML0.png) -->
![Agent-Argo](https://github.com/Sakuard/k8s/blob/main/src/argo/Agent-Argo.png)
### :point_right: Deploy with ArgoCD
1. install ArgoCD in kubernetes cluster
2. Login ArgoCD
3. Deploy setup

### :heavy_check_mark: Install
```bash=
# install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
###  :heavy_check_mark: Login ArgoCD
1. Set ArgoCD-Server into LoadBalancer
```yaml=
# edit ArgoCD-svc
kubectl edit svc argocd-server -n argocd
# update `sepc.type` to LoadBalancer
spec:
  type: LoadBalancer
```
2. Get init admin password
```bash=
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode; echo
```
3. Login with admin
<!-- ![Argo-Login](https://hackmd.io/_uploads/SyQ1a_MIA.png) -->
![Argo-Login](https://github.com/Sakuard/k8s/blob/main/src/argo/Argo-Login.png)

### :heavy_check_mark: Deploy setup
- Repo Connect Setup <br/>:point_right: :gear:Settings > Repository > Connect REPO
    - repo url
    - user
    - user-pass
- Add up service CD setup

![argo-demo](https://github.com/Sakuard/k8s/blob/main/src/argo/argo-demo.gif)