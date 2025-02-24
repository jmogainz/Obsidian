Okay here is my current codebase tree and all relevant code that I think you are going to need to help me make modifications and some stuff to it. It is an auth service for the backend of my gig economy style application. 

Below is my project information that will provide you with enough context in my codebase to be able to add new features, enhance old ones, help me find bugs, and answer general questions. This is a backend auth service in a SOA architecture for my gig economy application. 

File tree:

.
├── cmd
│   └── main.go
├── codebase_content.txt
├── config
│   └── config.go
├── docker-compose.yml
├── Dockerfile
├── Dockerfile.migrate
├── fly.toml
├── go.mod
├── go.sum
├── install
│   └── poof-back-auth
├── internal
│   ├── app
│   │   └── app.go
│   ├── controllers
│   │   ├── health_controller.go
│   │   ├── pm_controller.go
│   │   ├── registration_controller.go
│   │   ├── validators.go
│   │   ├── verification_controller.go
│   │   └── worker_controller.go
│   ├── dtos
│   │   ├── login_pm_request.go
│   │   ├── login_worker_request.go
│   │   ├── register_pm_request.go
│   │   └── register_worker_request.go
│   ├── integration_test
│   │   └── auth_flow_test.go
│   ├── services
│   │   ├── email_service.go
│   │   ├── jwt_service.go
│   │   ├── jwt_service_test.go
│   │   ├── password_service.go
│   │   ├── pm_auth_service.go
│   │   ├── sms_service.go
│   │   ├── sms_service_test.go
│   │   └── worker_auth_service.go
│   └── utils
│       └── totp.go
├── Makefile
├── migrations
│   ├── 000001_initial.up.sql
│   └── README.md
├── README.md
<<<<<<< Updated upstream
└── scripts
    └── migrate.sh

12 directories, 35 files

File contents:

## .env

## Makefile

# Makefile

.PHONY: build up migrate integration-test unit-test down ci clean ssh-setup

## 1) SSH Setup: Start agent & add all keys from ~/.ssh
ssh-setup:
	@if [ -z "$$SSH_AUTH_SOCK" ]; then \
	  echo "[SSH Setup] No SSH agent found. Starting one..."; \
	  eval `ssh-agent -s`; \
	  echo "[SSH Setup] Adding all private keys from ~/.ssh..."; \
	  for key in $$(ls ~/.ssh/id_* 2>/dev/null || true); do \
	    if [ -f $$key ]; then \
	      echo "   -> Adding key: $$key"; \
	      ssh-add $$key >/dev/null 2>&1 || true; \
	    fi; \
	  done; \
	else \
	  echo "[SSH Setup] SSH agent already running at $$SSH_AUTH_SOCK"; \
	fi

## 2) Build all Docker images, passing the SSH key automatically
build: ssh-setup
	@echo "[Build] Using BuildKit SSH forwarding..."
	COMPOSE_DOCKER_CLI_BUILD=1 DOCKER_BUILDKIT=1 \
	docker-compose build --ssh default

## Start db + auth in the background
up:
	docker-compose up -d db poof-back-auth

## Run migrations in a one-off container
migrate:
	docker-compose run --rm migrate

## Run unit tests inside the "unit-tests" container
unit-test:
	docker-compose run --rm poof-back-auth-unit-tests

## Run integration tests (depends on poof-back-auth: healthy)
integration-test:
	docker-compose run --rm poof-back-auth-test

## Shut down all containers
down:
	docker-compose down

## Clean everything so next build is totally fresh
clean:
	@echo "Cleaning up Docker containers, images, volumes, and networks for this Compose project..."
	docker-compose down --rmi local -v --remove-orphans
	@echo "Cleanup complete. Next build will be from scratch for THIS project."

## CI pipeline: build -> up -> migrate -> unit-test -> integration-test -> down
ci: build
	@echo "[CI] Starting CI pipeline..."
	$(MAKE) up
	$(MAKE) migrate
	$(MAKE) unit-test
	$(MAKE) integration-test
	$(MAKE) down
	@echo "[CI] Pipeline complete."


## Dockerfile.migrate

FROM alpine:latest

# Install the migrate CLI 
# (or you can copy from a builder stage if you want).
RUN apk add --no-cache ca-certificates bash \
    && apk add --no-cache curl \
    && wget https://github.com/golang-migrate/migrate/releases/download/v4.16.2/migrate.linux-amd64.tar.gz \
    && tar xvf migrate.linux-amd64.tar.gz -C /usr/local/bin \
    && rm migrate.linux-amd64.tar.gz

WORKDIR /app
COPY scripts/migrate.sh scripts/migrate.sh
COPY migrations/ migrations/

RUN chmod +x scripts/migrate.sh
CMD ["./scripts/migrate.sh", "up"]


=======
├── scripts
│   └── migrate.sh
└── update_poof_go_packages.sh

13 directories, 37 files

File contents:

>>>>>>> Stashed changes
## fly.toml

app = "poof-back-auth"
primary_region = "atl"

[env]
  ENV = "production"

[build]
  image = "docker.io/username/poof-back-auth:latest"

# Define the internal port for the backend service
[[services]]
  internal_port = 8082
  protocol = "tcp"
  [[services.ports]]
    handlers = ["http"]
    port = 80
  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443

# Health check for the auth service
[[services.http_checks]]
  interval = "10s"
  timeout = "2s"
  grace_period = "5s"
  method = "GET"
  path = "/health"    # Adjust this to the correct health endpoint for auth
  protocol = "http"
  restart_limit = 0

[vm]
  memory = "1gb"           # Allocate 1GB RAM for backend
  size = "shared-cpu-1x"   # Allocate 1 shared CPU


## README.md



## update_poof_go_packages.sh

#!/usr/bin/env bash
#
# Usage:
#   ./update_poof_go_packages.sh <branch>
#
# Example:
#   ./update_poof_go_packages.sh develop
#   ./update_poof_go_packages.sh main
#
# The script will fetch each configured Poof repository from GitHub
# at the specified branch/tag.

set -e  # Exit immediately if a command exits with a non-zero status.

BRANCH="$1"

if [ -z "$BRANCH" ]; then
  echo "[ERROR] No branch specified."
  echo "Usage: $0 <branch>"
  echo "Example: $0 develop"
  exit 1
fi

# Allowed branches - you can remove or add as you wish
# if you want to allow more branches without restriction, skip this block.
case "$BRANCH" in
  main|staging|develop)
    ;;
  *)
    echo "[ERROR] Invalid branch '$BRANCH'. Allowed: main, staging, develop."
    exit 1
    ;;
esac

echo "[INFO] Using branch: $BRANCH"

# The list of sub-packages you want to fetch from github.com/poofdev
# just put the "suffix" after "poof-back-"
PACKAGES=(
  "poof-back-middleware"
  "poof-back-utils"
  "poof-back-repositories"
  "poof-back-models"
)

# 1) Export GOPRIVATE
echo "[INFO] Exporting GOPRIVATE=github.com/poofdev/*"
export GOPRIVATE="github.com/poofdev/*"

# 2) Set git config for github.com to use SSH instead of HTTPS
if git config --global -l | grep -q 'url.git@github.com:.insteadof=https://github.com/'; then
  echo "[INFO] Git SSH config for github.com already set. Skipping."
else
  echo "[INFO] Config not found. Setting url.git@github.com:.insteadOf \"https://github.com/\""
  git config --global url."git@github.com:".insteadOf "https://github.com/"
fi

# 3) Fetch each package via go get
for PKG in "${PACKAGES[@]}"; do
  FULLNAME="github.com/poofdev/$PKG"
  echo "[INFO] go get $FULLNAME@$BRANCH"
  go get "$FULLNAME@$BRANCH"
done

echo "[INFO] Done. Successfully fetched all Poof repos at branch '$BRANCH'."
exit 0



## Makefile

# Makefile

.PHONY: build up migrate integration-test unit-test down ci clean

## 1) Build all Docker images, passing the SSH key automatically
build:
	@if [ -z "$$SSH_AUTH_SOCK" ]; then \
	  echo "[SSH Setup] No SSH agent found. Starting one..."; \
	  eval "$$(ssh-agent -s)"; \
	  echo "[SSH Setup] Agent started; using socket at $$SSH_AUTH_SOCK"; \
	  echo "[SSH Setup] Adding all private keys from ~/.ssh..."; \
	  for key in $$(ls ~/.ssh/id_* 2>/dev/null || true); do \
	    if [ -f $$key ] && [[ ! "$$key" =~ \.pub$$ ]]; then \
  			echo "   -> Adding key: $$key"; \
	      ssh-add $$key >/dev/null 2>&1 || true; \
	    fi; \
	  done; \
	else \
	  echo "[SSH Setup] SSH agent already running at $$SSH_AUTH_SOCK"; \
	fi; \
	echo "[Build] Using BuildKit SSH forwarding with SSH_AUTH_SOCK=$$SSH_AUTH_SOCK"; \
	COMPOSE_DOCKER_CLI_BUILD=1 DOCKER_BUILDKIT=1 \
	docker-compose build --ssh default=$$SSH_AUTH_SOCK

## Start db + auth in the background
up:
	docker-compose up -d db poof-back-auth

## Run migrations in a one-off container
migrate:
	docker-compose run --rm migrate

## Run unit tests inside the "unit-tests" container
unit-test:
	docker-compose run --rm poof-back-auth-unit-tests

## Run integration tests (depends on poof-back-auth: healthy)
integration-test:
	docker-compose run --rm poof-back-auth-test

## Shut down all containers
down:
	docker-compose down

## Clean everything so next build is totally fresh
clean:
	@echo "Cleaning up Docker containers, images, volumes, and networks for this Compose project..."
	docker-compose down --rmi local -v --remove-orphans
	@echo "Cleanup complete. Next build will be from scratch for THIS project."

## CI pipeline: build -> up -> migrate -> unit-test -> integration-test -> down
ci: build
	@echo "[CI] Starting CI pipeline..."
	$(MAKE) up
	$(MAKE) migrate
	$(MAKE) integration-test
	$(MAKE) down
	$(MAKE) clean
	@echo "[CI] Pipeline complete."



## .gitignore

codebase_content.txt
install/


## Dockerfile

# syntax=docker/dockerfile:1.4

# ---------------------------------
# Stage 1: Build
# ---------------------------------
FROM golang:1.23-alpine AS builder

# 1) Install git + openssh for private repo access
RUN apk update && apk add --no-cache git openssh

ENV GOPRIVATE=github.com/poofdev/*
RUN git config --global url."git@github.com:".insteadOf "https://github.com/"

WORKDIR /app

RUN mkdir -p /root/.ssh \
 && ssh-keyscan github.com >> /root/.ssh/known_hosts \
 && chmod 644 /root/.ssh/known_hosts

# Copy just go.mod and go.sum first for caching
COPY go.mod go.sum ./

# 3) Use BuildKit SSH mount to fetch private modules
RUN --mount=type=ssh go mod download

# Copy the rest of your source code
COPY . .

# Build the app
RUN go build -o /poof-back-auth ./cmd/main.go

# ---------------------------------
# Stage 2: Final minimal image
# ---------------------------------
FROM alpine:latest

# Install curl (and any other needed tools)
RUN apk add --no-cache curl

WORKDIR /root/
COPY --from=builder /poof-back-auth .

EXPOSE 8082
CMD ["./poof-back-auth"]


## .git

gitdir: ../.git/modules/poof-back-auth


## .env

AUTH_SERVICE_URL=http://poof-back-auth:8082
DATABASE_URL=postgres://user:password@db:5432/mydatabase?sslmode=disable
TWILIO_FROM_PHONE=+12568151975
SENDGRID_FROM_EMAIL=thetrashproject123@gmail.com
TEST_EMAIL=jlmoors001@gmail.com
TEST_PHONE_NUMBER=+12567013403
RSA_PRIVATE_KEY_BASE64=LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQzdEMUF4NlpMaU1VdXoKa2xpbGF0S0grekFoYkxFVjBEVzhTOHVPdDNQTkRKbC9EZzlzK2gxcGV6QnJVb1ZudXZGS2Q2VXdNeU5CQjZoYgpXSi9HM1hlTlJ4TVBpZFowMFdJRUR2QXN2YVkwSDhKVncyWGEwM2ljQmdvMUgyNVB1NC9MZXlXNkJWVVFKbTVKCnBpUGFmcml5TEw3U3NjSmZFVnlFaU5JS3ErYjBIYjM5Y2daanp4MnZONWt5NFVCNS9INUdCdFVUNytYYW0yN1gKSytNRHpBMHp1YWxMLzBjVUZwRHFydEo1Wk4zVmkyN2NLb1FBT1htZEx0dVBIalRGZEFyODBacTJpUm5sS3RpNQpSVEVSSTlTUUlobUo1MHpQRjlvMWhhZEhCOGF0MDBESTdQNGFrVmJoOEpNSW8xOHNhdTNUeDRERkw1MVBGUDRiCllKSi9GMENKQWdNQkFBRUNnZ0VBQTRQTFNmY1d1dU9hUElxeHRXNlFhWk5vYTRxbGxKdnJPTmxqdnV0VXJiUHkKNTQxb0ZNdU9VeEVSZ1k0cjl2aTRSYXNVSktuL0NVeWgwbG9tUFVidExtK0xjNUltVXBYamRaZ1AraG1HS2ZXMQpxczdnaVZ1QkJLanM5d2hJdXVvNzRPTHJmRVVKODVEZ3VhS3dabGpueWRGSHRLRTRKM2tsd1FmRnJ5ZnZLd09pCm9GYUxuOVkrMk5rT1h3dXMyUlhLbnNYZFFlbjhxaHVLTWhWRFFkeXU2SmxVOHgwUGN0c0I0cTY3ek9uMXVxeFAKaWRTWVk4UDdaQ014cVJ4TkNHbXVtTHBYVS9SYnRVcUxVOFRlb0ZwNmxhQXlmT0k4eXBBdzl3Z2tRWnFxVFJ3YgpUQWxSNDRwdzVWSy9tTERLMERvSUJ1TlV0WUdVM2R3RnoxRmJrZmRXUVFLQmdRRDNEZVVraE9OTVFzbFZKcElhCmVkcWpNTXJua2RGaGIyU3U3NTFiUFZWWVNSUWxDUXlVeUtNNm9kMlZ0ZXZDY1o3TUZGMlJYcklTbUE5YlIraFgKeFdJdTRUd0dQSkUyZlJoc2lGTDJuUkVhTVRxSnJRR1NXNmRzd250ck5IUElzUG94N3Nkd0diQnhkaWc4VGZIcQpQdVhUamxDVlFNYitMVnJ2MDNkcTRiY2pKUUtCZ1FEQjFVcDhDMiszQUpZYTdLUWJvRStuRFlmVW01Uk1yMXZlCmRCVlI1L25RTmNyNGxpa3FTeTF6ZzBEN0pOL0tBUVU5MWdjcms2dnR0UDUxaGdBeXZWWmZPdDZnV28yalkrdk8KRWtHdHFwdHhTbnpjbmNrNTNtTDdzQnQ5S2pSaUR2OTJhUXU3ajF5MUFKWUVTTTdtNEdWVkFIWFEyaTAwN1lLbAo0VFZwN0Z6Y2xRS0JnUUR3WmFIeURpa0svUFhpR0owVWpEdjJqYlR5N2s0YWpJVWhRR0lTOVRTRnF0NmlSeExwCjkrMzFVS3BJVW5RdGlkZm1aMjdBMUs5Y0xvREs1c2FzR1pJM1ljM0JsOUFKZ0dKeXdaaFJCbmNzMEhoUW5Yc3AKQ214NUJTbUpJTW9Gb0Voa3JCOSs4bEJocDRMeFl6c2lIOEFOUXE4Zy9KNWxtSWFqVlZjZk5yRzR6UUtCZ0dJQQpKQU9waUpjZkRjV1pKYlB1RHJlb2lLZCs3YkVEN0ZBQm94SGhWcFhsekxSNHYyRnAxeFlUSTVTVzVTcnQ3eWQvCmdlcVBaQnJ3S3NOaXQ5RHZsNjdZUmQwUFM1TnpuckoyMm93aXVTckRmWFBSdHY2eUtKdVdRNSs1NmZnMkd3VlUKUVNGWWI0ZjRQdUQxcXQ1aVQvbDFIUncyWXlyaWR1N0ZlY3NQUFRndEFvR0FUaktNajNzMXIrZW1CVjVpSW5ycgoxVWdMakVNejdxMGRnY0NoT2g4NVRFRCtnS3dIWjZpWHBobGgxUXBhdURTdmxXbUdxaVplQmwxa3Z2SUVJNTVzCmRVRndnZGlqNTVzS2ttVmlqY2FFQ2ZXQnFCUG9xYjJzVnE0VWhvanBsSExSSTl3MGxqQVlmOWpwZk9HaUVXOXkKMEF0ZmhEMmlpa1dpSTc5RVcrRklOaUU9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K
RSA_PUBLIC_KEY_BASE64=LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF1dzlRTWVtUzRqRkxzNUpZcFdyUwpoL3N3SVd5eEZkQTF2RXZManJkenpReVpmdzRQYlBvZGFYc3dhMUtGWjdyeFNuZWxNRE1qUVFlb1cxaWZ4dDEzCmpVY1RENG5XZE5GaUJBN3dMTDJtTkIvQ1ZjTmwydE40bkFZS05SOXVUN3VQeTNzbHVnVlZFQ1p1U2FZajJuNjQKc2l5KzBySENYeEZjaElqU0Nxdm05QjI5L1hJR1k4OGRyemVaTXVGQWVmeCtSZ2JWRSsvbDJwdHUxeXZqQTh3TgpNN21wUy85SEZCYVE2cTdTZVdUZDFZdHUzQ3FFQURsNW5TN2JqeDQweFhRSy9OR2F0b2taNVNyWXVVVXhFU1BVCmtDSVppZWRNenhmYU5ZV25Sd2ZHcmROQXlPeitHcEZXNGZDVENLTmZMR3J0MDhlQXhTK2RUeFQrRzJDU2Z4ZEEKaVFJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==


## Dockerfile.migrate

FROM alpine:latest

# Install the migrate CLI 
# (or you can copy from a builder stage if you want).
RUN apk add --no-cache ca-certificates bash \
    && apk add --no-cache curl \
    && wget https://github.com/golang-migrate/migrate/releases/download/v4.16.2/migrate.linux-amd64.tar.gz \
    && tar xvf migrate.linux-amd64.tar.gz -C /usr/local/bin \
    && rm migrate.linux-amd64.tar.gz

WORKDIR /app
COPY scripts/migrate.sh scripts/migrate.sh
COPY migrations/ migrations/

RUN chmod +x scripts/migrate.sh
CMD ["./scripts/migrate.sh", "up"]


## .gitmodules

[submodule "migrations"]
	path = migrations
	url = git@github.com:poofdev/poof-back-migrations.git
[submodule "scripts"]
	path = scripts
	url = git@github.com:poofdev/poof-back-scripts.git


## docker-compose.yml

services:
  # ----------------------
  # 1) Postgres DB
  # ----------------------
  db:
    image: postgres:13
    container_name: poof_auth_db
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydatabase
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    # Add a healthcheck so Docker knows when Postgres is actually ready
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydatabase || exit 1"]
      interval: 5s
      timeout: 5s
      retries: 5

  # ----------------------
  # 2) Auth Service
  # ----------------------
  poof-back-auth:
    build: .
    container_name: poof_back_auth
    env_file:
      - .env
<<<<<<< Updated upstream
    # The line below ensures that from inside the container,
    # any references to `AUTH_SERVICE_URL` are set to
    # http://poof-back-auth:8082 (Docker's internal network)
=======
>>>>>>> Stashed changes
    ports:
      - "8082:8082"
    # Wait for db to be healthy (only works if you are either on Compose v2.x
    # or Docker Swarm mode in Compose v3). If you’re purely on Compose v3 in
    # local mode, you can keep this, but it might not enforce waiting at runtime.
    depends_on:
<<<<<<< Updated upstream
      - db
    # Healthcheck so other containers can wait until it's truly ready
=======
      db:
        condition: service_healthy

    # Healthcheck so other containers (or you) can see if auth is actually ready
>>>>>>> Stashed changes
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8082/health"]
      interval: 5s
      timeout: 2s
      retries: 5

  # ----------------------
  # 3) Migration Container
  # ----------------------
<<<<<<< Updated upstream
  # This re-uses the same Docker build context, so it has
  # the code + scripts available. We override the command to run migrations.
=======
>>>>>>> Stashed changes
  migrate:
    build:
      context: .
      dockerfile: Dockerfile.migrate
    container_name: poof_back_auth_migrate
    depends_on:
<<<<<<< Updated upstream
      - db
=======
      db:
        condition: service_healthy
>>>>>>> Stashed changes
    env_file:
      - .env

  # ----------------------
  # 4) Integration Test Container
  # ----------------------
<<<<<<< Updated upstream
  # Runs your integration tests, pointing to 'poof-back-auth' service.
  poof-back-auth-test:
    build: .
=======
  poof-back-auth-test:
    build:
      context: .
      target: builder
>>>>>>> Stashed changes
    container_name: poof_back_auth_test
    depends_on:
      poof-back-auth:
        condition: service_healthy
    env_file:
      - .env
    command: sh -c "
<<<<<<< Updated upstream
      echo '[Integration Tests] Running against poof-back-auth...' &&
=======
      echo '[Integration Tests] Running...' &&
>>>>>>> Stashed changes
      go test -v ./internal/integration_test/... &&
      echo 'Integration tests finished successfully.'"

  # ----------------------
<<<<<<< Updated upstream
  # 5) Optional: Unit Test Container
  # ----------------------
  # In case you want to run short (unit) tests inside Docker as well.
  poof-back-auth-unit-tests:
    build: .
    container_name: poof_back_auth_unit_tests
    depends_on:
      - db
=======
  # 5) Unit Test Container
  # ----------------------
  poof-back-auth-unit-tests:
    build:
      context: .
      target: builder
    container_name: poof_back_auth_unit_tests
    depends_on:
      db:
        condition: service_healthy
>>>>>>> Stashed changes
    env_file:
      - .env
    command: sh -c "
      echo '[Unit Tests] Running...' &&
      go test -v ./internal/... -short &&
      echo 'Unit tests finished.'"

volumes:
  pgdata:
    driver: local


## migrations/README.md

<<<<<<< Updated upstream
# syntax=docker/dockerfile:1.4

# ---------------------------------
# Stage 1: Build
# ---------------------------------
FROM golang:1.23-alpine AS builder

# 1) Install git + openssh for private repo access
RUN apk update && apk add --no-cache git openssh

ENV GOPRIVATE=github.com/poofdev/*
RUN git config --global url."git@github.com:".insteadOf "https://github.com/"

WORKDIR /app

RUN mkdir -p /root/.ssh \
 && ssh-keyscan github.com >> /root/.ssh/known_hosts \
 && chmod 644 /root/.ssh/known_hosts

# Copy just go.mod and go.sum first for caching
COPY go.mod go.sum ./

# 3) Use BuildKit SSH mount to fetch private modules
RUN --mount=type=ssh go mod download

# Copy the rest of your source code
COPY . .

# Build the app
RUN go build -o /poof-back-auth ./cmd/main.go

# ---------------------------------
# Stage 2: Final minimal image
# ---------------------------------
FROM alpine:latest

WORKDIR /root/
COPY --from=builder /poof-back-auth .

EXPOSE 8082
CMD ["./poof-back-auth"]
=======
# Poof Migrations for use as submodule, early stage SOA, single db.
>>>>>>> Stashed changes


## migrations/.git

gitdir: ../../.git/modules/poof-back-auth/modules/migrations


## migrations/000001_initial.up.sql

-- 000001_initial.up.sql

-- -------------------------------------
-- Property Managers "account" table
-- -------------------------------------
CREATE TABLE property_managers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    phone_number VARCHAR(20) UNIQUE,
    auth_provider VARCHAR(20) NOT NULL DEFAULT 'email',
    totp_secret VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    business_name VARCHAR(255) NOT NULL,
    business_address VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(50) NOT NULL,
    zip_code VARCHAR(20) NOT NULL,
    persona_inquiry_id VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_pm_email ON property_managers(email);
CREATE INDEX idx_pm_phone_number ON property_managers(phone_number);

-- -------------------------------------
-- Workers "account" table
-- -------------------------------------
CREATE TABLE workers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    phone_number VARCHAR(20) UNIQUE NOT NULL,
    auth_provider VARCHAR(20) NOT NULL DEFAULT 'email',
    totp_secret VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    street_address VARCHAR(255) NOT NULL,
    apt_suite VARCHAR(50),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(50) NOT NULL,
    zip_code VARCHAR(20) NOT NULL,
    vehicle_year INT NOT NULL,
    vehicle_make VARCHAR(100) NOT NULL,
    vehicle_model VARCHAR(100) NOT NULL,
    persona_inquiry_id VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_worker_email ON workers(email);
CREATE INDEX idx_worker_phone_number ON workers(phone_number);

-- Now that PM table is our "account" for PM, link "properties" to property_managers.id
CREATE TABLE properties (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    manager_id UUID REFERENCES property_managers(id) ON DELETE CASCADE,
    property_name VARCHAR(255) NOT NULL,
    address VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(50) NOT NULL,
    zip_code VARCHAR(20) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_properties_manager_id ON properties(manager_id);

CREATE TABLE property_buildings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID REFERENCES properties(id) ON DELETE CASCADE,
    building_name VARCHAR(100),
    address VARCHAR(255) NOT NULL,
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6)
);

CREATE INDEX idx_property_buildings_property_id ON property_buildings(property_id);

CREATE TABLE units (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID REFERENCES properties(id) ON DELETE CASCADE,
    unit_number VARCHAR(50) NOT NULL,
    tenant_token VARCHAR(255) UNIQUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_units_tenant_token ON units(tenant_token);
CREATE INDEX idx_units_property_id ON units(property_id);

CREATE TABLE trash_compactors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID REFERENCES properties(id) ON DELETE CASCADE,
    latitude DECIMAL(9,6) NOT NULL,
    longitude DECIMAL(9,6) NOT NULL
);

CREATE INDEX idx_trash_compactors_property_id ON trash_compactors(property_id);

-- --------------------------------------
-- PM refresh tokens
-- --------------------------------------
CREATE TABLE pm_refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    pm_id UUID REFERENCES property_managers(id) ON DELETE CASCADE,
    refresh_token VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    revoked BOOLEAN DEFAULT FALSE,
    ip_address VARCHAR(45)
);

CREATE INDEX idx_pm_refresh_tokens ON pm_refresh_tokens(refresh_token);

-- --------------------------------------
-- Worker refresh tokens
-- --------------------------------------
CREATE TABLE worker_refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    worker_id UUID REFERENCES workers(id) ON DELETE CASCADE,
    refresh_token VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    revoked BOOLEAN DEFAULT FALSE,
    ip_address VARCHAR(45)
);

CREATE INDEX idx_worker_refresh_tokens ON worker_refresh_tokens(refresh_token);

-- --------------------------------------
-- PM login attempts
-- --------------------------------------
CREATE TABLE pm_login_attempts (
    pm_id UUID PRIMARY KEY REFERENCES property_managers(id) ON DELETE CASCADE,
    attempt_count INT NOT NULL DEFAULT 0,
    locked_until TIMESTAMP WITH TIME ZONE,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- --------------------------------------
-- Worker login attempts
-- --------------------------------------
CREATE TABLE worker_login_attempts (
    worker_id UUID PRIMARY KEY REFERENCES workers(id) ON DELETE CASCADE,
    attempt_count INT NOT NULL DEFAULT 0,
    locked_until TIMESTAMP WITH TIME ZONE,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- SMS Verification Codes
CREATE TABLE sms_verification_codes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone_number VARCHAR(20) NOT NULL,
    verification_code VARCHAR(10) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    attempts INTEGER DEFAULT 0
);

CREATE INDEX idx_sms_codes_phone_number ON sms_verification_codes(phone_number);

-- Email Verification Codes
CREATE TABLE email_verification_codes (
    id UUID PRIMARY KEY,
    email TEXT NOT NULL,
    verification_code TEXT NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    attempts INT DEFAULT 0,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_email_verification_codes_email ON email_verification_codes(email);

-- PM Blacklisted tokens (if you want to keep a blacklisting system)
CREATE TABLE pm_blacklisted_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    token_id VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_pm_blacklisted_tokens_token_id ON pm_blacklisted_tokens(token_id);

-- Worker Blacklisted tokens
CREATE TABLE worker_blacklisted_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    token_id VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_worker_blacklisted_tokens_token_id ON worker_blacklisted_tokens(token_id);

-- Done


## cmd/main.go

// cmd/main.go

package main

import (
    "net/http"

    "github.com/gorilla/mux"
    "github.com/rs/cors"

    "github.com/poofdev/poof-back-auth/config"
    "github.com/poofdev/poof-back-auth/internal/app"
    "github.com/poofdev/poof-back-auth/internal/controllers"
    "github.com/poofdev/poof-back-auth/internal/services"
    "github.com/poofdev/poof-back-repositories"
    "github.com/poofdev/poof-back-utils"
)

func main() {
    // 1) Init Logger + Load Config
    utils.InitLogger()
    cfg := config.LoadConfig()

    // 2) New App (DB connection, etc.)
    application, err := app.NewApp(cfg)
    if err != nil {
        utils.Logger.Fatal("Failed to initialize application:", err)
    }
    defer application.Close()

    // 3) Repositories
    pmRepo := repositories.NewPropertyManagerRepository(application.DB, cfg.DBEncryptionKey)
    pmTokenRepo := repositories.NewPMTokenRepository(application.DB)
    pmLoginRepo := repositories.NewPMLoginAttemptsRepository(application.DB)

    workerRepo := repositories.NewWorkerRepository(application.DB, cfg.DBEncryptionKey)
    workerTokenRepo := repositories.NewWorkerTokenRepository(application.DB)
    workerLoginRepo := repositories.NewWorkerLoginAttemptsRepository(application.DB)

    smsRepo := repositories.NewSMSRepository(application.DB)
    emailRepo := repositories.NewEmailRepository(application.DB)

    // 4) Core Services: JWT, SMS, Email
    smsService := services.NewSMSService(cfg, smsRepo)
    emailService := services.NewEmailService(cfg, emailRepo)

    // 5) Auth Services for PM + Worker
    pmAuthService := services.NewPMAuthService(
        pmRepo,
        pmTokenRepo,            // pass the PM-specific token repo
        pmLoginRepo,
        smsService,
        emailService,
        cfg,
    )

    workerAuthService := services.NewWorkerAuthService(
        workerRepo,
        workerTokenRepo,        // pass the Worker-specific token repo
        workerLoginRepo,
        smsService,
        emailService,
        cfg,
    )

    // 6) Controllers
    // Health
    healthController := controllers.NewHealthController(application)

    // PM
    pmController := controllers.NewPMController(pmAuthService, pmRepo)

    // Worker
    workerController := controllers.NewWorkerController(workerAuthService, workerRepo)

    // Verification (SMS/Email codes)
    verificationController := controllers.NewVerificationController(smsService, emailService)

    // TOTP secret generation (if you still want the single route)
    registrationController := controllers.NewRegistrationController() // or adapt if needed

    // 7) Set up Router
    router := mux.NewRouter()

    // Health
    router.HandleFunc("/health", healthController.HealthCheckHandler).Methods("GET")

    // Auth subrouter
    authRouter := router.PathPrefix("/auth").Subrouter()

    // ---- Property Manager Endpoints ----
    authRouter.HandleFunc("/pm/register", pmController.RegisterPM).Methods("POST")
    authRouter.HandleFunc("/pm/login", pmController.LoginPM).Methods("POST")
    authRouter.HandleFunc("/pm/exists/email", pmController.CheckPMEmailExists).Methods("POST")
    authRouter.HandleFunc("/pm/exists/phone", pmController.CheckPMPhoneNumberExists).Methods("POST")
    authRouter.HandleFunc("/pm/refresh_token", pmController.RefreshTokenPM).Methods("POST")
    authRouter.HandleFunc("/pm/logout", pmController.LogoutPM).Methods("POST")

    // ---- Worker Endpoints ----
    authRouter.HandleFunc("/worker/register", workerController.RegisterWorker).Methods("POST")
    authRouter.HandleFunc("/worker/login", workerController.LoginWorker).Methods("POST")
    authRouter.HandleFunc("/worker/exists/email", workerController.CheckWorkerEmailExists).Methods("POST")
    authRouter.HandleFunc("/worker/exists/phone", workerController.CheckWorkerPhoneNumberExists).Methods("POST")
    authRouter.HandleFunc("/worker/refresh_token", workerController.RefreshTokenWorker).Methods("POST")
    authRouter.HandleFunc("/worker/logout", workerController.LogoutWorker).Methods("POST")

    // ---- TOTP + Verification Endpoints (shared) ----
    authRouter.HandleFunc("/register/totp_secret", registrationController.GenerateTOTPSecret).Methods("POST")
    authRouter.HandleFunc("/verify/request_sms_code", verificationController.RequestSMSCode).Methods("POST")
    authRouter.HandleFunc("/verify/check_sms_code", verificationController.VerifySMSCode).Methods("POST")
    authRouter.HandleFunc("/verify/request_email_code", verificationController.RequestEmailCode).Methods("POST")
    authRouter.HandleFunc("/verify/check_email_code", verificationController.VerifyEmailCode).Methods("POST")

    // 8) CORS
    c := cors.New(cors.Options{
        AllowedOrigins:   []string{"https://poof.com"}, // Or adapt
        AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
        AllowedHeaders:   []string{"Authorization", "Content-Type"},
        AllowCredentials: true,
    })

    // 9) Start Server
    utils.Logger.Info("Starting server on port 8082...")
    if err := http.ListenAndServe(":8082", c.Handler(router)); err != nil {
        utils.Logger.Fatal("Failed to start server:", err)
    }
}

<<<<<<< Updated upstream
type OpenIDProviderConfig struct {
	ClientID     string
	ClientSecret string
	RedirectURL  string
	ProviderURL  string
}

func LoadConfig() *Config {
	privateKeyBase64 := os.Getenv("RSA_PRIVATE_KEY_BASE64")
    if privateKeyBase64 == "" {
        log.Fatal("RSA_PRIVATE_KEY_BASE64 not set")
    }
    // 1) Decode from base64
    privateKeyPEM, err := base64.StdEncoding.DecodeString(privateKeyBase64)
    if err != nil {
        log.Fatalf("Failed to decode base64 private key: %v", err)
    }
    // 2) Parse the PEM block
    block, _ := pem.Decode(privateKeyPEM)
    if block == nil {
        log.Fatal("Failed to decode PEM block for private key")
    }
    privateKey, err := jwt.ParseRSAPrivateKeyFromPEM(privateKeyPEM)
    if err != nil {
        log.Fatalf("Failed to parse RSA private key: %v", err)
    }

    publicKeyBase64 := os.Getenv("RSA_PUBLIC_KEY_BASE64")
    if publicKeyBase64 == "" {
        log.Fatal("RSA_PUBLIC_KEY_BASE64 not set")
    }
    publicKeyPEM, err := base64.StdEncoding.DecodeString(publicKeyBase64)
    if err != nil {
        log.Fatalf("Failed to decode base64 public key: %v", err)
    }
    block, _ = pem.Decode(publicKeyPEM)
    if block == nil {
        log.Fatal("Failed to decode PEM block for public key")
    }
    publicKey, err := jwt.ParseRSAPublicKeyFromPEM(block.Bytes)
    if err != nil {
        log.Fatalf("Failed to parse RSA public key: %v", err)
    }

	dbEncryptionKey := os.Getenv("DB_ENCRYPTION_KEY")
	if dbEncryptionKey == "" {
		log.Fatal("DBEncryptionKey not set in environment variables")
	}
	decodedKey, err := base64.StdEncoding.DecodeString(dbEncryptionKey)
	if err != nil {
		log.Fatal("Failed to decode DBEncryptionKey:", err)
	}
	if len(decodedKey) != 32 {
		log.Fatal("DBEncryptionKey must be 32 bytes for AES-256 encryption")
	}

	appName := os.Getenv("APP_NAME")
	if appName == "" {
		appName = "Poof"
	}

	// Return the complete configuration
	return &Config{
		AppName:            appName,
		DatabaseURL:        os.Getenv("DATABASE_URL"),
		DBEncryptionKey:  decodedKey,
		TokenExpiry:        10 * time.Minute,
		RefreshTokenExpiry: 7 * 24 * time.Hour,
		MaxLoginAttempts: 10,
		AttemptWindow: 5 * time.Minute,
		LockDuration: 10 * time.Minute,
		TwilioAccountSID: os.Getenv("TWILIO_ACCOUNT_SID"),
		TwilioAuthToken:  os.Getenv("TWILIO_AUTH_TOKEN"),
		TwilioFromPhone:  os.Getenv("TWILIO_FROM_PHONE"),
		RSAPrivateKey:    privateKey,
		RSAPublicKey:     publicKey,
		SendGridAPIKey:   os.Getenv("SENDGRID_API_KEY"),
		SendGridFromEmail: os.Getenv("SENDGRID_FROM_EMAIL"),
	}
}


## scripts/migrate.sh

#!/bin/bash
set -e

# 1) Ensure DATABASE_URL is set
if [ -z "$DATABASE_URL" ]; then
  echo "[ERROR] DATABASE_URL is not set. Please export a valid Postgres connection string."
  exit 1
fi

# 2) Check that it looks like a PostgreSQL URL
#    (Naive check: just ensures it starts with postgres:// or postgresql://)
if ! [[ "$DATABASE_URL" =~ ^postgres(ql)?:// ]]; then
  echo "[ERROR] DATABASE_URL doesn’t appear to be a valid PostgreSQL address: $DATABASE_URL"
  exit 1
fi

# 3) Run migrate with the provided arguments ($@)
migrate -database "$DATABASE_URL" -path ./migrations "$@"

=======
>>>>>>> Stashed changes


## scripts/.git

gitdir: ../../.git/modules/poof-back-auth/modules/scripts


## internal/utils/totp.go

package utils

import (
    "encoding/base64"
    "fmt"

    "github.com/pquerna/otp/totp"
    "github.com/skip2/go-qrcode"
)

func GenerateTOTPSecret(appName string, accountName string) (string, error) {
    key, err := totp.Generate(totp.GenerateOpts{
        Issuer:      appName,
        AccountName: accountName,
    })
    if err != nil {
        return "", err
    }
    return key.Secret(), nil
}

func GenerateTOTPQRCode(secret string, appName string, accountName string) (string, error) {
    otpauthURL := fmt.Sprintf("otpauth://totp/%s?secret=%s&issuer=%s", accountName, secret, appName)
    png, err := qrcode.Encode(otpauthURL, qrcode.Medium, 256)
    if err != nil {
        return "", err
    }
    // Encode PNG to Base64 string
    encoded := base64.StdEncoding.EncodeToString(png)
    return encoded, nil
}

func ValidateTOTPCode(secret, code string) bool {
    return totp.Validate(code, secret)
}


## internal/controllers/health_controller.go

// controllers/health_controller.go

package controllers

import (
    "net/http"
    "context"

    "github.com/poofdev/poof-back-auth/internal/app"
    "github.com/poofdev/poof-back-utils"
)

type HealthController struct {
    app *app.App
}

func NewHealthController(app *app.App) *HealthController {
    return &HealthController{
        app: app,
    }
}

func (c *HealthController) HealthCheckHandler(w http.ResponseWriter, r *http.Request) {
    // Check database connectivity using Pool's Ping method
    if err := c.app.DB.Ping(context.Background()); err != nil {
        utils.Logger.WithError(err).Error("Database unreachable")
        utils.RespondWithError(w, http.StatusServiceUnavailable, "Database unreachable", err)
        return
    }

    // Health check passed
    utils.RespondWithJSON(w, http.StatusOK, map[string]string{"status": "OK"})
}


## internal/controllers/validators.go

package controllers

import (
    "github.com/go-playground/validator/v10"
)

var validate = validator.New()


## internal/controllers/pm_controller.go

package controllers

import (
    "encoding/json"
    "net/http"

    "github.com/go-playground/validator/v10"
    "github.com/poofdev/poof-back-auth/internal/dtos"
    "github.com/poofdev/poof-back-auth/internal/services"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-repositories"
    "github.com/poofdev/poof-back-utils"
    auth_utils "github.com/poofdev/poof-back-auth/internal/utils"
)

type PMController struct {
    pmAuthService services.PMAuthService
    pmRepo        repositories.PropertyManagerRepository
}

func NewPMController(
    pmAuth services.PMAuthService,
    pmRepo repositories.PropertyManagerRepository,
) *PMController {
    return &PMController{
        pmAuthService: pmAuth,
        pmRepo:        pmRepo,
    }
}

var pmValidate = validator.New()

// ----------- Registration -----------
func (c *PMController) RegisterPM(w http.ResponseWriter, r *http.Request) {
    var req dtos.RegisterPMRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid payload", err)
        return
    }
    if err := pmValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    // Convert DTO -> models.PropertyManager
    pm := &models.PropertyManager{
        Email:        req.Email,
        PhoneNumber:  req.PhoneNumber,
        AuthProvider: "email_verification",
        TOTPSecret:   req.TOTPSecret,
        BusinessName: req.BusinessName,
        BusinessAddress: req.BusinessAddress,
        City:         req.City,
        State:        req.State,
        ZipCode:      req.ZipCode,
        PersonaVerificationInquiryID: req.PersonaInquiryID,
    }

    // Validate TOTP code
    if !auth_utils.ValidateTOTPCode(pm.TOTPSecret, req.TOTPToken) {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid TOTP code", nil)
        return
    }

    // Register
    err := c.pmAuthService.Register(r.Context(), pm)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Failed to register PM", err)
        return
    }

    utils.RespondWithJSON(w, http.StatusCreated, map[string]string{"message": "PM registered successfully"})
}

// ----------- Login -----------
func (c *PMController) LoginPM(w http.ResponseWriter, r *http.Request) {
    var req dtos.LoginPMRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request", err)
        return
    }
    if err := pmValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    ip := utils.GetIPAddress(r)
    pm, accessToken, refreshToken, err := c.pmAuthService.Login(
        r.Context(),
        req.Email,
        req.TOTPCode,
        ip,
    )
    if err != nil {
        utils.RespondWithError(w, http.StatusUnauthorized, "Login failed", err)
        return
    }

    utils.RespondWithJSON(w, http.StatusOK, map[string]interface{}{
        "pm":            pm,
        "access_token":  accessToken,
        "refresh_token": refreshToken,
    })
}

// ----------- Check Email -----------
func (c *PMController) CheckPMEmailExists(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Email string `json:"email" validate:"required,email"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }
    if err := pmValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    ctx := r.Context()
    pm, err := c.pmRepo.GetByEmail(ctx, req.Email)
    if err != nil {
        // If not found, pm is nil or error is something other than "no rows"
        utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": false})
        return
    }
    if pm != nil {
        utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": true})
        return
    }
    utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": false})
}

// ----------- Check Phone -----------
func (c *PMController) CheckPMPhoneNumberExists(w http.ResponseWriter, r *http.Request) {
    var req struct {
        PhoneNumber string `json:"phone_number" validate:"required"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }
    if err := pmValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    ctx := r.Context()

    pm, err := c.pmRepo.GetByPhoneNumber(ctx, req.PhoneNumber)
    if err == nil && pm != nil {
        utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": true})
        return
    }
    utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": false})
}

// ----------- Refresh Token -----------
func (c *PMController) RefreshTokenPM(w http.ResponseWriter, r *http.Request) {
    var req struct {
        RefreshToken string `json:"refresh_token" validate:"required,len=64"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid payload", err)
        return
    }
    if err := pmValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    ip := utils.GetIPAddress(r)
    accessToken, newRefresh, err := c.pmAuthService.RefreshToken(r.Context(), req.RefreshToken, ip)
    if err != nil {
        utils.Logger.WithError(err).Error("PM refresh token error")
        utils.RespondWithError(w, http.StatusUnauthorized, "Refresh token failed", err)
        return
    }

    resp := map[string]string{
        "access_token":  accessToken,
        "refresh_token": newRefresh,
    }
    utils.RespondWithJSON(w, http.StatusOK, resp)
}

// ----------- Logout -----------
func (c *PMController) LogoutPM(w http.ResponseWriter, r *http.Request) {
    var req struct {
        RefreshToken string `json:"refresh_token" validate:"required,len=64"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid payload", err)
        return
    }
    if err := pmValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    err := c.pmAuthService.Logout(r.Context(), req.RefreshToken)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Failed to logout", err)
        return
    }
    utils.RespondWithJSON(w, http.StatusOK, map[string]string{"message": "Logged out successfully"})
}



## internal/controllers/worker_controller.go

package controllers

import (
    "encoding/json"
    "net/http"

    "github.com/go-playground/validator/v10"
    "github.com/poofdev/poof-back-auth/internal/dtos"
    "github.com/poofdev/poof-back-auth/internal/services"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-repositories"
    "github.com/poofdev/poof-back-utils"
    auth_utils "github.com/poofdev/poof-back-auth/internal/utils"
)

type WorkerController struct {
    workerAuthService services.WorkerAuthService
    workerRepo        repositories.WorkerRepository
}

func NewWorkerController(
    workerAuth services.WorkerAuthService,
    workerRepo repositories.WorkerRepository,
) *WorkerController {
    return &WorkerController{
        workerAuthService: workerAuth,
        workerRepo:        workerRepo,
    }
}

var workerValidate = validator.New()

// ----------- Registration -----------
func (c *WorkerController) RegisterWorker(w http.ResponseWriter, r *http.Request) {
    var req dtos.RegisterWorkerRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid payload", err)
        return
    }

    if err := workerValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    worker := &models.Worker{
        Email:       req.Email,
        PhoneNumber: req.PhoneNumber,
        AuthProvider: "sms_2fa", // or "email_verification" 
        TOTPSecret:  req.TOTPSecret,
        FirstName:   req.FirstName,
        LastName:    req.LastName,
        StreetAddress: req.StreetAddress,
        AptSuite:    req.AptSuite,
        City:        req.City,
        State:       req.State,
        ZipCode:     req.ZipCode,
        VehicleYear: req.VehicleYear,
        VehicleMake: req.VehicleMake,
        VehicleModel: req.VehicleModel,
        PersonaVerificationInquiryID: req.PersonaInquiryID,
    }

    if !auth_utils.ValidateTOTPCode(worker.TOTPSecret, req.TOTPToken) {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid TOTP code", nil)
        return
    }

    err := c.workerAuthService.Register(r.Context(), worker)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Failed to register Worker", err)
        return
    }

    utils.RespondWithJSON(w, http.StatusCreated, map[string]string{"message": "Worker registered successfully"})
}

// ----------- Login -----------
func (c *WorkerController) LoginWorker(w http.ResponseWriter, r *http.Request) {
    var req dtos.LoginWorkerRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request", err)
        return
    }
    if err := workerValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    ip := utils.GetIPAddress(r)
    worker, accessToken, refreshToken, err := c.workerAuthService.Login(
        r.Context(),
        req.PhoneNumber,
        req.TOTPCode,
        ip,
    )
    if err != nil {
        utils.RespondWithError(w, http.StatusUnauthorized, "Login failed", err)
        return
    }

    utils.RespondWithJSON(w, http.StatusOK, map[string]interface{}{
        "worker":        worker,
        "access_token":  accessToken,
        "refresh_token": refreshToken,
    })
}

// ----------- Check Email -----------
func (c *WorkerController) CheckWorkerEmailExists(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Email string `json:"email" validate:"required,email"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }
    if err := workerValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    ctx := r.Context()
    wkr, err := c.workerRepo.GetByEmail(ctx, req.Email)
    if err != nil {
        utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": false})
        return
    }
    if wkr != nil {
        utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": true})
        return
    }
    utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": false})
}

// ----------- Check Phone -----------
func (c *WorkerController) CheckWorkerPhoneNumberExists(w http.ResponseWriter, r *http.Request) {
    var req struct {
        PhoneNumber string `json:"phone_number" validate:"required"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }
    if err := workerValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    ctx := r.Context()
    wkr, err := c.workerRepo.GetByPhoneNumber(ctx, req.PhoneNumber)
    if err == nil && wkr != nil {
        utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": true})
        return
    }
    utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": false})
}

// ----------- Refresh Token -----------
func (c *WorkerController) RefreshTokenWorker(w http.ResponseWriter, r *http.Request) {
    var req struct {
        RefreshToken string `json:"refresh_token" validate:"required,len=64"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid payload", err)
        return
    }
    if err := workerValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    ip := utils.GetIPAddress(r)
    accessToken, newRefresh, err := c.workerAuthService.RefreshToken(r.Context(), req.RefreshToken, ip)
    if err != nil {
        utils.Logger.WithError(err).Error("Worker refresh token error")
        utils.RespondWithError(w, http.StatusUnauthorized, "Refresh token failed", err)
        return
    }

    resp := map[string]string{
        "access_token":  accessToken,
        "refresh_token": newRefresh,
    }
    utils.RespondWithJSON(w, http.StatusOK, resp)
}

// ----------- Logout -----------
func (c *WorkerController) LogoutWorker(w http.ResponseWriter, r *http.Request) {
    var req struct {
        RefreshToken string `json:"refresh_token" validate:"required,len=64"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid payload", err)
        return
    }
    if err := workerValidate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    err := c.workerAuthService.Logout(r.Context(), req.RefreshToken)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Failed to logout", err)
        return
    }
    utils.RespondWithJSON(w, http.StatusOK, map[string]string{"message": "Logged out successfully"})
}



## internal/controllers/verification_controller.go

package controllers

import (
    "encoding/json"
    "net/http"

    "github.com/poofdev/poof-back-auth/internal/services"
    "github.com/poofdev/poof-back-utils"
)

type VerificationController struct {
    smsService   services.SMSService
    emailService services.EmailService
}

func NewVerificationController(smsService services.SMSService, emailService services.EmailService) *VerificationController {
    return &VerificationController{
        smsService:   smsService,
        emailService: emailService,
    }
}

// RequestSMSCode handles POST /verify/request_sms_code
func (c *VerificationController) RequestSMSCode(w http.ResponseWriter, r *http.Request) {
    var req struct {
        PhoneNumber string `json:"phone_number" validate:"required,e164"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid phone number format", err)
        return
    }

    ctx := r.Context()
    err := c.smsService.GenerateAndSendCode(ctx, req.PhoneNumber)
    if err != nil {
        // If there's a phone number format or Twilio lookup error, 
        // the service returns it here; we can respond with 400.
        utils.Logger.WithError(err).Error("Failed to send SMS code")
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid or unreachable phone number", err)
        return
    }
    utils.RespondWithJSON(w, http.StatusOK, map[string]string{"message": "Verification code sent"})
}

// VerifySMSCode handles POST /verify/check_sms_code
func (c *VerificationController) VerifySMSCode(w http.ResponseWriter, r *http.Request) {
    var req struct {
        PhoneNumber string `json:"phone_number" validate:"required,e164"`
        Code        string `json:"code" validate:"required,len=6,numeric"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid phone number or code format", err)
        return
    }

    ctx := r.Context()
    valid, err := c.smsService.VerifyCode(ctx, req.PhoneNumber, req.Code)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to verify SMS code")
        utils.RespondWithError(w, http.StatusInternalServerError, "Verification failed", err)
        return
    }

    if !valid {
        utils.RespondWithError(w, http.StatusUnauthorized, "Invalid or expired verification code", nil)
        return
    }

    utils.RespondWithJSON(w, http.StatusOK, map[string]string{"message": "Phone number verified"})
}

// RequestEmailCode handles POST /verify/request_email_code
func (c *VerificationController) RequestEmailCode(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Email string `json:"email" validate:"required,email"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid email format", err)
        return
    }

    ctx := r.Context()
    err := c.emailService.GenerateAndSendCode(ctx, req.Email)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to send email code")
        utils.RespondWithError(w, http.StatusInternalServerError, "Failed to send email code", err)
        return
    }

    utils.RespondWithJSON(w, http.StatusOK, map[string]string{"message": "Verification code sent"})
}

// VerifyEmailCode handles POST /verify/check_email_code
func (c *VerificationController) VerifyEmailCode(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Email string `json:"email" validate:"required,email"`
        Code  string `json:"code" validate:"required,len=6,numeric"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid email or code format", err)
        return
    }

    ctx := r.Context()
    valid, err := c.emailService.VerifyCode(ctx, req.Email, req.Code)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to verify email code")
        utils.RespondWithError(w, http.StatusInternalServerError, "Verification failed", err)
        return
    }

    if !valid {
        utils.RespondWithError(w, http.StatusUnauthorized, "Invalid or expired verification code", nil)
        return
    }

    utils.RespondWithJSON(w, http.StatusOK, map[string]string{"message": "Email verified"})
}


## internal/controllers/registration_controller.go

package controllers

import (
	"net/http"

	"github.com/poofdev/poof-back-auth/config"
	auth_utils "github.com/poofdev/poof-back-auth/internal/utils"
	"github.com/poofdev/poof-back-utils"
)

type RegistrationController struct {
}

func NewRegistrationController() *RegistrationController {
    return &RegistrationController{
    }
}

func (c *RegistrationController) GenerateTOTPSecret(w http.ResponseWriter, r *http.Request) {
    // Generate a new TOTP secret
    appName := config.LoadConfig().AppName
    accountName := appName + " User"

    secret, err := auth_utils.GenerateTOTPSecret(appName, accountName)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Failed to generate TOTP secret", err)
        return
    }

    // Generate QR code
    qrCode, err := auth_utils.GenerateTOTPQRCode(secret, appName, accountName)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Failed to generate QR code", err)
        return
    }

    // Return the secret and QR code to the frontend
    response := map[string]string{
        "secret":  secret,
        "qr_code": qrCode, // Base64-encoded PNG image
    }
    utils.RespondWithJSON(w, http.StatusOK, response)
}


## internal/services/sms_service.go

package services

import (
    "context"
    "crypto/rand"
    "errors"
    "fmt"
    "math/big"
    "regexp"
    "time"

    "github.com/twilio/twilio-go"
    api "github.com/twilio/twilio-go/rest/api/v2010"
    lookups "github.com/twilio/twilio-go/rest/lookups/v2"

    "github.com/poofdev/poof-back-auth/config"
    "github.com/poofdev/poof-back-repositories"
)

// 1) Twilio client interface (sending SMS)
type TwilioClientInterface interface {
    CreateMessage(params *api.CreateMessageParams) (*api.ApiV2010Message, error)
}

type twilioClientWrapper struct {
    client *twilio.RestClient
}

func (t *twilioClientWrapper) CreateMessage(params *api.CreateMessageParams) (*api.ApiV2010Message, error) {
    return t.client.Api.CreateMessage(params)
}

type TwilioLookupClientInterface interface {
    FetchPhoneNumber(ctx context.Context, phoneNumber string) (*lookups.LookupsV2PhoneNumber, error)
}

type twilioLookupClientWrapper struct {
    client *twilio.RestClient
}

func (l *twilioLookupClientWrapper) FetchPhoneNumber(
    ctx context.Context,
    phoneNumber string,
) (*lookups.LookupsV2PhoneNumber, error) {

    params := &lookups.FetchPhoneNumberParams{
        // Fields: "line_type_intelligence" etc. if you want paid data
    }
    return l.client.LookupsV2.FetchPhoneNumber(phoneNumber, params)
}

// E.164 regex for a basic local format check
var e164Regex = regexp.MustCompile(`^\+?[1-9]\d{1,14}$`)

// SMSService interface, unchanged
type SMSService interface {
    GenerateAndSendCode(ctx context.Context, phoneNumber string) error
    VerifyCode(ctx context.Context, phoneNumber, code string) (bool, error)
}

// smsService struct
type smsService struct {
    client       TwilioClientInterface
    lookupClient TwilioLookupClientInterface
    fromPhone    string
    repo         repositories.SMSRepository
}

// Constructor: supply both the messaging client and lookup client
func NewSMSService(cfg *config.Config, repo repositories.SMSRepository) SMSService {
    // Reuse the same underlying Twilio RestClient for both
    twilioRestClient := twilio.NewRestClientWithParams(twilio.ClientParams{
        Username: cfg.TwilioAccountSID,
        Password: cfg.TwilioAuthToken,
    })

    messageClient := &twilioClientWrapper{
        client: twilioRestClient,
    }
    lookupClient := &twilioLookupClientWrapper{
        client: twilioRestClient,
    }

    return &smsService{
        client:       messageClient,
        lookupClient: lookupClient,
        fromPhone:    cfg.TwilioFromPhone,
        repo:         repo,
    }
}

// 1) Validate phone number locally + Twilio Lookup, then send SMS
func (s *smsService) GenerateAndSendCode(ctx context.Context, phoneNumber string) error {
    // 1. Local E.164 format check
    if !e164Regex.MatchString(phoneNumber) {
        return errors.New("phone number is not in valid E.164 format (local check)")
    }

    // 2. Twilio Lookup v2 call
    lookupResp, err := s.lookupClient.FetchPhoneNumber(ctx, phoneNumber)
    if err != nil {
        return fmt.Errorf("twilio lookup error: %w", err)
    }

    // 3. Check if valid
    if lookupResp.Valid == nil || !(*lookupResp.Valid) {
        return errors.New("phone number is invalid according to Twilio Lookup")
    }

    // 4. Use Twilio’s normalized E.164 number if available
    normalizedNumber := phoneNumber
    if lookupResp.PhoneNumber != nil {
        normalizedNumber = *lookupResp.PhoneNumber
    }

    // 5. Generate code, store in DB
    code, err := generateVerificationCode(6)
    if err != nil {
        return err
    }
    expiresAt := time.Now().Add(5 * time.Minute)
    if err := s.repo.CreateCode(ctx, normalizedNumber, code, expiresAt); err != nil {
        return err
    }

    // 6. Send the SMS
    params := &api.CreateMessageParams{}
    params.SetTo(normalizedNumber)
    params.SetFrom(s.fromPhone)
    params.SetBody(fmt.Sprintf("Your verification code is %s", code))

    if _, err := s.client.CreateMessage(params); err != nil {
        return err
    }

    return nil
}

// 2) Verify the code
func (s *smsService) VerifyCode(ctx context.Context, phoneNumber, code string) (bool, error) {
    // For consistency, you could do a Twilio Lookup again or at least do a local
    // E.164 check to see if phoneNumber is well-formed and normalized similarly.
    // We'll keep it simple here:
    smsCode, err := s.repo.GetCode(ctx, phoneNumber)
    if err != nil {
        return false, err
    }

    // Compare codes, check expiration
    if smsCode.VerificationCode != code || time.Now().After(smsCode.ExpiresAt) {
        if incErr := s.repo.IncrementAttempts(ctx, smsCode.ID); incErr != nil {
            return false, incErr
        }
        return false, nil
    }

    // Delete code on success
    if delErr := s.repo.DeleteCode(ctx, smsCode.ID); delErr != nil {
        return false, delErr
    }
    return true, nil
}

// Helper function for generating numeric codes
func generateVerificationCode(length int) (string, error) {
    const digits = "0123456789"
    code := make([]byte, length)
    for i := 0; i < length; i++ {
        num, err := rand.Int(rand.Reader, big.NewInt(int64(len(digits))))
        if err != nil {
            return "", err
        }
        code[i] = digits[num.Int64()]
    }
    return string(code), nil
}



## internal/services/pm_auth_service.go

package services

import (
    "context"
    "errors"
    "time"

    "github.com/google/uuid"
    "github.com/poofdev/poof-back-auth/config"
    "github.com/poofdev/poof-back-utils"
    auth_utils "github.com/poofdev/poof-back-auth/internal/utils"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-repositories"
)

// PMAuthService defines methods for PM registration, login, refresh, logout, etc.
type PMAuthService interface {
    Register(ctx context.Context, pm *models.PropertyManager) error
    Login(ctx context.Context, email, totpCode, ipAddress string) (*models.PropertyManager, string, string, error)
    RefreshToken(ctx context.Context, refreshTokenString, ipAddress string) (string, string, error)
    Logout(ctx context.Context, refreshTokenString string) error
}

// pmAuthService implements PMAuthService
type pmAuthService struct {
    pmRepo      repositories.PropertyManagerRepository
    pmLoginRepo repositories.PMLoginAttemptsRepository

    // The JWTService referencing the pmTokenRepo
    pmJWTService JWTService

    smsService   SMSService
    emailService EmailService
    cfg          *config.Config
}

// NewPMAuthService constructs an instance of PMAuthService.
// We pass in pmTokenRepo as a "repositories.TokenRepository" so the JWTService
// writes to pm_refresh_tokens, etc.
func NewPMAuthService(
    pmRepo repositories.PropertyManagerRepository,
    pmTokenRepo repositories.TokenRepository,
    pmLoginRepo repositories.PMLoginAttemptsRepository,
    smsService SMSService,
    emailService EmailService,
    cfg *config.Config,
) PMAuthService {
    // Create a JWT service specifically for PM, referencing pmTokenRepo
    pmJWTService := NewJWTService(cfg, pmTokenRepo)

    return &pmAuthService{
        pmRepo:       pmRepo,
        pmLoginRepo:  pmLoginRepo,
        pmJWTService: pmJWTService,
        smsService:   smsService,
        emailService: emailService,
        cfg:          cfg,
    }
}

// ------------------- Register -------------------
func (s *pmAuthService) Register(ctx context.Context, pm *models.PropertyManager) error {
    pm.ID = uuid.New()
    pm.IsActive = true
    return s.pmRepo.Create(ctx, pm)
}

// ------------------- Login -------------------
func (s *pmAuthService) Login(
    ctx context.Context,
    email string,
    totpCode string,
    ipAddress string,
) (*models.PropertyManager, string, string, error) {

    // 1) Lookup PM by email
    pm, err := s.pmRepo.GetByEmail(ctx, email)
    if err != nil || pm == nil {
        return nil, "", "", errors.New("invalid credentials")
    }

    // 2) Check if locked
    locked, lockedUntil, err := s.pmLoginRepo.IsLocked(ctx, pm.ID)
    if err == nil && locked {
        return nil, "", "", errors.New("PM account locked until " + lockedUntil.Format(time.RFC3339))
    }

    // 3) Validate TOTP
    if !auth_utils.ValidateTOTPCode(pm.TOTPSecret, totpCode) {
        _ = s.pmLoginRepo.Increment(ctx, pm.ID, s.cfg.LockDuration, s.cfg.AttemptWindow, s.cfg.MaxLoginAttempts)
        return nil, "", "", errors.New("invalid credentials")
    }
    _ = s.pmLoginRepo.Reset(ctx, pm.ID)

    // 4) Generate Access Token
    accessToken, err := s.pmJWTService.GenerateAccessToken(pm.ID, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate PM access token")
        return nil, "", "", errors.New("token generation failed")
    }

    // 5) Generate Refresh Token
    refreshToken, err := s.pmJWTService.GenerateRefreshToken(pm.ID, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate PM refresh token")
        return nil, "", "", errors.New("token generation failed")
    }

    return pm, accessToken, refreshToken.Token, nil
}

// ------------------- RefreshToken -------------------
func (s *pmAuthService) RefreshToken(
    ctx context.Context,
    refreshTokenString string,
    ipAddress string,
) (string, string, error) {

    // Delegates to pmJWTService which references pmTokenRepo
    newAccess, newRefresh, err := s.pmJWTService.RefreshToken(ctx, refreshTokenString, ipAddress)
    if err != nil {
        return "", "", err
    }
    return newAccess, newRefresh, nil
}

// ------------------- Logout -------------------
func (s *pmAuthService) Logout(
    ctx context.Context,
    refreshTokenString string,
) error {
    // pmJWTService will revoke the old token in pm_refresh_tokens
    return s.pmJWTService.Logout(ctx, refreshTokenString)
}



## internal/services/worker_auth_service.go

package services

import (
    "context"
    "errors"
    "time"

    "github.com/google/uuid"
    "github.com/poofdev/poof-back-auth/config"
    auth_utils "github.com/poofdev/poof-back-auth/internal/utils"
    "github.com/poofdev/poof-back-utils"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-repositories"
)

// WorkerAuthService for registering, logging in, refreshing tokens, etc., for Workers
type WorkerAuthService interface {
    Register(ctx context.Context, w *models.Worker) error
    Login(ctx context.Context, phoneNumber, totpCode, ipAddress string) (*models.Worker, string, string, error)
    RefreshToken(ctx context.Context, refreshTokenString, ipAddress string) (string, string, error)
    Logout(ctx context.Context, refreshTokenString string) error
}

type workerAuthService struct {
    workerRepo      repositories.WorkerRepository
    workerLoginRepo repositories.WorkerLoginAttemptsRepository

    // The JWTService referencing the workerTokenRepo
    workerJWTService JWTService

    smsService   SMSService
    emailService EmailService
    cfg          *config.Config
}

func NewWorkerAuthService(
    workerRepo repositories.WorkerRepository,
    workerTokenRepo repositories.TokenRepository,
    workerLoginRepo repositories.WorkerLoginAttemptsRepository,
    smsService SMSService,
    emailService EmailService,
    cfg *config.Config,
) WorkerAuthService {
    // Create a JWT service specifically for Worker, referencing workerTokenRepo
    workerJWTService := NewJWTService(cfg, workerTokenRepo)

    return &workerAuthService{
        workerRepo:       workerRepo,
        workerLoginRepo:  workerLoginRepo,
        workerJWTService: workerJWTService,
        smsService:       smsService,
        emailService:     emailService,
        cfg:              cfg,
    }
}

// ------------------- Register -------------------
func (s *workerAuthService) Register(ctx context.Context, w *models.Worker) error {
    w.ID = uuid.New()
    w.IsActive = true
    return s.workerRepo.Create(ctx, w)
}

// ------------------- Login -------------------
func (s *workerAuthService) Login(
    ctx context.Context,
    phoneNumber string,
    totpCode string,
    ipAddress string,
) (*models.Worker, string, string, error) {

    // 1) Lookup worker
    worker, err := s.workerRepo.GetByPhoneNumber(ctx, phoneNumber)
    if err != nil || worker == nil {
        return nil, "", "", errors.New("invalid credentials")
    }

    // 2) Check if locked
    locked, lockedUntil, err := s.workerLoginRepo.IsLocked(ctx, worker.ID)
    if err == nil && locked {
        return nil, "", "", errors.New("Worker account locked until " + lockedUntil.Format(time.RFC3339))
    }

    // 3) Validate TOTP
    if !auth_utils.ValidateTOTPCode(worker.TOTPSecret, totpCode) {
        _ = s.workerLoginRepo.Increment(ctx, worker.ID, s.cfg.LockDuration, s.cfg.AttemptWindow, s.cfg.MaxLoginAttempts)
        return nil, "", "", errors.New("invalid credentials")
    }
    _ = s.workerLoginRepo.Reset(ctx, worker.ID)

    // 4) Generate Access Token
    accessToken, err := s.workerJWTService.GenerateAccessToken(worker.ID, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate worker access token")
        return nil, "", "", errors.New("token generation failed")
    }

    // 5) Generate Refresh Token
    refreshToken, err := s.workerJWTService.GenerateRefreshToken(worker.ID, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate worker refresh token")
        return nil, "", "", errors.New("token generation failed")
    }

    return worker, accessToken, refreshToken.Token, nil
}

// ------------------- RefreshToken -------------------
func (s *workerAuthService) RefreshToken(
    ctx context.Context,
    refreshTokenString string,
    ipAddress string,
) (string, string, error) {

    // workerJWTService handles reading/writing worker_refresh_tokens
    newAccess, newRefresh, err := s.workerJWTService.RefreshToken(ctx, refreshTokenString, ipAddress)
    if err != nil {
        return "", "", err
    }
    return newAccess, newRefresh, nil
}

// ------------------- Logout -------------------
func (s *workerAuthService) Logout(
    ctx context.Context,
    refreshTokenString string,
) error {
    return s.workerJWTService.Logout(ctx, refreshTokenString)
}



## internal/services/password_service.go

package services

import (
    "golang.org/x/crypto/bcrypt"
)

type PasswordService interface {
    HashPassword(password string) (string, error)
    VerifyPassword(hashedPassword, password string) error
}

type passwordService struct{}

func NewPasswordService() PasswordService {
    return &passwordService{}
}

func (p *passwordService) HashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
    return string(bytes), err
}

func (p *passwordService) VerifyPassword(hashedPassword, password string) error {
    return bcrypt.CompareHashAndPassword([]byte(hashedPassword), []byte(password))
}


## internal/services/email_service.go

package services

import (
    "context"
    "fmt"
    "github.com/sendgrid/sendgrid-go"
    "github.com/sendgrid/sendgrid-go/helpers/mail"
    "github.com/poofdev/poof-back-auth/config"
    "github.com/poofdev/poof-back-repositories"
    "time"
)

type EmailService interface {
    GenerateAndSendCode(ctx context.Context, email string) error
    VerifyCode(ctx context.Context, email, code string) (bool, error)
}

type emailService struct {
    client  *sendgrid.Client
    fromEmail string
    repo    repositories.EmailRepository
}

func NewEmailService(cfg *config.Config, repo repositories.EmailRepository) EmailService {
    client := sendgrid.NewSendClient(cfg.SendGridAPIKey)
    return &emailService{
        client:    client,
        fromEmail: cfg.SendGridFromEmail,
        repo:      repo,
    }
}

func (e *emailService) GenerateAndSendCode(ctx context.Context, email string) error {
    code, err := generateVerificationCode(6)
    if err != nil {
        return err
    }
    expiresAt := time.Now().Add(5 * time.Minute)

    // Store the code in the database
    err = e.repo.CreateCode(ctx, email, code, expiresAt)
    if err != nil {
        return err
    }

    // Send Email via SendGrid
    from := mail.NewEmail("Your App Name", e.fromEmail)
    to := mail.NewEmail("", email)
    subject := "Your Verification Code"
    content := fmt.Sprintf("Your verification code is %s", code)
    message := mail.NewSingleEmail(from, subject, to, content, content)
    _, err = e.client.Send(message)
    if err != nil {
        return err
    }

    return nil
}

func (e *emailService) VerifyCode(ctx context.Context, email, code string) (bool, error) {
    emailCode, err := e.repo.GetCode(ctx, email)
    if err != nil {
        return false, err
    }

    if emailCode.VerificationCode != code || time.Now().After(emailCode.ExpiresAt) {
        // Increment attempts
        err = e.repo.IncrementAttempts(ctx, emailCode.ID)
        if err != nil {
            return false, err
        }
        return false, nil
    }

    // Delete the code after successful verification
    err = e.repo.DeleteCode(ctx, emailCode.ID)
    if err != nil {
        return false, err
    }

    return true, nil
}


## internal/services/jwt_service.go

package services

import (
    "context"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "encoding/base64"
    "errors"
    "math/big"
    "net"
    "time"

    "github.com/golang-jwt/jwt/v5"
    "github.com/google/uuid"
    "github.com/poofdev/poof-back-auth/config"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-repositories"
    "github.com/poofdev/poof-back-utils"
)

// JWTService is used by each role-specific AuthService (PM or Worker)
// to generate access/refresh tokens, refresh them, and handle logout.
type JWTService interface {
    GenerateAccessToken(subjectID uuid.UUID, ipAddress string) (string, error)
    GenerateRefreshToken(subjectID uuid.UUID, ipAddress string) (*models.RefreshToken, error)
    RefreshToken(ctx context.Context, refreshTokenString, ipAddress string) (string, string, error)
    Logout(ctx context.Context, refreshTokenString string) error
}

// jwtService implements JWTService for a given tokenRepo (PM or Worker).
type jwtService struct {
    privateKey    *rsa.PrivateKey
    publicKey     *rsa.PublicKey
    tokenExpiry   time.Duration
    refreshExpiry time.Duration
    tokenRepo     repositories.TokenRepository // e.g. pmTokenRepo or workerTokenRepo
}

func NewJWTService(cfg *config.Config, tokenRepo repositories.TokenRepository) JWTService {
    return &jwtService{
        privateKey:    cfg.RSAPrivateKey,
        publicKey:     cfg.RSAPublicKey,
        tokenExpiry:   cfg.TokenExpiry,
        refreshExpiry: cfg.RefreshTokenExpiry,
        tokenRepo:     tokenRepo,
    }
}

// -------------------------------------------------------------------
// 1) Generate Access Token
// -------------------------------------------------------------------
func (j *jwtService) GenerateAccessToken(
    subjectID uuid.UUID,
    ipAddress string,
) (string, error) {

    // Validate IP
    parsedIP := net.ParseIP(ipAddress)
    if parsedIP == nil {
        return "", errors.New("invalid IP address")
    }

    tokenID := uuid.New().String()
    claims := jwt.MapClaims{
        "iss":  "poof-service",
        "sub":  subjectID.String(),
        "exp":  time.Now().Add(j.tokenExpiry).Unix(),
        "iat":  time.Now().Unix(),
        "jti":  tokenID,
        "ip":   parsedIP.String(), // IP claim, if you want to verify in middleware
    }

    token := jwt.NewWithClaims(jwt.SigningMethodRS256, claims)
    signedToken, err := token.SignedString(j.privateKey)
    if err != nil {
        return "", err
    }
    return signedToken, nil
}

// -------------------------------------------------------------------
// 2) Generate Refresh Token
// -------------------------------------------------------------------
func (j *jwtService) GenerateRefreshToken(
    subjectID uuid.UUID,
    ipAddress string,
) (*models.RefreshToken, error) {

    if j.tokenRepo == nil {
        return nil, errors.New("jwtService has nil tokenRepo")
    }

    rawToken := generateSecureToken(64) // random 64-char token for client
    hasher := sha256.New()
    hasher.Write([]byte(rawToken))
    hashedToken := base64.URLEncoding.EncodeToString(hasher.Sum(nil))

    expiresAt := time.Now().Add(j.refreshExpiry)

    // Prepare the DB record with hashed token
    refreshToken := &models.RefreshToken{
        ID:        uuid.New(),
        UserID:    subjectID, // For PM/Worker, your RefreshToken struct calls it "UserID"
        Token:     hashedToken,
        ExpiresAt: expiresAt,
        CreatedAt: time.Now(),
        Revoked:   false,
        IPAddress: ipAddress,
    }

    // Store in DB
    err := j.tokenRepo.CreateRefreshToken(context.Background(), refreshToken)
    if err != nil {
        return nil, err
    }

    // Return a copy containing the raw token for the client
    return &models.RefreshToken{
        ID:        refreshToken.ID,
        UserID:    refreshToken.UserID,
        Token:     rawToken,
        ExpiresAt: refreshToken.ExpiresAt,
        CreatedAt: refreshToken.CreatedAt,
        Revoked:   refreshToken.Revoked,
        IPAddress: ipAddress,
    }, nil
}

// -------------------------------------------------------------------
// 3) Refresh an Existing Token
// -------------------------------------------------------------------
func (j *jwtService) RefreshToken(
    ctx context.Context,
    refreshTokenString string,
    ipAddress string,
) (string, string, error) {

    if j.tokenRepo == nil {
        return "", "", errors.New("jwtService has nil tokenRepo")
    }

    // 1) Retrieve old token from DB
    oldToken, err := j.tokenRepo.GetRefreshToken(ctx, refreshTokenString)
    if err != nil || oldToken == nil || oldToken.Revoked {
        utils.Logger.WithError(err).Error("invalid or missing refresh token in jwtService.RefreshToken")
        return "", "", errors.New("invalid refresh token")
    }

    // 2) Check expiration
    if oldToken.IsExpired() {
        utils.Logger.Error("refresh token expired in jwtService.RefreshToken")
        return "", "", errors.New("refresh token expired")
    }

    // 3) Optional IP check
    if oldToken.IPAddress != "" && oldToken.IPAddress != ipAddress {
        utils.Logger.Error("IP mismatch for refresh token in jwtService.RefreshToken")
        return "", "", errors.New("ip mismatch")
    }

    // 4) Revoke old token
    err = j.tokenRepo.RevokeRefreshToken(ctx, oldToken.ID)
    if err != nil {
        utils.Logger.WithError(err).Error("failed to revoke refresh token in jwtService.RefreshToken")
        return "", "", errors.New("failed to revoke old token")
    }

    // 5) Generate new Access Token
    accessToken, err := j.GenerateAccessToken(oldToken.UserID, ipAddress)
    if err != nil {
        return "", "", err
    }

    // 6) Generate new Refresh Token
    newRT, err := j.GenerateRefreshToken(oldToken.UserID, ipAddress)
    if err != nil {
        return "", "", err
    }

    return accessToken, newRT.Token, nil
}

// -------------------------------------------------------------------
// 4) Logout (Revoke) a Refresh Token
// -------------------------------------------------------------------
func (j *jwtService) Logout(
    ctx context.Context,
    refreshTokenString string,
) error {

    if j.tokenRepo == nil {
        return errors.New("jwtService has nil tokenRepo")
    }

    oldToken, err := j.tokenRepo.GetRefreshToken(ctx, refreshTokenString)
    if err != nil {
        utils.Logger.WithError(err).Error("logout fetch refresh token error in jwtService")
        return errors.New("logout server error")
    }
    if oldToken == nil {
        // Already not found => do nothing
        return nil
    }

    // Revoke
    err = j.tokenRepo.RevokeRefreshToken(ctx, oldToken.ID)
    if err != nil {
        utils.Logger.WithError(err).Error("failed to revoke token in jwtService.Logout")
        return errors.New("logout server error")
    }

    return nil
}

// -------------------------------------------------------------------
// Utility: Secure random string generator
// -------------------------------------------------------------------
func generateSecureToken(length int) string {
    const charset = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
    b := make([]byte, length)
    for i := range b {
        b[i] = charset[secureRandomInt(len(charset))]
    }
    return string(b)
}

func secureRandomInt(max int) int {
    n, err := rand.Int(rand.Reader, big.NewInt(int64(max)))
    if err != nil {
        panic(err) // handle gracefully in real code
    }
    return int(n.Int64())
}



## internal/app/app.go

package app

import (
    "context"
    "github.com/jackc/pgx/v4/pgxpool"
    "github.com/poofdev/poof-back-auth/config"
    "github.com/poofdev/poof-back-utils"
)

type App struct {
    Config *config.Config
    DB     *pgxpool.Pool
}

func NewApp(cfg *config.Config) (*App, error) {
    ctx := context.Background()

    dbpool, err := pgxpool.Connect(ctx, cfg.DatabaseURL)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to connect to database")
        return nil, err
    }

    return &App{
        Config: cfg,
        DB:     dbpool,
    }, nil
}

func (a *App) Close() {
    a.DB.Close()
}


## internal/dtos/register_worker_request.go

package dtos

type RegisterWorkerRequest struct {
    FirstName         string  `json:"first_name" validate:"required,min=1,max=100"`
    LastName          string  `json:"last_name" validate:"required,min=1,max=100"`
    Email             string  `json:"email" validate:"required,email"`
    PhoneNumber       string  `json:"phone_number" validate:"required"`
    PersonaInquiryID  string  `json:"persona_inquiry_id" validate:"required,uuid"`
    StreetAddress     string  `json:"street_address" validate:"required,min=1,max=255"`
    AptSuite          *string `json:"apt_suite,omitempty"`
    City              string  `json:"city" validate:"required,min=1,max=100"`
    State             string  `json:"state" validate:"required,min=1,max=50"`
    ZipCode           string  `json:"zip_code" validate:"required,min=1,max=20"`
    VehicleYear       int     `json:"vehicle_year" validate:"required"`
    VehicleMake       string  `json:"vehicle_make" validate:"required,min=1,max=100"`
    VehicleModel      string  `json:"vehicle_model" validate:"required,min=1,max=100"`
    TOTPSecret        string  `json:"totp_secret" validate:"required"`
    TOTPToken         string  `json:"totp_token" validate:"required"`
}


## internal/dtos/register_pm_request.go

package dtos

type RegisterPMRequest struct {
    FirstName                string  `json:"first_name" validate:"required,min=1,max=100"`
    LastName                 string  `json:"last_name" validate:"required,min=1,max=100"`
    Email                    string  `json:"email" validate:"required,email"`
    PhoneNumber              *string `json:"phone_number,omitempty" validate:"omitempty"`
    PersonaInquiryID         string  `json:"persona_inquiry_id" validate:"required,uuid"`
    BusinessName             string  `json:"business_name" validate:"required,min=1,max=255"`
    BusinessAddress          string  `json:"business_address" validate:"required,min=1,max=255"`
    City                     string  `json:"city" validate:"required,min=1,max=100"`
    State                    string  `json:"state" validate:"required,min=1,max=50"`
    ZipCode                  string  `json:"zip_code" validate:"required,min=1,max=20"`
    TOTPSecret               string  `json:"totp_secret" validate:"required"`
    TOTPToken                string  `json:"totp_token" validate:"required"`
}


## internal/dtos/login_pm_request.go

package dtos

type LoginPMRequest struct {
    Email    string `json:"email" validate:"required,email"`
    TOTPCode string `json:"totp_code" validate:"required,len=6,numeric"`
}


## internal/dtos/login_worker_request.go

package dtos

type LoginWorkerRequest struct {
    PhoneNumber string `json:"phone_number" validate:"required"`
    TOTPCode    string `json:"totp_code" validate:"required,len=6,numeric"`
}


## config/config.go

package config

import (
	"crypto/rsa"
	"encoding/base64"
	"encoding/pem"
	"log"
	"os"
	"time"

	"github.com/golang-jwt/jwt/v5"
)

type Config struct {
	AppName            string
	DatabaseURL        string
	DBEncryptionKey    []byte
	TokenExpiry        time.Duration
	RefreshTokenExpiry time.Duration
	MaxLoginAttempts   int
	AttemptWindow      time.Duration
	LockDuration 	   time.Duration
	TwilioAccountSID   string
	TwilioAuthToken    string
	TwilioFromPhone    string
	RSAPrivateKey      *rsa.PrivateKey
	RSAPublicKey       *rsa.PublicKey
	SendGridAPIKey     string
	SendGridFromEmail  string
}

type OpenIDProviderConfig struct {
	ClientID     string
	ClientSecret string
	RedirectURL  string
	ProviderURL  string
}

func LoadConfig() *Config {
	privateKeyBase64 := os.Getenv("RSA_PRIVATE_KEY_BASE64")
    if privateKeyBase64 == "" {
        log.Fatal("RSA_PRIVATE_KEY_BASE64 not set")
    }
    // 1) Decode from base64
    privateKeyPEM, err := base64.StdEncoding.DecodeString(privateKeyBase64)
    if err != nil {
        log.Fatalf("Failed to decode base64 private key: %v", err)
    }
    // 2) Parse the PEM block
    block, _ := pem.Decode(privateKeyPEM)
    if block == nil {
        log.Fatal("Failed to decode PEM block for private key")
    }
    privateKey, err := jwt.ParseRSAPrivateKeyFromPEM(privateKeyPEM)
    if err != nil {
        log.Fatalf("Failed to parse RSA private key: %v", err)
    }

    publicKeyBase64 := os.Getenv("RSA_PUBLIC_KEY_BASE64")
    if publicKeyBase64 == "" {
        log.Fatal("RSA_PUBLIC_KEY_BASE64 not set")
    }
    publicKeyPEM, err := base64.StdEncoding.DecodeString(publicKeyBase64)
    if err != nil {
        log.Fatalf("Failed to decode base64 public key: %v", err)
    }
    block, _ = pem.Decode(publicKeyPEM)
    if block == nil {
        log.Fatal("Failed to decode PEM block for public key")
    }
    publicKey, err := jwt.ParseRSAPublicKeyFromPEM(publicKeyPEM)
    if err != nil {
        log.Fatalf("Failed to parse RSA public key: %v", err)
    }

	dbEncryptionKey := os.Getenv("DB_ENCRYPTION_KEY")
	if dbEncryptionKey == "" {
		log.Fatal("DBEncryptionKey not set in environment variables")
	}
	decodedKey, err := base64.StdEncoding.DecodeString(dbEncryptionKey)
	if err != nil {
		log.Fatal("Failed to decode DBEncryptionKey:", err)
	}
	if len(decodedKey) != 32 {
		log.Fatal("DBEncryptionKey must be 32 bytes for AES-256 encryption")
	}

	appName := os.Getenv("APP_NAME")
	if appName == "" {
		appName = "Poof"
	}

	// Return the complete configuration
	return &Config{
		AppName:            appName,
		DatabaseURL:        os.Getenv("DATABASE_URL"),
		DBEncryptionKey:  decodedKey,
		TokenExpiry:        10 * time.Minute,
		RefreshTokenExpiry: 7 * 24 * time.Hour,
		MaxLoginAttempts: 10,
		AttemptWindow: 5 * time.Minute,
		LockDuration: 10 * time.Minute,
		TwilioAccountSID: os.Getenv("TWILIO_ACCOUNT_SID"),
		TwilioAuthToken:  os.Getenv("TWILIO_AUTH_TOKEN"),
		TwilioFromPhone:  os.Getenv("TWILIO_FROM_PHONE"),
		RSAPrivateKey:    privateKey,
		RSAPublicKey:     publicKey,
		SendGridAPIKey:   os.Getenv("SENDGRID_API_KEY"),
		SendGridFromEmail: os.Getenv("SENDGRID_FROM_EMAIL"),
	}
}


## scripts/migrate.sh

#!/bin/bash
set -e

# 1) Ensure DATABASE_URL is set
if [ -z "$DATABASE_URL" ]; then
  echo "[ERROR] DATABASE_URL is not set. Please export a valid Postgres connection string."
  exit 1
fi

# 2) Check that it looks like a PostgreSQL URL
#    (Naive check: just ensures it starts with postgres:// or postgresql://)
if ! [[ "$DATABASE_URL" =~ ^postgres(ql)?:// ]]; then
  echo "[ERROR] DATABASE_URL doesn’t appear to be a valid PostgreSQL address: $DATABASE_URL"
  exit 1
fi

# 3) Run migrate with the provided arguments ($@)
migrate -database "$DATABASE_URL" -path ./migrations "$@"



## migrations/README.md

# Poof Migrations for use as submodule, early stage SOA, single db.


## migrations/.git

gitdir: ../../.git/modules/poof-back-auth/modules/migrations


--------------------------------------------------------
#### Middleware Package ######

## jwt.go

package middleware

import (
    "context"
    "errors"
    "time"
    "crypto/rsa"

    "github.com/golang-jwt/jwt/v5"
)

func ValidateToken(ctx context.Context, tokenString, ipAddress string, publicKey *rsa.PublicKey) (*jwt.Token, error) {
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodRSA); !ok {
            return nil, errors.New("unexpected signing method")
        }
        return publicKey, nil
    })
    if err != nil {
        return nil, err
    }

    claims, ok := token.Claims.(jwt.MapClaims)
    if !ok || !token.Valid {
        return nil, errors.New("invalid token claims")
    }

    // Validate expiration
    if exp, ok := claims["exp"].(float64); ok {
        if time.Unix(int64(exp), 0).Before(time.Now()) {
            return nil, jwt.ErrTokenExpired
        }
    } else {
        return nil, errors.New("missing expiration claim")
    }

    // Validate issuer
    if iss, ok := claims["iss"].(string); ok {
        if iss != "poof-service" { // Replace with your issuer name
            return nil, errors.New("invalid token issuer")
        }
    } else {
        return nil, errors.New("missing issuer claim")
    }

    // Validate IP address
    if ipClaim, ok := claims["ip"].(string); ok {
        if ipClaim != ipAddress {
            return nil, errors.New("IP address mismatch")
        }
    } else {
        return nil, errors.New("missing IP claim")
    }

    return token, nil
}


## auth_middleware.go

package middleware

import (
	"context"
	"crypto/rsa"
	"errors"
	"net/http"
	"strings"

	"github.com/golang-jwt/jwt/v5"
	"github.com/poofdev/poof-back-utils"
)

type contextKey string

const ContextKeyUserID = contextKey("userID")

func AuthMiddleware(publicKey *rsa.PublicKey) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            authHeader := r.Header.Get("Authorization")
            if authHeader == "" {
                utils.RespondWithError(w, http.StatusUnauthorized, "Missing Authorization header", nil)
                return
            }

            tokenString := strings.TrimPrefix(authHeader, "Bearer ")

            // Pass IP address to token validation
            ipAddress := utils.GetIPAddress(r)
            ctx := r.Context()
            token, err := ValidateToken(ctx, tokenString, ipAddress, publicKey)
            if err != nil || !token.Valid {
                if errors.Is(err, jwt.ErrTokenExpired) {
                    utils.RespondWithError(w, http.StatusUnauthorized, "Token expired", errors.New("token_expired"))
                    return
                }
                utils.RespondWithError(w, http.StatusUnauthorized, "Invalid or expired token", err)
                return
            }

            claims, ok := token.Claims.(jwt.MapClaims)
            if !ok {
                utils.RespondWithError(w, http.StatusUnauthorized, "Invalid token claims", nil)
                return
            }

            userID, ok := claims["sub"].(string)
            if !ok {
                utils.RespondWithError(w, http.StatusUnauthorized, "Invalid token claims", nil)
                return
            }

            // Add user ID to context
            ctx = context.WithValue(ctx, ContextKeyUserID, userID)
            r = r.WithContext(ctx)

            next.ServeHTTP(w, r)
        })
    }
}

------------------------------------------------------------------------

### Models Package ###

## trash_compactor.go

package models

import "github.com/google/uuid"

type TrashCompactor struct {
    ID         uuid.UUID `json:"id"`
    PropertyID uuid.UUID `json:"property_id"`
    Latitude   float64   `json:"latitude"`
    Longitude  float64   `json:"longitude"`
}


## sms_code.go

package models

import (
    "time"

    "github.com/google/uuid"
)

// SMSVerificationCode represents an SMS verification code sent to a user.
type SMSVerificationCode struct {
    ID              uuid.UUID `json:"id"`
    PhoneNumber     string    `json:"phone_number"`
    VerificationCode string   `json:"verification_code"`
    ExpiresAt       time.Time `json:"expires_at"`
    CreatedAt       time.Time `json:"created_at"`
    Attempts        int       `json:"attempts"`
}


## worker.go

package models

import (
    "time"
    "github.com/google/uuid"
)

type Worker struct {
    ID                           uuid.UUID `json:"id"`
    Email                        string    `json:"email"`
    PhoneNumber                  string    `json:"phone_number"`
    AuthProvider                 string    `json:"auth_provider"`
    TOTPSecret                   string    `json:"totp_secret,omitempty"`
    IsActive                     bool      `json:"is_active"`
    FirstName                    string    `json:"first_name"`
    LastName                     string    `json:"last_name"`
    StreetAddress                string    `json:"street_address"`
    AptSuite                     *string   `json:"apt_suite,omitempty"`
    City                         string    `json:"city"`
    State                        string    `json:"state"`
    ZipCode                      string    `json:"zip_code"`
    VehicleYear                  int       `json:"vehicle_year"`
    VehicleMake                  string    `json:"vehicle_make"`
    VehicleModel                 string    `json:"vehicle_model"`
    PersonaVerificationInquiryID string    `json:"persona_inquiry_id"`
    CreatedAt                    time.Time `json:"created_at"`
    UpdatedAt                    time.Time `json:"updated_at"`
}


## property.go

package models

import (
    "time"
    "github.com/google/uuid"
)

type Property struct {
    ID           uuid.UUID         `json:"id"`
    ManagerID    uuid.UUID         `json:"manager_id"`
    PropertyName string            `json:"property_name"`
    Address      string            `json:"address"`
    City         string            `json:"city"`
    State        string            `json:"state"`
    ZipCode      string            `json:"zip_code"`
    Buildings    []PropertyBuilding `json:"buildings,omitempty"`
    Units        []Unit            `json:"units,omitempty"`
    Compactors   []TrashCompactor  `json:"compactors,omitempty"`
    CreatedAt    time.Time         `json:"created_at"`
}


## property_manager.go

package models

import (
    "time"
    "github.com/google/uuid"
)

type PropertyManager struct {
    ID                           uuid.UUID  `json:"id"`
    Email                        string     `json:"email"`
    PhoneNumber                  *string    `json:"phone_number,omitempty"`
    AuthProvider                 string     `json:"auth_provider"`
    TOTPSecret                   string     `json:"totp_secret,omitempty"`
    IsActive                     bool       `json:"is_active"`
    BusinessName                 string     `json:"business_name"`
    BusinessAddress              string     `json:"business_address"`
    City                         string     `json:"city"`
    State                        string     `json:"state"`
    ZipCode                      string     `json:"zip_code"`
    PersonaVerificationInquiryID string     `json:"persona_inquiry_id"`
    CreatedAt                    time.Time  `json:"created_at"`
    UpdatedAt                    time.Time  `json:"updated_at"`
}


## blacklisted_token.go

package models

import (
    "time"

    "github.com/google/uuid"
)

// BlacklistedToken represents a revoked or invalidated access token
type BlacklistedToken struct {
    ID        uuid.UUID `json:"id"`
    TokenID   string    `json:"token_id"`   // JTI (JWT ID) claim from the token
    ExpiresAt time.Time `json:"expires_at"` // Token expiration time
    CreatedAt time.Time `json:"created_at"` // Time when token was blacklisted
}


## .gitignore

codebase_content.txt


## property_building.go

package models

import "github.com/google/uuid"

type PropertyBuilding struct {
    ID          uuid.UUID         `json:"id"`
    PropertyID  uuid.UUID         `json:"property_id"`
    BuildingName string            `json:"building_name,omitempty"`
    Address     string            `json:"address"`
    Latitude    float64           `json:"latitude,omitempty"`
    Longitude   float64           `json:"longitude,omitempty"`
}


## email_verification_code.go

package models

import (
    "time"

    "github.com/google/uuid"
)

type EmailVerificationCode struct {
    ID               uuid.UUID
    Email            string
    VerificationCode string
    ExpiresAt        time.Time
    Attempts         int
    CreatedAt        time.Time
}


## unit.go

package models

import (
    "time"
    "github.com/google/uuid"
)

type Unit struct {
    ID          uuid.UUID  `json:"id"`
    PropertyID  uuid.UUID  `json:"property_id"`
    UnitNumber  string     `json:"unit_number"`
    TenantToken *string    `json:"tenant_token,omitempty"` // Optional, for tenant identification
    CreatedAt   time.Time  `json:"created_at"`
}


## refresh_token.go

package models

import (
    "time"

    "github.com/google/uuid"
)

// RefreshToken represents a refresh token issued to a user.
type RefreshToken struct {
    ID        uuid.UUID `json:"id"`
    UserID    uuid.UUID `json:"user_id"`
    Token     string    `json:"token"`
    ExpiresAt time.Time `json:"expires_at"`
    CreatedAt time.Time `json:"created_at"`
    Revoked   bool      `json:"revoked"`
    IPAddress string    `json:"ip_address"`
}

func (rt *RefreshToken) IsExpired() bool {
    return time.Now().After(rt.ExpiresAt)
}


----------------------------------------------
####  Utils Package #####

## random.go

package utils

import (
    "crypto/rand"
    "encoding/hex"
)

func RandomString(length int) string {
    bytes := make([]byte, length)
    _, err := rand.Read(bytes)
    if err != nil {
        panic(err) // Handle error appropriately in production
    }
    return hex.EncodeToString(bytes)[:length]
}


## encryption.go

package utils

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/base64"
	"errors"
	"io"
)

// Encrypt encrypts the provided plaintext using the given encryption key (must be 32 bytes for AES-256).
func Encrypt(encryptionKey []byte, text string) (string, error) {
	if len(encryptionKey) != 32 {
		return "", errors.New("encryption key must be 32 bytes long")
	}

	block, err := aes.NewCipher(encryptionKey)
	if err != nil {
		return "", err
	}

	plaintext := []byte(text)
	ciphertext := make([]byte, aes.BlockSize+len(plaintext))
	iv := ciphertext[:aes.BlockSize]

	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		return "", err
	}

	stream := cipher.NewCFBEncrypter(block, iv)
	stream.XORKeyStream(ciphertext[aes.BlockSize:], plaintext)

	return base64.URLEncoding.EncodeToString(ciphertext), nil
}

// Decrypt decrypts the provided ciphertext using the given encryption key (must be 32 bytes for AES-256).
func Decrypt(encryptionKey []byte, encryptedText string) (string, error) {
	if len(encryptionKey) != 32 {
		return "", errors.New("encryption key must be 32 bytes long")
	}

	ciphertext, err := base64.URLEncoding.DecodeString(encryptedText)
	if err != nil {
		return "", err
	}

	block, err := aes.NewCipher(encryptionKey)
	if err != nil {
		return "", err
	}

	if len(ciphertext) < aes.BlockSize {
		return "", errors.New("ciphertext too short")
	}

	iv := ciphertext[:aes.BlockSize]
	ciphertext = ciphertext[aes.BlockSize:]

	stream := cipher.NewCFBDecrypter(block, iv)
	stream.XORKeyStream(ciphertext, ciphertext)

	return string(ciphertext), nil
}



## logger.go

package utils

import (
    "github.com/sirupsen/logrus"
    "os"
)

var Logger = logrus.New()

func InitLogger() {
    Logger.SetOutput(os.Stdout)
    Logger.SetLevel(logrus.InfoLevel)
    Logger.SetFormatter(&logrus.TextFormatter{
        FullTimestamp: true,
    })
}



## response.go

package utils

import (
    "encoding/json"
    "net/http"
    "github.com/sirupsen/logrus"
)

type ErrorResponse struct {
    Code    string `json:"code"`
    Message string `json:"message"`
}

func RespondWithError(w http.ResponseWriter, status int, publicMessage string, err error) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)

    var code string
    if err != nil {
        code = err.Error()
    } else {
        code = "unknown_error"
    }

    // Send a structured response to the frontend
    json.NewEncoder(w).Encode(ErrorResponse{
        Code:    code,
        Message: publicMessage,
    })

    // Log the detailed error internally
    if err != nil {
        Logger.WithFields(logrus.Fields{
            "status": status,
            "error":  err.Error(),
        }).Error(publicMessage)
    } else {
        Logger.WithFields(logrus.Fields{
            "status": status,
        }).Error(publicMessage)
    }
}

func RespondWithJSON(w http.ResponseWriter, status int, payload interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(payload)
}


## ip.go

package utils

import (
	"net"
	"net/http"
	"strings"
)

func GetIPAddress(r *http.Request) string {
    // 1. Check the X-Forwarded-For header (can contain multiple IPs)
    forwardedFor := r.Header.Get("X-Forwarded-For")
    if forwardedFor != "" {
        // Split by commas in case there are multiple proxies
        ips := strings.Split(forwardedFor, ",")
        for _, ip := range ips {
            cleanIP := strings.TrimSpace(ip)
            if isValidIP(cleanIP) {
                return cleanIP
            }
        }
    }

    // 2. Check CF-Connecting-IP (Cloudflare-specific)
    cfConnectingIP := r.Header.Get("CF-Connecting-IP")
    if cfConnectingIP != "" && isValidIP(cfConnectingIP) {
        return cfConnectingIP
    }

    // 3. Check the X-Real-IP header
    realIP := r.Header.Get("X-Real-IP")
    if realIP != "" && isValidIP(realIP) {
        return realIP
    }

    // 4. Check the Forwarded header (standardized RFC 7239)
    forwarded := r.Header.Get("Forwarded")
    if forwarded != "" {
        forwardedParts := strings.Split(forwarded, ";")
        for _, part := range forwardedParts {
            if strings.HasPrefix(strings.TrimSpace(part), "for=") {
                ip := strings.TrimPrefix(strings.TrimSpace(part), "for=")
                // Remove surrounding quotes if present
                ip = strings.Trim(ip, "\"")
                if isValidIP(ip) {
                    return ip
                }
            }
        }
    }

    // 5. Fallback to RemoteAddr (direct connection address)
    ip, _, err := net.SplitHostPort(r.RemoteAddr)
    if err == nil && isValidIP(ip) {
        return ip
    }

    return ""
}

// isValidIP checks if the provided IP address is valid (both IPv4 and IPv6).
func isValidIP(ip string) bool {
    return net.ParseIP(ip) != nil
}


## .gitignore

codebase_content.txt


## hash.go

package utils

import (
	"crypto/sha256"
	"encoding/base64"
)

func HashToken(raw string) string {
    hasher := sha256.New()
    hasher.Write([]byte(raw))
    return base64.URLEncoding.EncodeToString(hasher.Sum(nil))
}



---------------------------------------------
#### Repositories Package ####

## worker_repository.go

package repositories

import (
    "github.com/jackc/pgx/v4"
    "context"
    "github.com/google/uuid"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-utils"
)

type WorkerRepository interface {
    Create(ctx context.Context, w *models.Worker) error
    GetByEmail(ctx context.Context, email string) (*models.Worker, error)
    GetByPhoneNumber(ctx context.Context, phoneNumber string) (*models.Worker, error)
    Update(ctx context.Context, w *models.Worker) error
    GetByID(ctx context.Context, id uuid.UUID) (*models.Worker, error)
}

type workerRepository struct {
    db             DB
    dbEncryptionKey []byte
}

func NewWorkerRepository(db DB, encryptionKey []byte) WorkerRepository {
    return &workerRepository{
        db: db,
        dbEncryptionKey: encryptionKey,
    }
}

func (r *workerRepository) Create(ctx context.Context, w *models.Worker) error {
    // Encrypt TOTP
    if w.TOTPSecret != "" {
        encryptedSecret, err := utils.Encrypt(r.dbEncryptionKey, w.TOTPSecret)
        if err != nil {
            return err
        }
        w.TOTPSecret = encryptedSecret
    }

    query := `
        INSERT INTO workers (
            id, email, phone_number, auth_provider, totp_secret, is_active,
            first_name, last_name, street_address, apt_suite, city, state,
            zip_code, vehicle_year, vehicle_make, vehicle_model,
            persona_inquiry_id, created_at, updated_at
        )
        VALUES (
            $1, $2, $3, $4, $5, $6,
            $7, $8, $9, $10, $11, $12,
            $13, $14, $15, $16, $17,
            NOW(), NOW()
        )
    `
    _, err := r.db.Exec(ctx, query,
        w.ID, w.Email, w.PhoneNumber, w.AuthProvider, w.TOTPSecret, w.IsActive,
        w.FirstName, w.LastName, w.StreetAddress, w.AptSuite, w.City, w.State,
        w.ZipCode, w.VehicleYear, w.VehicleMake, w.VehicleModel, w.PersonaVerificationInquiryID)
    return err
}

func (r *workerRepository) GetByEmail(ctx context.Context, email string) (*models.Worker, error) {
    query := `
        SELECT
            id, email, phone_number, auth_provider, totp_secret, is_active,
            first_name, last_name, street_address, apt_suite, city, state,
            zip_code, vehicle_year, vehicle_make, vehicle_model,
            persona_inquiry_id, created_at, updated_at
        FROM workers
        WHERE email = $1
    `
    row := r.db.QueryRow(ctx, query, email)
    return r.scanWorker(row)
}

func (r *workerRepository) GetByPhoneNumber(ctx context.Context, phoneNumber string) (*models.Worker, error) {
    query := `
        SELECT
            id, email, phone_number, auth_provider, totp_secret, is_active,
            first_name, last_name, street_address, apt_suite, city, state,
            zip_code, vehicle_year, vehicle_make, vehicle_model,
            persona_inquiry_id, created_at, updated_at
        FROM workers
        WHERE phone_number = $1
    `
    row := r.db.QueryRow(ctx, query, phoneNumber)
    return r.scanWorker(row)
}

func (r *workerRepository) GetByID(ctx context.Context, id uuid.UUID) (*models.Worker, error) {
    query := `
        SELECT
            id, email, phone_number, auth_provider, totp_secret, is_active,
            first_name, last_name, street_address, apt_suite, city, state,
            zip_code, vehicle_year, vehicle_make, vehicle_model,
            persona_inquiry_id, created_at, updated_at
        FROM workers
        WHERE id = $1
    `
    row := r.db.QueryRow(ctx, query, id)
    return r.scanWorker(row)
}

func (r *workerRepository) Update(ctx context.Context, w *models.Worker) error {
    if w.TOTPSecret != "" {
        encrypted, err := utils.Encrypt(r.dbEncryptionKey, w.TOTPSecret)
        if err != nil {
            return err
        }
        w.TOTPSecret = encrypted
    }

    query := `
        UPDATE workers
        SET email = $1,
            phone_number = $2,
            auth_provider = $3,
            totp_secret = $4,
            is_active = $5,
            first_name = $6,
            last_name = $7,
            street_address = $8,
            apt_suite = $9,
            city = $10,
            state = $11,
            zip_code = $12,
            vehicle_year = $13,
            vehicle_make = $14,
            vehicle_model = $15,
            persona_inquiry_id = $16,
            updated_at = NOW()
        WHERE id = $17
    `
    _, err := r.db.Exec(ctx, query,
        w.Email, w.PhoneNumber, w.AuthProvider, w.TOTPSecret, w.IsActive,
        w.FirstName, w.LastName, w.StreetAddress, w.AptSuite, w.City, w.State,
        w.ZipCode, w.VehicleYear, w.VehicleMake, w.VehicleModel,
        w.PersonaVerificationInquiryID, w.ID,
    )
    return err
}

func (r *workerRepository) scanWorker(row pgx.Row) (*models.Worker, error) {
    var w models.Worker
    var encryptedSecret *string

    err := row.Scan(
        &w.ID,
        &w.Email,
        &w.PhoneNumber,
        &w.AuthProvider,
        &encryptedSecret,
        &w.IsActive,
        &w.FirstName,
        &w.LastName,
        &w.StreetAddress,
        &w.AptSuite,
        &w.City,
        &w.State,
        &w.ZipCode,
        &w.VehicleYear,
        &w.VehicleMake,
        &w.VehicleModel,
        &w.PersonaVerificationInquiryID,
        &w.CreatedAt,
        &w.UpdatedAt,
    )
    if err != nil {
        return nil, err
    }

    // Decrypt TOTP if present
    if encryptedSecret != nil {
        decrypted, err := utils.Decrypt(r.dbEncryptionKey, *encryptedSecret)
        if err != nil {
            return nil, err
        }
        w.TOTPSecret = decrypted
    }
    return &w, nil
}


## .gitignore

codebase_content.txt


## worker_login_attempts_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/google/uuid"
)

type WorkerLoginAttempts struct {
    WorkerID    uuid.UUID
    AttemptCount int
    LockedUntil *time.Time
    UpdatedAt   time.Time
    CreatedAt   time.Time
}

type WorkerLoginAttemptsRepository interface {
    GetOrCreate(ctx context.Context, workerID uuid.UUID) (*WorkerLoginAttempts, error)
    Increment(ctx context.Context, workerID uuid.UUID, lockDuration, window time.Duration, maxAttempts int) error
    Reset(ctx context.Context, workerID uuid.UUID) error
    IsLocked(ctx context.Context, workerID uuid.UUID) (bool, time.Time, error)
}

type workerLoginAttemptsRepository struct {
    db DB
}

func NewWorkerLoginAttemptsRepository(db DB) WorkerLoginAttemptsRepository {
    return &workerLoginAttemptsRepository{db: db}
}

func (r *workerLoginAttemptsRepository) GetOrCreate(ctx context.Context, workerID uuid.UUID) (*WorkerLoginAttempts, error) {
    query := `
        SELECT worker_id, attempt_count, locked_until, updated_at, created_at
        FROM worker_login_attempts
        WHERE worker_id = $1
    `
    row := r.db.QueryRow(ctx, query, workerID)
    la := &WorkerLoginAttempts{}
    err := row.Scan(
        &la.WorkerID,
        &la.AttemptCount,
        &la.LockedUntil,
        &la.UpdatedAt,
        &la.CreatedAt,
    )
    if err == nil {
        return la, nil
    }
    // Insert fresh
    insert := `
        INSERT INTO worker_login_attempts (worker_id, attempt_count, locked_until, updated_at, created_at)
        VALUES ($1, 0, NULL, NOW(), NOW())
        RETURNING worker_id, attempt_count, locked_until, updated_at, created_at
    `
    row = r.db.QueryRow(ctx, insert, workerID)
    err = row.Scan(
        &la.WorkerID,
        &la.AttemptCount,
        &la.LockedUntil,
        &la.UpdatedAt,
        &la.CreatedAt,
    )
    return la, err
}

func (r *workerLoginAttemptsRepository) Increment(
    ctx context.Context,
    workerID uuid.UUID,
    lockDuration, window time.Duration,
    maxAttempts int,
) error {
    query := `
WITH current AS (
    SELECT worker_id,
           attempt_count,
           locked_until,
           updated_at
    FROM worker_login_attempts
    WHERE worker_id = $1
    FOR UPDATE
)
UPDATE worker_login_attempts
SET attempt_count = CASE
    WHEN (locked_until IS NOT NULL AND locked_until > NOW())
         THEN attempt_count
    ELSE CASE
        WHEN (NOW() - current.updated_at) > $3
            THEN 1
        ELSE attempt_count + 1
    END
END,
locked_until = CASE
    WHEN (locked_until IS NOT NULL AND locked_until > NOW())
         THEN locked_until
    ELSE CASE
        WHEN ((NOW() - current.updated_at) <= $3
              AND (current.attempt_count + 1) >= $4)
            THEN NOW() + $2
        ELSE NULL
    END
END,
updated_at = NOW()
FROM current
WHERE worker_login_attempts.worker_id = current.worker_id
RETURNING worker_login_attempts.worker_id
    `
    _, err := r.db.Exec(ctx, query, workerID, lockDuration, window, maxAttempts)
    return err
}

func (r *workerLoginAttemptsRepository) Reset(ctx context.Context, workerID uuid.UUID) error {
    query := `
        UPDATE worker_login_attempts
        SET attempt_count = 0,
            locked_until = NULL,
            updated_at = NOW()
        WHERE worker_id = $1
    `
    _, err := r.db.Exec(ctx, query, workerID)
    return err
}

func (r *workerLoginAttemptsRepository) IsLocked(ctx context.Context, workerID uuid.UUID) (bool, time.Time, error) {
    query := `
        SELECT locked_until
        FROM worker_login_attempts
        WHERE worker_id = $1
    `
    row := r.db.QueryRow(ctx, query, workerID)
    var lockedUntil *time.Time
    if err := row.Scan(&lockedUntil); err != nil {
        return false, time.Time{}, err
    }
    if lockedUntil != nil && lockedUntil.After(time.Now()) {
        return true, *lockedUntil, nil
    }
    return false, time.Time{}, nil
}



## pm_repository.go

package repositories

import (
    "context"

    "github.com/jackc/pgx/v4"
    "github.com/google/uuid"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-utils"
)

type PropertyManagerRepository interface {
    Create(ctx context.Context, pm *models.PropertyManager) error
    GetByEmail(ctx context.Context, email string) (*models.PropertyManager, error)
    GetByPhoneNumber(ctx context.Context, phone string) (*models.PropertyManager, error)
    GetByID(ctx context.Context, id uuid.UUID) (*models.PropertyManager, error)
    Update(ctx context.Context, pm *models.PropertyManager) error
}

type propertyManagerRepository struct {
    db             DB
    dbEncryptionKey []byte
}

func NewPropertyManagerRepository(db DB, encryptionKey []byte) PropertyManagerRepository {
    return &propertyManagerRepository{
        db: db,
        dbEncryptionKey: encryptionKey,
    }
}

func (r *propertyManagerRepository) Create(ctx context.Context, pm *models.PropertyManager) error {
    // Encrypt the TOTPSecret
    if pm.TOTPSecret != "" {
        encryptedSecret, err := utils.Encrypt(r.dbEncryptionKey, pm.TOTPSecret)
        if err != nil {
            return err
        }
        pm.TOTPSecret = encryptedSecret
    }

    query := `
        INSERT INTO property_managers (
            id, email, phone_number, auth_provider, totp_secret, is_active,
            business_name, business_address, city, state, zip_code, persona_inquiry_id,
            created_at, updated_at
        )
        VALUES ($1, $2, $3, $4, $5, $6,
                $7, $8, $9, $10, $11, $12,
                NOW(), NOW())
    `
    _, err := r.db.Exec(ctx, query,
        pm.ID, pm.Email, pm.PhoneNumber, pm.AuthProvider, pm.TOTPSecret, pm.IsActive,
        pm.BusinessName, pm.BusinessAddress, pm.City, pm.State, pm.ZipCode, pm.PersonaVerificationInquiryID,
    )
    return err
}

func (r *propertyManagerRepository) GetByEmail(ctx context.Context, email string) (*models.PropertyManager, error) {
    query := `
        SELECT
            id, email, phone_number, auth_provider, totp_secret, is_active,
            business_name, business_address, city, state, zip_code,
            persona_inquiry_id, created_at, updated_at
        FROM property_managers
        WHERE email = $1
    `
    row := r.db.QueryRow(ctx, query, email)
    return r.scanPropertyManager(row)
}

func (r *propertyManagerRepository) GetByPhoneNumber(ctx context.Context, phone string) (*models.PropertyManager, error) {
    query := `
        SELECT
            id, email, phone_number, auth_provider, totp_secret, is_active,
            business_name, business_address, city, state, zip_code,
            persona_inquiry_id, created_at, updated_at
        FROM property_managers
        WHERE phone_number = $1
    `
    row := r.db.QueryRow(ctx, query, phone)
    return r.scanPropertyManager(row)
}

func (r *propertyManagerRepository) GetByID(ctx context.Context, id uuid.UUID) (*models.PropertyManager, error) {
    query := `
        SELECT
            id, email, phone_number, auth_provider, totp_secret, is_active,
            business_name, business_address, city, state, zip_code,
            persona_inquiry_id, created_at, updated_at
        FROM property_managers
        WHERE id = $1
    `
    row := r.db.QueryRow(ctx, query, id)
    return r.scanPropertyManager(row)
}

func (r *propertyManagerRepository) Update(ctx context.Context, pm *models.PropertyManager) error {
    // Re-encrypt TOTP if changed
    if pm.TOTPSecret != "" {
        encryptedSecret, err := utils.Encrypt(r.dbEncryptionKey, pm.TOTPSecret)
        if err != nil {
            return err
        }
        pm.TOTPSecret = encryptedSecret
    }

    query := `
        UPDATE property_managers
        SET email = $1,
            phone_number = $2,
            auth_provider = $3,
            totp_secret = $4,
            is_active = $5,
            business_name = $6,
            business_address = $7,
            city = $8,
            state = $9,
            zip_code = $10,
            persona_inquiry_id = $11,
            updated_at = NOW()
        WHERE id = $12
    `
    _, err := r.db.Exec(ctx, query,
        pm.Email, pm.PhoneNumber, pm.AuthProvider, pm.TOTPSecret, pm.IsActive,
        pm.BusinessName, pm.BusinessAddress, pm.City, pm.State, pm.ZipCode,
        pm.PersonaVerificationInquiryID, pm.ID)
    return err
}

func (r *propertyManagerRepository) scanPropertyManager(row pgx.Row) (*models.PropertyManager, error) {
    var pm models.PropertyManager
    var encryptedSecret *string

    err := row.Scan(
        &pm.ID,
        &pm.Email,
        &pm.PhoneNumber,
        &pm.AuthProvider,
        &encryptedSecret,
        &pm.IsActive,
        &pm.BusinessName,
        &pm.BusinessAddress,
        &pm.City,
        &pm.State,
        &pm.ZipCode,
        &pm.PersonaVerificationInquiryID,
        &pm.CreatedAt,
        &pm.UpdatedAt,
    )
    if err != nil {
        return nil, err
    }

    if encryptedSecret != nil {
        decrypted, err := utils.Decrypt(r.dbEncryptionKey, *encryptedSecret)
        if err != nil {
            return nil, err
        }
        pm.TOTPSecret = decrypted
    }

    return &pm, nil
}


## pm_token_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/google/uuid"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-utils"
)

type PMTokenRepository interface {
    CreateRefreshToken(ctx context.Context, token *models.RefreshToken) error
    GetRefreshToken(ctx context.Context, rawToken string) (*models.RefreshToken, error)
    RevokeRefreshToken(ctx context.Context, id uuid.UUID) error

    // If you want blacklisting:
    BlacklistToken(ctx context.Context, tokenID string, expiresAt time.Time) error
    IsTokenBlacklisted(ctx context.Context, tokenID string) (bool, error)
}

type pmTokenRepository struct {
    db DB
}

func NewPMTokenRepository(db DB) TokenRepository {
    return &pmTokenRepository{db: db}
}

func (r *pmTokenRepository) CreateRefreshToken(ctx context.Context, token *models.RefreshToken) error {
    // We store hashed refresh_token in pm_refresh_tokens table
    // Note that token.UserID in this context is the pm_id
    query := `
        INSERT INTO pm_refresh_tokens (id, pm_id, refresh_token, expires_at, created_at, revoked, ip_address)
        VALUES ($1, $2, $3, $4, NOW(), $5, $6)
    `
    _, err := r.db.Exec(ctx, query,
        token.ID,
        token.UserID, // pm_id
        utils.HashToken(token.Token),
        token.ExpiresAt,
        token.Revoked,
        token.IPAddress,
    )
    return err
}

func (r *pmTokenRepository) GetRefreshToken(ctx context.Context, rawToken string) (*models.RefreshToken, error) {
    hashed := utils.HashToken(rawToken)
    query := `
        SELECT id, pm_id, refresh_token, expires_at, created_at, revoked, ip_address
        FROM pm_refresh_tokens
        WHERE refresh_token = $1
    `
    row := r.db.QueryRow(ctx, query, hashed)

    var rt models.RefreshToken
    err := row.Scan(
        &rt.ID,
        &rt.UserID,    // store pm_id in .UserID
        &rt.Token,
        &rt.ExpiresAt,
        &rt.CreatedAt,
        &rt.Revoked,
        &rt.IPAddress,
    )
    if err != nil {
        return nil, err
    }
    // Re-inject the "raw" token so the service can handle it, if needed
    // but all we stored was the hashed value in the DB
    rt.Token = rawToken
    return &rt, nil
}

func (r *pmTokenRepository) RevokeRefreshToken(ctx context.Context, id uuid.UUID) error {
    query := `
        UPDATE pm_refresh_tokens
        SET revoked = TRUE
        WHERE id = $1
    `
    _, err := r.db.Exec(ctx, query, id)
    return err
}

// Optional blacklisting for access tokens:
func (r *pmTokenRepository) BlacklistToken(ctx context.Context, tokenID string, expiresAt time.Time) error {
    query := `
        INSERT INTO pm_blacklisted_tokens (id, token_id, expires_at, created_at)
        VALUES ($1, $2, $3, NOW())
    `
    _, err := r.db.Exec(ctx, query, uuid.New(), tokenID, expiresAt)
    return err
}

func (r *pmTokenRepository) IsTokenBlacklisted(ctx context.Context, tokenID string) (bool, error) {
    query := `
        SELECT EXISTS (
            SELECT 1 FROM pm_blacklisted_tokens
            WHERE token_id = $1 AND expires_at > NOW()
        )
    `
    var exists bool
    err := r.db.QueryRow(ctx, query, tokenID).Scan(&exists)
    return exists, err
}


## sms_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/poofdev/poof-back-models"
    "github.com/google/uuid"
)

type SMSRepository interface {
    CreateCode(ctx context.Context, phoneNumber, code string, expiresAt time.Time) error
    GetCode(ctx context.Context, phoneNumber string) (*models.SMSVerificationCode, error)
    IncrementAttempts(ctx context.Context, id uuid.UUID) error
    DeleteCode(ctx context.Context, id uuid.UUID) error
}

type smsRepository struct {
    db DB
}

func NewSMSRepository(db DB) SMSRepository {
    return &smsRepository{db: db}
}

func (r *smsRepository) CreateCode(ctx context.Context, phoneNumber, code string, expiresAt time.Time) error {
    query := `
        INSERT INTO sms_verification_codes (id, phone_number, verification_code, expires_at, created_at, attempts)
        VALUES ($1, $2, $3, $4, NOW(), 0)
    `
    _, err := r.db.Exec(ctx, query, uuid.New(), phoneNumber, code, expiresAt)
    return err
}

func (r *smsRepository) GetCode(ctx context.Context, phoneNumber string) (*models.SMSVerificationCode, error) {
    query := `
        SELECT id, phone_number, verification_code, expires_at, created_at, attempts
        FROM sms_verification_codes
        WHERE phone_number = $1
        ORDER BY created_at DESC
        LIMIT 1
    `
    row := r.db.QueryRow(ctx, query, phoneNumber)

    var smsCode models.SMSVerificationCode
    err := row.Scan(
        &smsCode.ID,
        &smsCode.PhoneNumber,
        &smsCode.VerificationCode,
        &smsCode.ExpiresAt,
        &smsCode.CreatedAt,
        &smsCode.Attempts,
    )
    if err != nil {
        return nil, err
    }
    return &smsCode, nil
}

func (r *smsRepository) IncrementAttempts(ctx context.Context, id uuid.UUID) error {
    query := `
        UPDATE sms_verification_codes
        SET attempts = attempts + 1
        WHERE id = $1
    `
    _, err := r.db.Exec(ctx, query, id)
    return err
}

func (r *smsRepository) DeleteCode(ctx context.Context, id uuid.UUID) error {
    query := `
        DELETE FROM sms_verification_codes
        WHERE id = $1
    `
    _, err := r.db.Exec(ctx, query, id)
    return err
}


## worker_token_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/google/uuid"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-utils"
)

type WorkerTokenRepository interface {
    CreateRefreshToken(ctx context.Context, token *models.RefreshToken) error
    GetRefreshToken(ctx context.Context, rawToken string) (*models.RefreshToken, error)
    RevokeRefreshToken(ctx context.Context, id uuid.UUID) error

    // If you want blacklisting for workers
    BlacklistToken(ctx context.Context, tokenID string, expiresAt time.Time) error
    IsTokenBlacklisted(ctx context.Context, tokenID string) (bool, error)
}

type workerTokenRepository struct {
    db DB
}

func NewWorkerTokenRepository(db DB) TokenRepository {
    return &workerTokenRepository{db: db}
}

func (r *workerTokenRepository) CreateRefreshToken(ctx context.Context, token *models.RefreshToken) error {
    query := `
        INSERT INTO worker_refresh_tokens (id, worker_id, refresh_token, expires_at, created_at, revoked, ip_address)
        VALUES ($1, $2, $3, $4, NOW(), $5, $6)
    `
    _, err := r.db.Exec(ctx, query,
        token.ID,
        token.UserID, // worker_id
        utils.HashToken(token.Token),
        token.ExpiresAt,
        token.Revoked,
        token.IPAddress,
    )
    return err
}

func (r *workerTokenRepository) GetRefreshToken(ctx context.Context, rawToken string) (*models.RefreshToken, error) {
    hashed := utils.HashToken(rawToken)
    query := `
        SELECT id, worker_id, refresh_token, expires_at, created_at, revoked, ip_address
        FROM worker_refresh_tokens
        WHERE refresh_token = $1
    `
    row := r.db.QueryRow(ctx, query, hashed)

    var rt models.RefreshToken
    err := row.Scan(
        &rt.ID,
        &rt.UserID,
        &rt.Token,
        &rt.ExpiresAt,
        &rt.CreatedAt,
        &rt.Revoked,
        &rt.IPAddress,
    )
    if err != nil {
        return nil, err
    }
    rt.Token = rawToken
    return &rt, nil
}

func (r *workerTokenRepository) RevokeRefreshToken(ctx context.Context, id uuid.UUID) error {
    query := `
        UPDATE worker_refresh_tokens
        SET revoked = TRUE
        WHERE id = $1
    `
    _, err := r.db.Exec(ctx, query, id)
    return err
}

func (r *workerTokenRepository) BlacklistToken(ctx context.Context, tokenID string, expiresAt time.Time) error {
    query := `
        INSERT INTO worker_blacklisted_tokens (id, token_id, expires_at, created_at)
        VALUES ($1, $2, $3, NOW())
    `
    _, err := r.db.Exec(ctx, query, uuid.New(), tokenID, expiresAt)
    return err
}

func (r *workerTokenRepository) IsTokenBlacklisted(ctx context.Context, tokenID string) (bool, error) {
    query := `
        SELECT EXISTS (
            SELECT 1 FROM worker_blacklisted_tokens
            WHERE token_id = $1 AND expires_at > NOW()
        )
    `
    var exists bool
    err := r.db.QueryRow(ctx, query, tokenID).Scan(&exists)
    return exists, err
}


## pm_login_attempts_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/google/uuid"
)

type PMLoginAttempts struct {
    PMID        uuid.UUID
    AttemptCount int
    LockedUntil *time.Time
    UpdatedAt   time.Time
    CreatedAt   time.Time
}

type PMLoginAttemptsRepository interface {
    GetOrCreate(ctx context.Context, pmID uuid.UUID) (*PMLoginAttempts, error)
    Increment(ctx context.Context, pmID uuid.UUID, lockDuration, window time.Duration, maxAttempts int) error
    Reset(ctx context.Context, pmID uuid.UUID) error
    IsLocked(ctx context.Context, pmID uuid.UUID) (bool, time.Time, error)
}

type pmLoginAttemptsRepository struct {
    db DB
}

func NewPMLoginAttemptsRepository(db DB) PMLoginAttemptsRepository {
    return &pmLoginAttemptsRepository{db: db}
}

func (r *pmLoginAttemptsRepository) GetOrCreate(ctx context.Context, pmID uuid.UUID) (*PMLoginAttempts, error) {
    query := `
        SELECT pm_id, attempt_count, locked_until, updated_at, created_at
        FROM pm_login_attempts
        WHERE pm_id = $1
    `
    row := r.db.QueryRow(ctx, query, pmID)
    la := &PMLoginAttempts{}
    err := row.Scan(
        &la.PMID,
        &la.AttemptCount,
        &la.LockedUntil,
        &la.UpdatedAt,
        &la.CreatedAt,
    )
    if err == nil {
        return la, nil
    }
    // If no record => insert fresh
    insert := `
        INSERT INTO pm_login_attempts (pm_id, attempt_count, locked_until, updated_at, created_at)
        VALUES ($1, 0, NULL, NOW(), NOW())
        RETURNING pm_id, attempt_count, locked_until, updated_at, created_at
    `
    row = r.db.QueryRow(ctx, insert, pmID)
    err = row.Scan(
        &la.PMID,
        &la.AttemptCount,
        &la.LockedUntil,
        &la.UpdatedAt,
        &la.CreatedAt,
    )
    return la, err
}

func (r *pmLoginAttemptsRepository) Increment(
    ctx context.Context,
    pmID uuid.UUID,
    lockDuration, window time.Duration,
    maxAttempts int,
) error {
    query := `
WITH current AS (
    SELECT pm_id,
           attempt_count,
           locked_until,
           updated_at
    FROM pm_login_attempts
    WHERE pm_id = $1
    FOR UPDATE
)
UPDATE pm_login_attempts
SET attempt_count = CASE
    WHEN (locked_until IS NOT NULL AND locked_until > NOW())
         THEN attempt_count
    ELSE CASE
        WHEN (NOW() - current.updated_at) > $3
            THEN 1
        ELSE attempt_count + 1
    END
END,
locked_until = CASE
    WHEN (locked_until IS NOT NULL AND locked_until > NOW())
         THEN locked_until
    ELSE CASE
        WHEN ((NOW() - current.updated_at) <= $3
              AND (current.attempt_count + 1) >= $4)
            THEN NOW() + $2
        ELSE NULL
    END
END,
updated_at = NOW()
FROM current
WHERE pm_login_attempts.pm_id = current.pm_id
RETURNING pm_login_attempts.pm_id
    `
    _, err := r.db.Exec(ctx, query, pmID, lockDuration, window, maxAttempts)
    return err
}

func (r *pmLoginAttemptsRepository) Reset(ctx context.Context, pmID uuid.UUID) error {
    query := `
        UPDATE pm_login_attempts
        SET attempt_count = 0,
            locked_until = NULL,
            updated_at = NOW()
        WHERE pm_id = $1
    `
    _, err := r.db.Exec(ctx, query, pmID)
    return err
}

func (r *pmLoginAttemptsRepository) IsLocked(ctx context.Context, pmID uuid.UUID) (bool, time.Time, error) {
    query := `
        SELECT locked_until
        FROM pm_login_attempts
        WHERE pm_id = $1
    `
    row := r.db.QueryRow(ctx, query, pmID)
    var lockedUntil *time.Time
    if err := row.Scan(&lockedUntil); err != nil {
        return false, time.Time{}, err
    }
    if lockedUntil != nil && lockedUntil.After(time.Now()) {
        return true, *lockedUntil, nil
    }
    return false, time.Time{}, nil
}



## db.go

package repositories

import (
    "context"

    "github.com/jackc/pgconn"
	"github.com/jackc/pgx/v4"
)

type DB interface {
	Exec(ctx context.Context, sql string, arguments ...interface{}) (pgconn.CommandTag, error)
	Query(ctx context.Context, sql string, optionsAndArgs ...interface{}) (pgx.Rows, error)
	QueryRow(ctx context.Context, sql string, optionsAndArgs ...interface{}) pgx.Row
}



## token_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/google/uuid"
    "github.com/poofdev/poof-back-models"
)

// TokenRepository is the interface used by the JWT service
// to create, retrieve, revoke, and optionally blacklist refresh tokens.
type TokenRepository interface {
    CreateRefreshToken(ctx context.Context, token *models.RefreshToken) error
    GetRefreshToken(ctx context.Context, rawToken string) (*models.RefreshToken, error)
    RevokeRefreshToken(ctx context.Context, id uuid.UUID) error

    // If you do blacklisting:
    BlacklistToken(ctx context.Context, tokenID string, expiresAt time.Time) error
    IsTokenBlacklisted(ctx context.Context, tokenID string) (bool, error)
}


## email_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/google/uuid"
    "github.com/jackc/pgx/v4/pgxpool"
    "github.com/poofdev/poof-back-models"
)

type EmailRepository interface {
    CreateCode(ctx context.Context, email, code string, expiresAt time.Time) error
    GetCode(ctx context.Context, email string) (*models.EmailVerificationCode, error)
    DeleteCode(ctx context.Context, id uuid.UUID) error
    IncrementAttempts(ctx context.Context, id uuid.UUID) error
}

type emailRepository struct {
    db *pgxpool.Pool
}

func NewEmailRepository(db *pgxpool.Pool) EmailRepository {
    return &emailRepository{
        db: db,
    }
}

// CreateCode inserts a new email verification code into the database
func (e *emailRepository) CreateCode(ctx context.Context, email, code string, expiresAt time.Time) error {
    query := `
        INSERT INTO email_verification_codes (id, email, verification_code, expires_at, created_at, attempts)
        VALUES ($1, $2, $3, $4, NOW(), 0)
    `
    _, err := e.db.Exec(ctx, query, uuid.New(), email, code, expiresAt)
    return err
}

// GetCode retrieves the most recent verification code for the given email address
func (e *emailRepository) GetCode(ctx context.Context, email string) (*models.EmailVerificationCode, error) {
    query := `
        SELECT id, email, verification_code, expires_at, created_at, attempts
        FROM email_verification_codes
        WHERE email = $1
        ORDER BY created_at DESC
        LIMIT 1
    `
    row := e.db.QueryRow(ctx, query, email)

    var emailCode models.EmailVerificationCode
    err := row.Scan(
        &emailCode.ID,
        &emailCode.Email,
        &emailCode.VerificationCode,
        &emailCode.ExpiresAt,
        &emailCode.CreatedAt,
        &emailCode.Attempts,
    )
    if err != nil {
        return nil, err
    }
    return &emailCode, nil
}

// DeleteCode removes an email verification code from the database after successful verification
func (e *emailRepository) DeleteCode(ctx context.Context, id uuid.UUID) error {
    query := `
        DELETE FROM email_verification_codes
        WHERE id = $1
    `
    _, err := e.db.Exec(ctx, query, id)
    return err
}

// IncrementAttempts increments the attempt count for a verification code
func (e *emailRepository) IncrementAttempts(ctx context.Context, id uuid.UUID) error {
    query := `
        UPDATE email_verification_codes
        SET attempts = attempts + 1
        WHERE id = $1
    `
    _, err := e.db.Exec(ctx, query, id)
    return err
<<<<<<< Updated upstream
}


## sms_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/poofdev/poof-back-models"
    "github.com/google/uuid"
)

type SMSRepository interface {
    CreateCode(ctx context.Context, phoneNumber, code string, expiresAt time.Time) error
    GetCode(ctx context.Context, phoneNumber string) (*models.SMSVerificationCode, error)
    IncrementAttempts(ctx context.Context, id uuid.UUID) error
    DeleteCode(ctx context.Context, id uuid.UUID) error
}

type smsRepository struct {
    db DB
}

func NewSMSRepository(db DB) SMSRepository {
    return &smsRepository{db: db}
}

func (r *smsRepository) CreateCode(ctx context.Context, phoneNumber, code string, expiresAt time.Time) error {
    query := `
        INSERT INTO sms_verification_codes (id, phone_number, verification_code, expires_at, created_at, attempts)
        VALUES ($1, $2, $3, $4, NOW(), 0)
    `
    _, err := r.db.Exec(ctx, query, uuid.New(), phoneNumber, code, expiresAt)
    return err
}

func (r *smsRepository) GetCode(ctx context.Context, phoneNumber string) (*models.SMSVerificationCode, error) {
    query := `
        SELECT id, phone_number, verification_code, expires_at, created_at, attempts
        FROM sms_verification_codes
        WHERE phone_number = $1
        ORDER BY created_at DESC
        LIMIT 1
    `
    row := r.db.QueryRow(ctx, query, phoneNumber)

    var smsCode models.SMSVerificationCode
    err := row.Scan(
        &smsCode.ID,
        &smsCode.PhoneNumber,
        &smsCode.VerificationCode,
        &smsCode.ExpiresAt,
        &smsCode.CreatedAt,
        &smsCode.Attempts,
    )
    if err != nil {
        return nil, err
    }
    return &smsCode, nil
}

func (r *smsRepository) IncrementAttempts(ctx context.Context, id uuid.UUID) error {
    query := `
        UPDATE sms_verification_codes
        SET attempts = attempts + 1
        WHERE id = $1
    `
    _, err := r.db.Exec(ctx, query, id)
    return err
}

func (r *smsRepository) DeleteCode(ctx context.Context, id uuid.UUID) error {
    query := `
        DELETE FROM sms_verification_codes
        WHERE id = $1
    `
    _, err := r.db.Exec(ctx, query, id)
    return err
}


## user_repository.go

package repositories

import (
    "context"

    "github.com/google/uuid"
    "github.com/jackc/pgx/v4"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-utils"
)

type UserRepository interface {
    CreateUser(ctx context.Context, user *models.User) error
    CreateUserProfile(ctx context.Context, userID uuid.UUID, firstName, lastName string) error
    GetByID(ctx context.Context, id uuid.UUID) (*models.User, error)
    GetByPhoneNumber(ctx context.Context, phoneNumber string) (*models.User, error)
    GetByEmail(ctx context.Context, email string) (*models.User, error)
    UpdateUser(ctx context.Context, user *models.User) error
    GetByEmailAndRole(ctx context.Context, email string, role models.RoleType) (*models.User, error)
    GetByPhoneNumberAndRole(ctx context.Context, phoneNumber string, role models.RoleType) (*models.User, error)
}

type userRepository struct {
    db             DB
    dbEncryptionKey []byte
}

func NewUserRepository(db DB, dbEncryptionKey []byte) UserRepository {
    return &userRepository{db: db, dbEncryptionKey: dbEncryptionKey}
}

func (r *userRepository) CreateUser(ctx context.Context, user *models.User) error {
    encryptedSecret, err := utils.Encrypt(r.dbEncryptionKey, user.TOTPSecret)
    if err != nil {
        return err
    }
    user.TOTPSecret = encryptedSecret

    query := `
        INSERT INTO users (
            id, email, phone_number, totp_secret,
            created_at, updated_at, auth_provider, roles, is_active
        )
        VALUES ($1, $2, $3, $4, NOW(), NOW(), $5, $6, $7)
    `

    _, execErr := r.db.Exec(ctx, query,
        user.ID, user.Email, user.PhoneNumber, user.TOTPSecret,
        user.AuthProvider, user.Roles, user.IsActive)
    return execErr
}

func (r *userRepository) CreateUserProfile(ctx context.Context, userID uuid.UUID, firstName, lastName string) error {
    query := `
        INSERT INTO user_profiles (id, first_name, last_name, created_at, updated_at)
        VALUES ($1, $2, $3, NOW(), NOW())
    `
    _, err := r.db.Exec(ctx, query, userID, firstName, lastName)
    return err
}

func (r *userRepository) GetByID(ctx context.Context, id uuid.UUID) (*models.User, error) {
    query := `
        SELECT id, email, phone_number, totp_secret, created_at, updated_at,
               auth_provider, roles, is_active
        FROM users
        WHERE id = $1
    `
    row := r.db.QueryRow(ctx, query, id)
    return r.scanUser(row)
}

func (r *userRepository) GetByPhoneNumber(ctx context.Context, phoneNumber string) (*models.User, error) {
    query := `
        SELECT id, email, phone_number, totp_secret, created_at, updated_at,
               auth_provider, roles, is_active
        FROM users
        WHERE phone_number = $1
    `
    row := r.db.QueryRow(ctx, query, phoneNumber)
    return r.scanUser(row)
}

func (r *userRepository) GetByEmail(ctx context.Context, email string) (*models.User, error) {
    query := `
        SELECT id, email, phone_number, totp_secret, created_at, updated_at,
               auth_provider, roles, is_active
        FROM users
        WHERE email = $1
    `
    row := r.db.QueryRow(ctx, query, email)
    return r.scanUser(row)
}

func (r *userRepository) GetByEmailAndRole(ctx context.Context, email string, role models.RoleType) (*models.User, error) {
    query := `
        SELECT id, email, phone_number, totp_secret, created_at, updated_at,
               auth_provider, roles, is_active
        FROM users
        WHERE email = $1 AND $2 = ANY(roles)
    `
    row := r.db.QueryRow(ctx, query, email, role)
    return r.scanUser(row)
}

func (r *userRepository) GetByPhoneNumberAndRole(ctx context.Context, phoneNumber string, role models.RoleType) (*models.User, error) {
    query := `
        SELECT id, email, phone_number, totp_secret, created_at, updated_at,
               auth_provider, roles, is_active
        FROM users
        WHERE phone_number = $1 AND $2 = ANY(roles)
    `
    row := r.db.QueryRow(ctx, query, phoneNumber, role)
    return r.scanUser(row)
}

func (r *userRepository) UpdateUser(ctx context.Context, user *models.User) error {
    query := `
        UPDATE users
        SET email = $1,
            phone_number = $2,
            updated_at = NOW(),
            roles = $3,
            is_active = $4
        WHERE id = $5
    `
    _, err := r.db.Exec(ctx, query,
        user.Email, user.PhoneNumber, user.Roles, user.IsActive, user.ID)
    return err
}

func (r *userRepository) scanUser(row pgx.Row) (*models.User, error) {
    var user models.User
    var email, phoneNumber, totpSecret *string

    err := row.Scan(
        &user.ID,
        &email,
        &phoneNumber,
        &totpSecret,
        &user.CreatedAt,
        &user.UpdatedAt,
        &user.AuthProvider,
        &user.Roles,
        &user.IsActive,
    )
    if err != nil {
        return nil, err
    }

    if email != nil {
        user.Email = email
    }
    if phoneNumber != nil {
        user.PhoneNumber = phoneNumber
    }
    if totpSecret != nil {
        decryptedSecret, err := utils.Decrypt(r.dbEncryptionKey, *totpSecret)
        if err != nil {
            return nil, err
        }
        user.TOTPSecret = decryptedSecret
    }

    return &user, nil
}


## pm_repository.go

package repositories

import (
    "context"

    "github.com/poofdev/poof-back-models"
)

type PropertyManagerRepository interface {
    Create(ctx context.Context, pm *models.PropertyManager) error
}

type propertyManagerRepository struct {
    db DB
}

func NewPropertyManagerRepository(db DB) PropertyManagerRepository {
    return &propertyManagerRepository{db: db}
}

func (r *propertyManagerRepository) Create(ctx context.Context, pm *models.PropertyManager) error {
    query := `
        INSERT INTO property_managers (
            id, business_name, business_address, city, state, zip_code, persona_inquiry_id
        )
        VALUES ($1, $2, $3, $4, $5, $6, $7)
    `
    _, err := r.db.Exec(ctx, query,
        pm.ID, pm.BusinessName, pm.BusinessAddress, pm.City, pm.State, pm.ZipCode, pm.PersonaVerificationInquiryID)
    return err
}


## token_repository.go

package repositories

import (
    "context"
    "time"
    "crypto/sha256"
    "encoding/base64"

    "github.com/poofdev/poof-back-models"
    "github.com/google/uuid"
)

type TokenRepository interface {
    // Refresh Tokens
    CreateRefreshToken(ctx context.Context, token *models.RefreshToken) error
    GetRefreshToken(ctx context.Context, tokenString string) (*models.RefreshToken, error)
    RevokeRefreshToken(ctx context.Context, id uuid.UUID) error

    // Blacklisted Tokens
    BlacklistToken(ctx context.Context, tokenID string, expiresAt time.Time) error
    IsTokenBlacklisted(ctx context.Context, tokenID string) (bool, error)
}

type tokenRepository struct {
    db DB
}

func NewTokenRepository(db DB) TokenRepository {
    return &tokenRepository{db: db}
}

// Refresh Token Methods

func (r *tokenRepository) CreateRefreshToken(ctx context.Context, token *models.RefreshToken) error {
    query := `
        INSERT INTO refresh_tokens (id, user_id, refresh_token, expires_at, created_at, revoked, ip_address)
        VALUES ($1, $2, $3, $4, NOW(), $5, $6)
    `
    _, err := r.db.Exec(ctx, query,
        token.ID, token.UserID, token.Token, token.ExpiresAt, token.Revoked, token.IPAddress)
    return err
}

func (r *tokenRepository) GetRefreshToken(ctx context.Context, rawToken string) (*models.RefreshToken, error) {
    // Hash the provided token
    hasher := sha256.New()
    hasher.Write([]byte(rawToken))
    hashedToken := base64.URLEncoding.EncodeToString(hasher.Sum(nil))

    // Query the database for the hashed token
    query := `
        SELECT id, user_id, refresh_token, expires_at, created_at, revoked, ip_address
        FROM refresh_tokens
        WHERE refresh_token = $1
    `
    row := r.db.QueryRow(ctx, query, hashedToken)

    var token models.RefreshToken
    err := row.Scan(
        &token.ID,
        &token.UserID,
        &token.Token,
        &token.ExpiresAt,
        &token.CreatedAt,
        &token.Revoked,
        &token.IPAddress,
    )
    if err != nil {
        return nil, err
    }
    return &token, nil
}

func (r *tokenRepository) RevokeRefreshToken(ctx context.Context, id uuid.UUID) error {
    query := `
        UPDATE refresh_tokens
        SET revoked = TRUE
        WHERE id = $1
    `
    _, err := r.db.Exec(ctx, query, id)
    return err
}

// Blacklisted Token Methods

func (r *tokenRepository) BlacklistToken(ctx context.Context, tokenID string, expiresAt time.Time) error {
    query := `
        INSERT INTO blacklisted_tokens (id, token_id, expires_at, created_at)
        VALUES ($1, $2, $3, NOW())
    `
    _, err := r.db.Exec(ctx, query, uuid.New(), tokenID, expiresAt)
    return err
}

func (r *tokenRepository) IsTokenBlacklisted(ctx context.Context, tokenID string) (bool, error) {
    query := `
        SELECT EXISTS (
            SELECT 1
            FROM blacklisted_tokens
            WHERE token_id = $1 AND expires_at > NOW()
        )
    `
    var exists bool
    err := r.db.QueryRow(ctx, query, tokenID).Scan(&exists)
    return exists, err
}


Okay analyze and process why my docker container for my auth service appears to be failing at the following error:
▷ docker logs poof_back_auth
2024/12/29 22:00:49 Failed to decode PEM block containing the private key
=======
}
>>>>>>> Stashed changes
