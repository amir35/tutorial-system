**Tutorial Service**

This is tutorial backend service written in Spring Boot Java.

**Tutorial UI**

This is tutorial frontend ui written in Angular.

**Branch - feature/openshift**
oc apply -f .\01_pvc.yaml
oc get pvc
tutorial-system-pvc created with STATUS as Pending

**ImageStream will manage/reference the image build from Openshift**
oc apply -f .\01_imagestream.yaml

**BuildConfig will build the image based on the ImageStream**
oc apply -f .\02_buildconfig.yaml
oc get buildconfig

**Now run the Build from the BuildConfig**
oc start-build tutorial-service --follow
oc get istag

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