# Kubernetes GitOps Workshop

******

## Setup

### Create K8s cluster

```
$ kind create cluster --image=kindest/node:v1.19.7
```

### Install ArgoCD

```
$ docker run --rm -v $(pwd)/argocd_deploy:/app/manifests k8s.gcr.io/kustomize/kustomize:v3.10.0 build manifests | kubectl apply -f -
```

ArgoCD is now being installed. You can follow the progress with:

```
$ kubectl get po -n argocd -w
```

Take this time to have a look a the resources in the `GitOps-workshop/argocd_deploy` folder.
You will find some commented lines and files, we will use them in the next steps.

### Login web interface

* `kubectl port-forward svc/argocd-server -n argocd 8080:443`
* `kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2` # Get admin password
* Login on localhost:8080 with username "admin" and password from previous step

### Login CLI (optional)

* `argocd login localhost:8080`

******

## Manage ArgoCD ... with ArgoCD!

It is possible to let ArgoCD manage itself! This is actually the recommended way of deploying ArgoCD, as described in the [docs](https://argoproj.github.io/argo-cd/operator-manual/declarative-setup/#manage-argo-cd-using-argo-cd).

### Installing Tools

ArgoCD supports several ways of generating Kubernetes manifests.
Defining manifests is supported using different [Tools](https://argoproj.github.io/argo-cd/user-guide/application_sources/).

ArgoCD is very unopinionated and leaves the operator total freedom regarding
which tool to use to generate manifests. We will see:

* Kustomize
* Helm
* Plain YAML

#### Kustomize

In the previous step, we installed ArgoCD manually using [Kustomize](https://github.com/kubernetes-sigs/kustomize).
In order to let ArgoCD manage itself, we need to make sure that it is able to run Kustomize

Open the `argocd_deploy/kustomization.yaml` file and uncomment the following section:

```
#patchesStrategicMerge:
#- overlays/argocd-cm.yaml
#- overlays/argocd-repo-server-deploy.yaml
```

After applying this change:
* Kustomize will be downloaded on boot
* ArgoCD will allow us to use it in our manifests

Apply the changes to the ArgoCD deployment by running:

```
$ docker run --rm -v $(pwd)/argocd_deploy:/app/manifests k8s.gcr.io/kustomize/kustomize:v3.10.0 build manifests | kubectl apply -f -
```

At this point, Kustomize is installed in our ArgoCD instance and ready to be used.
ArgoCD tools are useful also to test upgrades.

### Add ArgoCD to the managed applications

All the tools needed are now installed and we are ready to let ArgoCD manage itself.
Whenever possible we should define our applications in a declarative way, the same applies for ArgoCD

Find the ArgoCD definition at `argocd_deploy/apps/argocd.yaml`
Take some time to have a look at the file and replace the fields that need to be edited.

When you're done, uncomment this section from `argocd_deploy/kustomization.yaml`:

```
resources:
- resources/namespace.yaml
- apps/argocd.yaml
```

And apply the changes (last manual apply):

```
$ docker run --rm -v $(pwd)/argocd_deploy:/app/manifests k8s.gcr.io/kustomize/kustomize:v3.10.0 build manifests | kubectl apply -f -
```

With this change, ArgoCD is now managing itself using the definition in our repository.
Verify on ArgoCD interface that a new application has been installed and is working

### Test - ArgoCD upgrade

But... Will it actually work?

To test it out we will let ArgoCD upgrate itself!
The installed version is v1.8.6 but v1.8.7 is out... let's upgrade!

Have a look at the `argocd_deploy/kustomization.yaml` file... Can you figure out how to do it?

Apply the change by committing and pushing to `master`, no need to use the docker command line anymore.
After a couple of minutes you should see the new version deployed!

******

## Monitoring? Let's do it!



* `kubectl get secret --namespace monitoring kube-prometheus-stack-grafana
  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo`
* `k port-forward -n monitoring svc/kube-prometheus-stack-grafana 10000:80`

