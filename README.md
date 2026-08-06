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
docker build -t tutorial-service:v1 .

**Tutorial Service - Run the container**
docker run -d -p 8080:8080 --name tutorial-service tutorial-service:v1

**Tutorial UI - Build the image**
docker build -t tutorial-ui:v1 .

**Tutorial UI - Run the container**
docker run -d -p 4200:80 --name tutorial-ui tutorial-ui:v1

**Image created**
tutorial-service:v1
tutorial-ui:v1

**Push image to Docker Hub**
docker push amirdocker2204/tutorial-service:v1
docker push amirdocker2204/tutorial-ui:v1
