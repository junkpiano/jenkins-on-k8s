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
- Helm `brew install helm`

## Start Kubernetes

```
colima start --kubernetes --k3s-arg "" # start VM without k3s args
```

## Install with helm (Recommended)

```
kubectl create namespace jenkins
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm upgrade --install jenkins jenkins/jenkins --namespace jenkins --values jenkins-values.yaml
```

Open `http://jenkins.localhost`

## Install without helm

```
kubectl create namespace jenkins
kubectl create -f jenkins-pvc.yaml -n jenkins
kubectl create -f jenkins-service.yaml -n jenkins
kubectl create -f jenkins-serviceAccount.yaml -n jenkins
kubectl create -f jenkins.yaml -n jenkins
kubectl apply -f jenkins-ingress.yaml
```

Open `http://jenkins.localhost`.

## Stop and delete kubernetes

```
colima stop
colima delete # tear down the current VM
```

## Jenkins Configuration

Let's walk through Pod Template settings (in Jenkins UI)

---
**Where:**

Manage Jenkins → Configure System → Kubernetes Cloud → Pod Templates

---

### Basic Pod Template Settings

| Field           | Example / Description                                             |
|-----------------|-------------------------------------------------------------------|
| Name            | jnlp-agent                                                        |
| Labels          | k8s-agent                                                         |
| Namespace       | jenkins                                                           |
| Service Account | jenkins (with RBAC)                                               |
| Node Selector   | (optional)                                                        |
| Usage           | Only build jobs with label expressions matching this pod template |

### Container Settings (inside Pod Template)

Click Add Container, and fill:

| Field          | Example                                      |
|----------------|----------------------------------------------|
| Name           | jnlp (required)                              |
| Docker Image   | jenkins/inbound-agent:3107.v665000b_51092-15 |
| Working Dir    | /home/jenkins/agent                          |
| Command to run | (leave empty)                                |
| Args           | $(JENKINS_SECRET) $(JENKINS_NAME)            |

### Optional: YAML Override (inline pod definition)

You can also paste a full YAML to override the pod spec:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins/label: k8s-agent
spec:
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:3107.v665000b_51092-15
      args:
        - $(JENKINS_SECRET)
        - $(JENKINS_NAME)
      tty: true
```

💡 This is useful if you want to define multiple containers (e.g., jnlp, maven, node) in one pod.

### Full Working Setup: Jenkinsfile + Pod Template (UI)

Once this Pod Template exists in Jenkins:

```
pipeline {
    agent {
        label 'k8s-agent' // Match the Pod Template label
    }
    stages {
        stage('Run inside K8s agent') {
            steps {
                sh 'echo Hello from Kubernetes!'
            }
        }
    }
}
```

No need to inline YAML if you already defined the Pod Template in the UI.

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