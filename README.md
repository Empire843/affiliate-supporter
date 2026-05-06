# Affiliate Supporter

Nền tảng hỗ trợ người làm Affiliate Shopee — theo dõi giá tự động, nhận alert flash sale, quản lý link affiliate.

## Kiến trúc

```
api-gateway        :8080
link-service       :8081   — Quản lý link & lịch sử giá
crawler-service    :8082   — Crawl giá Shopee định kỳ
alert-service      :8083   — Rule engine & gửi Telegram/Email
```

## Chạy Local

```bash
cp .env.example .env
# Điền các giá trị vào .env

docker compose up -d
```

## Tech Stack

- Java 17 + Spring Boot 3.x
- Apache Kafka
- PostgreSQL 15
- Redis
- Docker + Docker Compose
