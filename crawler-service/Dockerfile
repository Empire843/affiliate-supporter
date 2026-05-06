# STAGE 1: BUILD
FROM maven:3.9-eclipse-temurin-21 as builder
WORKDIR /app
COPY pom.xml .

RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

RUN mvn clean package -DskipTests

# STAGE 2: RUN
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

RUN addgroup -S spring && adduser -S spring -G spring

USER spring:spring

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8082
ENTRYPOINT ["java", "-jar", "app.jar"]