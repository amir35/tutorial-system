**Tutorial Service**

This is tutorial backend service written in Spring Boot Java.

**Tutorial UI**

This is tutorial frontend ui written in Angular.

**Branch - feature/openshift**
oc apply -f .\01_pvc.yaml
oc get pvc
tutorial-system-pvc created with STATUS as Pending

oc apply -f .\01_imagestream.yaml


oc apply -f .\02_buildconfig.yaml
oc get buildconfig

oc apply -f .\03_configmap.yaml
oc get configmap

oc apply -f .\04_secret.yaml
oc get secret
oc describe secret tutorial-secret

oc apply -f .\05_deployment.yaml
oc get deployment
oc get pods
oc get pvc
Here, PVC tutorial-system-pvc STATUS changed to BOUND.