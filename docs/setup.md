# Setup

## Create K8s cluster

Use [kind](https://github.com/kubernetes-sigs/kind) to start a new local cluster

```
$ kind create cluster --image=kindest/node:v1.19.7
```

## Install ArgoCD

Now that the cluster is up and running, let's build the manifests of ArgoCD using kustomize and apply them.

```
$ docker run --rm -v $(pwd)/argocd_deploy:/app/manifests k8s.gcr.io/kustomize/kustomize:v3.10.0 build manifests | kubectl apply -f -
```

ArgoCD is now being installed. You can follow the progress with:

```
$ kubectl get po -n argocd -w
```

Take this time to have a look a the resources in the `GitOps-workshop/argocd_deploy` folder.
You will find some commented lines and files, we will use them in the next steps.

## Login web interface

* `kubectl port-forward svc/argocd-server -n argocd 8080:443`
* `kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2` # Get admin password
* Login on localhost:8080 with username "admin" and password from previous step

## Login CLI (optional)

* `argocd login localhost:8080`

