# Setup

Three clusters:
- global: will contain the control board
- primary: first cluster
- secondary: second cluster

This will allow testing applications that are spread over multiple k8s clusters.

## Bootstrap

Everything needed to set up argocd and link it to the other clusters.

Create the minikube clusters:
```bash
minikube start --addons=dashboard,ingress -p global
minikube start --addons=dashboard -p primary
minikube start --addons=dashboard -p secondary
```

Optional, but recommended: open a minikube dashboard for each of them.
```bash
minikube dashboard -p global
minikube dashboard -p primary
minikube dashboard -p secondary
```
Each command will hang, so open dedicated shells.

Switch over to global cluster, install argocd, and connect it to the other clusters:
```bash
minikube profile global
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -n argocd -f bootstrap/argocd-ingress.yaml
```
At this point the argocd UI should be reachable at `https://argocd.test`. You may have to add these lines in /etc/hosts:
```bash
${minikube ip -p global} argocd.test
```

Login via the UI then in CLI. Login is `admin`, password can be retrieved via:
```bash
argocd admin initial-password -n argocd
```

Actually login:
```bash
argocd login argocd.test
```

Allow the clusters to communicate together:
```bash
sudo iptables -I FORWARD 1 -j ACCEPT
```

Finally, link the clusters together:
```bash
argocd cluster add primary
argocd cluster add secondary
```

Bootstrap is now done, everything else can be driven via argocd now.

== Prerequisites ==

This contains most pre-required definitions for all other apps, including basic namespaces.
```bash
argocd appset create prerequisites/appset-prerequisites.yaml
argocd app sync prerequisites-{primary,secondary}
```

== Guestbook sample app ==

Taken from https://github.com/argoproj/argocd-example-apps.git.

Install the application on the two clusters:
```bash
argocd appset create guestbook/appset-guestbook.yaml
```
