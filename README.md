# 🍕 Pizza-Net — Platforma do zarządzania siecią pizzerii

**Pizza-Net** to pełnoprawna platforma backendowa i frontendowa do zarządzania siecią pizzerii, zbudowana w architekturze mikrousług. System obsługuje zamawianie pizzy, zarządzanie menu, płatności, dostawy oraz powiadomienia e-mail — wszystko przez jeden spójny API Gateway.

---

## 📁 Repozytoria organizacji

| Repozytorium | Opis | Technologia |
|---|---|---|
| ⭐ **[pizza-net_architecture](https://github.com/pizza-net/pizza-net_architecture)** | Dokumentacja architektoniczna całego projektu (to repo) | — |
| 🌐 **[pizza-frontend](https://github.com/pizza-net/pizza-frontend)** | Aplikacja frontendowa dla klientów i administratorów | React 19 + Vite |
| 🔀 **[gateway-service](https://github.com/pizza-net/gateway-service)** | Brama API — routing, JWT validation, CORS | Spring Cloud Gateway |
| 🔐 **[auth-service](https://github.com/pizza-net/auth-service)** | Autentykacja i autoryzacja, rejestracja, JWT, RBAC | Spring Boot + Security |
| 🍕 **[menu-service](https://github.com/pizza-net/menu-service)** | Zarządzanie menu — pizze, składniki, ceny, kategorie | Spring Boot + JPA |
| 📦 **[order-service](https://github.com/pizza-net/order-service)** | Tworzenie i śledzenie zamówień, integracja z RabbitMQ | Spring Boot + AMQP |
| 💳 **[payment-service](https://github.com/pizza-net/payment-service)** | Obsługa płatności online (Stripe), statusy transakcji | Spring Boot |
| 🚴 **[delivery-service](https://github.com/pizza-net/delivery-service)** | Zarządzanie kurierami i śledzenie dostaw | Spring Boot |
| 📧 **[notification-service](https://github.com/pizza-net/notification-service)** | Wysyłanie powiadomień e-mail przez RabbitMQ | Spring Boot + Mail |
| 🔍 **[discovery-server](https://github.com/pizza-net/discovery-server)** | Rejestr mikrousług (Eureka Server) | Spring Cloud Netflix |
| ⚙️ **[config-server](https://github.com/pizza-net/config-server)** | Centralny serwer konfiguracji dla wszystkich usług | Spring Cloud Config |
| 📄 **[pizza-config-repo](https://github.com/pizza-net/pizza-config-repo)** | Repozytorium plików konfiguracyjnych (YAML/properties) | — |

---

## 🏛️ Diagram architektury

```
                         ┌──────────────────────────┐
                         │      PIZZA-FRONTEND       │
                         │  React 19 + Vite          │
                         │  Port: 5173 (dev) / 80    │
                         └────────────┬─────────────┘
                                      │ HTTP
                                      ▼
                         ┌──────────────────────────┐
                         │      GATEWAY-SERVICE      │
                         │  Spring Cloud Gateway     │
                         │  Port: 8080               │
                         │  JWT validation, CORS     │
                         └──┬──────┬──────┬──────┬──┘
                            │      │      │      │
              ┌─────────────┘      │      │      └──────────────┐
              ▼                    ▼      ▼                      ▼
  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
  │   AUTH-SERVICE   │  │   MENU-SERVICE   │  │    ORDER-SERVICE     │
  │  Port: 8081      │  │  Port: 8082*     │  │    Port: 8083*       │
  │  JWT, BCrypt     │  │  CRUD menu       │  │    RabbitMQ publish  │
  │  PostgreSQL      │  │  PostgreSQL      │  │    PostgreSQL        │
  └──────────────────┘  └──────────────────┘  └──────────┬───────────┘
                                                          │ AMQP
                              ┌───────────────────────────┼───────────────────┐
                              ▼                           ▼                   ▼
              ┌───────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐
              │    PAYMENT-SERVICE    │  │   DELIVERY-SERVICE   │  │  NOTIFICATION-SERVICE │
              │    Port: 8084*        │  │   Port: 8085*        │  │  Port: 8086*          │
              │    Stripe integration │  │   Courier tracking   │  │  Email via SMTP       │
              │    PostgreSQL         │  │   PostgreSQL         │  │  RabbitMQ consume     │
              └───────────────────────┘  └──────────────────────┘  └──────────────────────┘

  ┌──────────────────────────────────────────────────────────────────────────────────────┐
  │                            INFRASTRUKTURA                                            │
  │  ┌──────────────────────┐          ┌────────────────────────────────────────────┐   │
  │  │   DISCOVERY-SERVER   │          │              CONFIG-SERVER                 │   │
  │  │  Eureka Registry     │◄─────────│  Spring Cloud Config                       │   │
  │  │  Port: 8761          │ register │  Czyta z: pizza-config-repo (GitHub)       │   │
  │  └──────────────────────┘          └────────────────────────────────────────────┘   │
  │                                                                                      │
  │  ┌──────────────────────┐          ┌────────────────────────────────────────────┐   │
  │  │      RabbitMQ        │          │              PostgreSQL                     │   │
  │  │  Message Broker      │          │  Osobna instancja DB per mikrousługa        │   │
  │  │  Port: 5672 / 15672  │          │  Port: 5432                                │   │
  │  └──────────────────────┘          └────────────────────────────────────────────┘   │
  └──────────────────────────────────────────────────────────────────────────────────────┘

  * Porty orientacyjne — aktualne wartości znajdziesz w pizza-config-repo lub plikach application.yml
```

---

## 🔧 Stos technologiczny

### Backend
| Warstwa | Technologia |
|---|---|
| Framework | Spring Boot 3.x / 4.x |
| Język | Java 21 |
| Bezpieczeństwo | Spring Security + JWT (jjwt 0.12.x) |
| Baza danych | PostgreSQL (osobna instancja per usługa) |
| ORM | Spring Data JPA + Hibernate |
| Messaging | RabbitMQ (AMQP) |
| Service Discovery | Netflix Eureka |
| API Gateway | Spring Cloud Gateway |
| Konfiguracja | Spring Cloud Config |
| Budowanie | Maven (Maven Wrapper) |
| Konteneryzacja | Docker (Dockerfile per usługa) |

### Frontend
| Warstwa | Technologia |
|---|---|
| Framework | React 19 |
| Bundler | Vite 7 |
| Routing | React Router DOM 6 |
| HTTP Client | Axios |
| Płatności | Stripe.js |
| Styl kodu | ESLint |

---

## 🚀 Uruchomienie projektu

### Wymagania
- **Java 21** (JDK)
- **Node.js 18+** i npm
- **Docker** i Docker Compose
- **Git**

### Kolejność uruchamiania usług

Usługi należy uruchamiać **w podanej kolejności**, ponieważ część z nich zależy od innych:

```
1. config-server      (musi być dostępny jako pierwszy)
2. discovery-server   (Eureka — rejestr usług)
3. gateway-service    (po rejestracji w Eureka)
4. auth-service
5. menu-service
6. order-service
7. payment-service
8. delivery-service
9. notification-service
10. pizza-frontend    (po uruchomieniu backendu)
```

### Uruchomienie pojedynczej usługi Spring Boot

```bash
# Klonowanie wybranej usługi
git clone https://github.com/pizza-net/<nazwa-usługi>.git
cd <nazwa-usługi>

# Uruchomienie przez Maven Wrapper
./mvnw spring-boot:run
```

### Uruchomienie frontendu

```bash
git clone https://github.com/pizza-net/pizza-frontend.git
cd pizza-frontend
npm install
npm run dev
# Aplikacja dostępna pod: http://localhost:5173
```

---

## 🔐 Przepływ autentykacji

```
Użytkownik → Login Form
    ↓
POST /api/auth/login → gateway-service (port 8080)
    ↓ lb://auth-service
auth-service weryfikuje dane (BCrypt)
    ↓
Generuje JWT Token (HMAC SHA-256, 24h ważności)
    ↓
Frontend zapisuje token w localStorage
    ↓
Kolejne requesty: Authorization: Bearer <token>
    ↓
gateway-service waliduje JWT przed przekazaniem do usług
```

**Role użytkowników:** `CUSTOMER`, `COURIER`, `ADMIN`

---

## 📨 Przepływ zamówienia

```
Klient → Złożenie zamówienia (pizza-frontend)
    ↓
POST /api/orders → order-service
    ↓
order-service → RabbitMQ (zdarzenie: ORDER_CREATED)
    ↓
    ├─→ payment-service  (inicjalizacja płatności)
    ├─→ delivery-service (przydzielenie kuriera)
    └─→ notification-service (e-mail do klienta)
```

---

## ⚙️ Konfiguracja centralna

Konfiguracja wszystkich usług zarządzana jest przez **config-server**, który czyta pliki z repozytorium **[pizza-config-repo](https://github.com/pizza-net/pizza-config-repo)**.

Przykładowe pliki konfiguracyjne:
- `application.yml` — wspólna konfiguracja (Eureka URL)
- `auth-service.yml` — port 8081, baza danych, JWT secret
- `gateway-service.yml` — port 8080, routing do usług
- `discovery-server.yml` — konfiguracja Eureka Server

---

## 🐳 Docker

Każda usługa backendowa oraz frontend zawierają plik `Dockerfile`. Budowanie obrazu:

```bash
# Np. dla auth-service
cd auth-service
./mvnw clean package -DskipTests
docker build -t pizza-net/auth-service .
```

---

## 📌 Porty usług (domyślne)

| Usługa | Port |
|---|---|
| pizza-frontend | 5173 (dev) / 80 (Docker) |
| gateway-service | **8080** |
| auth-service | **8081** |
| discovery-server (Eureka UI) | **8761** |
| RabbitMQ Management UI | **15672** |
| PostgreSQL | 5432 |

---

## 📂 Struktura organizacji GitHub

```
pizza-net/
├── pizza-net_architecture/   ← dokumentacja projektu (to repo)
├── pizza-frontend/           ← aplikacja webowa (React)
├── gateway-service/          ← brama API
├── auth-service/             ← autentykacja & autoryzacja
├── menu-service/             ← zarządzanie menu
├── order-service/            ← zamówienia
├── payment-service/          ← płatności
├── delivery-service/         ← dostawy
├── notification-service/     ← powiadomienia e-mail
├── discovery-server/         ← rejestr usług (Eureka)
├── config-server/            ← serwer konfiguracji
└── pizza-config-repo/        ← pliki konfiguracyjne YAML
```

---

