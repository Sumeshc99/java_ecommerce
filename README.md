# Product API — Java Spring Boot + PostgreSQL

A ready-to-run REST API with JWT-based login/password authentication and
Product CRUD endpoints, backed by PostgreSQL.

## Tech Stack

- Java 17
- Spring Boot 3.3.2 (Web, Security, Data JPA, Validation)
- PostgreSQL
- JWT (jjwt 0.12.5)
- Lombok
- Maven

## Project Structure

```
product-api/
├── pom.xml
├── setup.sql
└── src/main/
    ├── java/com/example/productapi/
    │   ├── ProductApiApplication.java
    │   ├── config/
    │   │   ├── SecurityConfig.java
    │   │   └── GlobalExceptionHandler.java
    │   ├── security/
    │   │   ├── JwtUtil.java
    │   │   ├── JwtAuthFilter.java
    │   │   └── UserDetailsServiceImpl.java
    │   ├── model/
    │   │   ├── User.java
    │   │   └── Product.java
    │   ├── repository/
    │   │   ├── UserRepository.java
    │   │   └── ProductRepository.java
    │   ├── dto/
    │   │   ├── RegisterRequest.java
    │   │   ├── LoginRequest.java
    │   │   ├── AuthResponse.java
    │   │   └── ProductRequest.java
    │   ├── service/
    │   │   ├── AuthService.java
    │   │   └── ProductService.java
    │   └── controller/
    │       ├── AuthController.java
    │       └── ProductController.java
    └── resources/
        └── application.properties
```

## 1. Prerequisites

- JDK 17+
- Maven 3.8+
- PostgreSQL 13+ running locally (or update the URL for a remote DB)

## 2. Database Setup

```bash
psql -U postgres -f setup.sql
```

This creates a `productdb` database. Tables (`users`, `products`) are
auto-created on first run because `spring.jpa.hibernate.ddl-auto=update`.

## 3. Configure

Edit `src/main/resources/application.properties` if your PostgreSQL
username/password/port differ from the defaults:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/productdb
spring.datasource.username=postgres
spring.datasource.password=postgres
```

Also change `jwt.secret` to your own long random string before deploying
anywhere real.

## 4. Run

```bash
cd product-api
mvn spring-boot:run
```

The API starts on `http://localhost:8080`.

## 5. API Endpoints

### Auth (public)

**Register**

```
POST /api/auth/register
Content-Type: application/json

{
  "username": "john",
  "email": "john@example.com",
  "password": "secret123"
}
```

Response:

```json
{
  "token": "eyJhbGciOi...",
  "username": "john",
  "role": "ROLE_USER"
}
```

**Login**

```
POST /api/auth/login
Content-Type: application/json

{
  "username": "john",
  "password": "secret123"
}
```

Response: same shape as register, with a fresh JWT.

### Products (require `Authorization: Bearer <token>` header)

| Method | Endpoint                          | Description             |
| ------ | --------------------------------- | ----------------------- |
| GET    | /api/products                     | List all products       |
| GET    | /api/products/{id}                | Get one product         |
| GET    | /api/products/search?name=phone   | Search products by name |
| GET    | /api/products/category/{category} | Filter by category      |
| POST   | /api/products                     | Create a product        |
| PUT    | /api/products/{id}                | Update a product        |
| DELETE | /api/products/{id}                | Delete a product        |

**Create product example**

```
POST /api/products
Authorization: Bearer eyJhbGciOi...
Content-Type: application/json

{
  "name": "Wireless Mouse",
  "description": "Ergonomic 2.4GHz wireless mouse",
  "price": 19.99,
  "quantity": 150,
  "category": "Electronics"
}
```

## 6. Quick test with curl

```bash
# Register
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"john","email":"john@example.com","password":"secret123"}'

# Login (copy the token from the response)
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"john","password":"secret123"}'

# Create a product (replace TOKEN)
curl -X POST http://localhost:8080/api/products \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Laptop","description":"14-inch","price":999.99,"quantity":10,"category":"Electronics"}'

# List products
curl http://localhost:8080/api/products -H "Authorization: Bearer TOKEN"
```

## Notes

- Passwords are hashed with BCrypt before storing — never stored in plain text.
- Auth is stateless (JWT), so no server-side sessions.
- `ddl-auto=update` is convenient for development; use a migration tool
  (Flyway/Liquibase) and set it to `validate` in production.
- CORS is currently open (`*`) for easy local frontend testing — restrict
  `allowedOriginPatterns` in `SecurityConfig.java` before going to production.
