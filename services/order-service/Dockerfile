
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:21-jre

WORKDIR /app

# Create a non-root application user
RUN groupadd --system appgroup && \
    useradd --system --gid appgroup appuser

# Copy JAR with application user ownership
COPY --from=build --chown=appuser:appgroup /app/target/order-service-1.0.0.jar app.jar

# Switch to non-root user
USER appuser

EXPOSE 18084

ENTRYPOINT ["java", "-jar", "app.jar"]
