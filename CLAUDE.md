# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Affiliate Supporter is a Shopee affiliate platform that automatically tracks prices, triggers flash-sale alerts, and manages affiliate links. It is built as a microservices system using Java 21 + Spring Boot 3.5.

## Services and Ports

| Service | Port | Description |
|---|---|---|
| `api-gateway` | 8080 | Spring Cloud Gateway (MVC) — routes all external traffic |
| `link-service` | 8081 | Manages affiliate links and price history (PostgreSQL `link_db`) |
| `crawler-service` | 8082 | Scheduled Shopee price crawling, publishes price events to Kafka (`crawler_db`) |
| `alert-service` | 8083 | Rule engine, consumes Kafka events, sends Telegram/Email notifications (`alert_db`) |

Each service has its own dedicated PostgreSQL instance (ports 5432–5434). Inter-service communication uses Kafka (`kafka:29092` inside Docker, `localhost:9092` externally). Redis (6379) is available for caching.

## Build & Test Commands

Each service is an independent Maven project. Run commands from inside the service directory:

```bash
# Build and run tests for a single service
cd link-service && mvn clean test

# Package (skip tests, used by Docker build)
cd link-service && mvn clean package -DskipTests

# Run locally (requires infrastructure running)
cd link-service && mvn spring-boot:run
```

The `api-gateway` has no Kafka/DB dependencies and can be tested in isolation. The other three services require PostgreSQL and Kafka.

## Running the Full Stack

```bash
cp .env.example .env
# Fill in TELEGRAM_BOT_TOKEN, TELEGRAM_CHAT_ID, MAIL_USERNAME, MAIL_PASSWORD
docker compose up -d
```

Infrastructure services (postgres-link, postgres-crawler, postgres-alert, kafka, zookeeper, redis) have healthchecks; application services wait on them before starting.

## Architecture Notes

- **No shared parent POM** — each service is fully self-contained with its own `pom.xml` and `mvnw` wrapper.
- **api-gateway** uses `spring-cloud-starter-gateway-server-webmvc` (MVC-based, not reactive). Route config goes in `application.properties` or a `@Bean RouteLocator`.
- **Kafka topics** are auto-created (`KAFKA_AUTO_CREATE_TOPICS_ENABLE: true`). The Confluent Platform image (`confluentinc/cp-kafka:7.5.0`) is used with a single broker.
- **Dockerfiles** use a two-stage build: Maven builder (`maven:3.9-eclipse-temurin-21`) → runtime (`eclipse-temurin:21-jre-alpine`) running as a non-root `spring` user.
- **alert-service** alone requires `spring-boot-starter-mail` — it is the only service that sends external notifications.
- Configuration beyond `spring.application.name` is injected at runtime via environment variables (see `docker-compose.yml` for the full variable list).

## CI

Jenkins pipeline defined in `Jenkinsfile`:
1. Checkout `develop` branch
2. `mvn clean test` for link-service, crawler-service, alert-service (sequentially)
3. `docker compose build`

api-gateway is not tested in CI (no domain logic to test yet).
