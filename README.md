
# CKS-Study-Guide
  ![CKS](https://devopscube.com/wp-content/uploads/2021/04/CKS-Certification-min.png)

Study Guide for the Certified Kubernetes Security Specialist Exam
Exam Syllabus
-----------------------------------------------------------
| Topic	                                     | Weightage  |
|--------------------------------------------|------------|
| Cluster Setup	                             | 10%        |
| Cluster Hardening	                         | 15%        |
| System Hardening	                         | 15%        |
| Minimize Microservice Vulnerabilities	     | 20%        |
| Supply Chain Security	                     | 20%        |
| Monitoring, Logging, and Runtime Security	 | 20%        |
-----------------------------------------------------------

## Learning Path and Testing Environments 
You need to _PRACTICE!_ There is no way to pass this without getting your hands dirty. Because of that, here are a few:
1. [Killercoda](/killercoda.com)
2. [Killer.sh](killer.sh)
3. [KodeKloud](https://kodekloud.com/)

## Documentation Pages Allowed During Exam
1. [Kubernetes Documentations](https://kubernetes.io/docs/home/)
2. [Kubernetes Blog](https://kubernetes.io/blog/)
3. [Trivy](https://github.com/aquasecurity/trivy)
4. [falco](https://falco.org/docs/)
5. [apparmor](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)


Things to understand before going INTO your exam

* Ensure that you have a mouse with you for faster COPY/PASTE actions during the exam. Don't count on the key commands as they are different with the Virtual Machine given to you. 
* You will NOT be able to have a browser with preorganized bookmarks. There will be a virtual desktop that you can access to dive into the kubernetes documentation page. Always, and I mean ALWAYS, ensure that when you're working with the Kube API Yaml, you copy commands and directlt/file paths from the instructions to the
 file. Don't think that in your tired state you'll be able to remember every key.
* Speed is the key, there is no time to get lost in a question. Look at each question and mark the questions you think will take longer than two min to fix.
* When you move to a new question, ensure that you IMMEDIATELY switch contexts. From there, read the entire question as sometimes they will provide a pre-filled out template at the very bottom of the page. 
* When editing the KubeAPI config, ensure that you make a copy first. This will ensure that you're prepared incase the worst happens.
* Understand that because time is NOT on your side, you will need to cut corners for deploying/editing cluster info. Because of this, I encourage you to learn more Imperative Commands which you can reference the official kubernetes guide [here](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run). 


## [Common Commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run) to Know going in to any CK[A/S/D]
-----------------------------------------------------------
| Short Name |       Full Name            |
|------------|----------------------------|
| csr      	 | certificatesigningrequests |
| cs	       | componentstatuses          |
| cm	       | configmaps                 |
| ds	       | daemonsets                 |
| deploy	   | deployments                |
| ep	       | endpoints                  |
| ev	       | events                     |
| hpa      	 | horizontalpodautoscalers   |
| ing	       | ingresses                  |
| limits	   | limitranges                |
| ns	       | namespaces                 |
| no	       | nodes                      |
| nopol      | network policy             |
| pvc	       | persistentvolumeclaims     |
| pv	       | persistentvolumes          |
| po	       | pods                       |
| pdb	       | poddisruptionbudgets       |
| psp	       | podsecuritypolicies        |
| rs	       | replicasets                |
| rc	       | replicationcontrollers     |
| quota	     | resourcequotas             |
| sa	       | serviceaccounts            |
| svc	       | services                   |
-----------------------------------------------------------


## Troubleshooting Commands
* k exec sec -- dmesg | grep -i gvisor     Here we're able to inspect a pod to see if a runtiemclass has been applied     
* grep -ri "phrase you're looking for" /var/www/html      With anything Falco, you need to know how to search for specific rules in the multiple files. This will save you some time.
* kubectl get pods --as dev-user              Confirmation after you finish a section of RBAC is key to ensure you're completing everything. 
* kubectl auth can-i get pods


## Log Locations
```
/var/log/pods
/var/log/containers
crictl ps + crictl logs
docker ps + docker logs (in case when Docker is used)
/var/log/syslog or journalctl
journalctl -fu falco
```

## Cluster Setup & Hardening

### RBAC
Take your time and review the documentation on [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/). Understanding the differences between Role and ClusterRole will help you on the exam. [Sysdig Explaination](https://sysdig.com/learn-cloud-native/kubernetes-security/kubernetes-rbac/#:~:text=In%20Kubernetes%2C%20RBAC%20policies%20can,Kubernetes%20identifies%20as%20service%20accounts.)

A ClusterRole|Role defines a set of permissions and where it is available, in the whole cluster or just a single Namespace.

A ClusterRoleBinding|RoleBinding connects a set of permissions with an account and defines where it is applied, in the whole cluster or just a single Namespace.

Because of this there are 4 different RBAC combinations:

1. Role + RoleBinding (available in single Namespace, applied in single Namespace)
1. ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)
1. ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)
1. Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)

### Network Policies
I am not a fan of networking, but with Kubernetes, there's no way around it. Lucky, there are others who have issues as well and have developed a tool to help out create a Network Policy:
[Network Policy Editor](https://editor.networkpolicy.io/)
Think of the network policy as something you apply to a pod or specific set of pods. You can have multiple assigned to one pod if need be or just one. When you deploy a pod, it has access to everything else in that namespace by default, and this is not what we want in the security space. Here is an example taken from [kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/network-policies/#networkpolicy-resource):

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:                    # this is what you want to apply this particular policy to, and ensure that you are using the labels associated with that pod. ex. k describe pod 
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978

```

Important to know for the exam since there are multiple questions with Network Policies. Look at these two examples:

```
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
```

and: 

```
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
```

In the above exmaple, you are setting a policy to recieve traffic from pods with labels role=client in the namespace user=alice. The following ingress policy states that your pod will get traffic from namespace alice, and any pod with the label role=client.
* hint hint *


### Kubelet Security
Default location for the Kubelet file:
```
/var/lib/kubelet/config.yaml
```

In order to find where the Kubelet file is at, run the ps -aux command:
```
$ px -aux | grep kubelet
```

* When making any changes to the Kubelet service, ensure to RESTART.
```
$ systemctl restart kubelet.service
```

### Service Accounts
* If you add a Service Account to a pod, you MUST re-create the pod as you won't be able to edit a running instance.
* If you don't want the kubelet to automatically mount a ServiceAccount's API credentials, you can opt out of the default behavior. You can opt out of automounting API credentials on /var/run/secrets/kubernetes.io/serviceaccount/token for a service account by setting automountServiceAccountToken: false on the ServiceAccount:

```
spec:
  serviceAccountName: custom
  automountServiceAccountToken: false
  containers:
  - name: webserver
```

## System Hardening

### [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

[Decode Secrets](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/#decoding-secret)
You will no doubt create secrets in the exam, but you should verify if they actually took.
Ex. 
Applying the below command will show data, but it is already encrypted:

```
$ kubectl get secret db-user-pass -o jsonpath='{.data}'
```

output:
{ "password": "UyFCXCpkJHpEc2I9", "username": "YWRtaW4=" }

In order to decode, you'll need to run this command with Base64
$ kubectl get secret db-user-pass -o jsonpath='{.data.password}' | base64 --decode


### Kubesec
$ kubesec scan pod.yaml

Using online kubesec API
$ curl -sSX POST --data-binary @pod.yaml https://v2.kubesec.io/scan

Running the API locally
$ kubesec http 8080 &

$ kubesec scan pod.yaml -o pod_report.json -o json

### [_AppArmor_](https://kubernetes.io/docs/tutorials/security/apparmor/)
Default directory for profiles:
/etc/apparmor.d/
* AppArmor profiles will need to be applied to each node in the cluster. SCP the profile itself to the node of your choice then run the following:

Commands to know: [Also helpful here](https://ubuntu.com/server/docs/security-apparmor)
$ aa-status
  -Will show all the profiles loaded into the application

$ apparmor_parsers <flag> <path to profile>
  -will apply the profile to apparmor

Ensure you apply the profile to your specified pod in the yaml file:
```
_example_
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-apparmor
    annotations:                                                      <-- THIS IS WHERE IT GOES
      # Tell Kubernetes to apply the AppArmor profile "k8s-apparmor-example-deny-write".
      # Note that this is ignored if the Kubernetes node is not running version 1.4 or greater.
      container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write
```

### [Seccomp](https://kubernetes.io/docs/tutorials/security/seccomp/)
Used to white list or block all syscalls
Default profile location: /var/lib/kubelet/seccomp/profiles/


## Minimize Microservice Vulnerabilities
### [gvisor runtime](https://kubernetes.io/docs/concepts/containers/runtime-class/)
* the runtime engine will be already installed on the machines for this exam. You will need to create a yaml and apply to the cluster. Important for you to understand, for handler, you have to specify runsc for gvisor to run on the cluster. 

```
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor 
handler: runsc 
```
* When this is applied, you will need to enable pods/deployments this particular way:
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: gvisor
```
Note: When applying gvisor runtime to a deployment and the edit is not taking, either gvisor is not configured correctly or you are applying in the wrong spec. Look again.


## Supply Chain Security
### Trivy Scanning Engine
* Aqua (boooooo), developed a free image scanner for anyone to use. Best to understand the syntax of container scanning with this bad boy:

To begin, make your life easier with the command below to pull the image faster than kubectl edit.

```
$ k get pod -oyaml | grep image:
```

Ensure the tool is installed on your machine with:

```
$ trivy --v
```

As always you can see what your options are when scanning container images:

```
$ trivy image -h
```

THIS might be an example you'd need to know. The below command is 

```
trivy image --severity HIGH python:3.6.12-alpine3.11 --output /root/python.txt
```

### Minimize Base Image Footprint
* You will be presented with a dockerfile and YAML file with requirements to change two parts of each file so that it meets the minimum standard for security. As a refresher, here is a link to Sysdig's [Top 20 Dockerfile Best Practices](https://sysdig.com/blog/dockerfile-best-practices/)

```
FROM alpine:latest
# Create user and set ownership and permissions as required
RUN adduser -D myuser && chown -R myuser /myapp-data
# ... copy application files
USER root
ENTRYPOINT ["/myapp"]
```

Take a look at the above example. If you are being asked to change a few commands from the above, what stands out? You will need to specify a specific version of the base OS, and NEVER RUN AS ROOT!

```
FROM alpine:3.12
# Create user and set ownership and permissions as required
RUN adduser -D myuser && chown -R myuser /myapp-data
# ... copy application files
USER myuser
ENTRYPOINT ["/myapp"]
```

This is now the correct change to make a more secure footprint.

### ImagePolicyWebhook
The [ImagePolicyWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook) admission controller allows a backend webhook to make admission decisions. This admission controller is disabled by default.

This part is tricky due to modifying the kube-api manifest file. As being said before, making any edits to the manifest file will require a restart which is done automatically. What you'll need to do:
1. Define a admission_config file (yaml or json) 
1. kubeconf file that points to the correct server
1. ImagePolicyWebhook admission plugin that's enabled in the kube-api manifest file

Exmaple of /etc/kubernetes/policywebhook/admission_config.json
```
{
   "apiVersion": "apiserver.config.k8s.io/v1",
   "kind": "AdmissionConfiguration",
   "plugins": [
      {
         "name": "ImagePolicyWebhook",
         "configuration": {
            "imagePolicy": {
               "kubeConfigFile": "/etc/kubernetes/policywebhook/kubeconf",
               "allowTTL": 100,
               "denyTTL": 50,
               "retryBackoff": 500,
               "defaultAllow": false
            }
         }
      }
   ]
}
```

The /etc/kubernetes/policywebhook/kubeconf should contain the correct server:
```
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/policywebhook/external-cert.pem
    server: https://localhost:1234
  name: image-checker
...
```

Kubeapi config needs to have the ImagePolicyWebhook admission plugin:
```
spec:
  containers:
  - command:
    - kube-apiserver
    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
    - --admission-control-config-file=/etc/kubernetes/policywebhook/admission_config.json
```

## Monitoring, Logging, and Runtime Securtiy
Three things will be on your exam. Don't skirt learning this as you will need to activate these on three different questions, Falco, AppArmor, Kubernetes Auditing.

### [_Falco_](https://falco.org/docs/getting-started/installation/)
* Core config yaml file is located in /etc/falco/falco.yaml
You will use the falco.yaml file to list the different rules files you need loaded and enforced with Falco.
Output can be saved in JSON or text, if the _json_output_ is set to false, then output is saved as text.

* /etc/falco/falco_rules.yaml is the default file to list all of the rules you want to be used in Falco.

* In order to see events generated by Falco is to run:
```
$ journalctl -fu falco
```
note: the -f flag keeps the logs open and continuous


* Eventually you will have to locate a rule in the rules directory but you won't have the time to look at each file. So instead, use the grep command to search all files:
$ grep -ir 'Package management process launched in container' /etc/falco/ 



### Ensure Immutability of Containers at Runtime
* [Read Only Root Filesystem](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)


### [Enable Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
* Make sure to make a copy of the Kube API yaml file (ex. /etc/kubernetes/manifests/kube-api.yaml) as you messing with this will likely result in kubectl not working anymore. Save your sanity by making a copy, and re-applying when you get too frustrated. 




Written by Mathurin. 
If you're new to writing a simple README.md, venture [here](https://medium.com/@saumya.ranjan/how-to-write-a-readme-md-file-markdown-file-20cb7cbcd6f) for some good instructions. 
