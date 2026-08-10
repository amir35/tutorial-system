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

oc apply -f .\03_configmap.yaml
oc get configmap

oc apply -f .\04_secret.yaml
oc get secret
oc describe secret tutorial-secret

**Now run the Build from the BuildConfig**
oc start-build tutorial-service --follow
oc get istag

oc apply -f .\05_deployment.yaml
oc get deployment
oc get pods

**To check pod has started**
oc logs <pod-name>

oc get pvc
Here, PVC tutorial-system-pvc STATUS changed to BOUND.

oc apply -f 06_service.yaml
oc get service

**Get applciation endpoint**
oc get endpoints tutorial-service
NAME               ENDPOINTS           AGE
tutorial-service   10.129.5.220:8080   42s

**Expose the Service outside the OpenShift cluster**
oc apply -f 07_route.yaml

**tutorial-ui**
oc apply -f 01_imagestream.yaml
oc get imagestream

oc apply -f 02_buildconfig.yaml
oc get buildconfig

oc start-build tutorial-ui --follow

oc get imagestream

oc apply -f 03_deployment.yaml
oc get deployment
oc get pods