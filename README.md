# PizzaNet — Microservices Backend Architecture

A production-ready, microservices-based backend system for managing a pizzeria network, built with Java 21 and Spring Boot.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Microservices](#microservices)
- [My Contribution](#my-contribution)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Running Locally with Docker Compose](#running-locally-with-docker-compose)
- [API Overview](#api-overview)

---

## Project Overview

**PizzaNet** is a scalable backend platform designed to support the operations of a multi-location pizzeria network. The system handles user authentication, menu management, and order processing through a set of loosely coupled, independently deployable microservices.

Each service owns its domain logic and persists data in a dedicated PostgreSQL database, following the *database-per-service* pattern. Inter-service communication is handled via synchronous REST APIs, with traffic routed through a central API Gateway.

---

## Architecture

```
                        ┌─────────────────────────┐
                        │      API Gateway         │
                        │  (Spring Cloud Gateway)  │
                        └──────────┬──────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
     ┌────────────────┐  ┌─────────────────┐  ┌────────────────┐
     │  auth-service  │  │  menu-service   │  │ order-service  │
     │  (Port 8081)   │  │  (Port 8082)    │  │  (Port 8083)   │
     └───────┬────────┘  └────────┬────────┘  └───────┬────────┘
             │                    │                    │
     ┌───────▼──────┐    ┌────────▼───────┐   ┌───────▼───────┐
     │  PostgreSQL  │    │  PostgreSQL    │   │  PostgreSQL   │
     │  (auth-db)   │    │  (menu-db)     │   │  (order-db)   │
     └──────────────┘    └────────────────┘   └───────────────┘
```

All services register themselves with a **Eureka Service Registry**, enabling dynamic service discovery and load balancing.

---

## Technologies Used

| Category              | Technology                        |
|-----------------------|-----------------------------------|
| Language              | Java 21                           |
| Framework             | Spring Boot                       |
| Service Discovery     | Spring Cloud Netflix Eureka       |
| API Routing           | Spring Cloud Gateway              |
| Persistence           | Spring Data JPA / Hibernate       |
| Database              | PostgreSQL                        |
| Containerization      | Docker, Docker Compose            |
| Build Tool            | Maven                             |
| API Style             | RESTful JSON APIs                 |

---

## Microservices

### `auth-service`
Handles user registration, login, and JWT-based authentication. Issues and validates tokens consumed by other services.

### `menu-service`
Manages the pizzeria menu: pizza categories, individual items, ingredients, and pricing. Exposes read-only endpoints for customers and write endpoints for administrators.

### `order-service`
Processes customer orders end-to-end — from cart creation through payment confirmation to order status tracking. Communicates with `menu-service` to validate items and pricing at the time of order.

---

## My Contribution

- Designed and implemented the overall microservices architecture, including service boundaries and inter-service communication contracts.
- Built the `auth-service` with JWT authentication and Spring Security integration.
- Developed the `menu-service` REST API with full CRUD operations backed by PostgreSQL.
- Implemented the `order-service` with order lifecycle management and cross-service validation.
- Configured **Spring Cloud Gateway** as a unified entry point with route definitions for all downstream services.
- Set up **Eureka Service Registry** for dynamic service discovery.
- Authored `docker-compose.yml` to orchestrate all services and their databases in isolated containers for local development.

---

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) ≥ 24.x
- [Docker Compose](https://docs.docker.com/compose/) ≥ 2.x

> No local Java or Maven installation is required — all builds happen inside Docker containers.

### Running Locally with Docker Compose

1. **Clone the repository**

   ```bash
   git clone https://github.com/pizza-net/pizza-net_architecture.git
   cd pizza-net_architecture
   ```

2. **Start all services**

   ```bash
   docker compose up --build
   ```

   Docker Compose will:
   - Build each microservice image from its `Dockerfile`
   - Spin up dedicated PostgreSQL instances for each service
   - Start the Eureka registry, API Gateway, and all microservices

3. **Verify the services are running**

   | Service          | URL                              |
   |------------------|----------------------------------|
   | API Gateway      | http://localhost:8080            |
   | Eureka Dashboard | http://localhost:8761            |
   | auth-service     | http://localhost:8080/auth       |
   | menu-service     | http://localhost:8080/menu       |
   | order-service    | http://localhost:8080/orders     |

4. **Stop all services**

   ```bash
   docker compose down
   ```

   To also remove persistent database volumes:

   ```bash
   docker compose down -v
   ```

---

## API Overview

All requests are made through the **API Gateway** at `http://localhost:8080`.

| Method | Path                     | Service        | Description                    |
|--------|--------------------------|----------------|--------------------------------|
| POST   | `/auth/register`         | auth-service   | Register a new user            |
| POST   | `/auth/login`            | auth-service   | Authenticate and receive JWT   |
| GET    | `/menu/pizzas`           | menu-service   | List all available pizzas      |
| POST   | `/menu/pizzas`           | menu-service   | Add a new pizza (admin)        |
| GET    | `/orders`                | order-service  | List orders for current user   |
| POST   | `/orders`                | order-service  | Place a new order              |
| GET    | `/orders/{id}`           | order-service  | Get order details by ID        |
