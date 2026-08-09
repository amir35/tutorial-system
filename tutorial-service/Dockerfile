# =========================
# Stage 1 - Build
# =========================
FROM maven:3.9.9-eclipse-temurin-17 AS builder

WORKDIR /app

# Copy project
COPY . .

# Build Spring Boot jar
RUN mvn clean package -DskipTests

# =========================
# Stage 2 - Runtime
# =========================
FROM eclipse-temurin:17-jre

#Create a folder inside a container and further operations will be done inside this folder
WORKDIR /app

#Copy to /app/app.jar
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","app.jar"]