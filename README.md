# Java Spring Boot TODO — Fullstack (Backend + Static Frontend)

A compact, full-stack TODO application built with Spring Boot (Java) for the backend and simple static HTML/JS for the frontend. This README gives a short, clear walkthrough to run, test, and use the project.

## What this repo contains

- `TodoBackend/` — Spring Boot backend (REST API, JPA, security with JWT). Uses H2 (runtime) and has PostgreSQL settings in `application.properties`.
- `TodoFrontend/` — Static frontend pages (login.html, register.html, todos.html, script.js, style.css). The frontend calls the backend APIs and uses JWT from login.

## Tech stack

- Java 24, Spring Boot 3.5.x
- Spring Web, Spring Data JPA, Spring Security
- JWT (jjwt) for stateless auth
- H2 (runtime) and PostgreSQL (configurable)
- Lombok for model boilerplate
- OpenAPI (springdoc) present for API docs

## Quick contract (what to expect)

- Backend inputs: JSON REST calls (auth and todo payloads). Output: JSON responses and HTTP status codes.
- Auth: `/auth/register` and `/auth/login`. Login returns `{ "token": "..." }`.
- Protected endpoints: all `/api/**` endpoints require `Authorization: Bearer <token>`.

## Run the backend (local)

1. Open a terminal and go to the backend folder:

```bash
cd TodoBackend
```

2. Run using the included Maven wrapper:

```bash
./mvnw spring-boot:run
```

or build and run the jar:

```bash
./mvnw -DskipTests package
java -jar target/*.jar
```

Notes:
- By default `src/main/resources/application.properties` is configured for PostgreSQL (jdbc:postgresql://localhost:5432/tododb). If you don't have Postgres, either create the DB and set credentials or switch to H2 (see below).

## Quick: run with in-memory H2 (no DB install)

Edit `TodoBackend/src/main/resources/application.properties` and replace the datasource URL with:

```properties
spring.datasource.url=jdbc:h2:mem:todo-db;DB_CLOSE_DELAY=-1
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=update
```

Then restart the app. H2 will run in-memory (data clears on restart).

## Authentication & Security

- Public endpoints: `POST /auth/register` and `POST /auth/login`.
- All other endpoints require a JWT in header `Authorization: Bearer <token>`.
- Token lifetime in the shipped `JwtUtil` is short (check `JwtUtil.EXPIRATION_TIME`) — for development you may increase it.

Example: login request

POST /auth/login

Request body (JSON):

```json
{ "email": "alice@example.com", "password": "password" }
```

Successful response:

```json
{ "token": "eyJhbGciOi..." }
```

Use that token for Todo requests:

Header: `Authorization: Bearer eyJhbGciOi...`

## API (core)

- POST `/auth/register` — body: { "email","password" } — registers a user.
- POST `/auth/login` — body: { "email","password" } — returns `{token}`.

- GET `/api/v1/todo` — list all todos (authenticated)
- GET `/api/v1/todo/{id}` — get one todo by id
- GET `/api/v1/todo/page?page=0&size=10` — paged todos
- POST `/api/v1/todo/create` — create todo; request body (JSON) follows Todo model
- PUT `/api/v1/todo` — update todo (full todo JSON)
- DELETE `/api/v1/todo/{id}` — delete by id

Todo JSON shape (model):

```json
{
	"id": 1,               // omit on create
	"title": "Buy milk",
	"description": "2 liters",
	"isCompleted": false
}
```

API status codes: 200 OK, 201 CREATED (create), 401 UNAUTHORIZED (missing/invalid token), 404 NOT FOUND, 409 CONFLICT (duplicate registration), etc.

## Frontend (static)

- Files live in `TodoFrontend/` and are plain HTML/CSS/JS.
- For local testing you can open `login.html` and `register.html` in a browser or serve them via a static server (Live Server extension or `python -m http.server` in `TodoFrontend`).
- The frontend obtains a token on login and stores it (the shipped `script.js` uses that token when calling `/api/v1/todo`). Ensure the backend server is running at the base URL expected by the frontend (default `http://localhost:8080`).

## Run tests (backend)

From `TodoBackend` run:

```bash
./mvnw test
```

This runs the project's unit tests. If you prefer a quick build check, run `./mvnw -DskipTests package`


---


