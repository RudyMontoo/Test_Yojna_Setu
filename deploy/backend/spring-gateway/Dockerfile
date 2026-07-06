# ── Stage 1: Build ───────────────────────────────────────────────────────────────
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app

# Copy Maven wrapper and pom first (layer cache for dependencies)
COPY pom.xml .
COPY src ./src

# Install Maven and build the JAR (skip tests for faster deploy)
RUN apk add --no-cache maven && \
    mvn package -DskipTests -q

# ── Stage 2: Run ─────────────────────────────────────────────────────────────────
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/gateway-2.0.0.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-Xmx300m", "-jar", "app.jar"]
