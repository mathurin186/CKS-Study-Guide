# CKS-Study-Guide
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

--------------------------------------------------------

Things to understand before going INTO your exam

-Ensure that you have a mouse with you for faster COPY/PASTE actions during the exam. Don't count on the key commands as they are different with the Virtual Machine given to you. 
-You will NOT be able to have a browser with preorganized bookmarks. There will be a virtual desktop that you can access to dive into the kubernetes documentation page.
-Speed is the key, there is no time to get lost in a question. Look at each question and make the ones you think will take longer than two min to fix.
-When you move to a new question, ensure that you IMMEDIATELY switch contexts. From there, read the entire question as sometimes they will provide a pre-filled out template at the very bottom of the page. 
-To edit any pod that's running, copy the contents of the active pod, then paste to a new Yaml file and create. 
-Understand that because time is NOT on your side, you will need to cut corners for deploying/editing cluster info. Because of this, I encourage you to learn more Imperial Commands which you can reference the official kubernetes guide [here](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run). 



Common Commands to Know going in to any CK[A/S/D]


##Cluster Setup



##Cluster Hardening
Default location for the Kubelet file:
/var/lib/kubelet/config.yaml


##System Hardening
[_AppArmor_](https://kubernetes.io/docs/tutorials/security/apparmor/)
Default directory for profiles:
/etc/apparmor.d/

Commands to know:
$ aa-status
  -Will show all the profiles loaded into the application


[Seccomp](https://kubernetes.io/docs/tutorials/security/seccomp/)
Used to white list or block all syscalls
Default profile location: /var/lib/kubelet/seccomp/profiles/


##Minimize Microservice Vulnerabilities



##Supply Chain Security




##Monitoring, Logging, and Runtime Securtiy
Three things will be on your exam. Don't skirt learning this as you will need to activate these on three different questions, Falco, AppArmor, Kubernetes Auditing.

[_Falco_](https://falco.org/docs/getting-started/installation/)

[_AppArmor_](https://kubernetes.io/docs/tutorials/security/apparmor/)

[Enable Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
