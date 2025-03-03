# Kong Gateway Configuration Documentation

## Overview
This documentation provides a comprehensive guide to setting up and configuring Kong Gateway using Docker Compose with various services, plugins, and routing rules. The configuration uses **Kong Gateway 3.9.0.1** alongside **PostgreSQL 13** as the database.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Folder Structure](#folder-structure)
3. [Environment Variables](#environment-variables)
4. [Services Description](#services-description)
5. [Plugins Overview](#plugins-overview)
6. [Docker Compose Configuration](#docker-compose-configuration)
7. [Health Checks](#health-checks)
8. [Kong Routes and Services](#kong-routes-and-services)
9. [Logging](#logging)
10. [Network Configuration](#network-configuration)

## Prerequisites
- Docker
- Docker Compose
- PostgreSQL client (optional, for database management)
- Basic knowledge of Kong API Gateway

## Folder Structure
```
├── .env                 # Environment variables
├── docker-compose.yml    # Docker Compose configuration
├── declarative           # Kong Declarative Configuration
│   └── kong.yml         # Kong service and routes definitions
└── nginx                # Nginx services
    ├── nginx1           # Static files for nginx1
    └── nginx2           # Static files for nginx2
```

## Environment Variables
The `.env` file is used to manage environment variables.

Example:
```
POSTGRES_USER=kong
POSTGRES_DB=kong
POSTGRES_PASSWORD=kong
KONG_DATABASE=postgres
KONG_PG_HOST=kong-database
KONG_PG_USER=kong
KONG_PG_PASSWORD=kong
KONG_DECLARATIVE_CONFIG=/kong/declarative/kong.yml
```

## Services Description
### 1. **kong-database**
PostgreSQL database for Kong Gateway.

### 2. **kong-setup**
Service to bootstrap Kong's database and import declarative configuration.

### 3. **kong-gateway**
Main Kong Gateway service for API proxying and plugin execution.

### 4. **nginx1**
Nginx service serving static content on port `8081`.

### 5. **nginx2**
Nginx service serving static content on port `8082`.

## Plugins Overview
### Key Plugins
- **key-auth**: API key authentication.
- **rate-limiting**: Limit the number of requests in a given timeframe.
- **request-transformer**: Modify request headers, query strings, or body.
- **response-transformer**: Modify response headers or body.
- **file-log**: Log requests and responses to a file.
- **prometheus**: Expose Prometheus metrics for monitoring.

## Docker Compose Configuration
The `docker-compose.yml` file orchestrates all services.

### Service Dependencies
- `kong-database` is the main database service.
- `kong-setup` initializes the database and imports configuration.
- `kong-gateway` depends on both `kong-database` and `kong-setup`.

### Ports Mapping
| Service       | Port   | Description         |
|--------------|-------|------------------|
| Kong Proxy   | 8000  | HTTP Proxy       |
| Kong Proxy   | 8443  | HTTPS Proxy      |
| Kong Admin   | 8001  | HTTP Admin API   |
| Kong Admin   | 8444  | HTTPS Admin API  |
| Nginx1       | 8081  | Static Content   |
| Nginx2       | 8082  | Static Content   |

## Health Checks
PostgreSQL health checks:
```yaml
test: ["CMD", "pg_isready", "-U", "kong"]
interval: 5s
retries: 5
timeout: 3s
```

## Kong Routes and Services
The `kong.yml` file defines all services and routes.

Example Route:
```yaml
services:
  - name: admin-api
    url: http://localhost:8002
    routes:
      - name: admin-api
        paths:
          - /admin-api
    plugins:
      - name: key-auth
```

## Logging
The `file-log` plugin logs requests to `/logs/kong.log`.

Example Log Configuration:
```yaml
plugins:
  - name: file-log
    config:
      path: /logs/kong.log
```

## Network Configuration
All services are connected to the `kong-net` network.

Docker network definition:
```yaml
networks:
  kong-net:
    driver: bridge
```

## Conclusion
This configuration sets up Kong Gateway with declarative configuration, API routing, rate limiting, logging, and authentication. Customize the `kong.yml` and `.env` files to suit your API gateway requirements.

For more information, visit the [Kong Gateway Documentation](https://docs.konghq.com/).

