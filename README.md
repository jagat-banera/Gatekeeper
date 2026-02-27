# Gatekeeper

Gatekeeper is a custom-built API Gateway system implemented using **Java 17** and **Spring Boot**.

It consists of two independent microservices:

- **Gatekeeper Admin (Control Plane)**
- **Gatekeeper Gateway (Data Plane)**

The system enables dynamic route management with runtime hot reloading, custom Netty-based traffic routing, and containerized deployment using Docker and Docker Compose.

---

# Architecture Overview

Gatekeeper follows a two-service architecture.

---

## 1. Gatekeeper Admin

The Admin service acts as the configuration and control layer.

### Responsibilities

- Admin signup and login
- Route creation, update, deletion
- Persistent route storage oon SQLite
- UI for managing gateway configuration
- Health endpoint exposure for orchestration

### Technologies Used

- Spring Boot
- Sqlite
- Spring Security
- Spring Data JPA
- Thymeleaf (Admin UI)
- Spring Actuator

The Admin service maintains the source of truth for all route configurations.

---

## 2. Gatekeeper Gateway

The Gateway service is the traffic routing engine.

### Responsibilities

- Fetch routes from Admin service at startup
- Build in-memory routing table using `ConcurrentHashMap`
- Handle runtime hot reloading of routes
- Perform request routing via embedded Netty server

### Ports

| Port | Purpose |
|------|---------|
| 8082 | Spring Boot application port (internal logic & route sync) |
| 8080 | Embedded Netty gateway port (actual traffic routing) |

Unlike Spring Cloud Gateway, this project embeds Netty directly to implement a custom routing layer.

---

# Communication Flow

1. Admin service starts and becomes healthy.
2. Gateway service starts after Admin health check passes.
3. On startup, Gateway fetches all routes from Admin.
4. Gateway builds an in-memory `ConcurrentHashMap` for fast lookups.
5. Netty server routes requests based on in-memory configuration.
6. Routes can be added, removed, or updated dynamically (hot reloading supported).

---

# Key Design Decisions

## Separation of Control and Data Plane

- Admin handles configuration and persistence.
- Gateway handles runtime traffic routing.

This separation improves modularity and mirrors real-world API gateway designs.

---

## In-Memory Routing Table

Using `ConcurrentHashMap` ensures:

- Thread-safe access
- High-performance route lookup
- Minimal request latency

---

## Custom Netty Integration

Instead of using Spring Cloud Gateway, this project embeds Netty directly to:

- Gain deeper control over the request lifecycle
- Improve performance
- Understand low-level HTTP routing mechanics

---

## Startup Health Dependency

Docker Compose ensures:

- Admin service starts first
- Gateway waits until Admin health endpoint reports `UP`
- Gateway fetches routes only when Admin is ready

This prevents partial initialization failures.

---

# Technology Stack

- Java 17
- Spring Boot
- Spring Security
- Spring Data JPA
- SQLite
- Thymeleaf
- Embedded Netty
- Maven
- Docker
- Docker Compose
- Eclipse Temurin 17 (Alpine base image)


---
# How to Download and Run

Gatekeeper can be started using Docker Compose without manually building individual services.



## Option 1: Run Using Docker Compose (Recommended)

### Prerequisites

- Docker installed
- Docker Compose installed

Verify installation:

```bash
docker --version
docker compose version 
```

Clone the Repository:

```bash
git clone https://github.com/jagat-banera/Gatekeeper.git
cd gatekeeper
```

Start the Containers:

```bash
docker compose up --build
```

---

## Access the Services

Once the application is running, you can access the following services:

| Service               | URL                                 | Description                                                      | 
|-----------------------|-------------------------------------|------------------------------------------------------------------|
| Admin UI Signup       | http://localhost:8081/admin-signup  | For Admin Credentials Setup                                      |
| Admin UI Login        | http://localhost:8081/login         | For Login Into UI Panel                                          |
| Admin UI List API     | http://localhost:8081/ui/list-api   | For Listing all APis ( Active and Inactive )                     |
| Admin UI Manage API   | http://localhost:8081/ui/manage-api | For Activating , Deactivating and Deleting Endpoints             |
| Admin UI Register APi | http://localhost:8081/ui/list-api | For Regstering new Routes ( By Default , New Routes are Inactive |


---

## Accessing a Created Route

Once a route is created and activated successfully in the Admin service, the configured target URL becomes accessible through the Gateway endpoint.

The Gateway listens on:

```bash
http://localhost:8080
```

All incoming requests to this port are evaluated against the in-memory routing table and forwarded to the configured target service.



### Example

Assume the following route configuration is created in the Admin UI:

- Route Path: `/users`
- Target URL: `http://example-service:9000`
- Status: Active

After saving and activating the route, you can access the target service through the Gateway as follows:

```bash
http://localhost:8080/users
```

---

# Future Scope

The architecture is intentionally designed to be extensible. Potential enhancements include:

- Observability integration (Prometheus, Grafana)
- Circuit breaker and resilience patterns
- Rate limiting and traffic shaping

Feedback, discussions, and contributions are welcome.



