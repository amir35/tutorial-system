**Tutorial Service**

This is tutorial backend service written in Spring Boot Java.

**Tutorial UI**

This is tutorial frontend ui written in Angular.

**Branch : feature/docker**
**application.properties**
spring.datasource.url=jdbc:oracle:thin:@host.docker.internal:1521/orcl

**angular.json**
Check "outputPath": "dist/tutorial-front-end" for DockerImage build copy

**Tutorial Service - Build the image**
docker build -t amirdocker2204/tutorial-service:v1 .

**Tutorial Service - Run the container**
docker run -d -p 8080:8080 --name tutorial-service amirdocker2204/tutorial-service:v1

**Tutorial UI - Build the image**
docker build -t amirdocker2204/tutorial-ui:v1 .

**Tutorial UI - Run the container**
docker run -d -p 4200:80 --name tutorial-ui amirdocker2204/tutorial-ui:v1

**Image created**
amirdocker2204/tutorial-service:v1
amirdocker2204/tutorial-ui:v1

**Push image to Docker Hub**
docker push amirdocker2204/tutorial-service:v1
docker push amirdocker2204/tutorial-ui:v1

**Branch : feature/kubernetes**
**Tutorial Service - Create kubernetes folder with below yaml files:**
├──01-configmap.yaml
├──02-secret.yaml
├──03-deployment.yaml
├──04-service.yaml
├──05-ingress.yaml

**configmap.yaml**
kubectl apply -f 01_configmap.yaml
kubectl get configmap
kubectl describe configmap tutorial-config

**secret.yaml**
kubectl apply -f 02_secret.yaml
kubectl get secrets
kubectl describe secret tutorial-service-secret

**deployment.yaml**
kubectl apply -f 03_deployment.yaml
Deployment -> ReplicaSet -> 3 Pods -> 3 Containers
kubectl get deployment
kubectl get replicaset
kubectl get pods
kubectl logs <pod-name>

**service.yaml**
kubectl apply -f 04_service.yaml
kubectl get service

**Access backend**
minikube service tutorial-service --url

**Tutorial UI - Create kubernetes folder with below yaml files:**
├──01-deployment.yaml
├──02-service.yaml
└──03-ingress.yaml

**deployment.yaml**
kubectl apply -f 01_deployment.yaml

**service.yaml**
kubectl apply -f 02_service.yaml

**Enable ingress**
minikube addons enable ingress
kubectl get pods -n ingress-nginx

**ingress.yaml**
We do not need to have 2 different ingress.yaml in both backend and frontend. We can have one ingress.yaml in the root folder of the project or in frontend folder. But for the sake of this tutorial, we will keep it in both backend and frontend folders.
kubectl apply -f 03_ingress.yaml
kubectl get ingress

minikube tunnel

kubectl get ingress

**Update Host File**
C:\Windows\System32\drivers\etc\hosts
Add '127.0.0.1 tutorial.local'
