Jenkins setup on kubernetes
============================

Setup Jenkins on kubernetes on MacOS.

- practice Kubernetes basics 
- practice Jenkins basics
- Certified Jenkins Engineer(CJE) exam prep
- Certified Kubernetes Adminitorator(CKA) exam prep

## Prerequiresite

- MacOS
- Colima `brew install colima`
- kubectl `brew install kubectl`
- Docker `brew install docker`
- Minikube([install](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download))

## Start Kubernetes

```
colima start # start VM
minikube start # start kubernetes inside the VM
```

## Enable minikube addons

```
minikube addons enable metrics-server # for dashboard
minikube addons enable ingress # for routing
```

## Install 

```
kubectl create -f jenkins-pvc.yaml -n jenkins
kubectl create -f jenkins.yaml -n jenkins
kubectl create -f jenkins-service.yaml -n jenkins
kubectl apply -f jenkins-ingress.yaml
```

Open `http://jenkins.localhost`.

## Minikube Dashboard

```
minikube dashboard # monitor pods, services, and so forth
```

## Stop and delete kubernetes

```
minikube stop
minikube delete # tear down the current k8s

colima stop
colima delete # tear down the current VM
```