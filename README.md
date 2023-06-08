
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



Things to understand before going INTO your exam

* Don't even think, immediately ensure that you have [Auto Complete and Alias's](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) set up for speedy typing of commands. kubectl create can be a bitch when you're on a time limit.
* Ensure that you have a mouse with you for faster COPY/PASTE actions during the exam. Don't count on the key commands as they are different with the Virtual Machine given to you. 
* You will NOT be able to have a browser with preorganized bookmarks. There will be a virtual desktop that you can access to dive into the kubernetes documentation page.
* Speed is the key, there is no time to get lost in a question. Look at each question and make the ones you think will take longer than two min to fix.
* When you move to a new question, ensure that you IMMEDIATELY switch contexts. From there, read the entire question as sometimes they will provide a pre-filled out template at the very bottom of the page. 
* To edit any pod that's running, copy the contents of the active pod, then paste to a new Yaml file and create. 
* Understand that because time is NOT on your side, you will need to cut corners for deploying/editing cluster info. Because of this, I encourage you to learn more Imperial Commands which you can reference the official kubernetes guide [here](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run). 


## Testing Environments 
You need to _PRACTICE!_ There is no way to pass this without getting your hands dirty. Because of that, here are a few:
1. [Killercoda](/killercoda.com)
2. [Killer.sh](killer.sh)
3. [KodeKloud](https://kodekloud.com/)



## Common Commands to Know going in to any CK[A/S/D]
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
* k exec sec -- dmesg | grep -i gvisor        Here we're able to inspect a pod to see if a runtiemclass has been applied>
* grep -ri "phrase you're looking for" /var/www/html      With anything Falco, you need to know how to search for specific rules in the multiple files. This will save you some time.
* kubectl get pods --as dev-user              Confirmation after you finish a section of RBAC is key to ensure you're completing everything. 


## Log Locations

/var/log/pods
/var/log/containers
crictl ps + crictl logs
docker ps + docker logs (in case when Docker is used)
kubelet logs: /var/log/syslog or journalctl


## Cluster Setup & Hardening

### RBAC
Assuming you aren't one to sit through documentation, I found this video from [School of DevOps](https://www.redfin.com/CA/Pacifica/547-Vista-Mar-Ave-94044/home/1950478) to give a better way to simplify when you'd use Roles vs ClusterRoles.

* check on users ability to perform a function in a namespaces or cluster:
$ kubectl auth can-i get pods

### Kubelet Security
Default location for the Kubelet file:
/var/lib/kubelet/config.yaml

In order to find where the Kubelet file is at, run the ps -aux command:
$ px -aux | grep kubelet

* When making any changes to the Kubelet service, ensure to RESTART. 
$ systemctl restart kubelet.service


### Service Accounts
* If you add a Service Account to a pod, you MUST re-create the pod as you won't be able to edit a running instance.

## System Hardening

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

[Decode Secrets](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/#decoding-secret)
You will no doubt create secrets in the exam, but you should verify if they actually took.
Ex. 
Applying the below command will show data, but it is already encrypted:
$ kubectl get secret db-user-pass -o jsonpath='{.data}'

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

Commands to know: [Also helpful here](https://ubuntu.com/server/docs/security-apparmor)
$ aa-status
  -Will show all the profiles loaded into the application

$ apparmor_parsers <flag> <path to profile>
  -will apply the profile to apparmor

Ensure you apply the profile to your specified pod in the yaml file:
_example_
  apiVersion: v1
  kind: Pod
  metadata:
    name: hello-apparmor
    annotations:
      # Tell Kubernetes to apply the AppArmor profile "k8s-apparmor-example-deny-write".
      # Note that this is ignored if the Kubernetes node is not running version 1.4 or greater.
      container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write


### [Seccomp](https://kubernetes.io/docs/tutorials/security/seccomp/)
Used to white list or block all syscalls
Default profile location: /var/lib/kubelet/seccomp/profiles/


## Minimize Microservice Vulnerabilities

### Trivy Scanning Engine
* Aqua (boooooo), developed a free image scanner for anyone to use. Best to understand the syntax of container scanning with this bad boy:

Ensure the tool is installed on your machine with:
$ trivy --v

As always you can see what your options are when scanning container images:
$ trivy image -h`

THIS might be an example you'd need to know. The below command is 
trivy image --severity HIGH python:3.6.12-alpine3.11 --output /root/python.txt


## Supply Chain Security




## Monitoring, Logging, and Runtime Securtiy
Three things will be on your exam. Don't skirt learning this as you will need to activate these on three different questions, Falco, AppArmor, Kubernetes Auditing.

### [_Falco_](https://falco.org/docs/getting-started/installation/)
* Core config yaml file is located in /etc/falco/falco.yaml
You will use the falco.yaml file to list the different rules files you need loaded and enforced with Falco.
Output can be saved in JSON or text, if the _json_output_ is set to false, then output is saved as text.

* /etc/falco/falco_rules.yaml is the default file to list all of the rules you want to be used in Falco.

* In order to see events generated by Falco is to run the _journalctl -u falco_ command.

* Eventually you will have to locate a rule in the rules directory but you won't have the time to look at each file. So instead, use the grep command to search all files:
$ grep -ir 'Package management process launched in container' /etc/falco/ 



### Ensure Immutability of Containers at Runtime
* [Read Only Root Filesystem](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)


### [Enable Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
* Make sure to make a copy of the Kube API yaml file (ex. /etc/kubernetes/manifests/kube-api.yaml) as you messing with this will likely result in kubectl not working anymore. Save your sanity by making a copy, and re-applying when you get too frustrated. 




Written by Mathurin. If you're new to writing a simple README.md, venture [here](https://medium.com/@saumya.ranjan/how-to-write-a-readme-md-file-markdown-file-20cb7cbcd6f) for some good instructions. 
