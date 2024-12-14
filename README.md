# Jenkins-Lab1
Jenkins-Lab
- insatll kuburnetes and jenkins plugin
on ur machine create a service account for jenkins with admin role
```
oc create rolebinding jenkins-admin-binding --clusterrole=admin --serviceaccount=jenkins:jenkins --namespace=jenkins
```
then generate the token and create a new credentials kind:secert text give it ID so u can use it further more 
