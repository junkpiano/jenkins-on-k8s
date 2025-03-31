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
kubectl create namespace jenkins
kubectl create -f jenkins-pvc.yaml -n jenkins
kubectl create -f jenkins-service.yaml -n jenkins
kubectl create -f jenkins-serviceAccount.yaml -n jenkins
kubectl create -f jenkins.yaml -n jenkins
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

## Lesson Learned

I faced many pitfalls and gave a lot of attempts to get it right.
Here is the lesson I learned through this hands-on.
This might help you as well. Once you finish your environemnt setup,
read below to review your work.

<details>
<summary>click to read</summary>

## Summary: Lessons I Learned — Running Jenkins Agents on Kubernetes

### 1. Jenkins Kubernetes Plugin = Dynamic Agent Pods

You learned how to:

* Use the Kubernetes plugin to launch agents as Kubernetes pods
* Set up a Pod Template that defines what the agent pod should look like
* Let Jenkins automatically schedule builds inside these pods

This is the foundation for scalable, cloud-native CI/CD.

### 🔐 2. Jenkins agents connect via port 50000 (JNLP)

You learned:

* The Jenkins controller must expose port 50000
* The Kubernetes Service must route both 8080 and 50000
* The agent pod connects back to the controller using JNLP with:

### 🧰 3. You need proper Pod Template configuration

Specifically:

* Container name must be jnlp
* No command override (leave blank)
* Proper arguments ($(JENKINS_SECRET) $(JENKINS_NAME))

Use a recent and compatible image:

```
jenkins/inbound-agent:latest
```

And you must match the label:

```
metadata:
  labels:
    jenkins/label: k8s-agent
```

With the label used in your pipeline.

### 🛡️ 4. Jenkins needs RBAC credentials to talk to Kubernetes

You created:

* A ServiceAccount for Jenkins
* A ClusterRole granting access to create/manage pods
* A ClusterRoleBinding to connect them

This is what allowed Jenkins to actually create agent pods in your cluster.

### 🧠 5. Jenkins controller != agent

You got this important lesson:

“Jenkins doesn’t have label 'foo'” does not mean Jenkins controller is broken
It means Jenkins couldn’t find a matching agent pod that connected back.

And to fix it:

* Add correct labels
* Use the node() block when needed
* Confirm that the agent actually connected successfully

### ✅ 6. Final Result: CI builds run dynamically on K8s pods!

You now have:

* Jenkins running in Kubernetes
* Agent pods launching dynamically
* Builds executing cleanly inside isolated pods

A truly cloud-native CI/CD pipeline setup 🔥
</details>