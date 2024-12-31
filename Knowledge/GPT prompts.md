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
├── internal
│   ├── app
│   │   └── app.go
│   ├── controllers
│   │   ├── auth_controller.go
│   │   ├── auth_controller_test.go
│   │   ├── health_controller.go
│   │   ├── registration_controller.go
│   │   └── verification_controller.go
│   ├── dtos
│   │   ├── login_pm_request.go
│   │   ├── login_worker_request.go
│   │   ├── register_pm_request.go
│   │   └── register_worker_request.go
│   ├── integration_test
│   │   └── auth_flow_test.go
│   ├── services
│   │   ├── auth_service.go
│   │   ├── auth_service_test.go
│   │   ├── email_service.go
│   │   ├── jwt_service.go
│   │   ├── jwt_service_test.go
│   │   ├── password_service.go
│   │   ├── registration_service.go
│   │   ├── sms_service.go
│   │   └── sms_service_test.go
│   └── utils
│       └── totp.go
├── Makefile
├── migrations
│   ├── 000001_initial.up.sql
│   └── README.md
├── README.md
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



## .gitignore

codebase_content.txt
install/


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

  # ----------------------
  # 2) Auth Service
  # ----------------------
  poof-back-auth:
    build: .
    container_name: poof_back_auth
    env_file:
      - .env
    # The line below ensures that from inside the container,
    # any references to `AUTH_SERVICE_URL` are set to
    # http://poof-back-auth:8082 (Docker's internal network)
    ports:
      - "8082:8082"
    depends_on:
      - db
    # Healthcheck so other containers can wait until it's truly ready
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8082/health"]
      interval: 5s
      timeout: 2s
      retries: 5

  # ----------------------
  # 3) Migration Container
  # ----------------------
  # This re-uses the same Docker build context, so it has
  # the code + scripts available. We override the command to run migrations.
  migrate:
    build:
      context: .
      dockerfile: Dockerfile.migrate
    container_name: poof_back_auth_migrate
    depends_on:
      - db
    env_file:
      - .env

  # ----------------------
  # 4) Integration Test Container
  # ----------------------
  # Runs your integration tests, pointing to 'poof-back-auth' service.
  poof-back-auth-test:
    build: .
    container_name: poof_back_auth_test
    depends_on:
      poof-back-auth:
        condition: service_healthy
    env_file:
      - .env
    command: sh -c "
      echo '[Integration Tests] Running against poof-back-auth...' &&
      go test -v ./internal/integration_test/... &&
      echo 'Integration tests finished successfully.'"

  # ----------------------
  # 5) Optional: Unit Test Container
  # ----------------------
  # In case you want to run short (unit) tests inside Docker as well.
  poof-back-auth-unit-tests:
    build: .
    container_name: poof_back_auth_unit_tests
    depends_on:
      - db
    env_file:
      - .env
    command: sh -c "
      echo '[Unit Tests] Running...' &&
      go test -v ./internal/... -short &&
      echo 'Unit tests finished.'"

volumes:
  pgdata:
    driver: local


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

WORKDIR /root/
COPY --from=builder /poof-back-auth .

EXPOSE 8082
CMD ["./poof-back-auth"]


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



## internal/services/auth_service.go

package services

import (
	"context"
	"errors"
    "time"

	"github.com/jackc/pgx/v4"
	"github.com/poofdev/poof-back-auth/config"
	"github.com/poofdev/poof-back-auth/internal/dtos"
	auth_utils "github.com/poofdev/poof-back-auth/internal/utils"
	"github.com/poofdev/poof-back-models"
	"github.com/poofdev/poof-back-repositories"
	"github.com/poofdev/poof-back-utils"
	"github.com/sirupsen/logrus"
)

type AuthService interface {
    RequestSMSCode(ctx context.Context, phoneNumber string) error
    AuthenticateSMS(ctx context.Context, phoneNumber, code string, ipAddress string) (*models.User, string, string, error)
    RequestEmailCode(ctx context.Context, email string) error
    AuthenticateEmail(ctx context.Context, email, code string, ipAddress string) (*models.User, string, string, error)
    UserExistsByEmail(ctx context.Context, email string) (bool, error)
    UserExistsByPhone(ctx context.Context, phoneNumber string) (bool, error)
    LoginPropertyManager(ctx context.Context, req dtos.LoginPMRequest, ipAddress string) (*models.User, string, string, error)
    LoginWorker(ctx context.Context, req dtos.LoginWorkerRequest, ipAddress string) (*models.User, string, string, error)
    RefreshToken(ctx context.Context, refreshToken string, ipAddress string) (string, string, error)
    Logout(ctx context.Context, refreshToken string) error
}

type authService struct {
    userRepo      repositories.UserRepository
    tokenRepo     repositories.TokenRepository
    loginAttemptsRepo repositories.LoginAttemptsRepository
    jwtService    JWTService
    smsService    SMSService
    emailService  EmailService
    cfg           *config.Config
}

func NewAuthService(
    userRepo repositories.UserRepository,
    tokenRepo repositories.TokenRepository,
    loginAttemptsRepo repositories.LoginAttemptsRepository,
    jwtService JWTService,
    smsService SMSService,
    emailService EmailService,
    cfg *config.Config,
) AuthService {
    return &authService{
        userRepo:      userRepo,
        tokenRepo:     tokenRepo,
        loginAttemptsRepo: loginAttemptsRepo,
        jwtService:    jwtService,
        smsService:    smsService,
        emailService:  emailService,
        cfg:           cfg,
    }
}

func (a *authService) RequestSMSCode(ctx context.Context, phoneNumber string) error {
    return a.smsService.GenerateAndSendCode(ctx, phoneNumber)
}

func (a *authService) AuthenticateSMS(ctx context.Context, phoneNumber, code string, ipAddress string) (*models.User, string, string, error) {
    valid, err := a.smsService.VerifyCode(ctx, phoneNumber, code)
    if err != nil || !valid {
        utils.Logger.WithError(err).WithFields(logrus.Fields{
            "phone_number": phoneNumber,
            "error":        err,
        }).Error("Invalid verification code attempt")
        return nil, "", "", errors.New("authentication failed")
    }

    user, err := a.userRepo.GetByPhoneNumber(ctx, phoneNumber)
    if err != nil {
        if err == pgx.ErrNoRows {
            return nil, "", "", errors.New("user not found, please sign up")
        } else {
            utils.Logger.WithError(err).Error("Failed to fetch user")
            return nil, "", "", errors.New("internal server error")
        }
    }

    accessToken, err := a.jwtService.GenerateAccessToken(user, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate access token")
        return nil, "", "", errors.New("token generation failed")
    }

    refreshToken, err := a.jwtService.GenerateRefreshToken(user.ID, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate refresh token")
        return nil, "", "", errors.New("token generation failed")
    }

    return user, accessToken, refreshToken.Token, nil
}

func (a *authService) RefreshToken(ctx context.Context, refreshTokenString string, ipAddress string) (string, string, error) {
    refreshToken, err := a.tokenRepo.GetRefreshToken(ctx, refreshTokenString)
    if err != nil || refreshToken.Revoked {
        utils.Logger.WithError(err).Error("Invalid refresh token")
        return "", "", errors.New("invalid refresh token")
    }

    if refreshToken.IsExpired() {
        utils.Logger.Error("refresh_token_expired")
        return "", "", errors.New("refresh token expired")
    }

    // Check if IP addresses match
    if refreshToken.IPAddress != ipAddress {
        utils.Logger.Error("IP address mismatch for refresh token")
        return "", "", errors.New("invalid refresh token")
    }

    user, err := a.userRepo.GetByID(ctx, refreshToken.UserID)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to fetch user")
        return "", "", errors.New("internal server error")
    }

    // Revoke old refresh token
    err = a.tokenRepo.RevokeRefreshToken(ctx, refreshToken.ID)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to revoke refresh token")
        return "", "", errors.New("internal server error")
    }

    // Issue new tokens
    accessToken, err := a.jwtService.GenerateAccessToken(user, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate access token")
        return "", "", errors.New("token generation failed")
    }

    newRefreshToken, err := a.jwtService.GenerateRefreshToken(user.ID, ipAddress)
    if err != nil {
        return "", "", err
    }

    return accessToken, newRefreshToken.Token, nil
}

func (a *authService) Logout(ctx context.Context, refreshTokenString string) error {
    refreshToken, err := a.tokenRepo.GetRefreshToken(ctx, refreshTokenString)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to fetch refresh token")
        return errors.New("internal server error")
    }

    // Revoke the refresh token
    err = a.tokenRepo.RevokeRefreshToken(ctx, refreshToken.ID)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to revoke refresh token")
        return errors.New("internal server error")
    }

    return nil
}

func (a *authService) RequestEmailCode(ctx context.Context, email string) error {
    return a.emailService.GenerateAndSendCode(ctx, email)
}

func (a *authService) AuthenticateEmail(ctx context.Context, email, code string, ipAddress string) (*models.User, string, string, error) {
    valid, err := a.emailService.VerifyCode(ctx, email, code)
    if err != nil || !valid {
        // Handle error and logging
        return nil, "", "", errors.New("authentication failed")
    }

    user, err := a.userRepo.GetByEmail(ctx, email)
    if err != nil {
        if err == pgx.ErrNoRows {
            return nil, "", "", errors.New("user not found, please sign up") 
        } else {
            // Handle error and logging
            return nil, "", "", errors.New("internal server error")
        }
    }

    accessToken, err := a.jwtService.GenerateAccessToken(user, ipAddress)
    if err != nil {
        // Handle error and logging
        return nil, "", "", errors.New("token generation failed")
    }

    refreshToken, err := a.jwtService.GenerateRefreshToken(user.ID, ipAddress)
    if err != nil {
        // Handle error and logging
        return nil, "", "", errors.New("token generation failed")
    }

    return user, accessToken, refreshToken.Token, nil
}

func (a *authService) UserExistsByEmail(ctx context.Context, email string) (bool, error) {
    user, err := a.userRepo.GetByEmail(ctx, email)
    if err != nil {
        // If the DB returns no rows, user doesn't exist
        if err == pgx.ErrNoRows {
            return false, nil
        }
        return false, err
    }
    // If user is not nil, we have a match
    return user != nil, nil
}

func (a *authService) UserExistsByPhone(ctx context.Context, phoneNumber string) (bool, error) {
    user, err := a.userRepo.GetByPhoneNumber(ctx, phoneNumber)
    if err != nil {
        if err == pgx.ErrNoRows {
            return false, nil
        }
        return false, err
    }
    return user != nil, nil
}

func (a *authService) LoginPropertyManager(ctx context.Context, req dtos.LoginPMRequest, ipAddress string) (*models.User, string, string, error) {
    user, err := a.userRepo.GetByEmailAndRole(ctx, req.Email, models.RolePropertyManager)
    if err != nil {
        return nil, "", "", errors.New("invalid credentials")
    }

    // Rate-limiting check
    locked, lockedUntil, err := a.loginAttemptsRepo.IsLocked(ctx, user.ID)
    if err == nil && locked {
        return nil, "", "", errors.New("account locked until " + lockedUntil.Format(time.RFC3339))
    }

    if !auth_utils.ValidateTOTPCode(user.TOTPSecret, req.TOTPCode) {
        // Increment on invalid TOTP
        _ = a.loginAttemptsRepo.Increment(
            ctx,
            user.ID,
            a.cfg.LockDuration,     // from config
            a.cfg.AttemptWindow,    // from config
            a.cfg.MaxLoginAttempts, // from config
        )
        return nil, "", "", errors.New("invalid credentials")
    }

    // If valid TOTP => reset attempts
    _ = a.loginAttemptsRepo.Reset(ctx, user.ID)

    // Generate tokens
    accessToken, err := a.jwtService.GenerateAccessToken(user, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate access token")
        return nil, "", "", errors.New("token generation failed")
    }

    refreshToken, err := a.jwtService.GenerateRefreshToken(user.ID, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate refresh token")
        return nil, "", "", errors.New("token generation failed")
    }

    return user, accessToken, refreshToken.Token, nil
}

func (a *authService) LoginWorker(ctx context.Context, req dtos.LoginWorkerRequest, ipAddress string) (*models.User, string, string, error) {
    user, err := a.userRepo.GetByPhoneNumberAndRole(ctx, req.PhoneNumber, models.RoleWorker)
    if err != nil {
        return nil, "", "", errors.New("invalid credentials")
    }

    // Rate-limiting check
    locked, lockedUntil, err := a.loginAttemptsRepo.IsLocked(ctx, user.ID)
    if err == nil && locked {
        return nil, "", "", errors.New("account locked until " + lockedUntil.Format(time.RFC3339))
    }

    if !auth_utils.ValidateTOTPCode(user.TOTPSecret, req.TOTPCode) {
        _ = a.loginAttemptsRepo.Increment(
            ctx,
            user.ID,
            a.cfg.LockDuration,
            a.cfg.AttemptWindow,
            a.cfg.MaxLoginAttempts,
        )
        return nil, "", "", errors.New("invalid credentials")
    }

    // If valid => reset attempts
    _ = a.loginAttemptsRepo.Reset(ctx, user.ID)

    // Generate tokens
    accessToken, err := a.jwtService.GenerateAccessToken(user, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate access token")
        return nil, "", "", errors.New("token generation failed")
    }

    refreshToken, err := a.jwtService.GenerateRefreshToken(user.ID, ipAddress)
    if err != nil {
        utils.Logger.WithError(err).Error("Failed to generate refresh token")
        return nil, "", "", errors.New("token generation failed")
    }

    return user, accessToken, refreshToken.Token, nil
}


## internal/services/registration_service.go

package services

import (
    "context"
    "errors"
    "net"
    "net/mail"
    "regexp"
    "strings"
    "time"

    "github.com/google/uuid"
    "github.com/poofdev/poof-back-auth/internal/dtos"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-repositories"
)

type RegistrationService interface {
    RegisterPropertyManager(ctx context.Context, req dtos.RegisterPMRequest) error
    RegisterWorker(ctx context.Context, req dtos.RegisterWorkerRequest) error
    ValidateEmail(email string) error
    ValidatePhoneNumber(phoneNumber string) error
}

type registrationService struct {
    userRepo            repositories.UserRepository
    propertyManagerRepo repositories.PropertyManagerRepository
    workerRepo          repositories.WorkerRepository
}

func NewRegistrationService(
    userRepo repositories.UserRepository,
    propertyManagerRepo repositories.PropertyManagerRepository,
    workerRepo repositories.WorkerRepository,
) RegistrationService {
    return &registrationService{
        userRepo:            userRepo,
        propertyManagerRepo: propertyManagerRepo,
        workerRepo:          workerRepo,
    }
}

func (s *registrationService) RegisterPropertyManager(ctx context.Context, req dtos.RegisterPMRequest) error {
    existingUser, _ := s.userRepo.GetByEmailAndRole(ctx, req.Email, models.RolePropertyManager)
    if existingUser != nil {
        return errors.New("email already in use")
    }

    if req.PhoneNumber != nil {
        existingUser, _ := s.userRepo.GetByPhoneNumberAndRole(ctx, *req.PhoneNumber, models.RolePropertyManager)
        if existingUser != nil {
            return errors.New("phone number already in use")
        }
    }

    user := &models.User{
        ID:           uuid.New(),
        Email:        &req.Email,
        PhoneNumber:  req.PhoneNumber,
        TOTPSecret:   req.TOTPSecret,
        AuthProvider: "email_verification",
        Roles:        []models.RoleType{models.RolePropertyManager},
        IsActive:     true,
        CreatedAt:    time.Now(),
        UpdatedAt:    time.Now(),
    }

    err := s.userRepo.CreateUser(ctx, user)
    if err != nil {
        return err
    }

    // Create user profile
    err = s.userRepo.CreateUserProfile(ctx, user.ID, req.FirstName, req.LastName)
    if err != nil {
        return err
    }

    propertyManager := &models.PropertyManager{
        ID:                          user.ID,
        BusinessName:                req.BusinessName,
        BusinessAddress:             req.BusinessAddress,
        City:                        req.City,
        State:                       req.State,
        ZipCode:                     req.ZipCode,
        PersonaVerificationInquiryID: req.PersonaInquiryID,
    }

    err = s.propertyManagerRepo.Create(ctx, propertyManager)
    if err != nil {
        return err
    }

    return nil
}

func (s *registrationService) RegisterWorker(ctx context.Context, req dtos.RegisterWorkerRequest) error {
    existingUser, _ := s.userRepo.GetByEmailAndRole(ctx, req.Email, models.RoleWorker)
    if existingUser != nil {
        return errors.New("email already in use")
    }

    existingUser, _ = s.userRepo.GetByPhoneNumberAndRole(ctx, req.PhoneNumber, models.RoleWorker)
    if existingUser != nil {
        return errors.New("phone number already in use")
    }

    user := &models.User{
        ID:           uuid.New(),
        Email:        &req.Email,
        PhoneNumber:  &req.PhoneNumber,
        AuthProvider: "email_verification",
        Roles:        []models.RoleType{models.RoleWorker},
        IsActive:     true,
        CreatedAt:    time.Now(),
        UpdatedAt:    time.Now(),
    }

    err := s.userRepo.CreateUser(ctx, user)
    if err != nil {
        return err
    }

    // Create user profile
    err = s.userRepo.CreateUserProfile(ctx, user.ID, req.FirstName, req.LastName)
    if err != nil {
        return err
    }

    worker := &models.Worker{
        ID:                             user.ID,
        FirstName:                      req.FirstName,
        LastName:                       req.LastName,
        StreetAddress:                  req.StreetAddress,
        AptSuite:                       req.AptSuite,
        City:                           req.City,
        State:                          req.State,
        ZipCode:                        req.ZipCode,
        VehicleYear:                    req.VehicleYear,
        VehicleMake:                    req.VehicleMake,
        VehicleModel:                   req.VehicleModel,
        PersonaVerificationInquiryID:   req.PersonaInquiryID,
    }

    err = s.workerRepo.Create(ctx, worker)
    if err != nil {
        return err
    }

    return nil
}

func (s *registrationService) ValidateEmail(email string) error {
    _, err := mail.ParseAddress(email)
    if err != nil {
        return err
    }

    domain := email[strings.LastIndex(email, "@")+1:]
    mxRecords, err := net.LookupMX(domain)
    if err != nil || len(mxRecords) == 0 {
        return errors.New("email domain has no MX records")
    }

    return nil
}

func (s *registrationService) ValidatePhoneNumber(phoneNumber string) error {
    re := regexp.MustCompile(`^(\+1[- ]?)?\d{3}[- ]?\d{3}[- ]?\d{4}$`)
    if !re.MatchString(phoneNumber) {
        return errors.New("invalid phone number format")
    }
    return nil
}


## internal/services/jwt_service.go

package services

import (
    "context"
    "errors"
    "time"
    "crypto/rand"
    "math/big"
    "crypto/rsa"
    "net"
    "crypto/sha256"
    "encoding/base64"

    "github.com/golang-jwt/jwt/v5"
    "github.com/google/uuid"
    "github.com/poofdev/poof-back-models"
    "github.com/poofdev/poof-back-auth/config"
    "github.com/poofdev/poof-back-repositories"
)

type JWTService interface {
    GenerateAccessToken(user *models.User, ipAddress string) (string, error)
    GenerateRefreshToken(userID uuid.UUID, ipAddress string) (*models.RefreshToken, error)
    ValidateToken(ctx context.Context, tokenString string, ipAddress string) (*jwt.Token, error)
}

type jwtService struct {
    privateKey    *rsa.PrivateKey
    publicKey     *rsa.PublicKey
    tokenExpiry   time.Duration
    refreshExpiry time.Duration
    tokenRepo     repositories.TokenRepository
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

func (j *jwtService) GenerateAccessToken(user *models.User, ipAddress string) (string, error) {
    parsedIP := net.ParseIP(ipAddress)
    if parsedIP == nil {
        return "", errors.New("invalid IP address")
    }

    tokenID := uuid.New().String()
    claims := jwt.MapClaims{
        "iss":   "poof-service",
        "sub":   user.ID.String(),
        "exp":   time.Now().Add(j.tokenExpiry).Unix(),
        "iat":   time.Now().Unix(),
        "jti":   tokenID,
        "roles": user.Roles,
        "ip":    parsedIP.String(), // Store the validated IP
    }
    token := jwt.NewWithClaims(jwt.SigningMethodRS256, claims)
    signedToken, err := token.SignedString(j.privateKey)
    if err != nil {
        return "", err
    }
    return signedToken, nil
}

func (j *jwtService) GenerateRefreshToken(userID uuid.UUID, ipAddress string) (*models.RefreshToken, error) {
    rawToken := generateSecureToken(64)

    // Hash the token
    hasher := sha256.New()
    hasher.Write([]byte(rawToken))
    hashedToken := base64.URLEncoding.EncodeToString(hasher.Sum(nil))

    expiresAt := time.Now().Add(j.refreshExpiry)

    refreshToken := &models.RefreshToken{
        ID:        uuid.New(),
        UserID:    userID,
        Token:     hashedToken, // Store the hash
        ExpiresAt: expiresAt,
        CreatedAt: time.Now(),
        Revoked:   false,
    }

    // Save the hashed token to the database
    err := j.tokenRepo.CreateRefreshToken(context.Background(), refreshToken)
    if err != nil {
        return nil, err
    }

    // Return the raw token to the caller
    return &models.RefreshToken{
        ID:        refreshToken.ID,
        UserID:    refreshToken.UserID,
        Token:     rawToken, // Return the plain token to the client
        ExpiresAt: refreshToken.ExpiresAt,
        CreatedAt: refreshToken.CreatedAt,
        Revoked:   refreshToken.Revoked,
        IPAddress: ipAddress,
    }, nil
}

func (j *jwtService) ValidateToken(ctx context.Context, tokenString, ipAddress string) (*jwt.Token, error) {
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodRSA); !ok {
            return nil, errors.New("unexpected signing method")
        }
        return j.publicKey, nil
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

// Helper function to generate secure random tokens
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
        panic(err) // Handle appropriately in production code
    }
    return int(n.Int64())
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


## internal/dtos/login_worker_request.go

package dtos

type LoginWorkerRequest struct {
    PhoneNumber string `json:"phone_number" validate:"required"`
    TOTPCode    string `json:"totp_code" validate:"required,len=6,numeric"`
}


## internal/dtos/login_pm_request.go

package dtos

type LoginPMRequest struct {
    Email    string `json:"email" validate:"required,email"`
    TOTPCode string `json:"totp_code" validate:"required,len=6,numeric"`
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
	"encoding/json"
	"net/http"

	"github.com/poofdev/poof-back-auth/config"
	"github.com/poofdev/poof-back-auth/internal/dtos"
	"github.com/poofdev/poof-back-auth/internal/services"
	auth_utils "github.com/poofdev/poof-back-auth/internal/utils"
	"github.com/poofdev/poof-back-utils"
)

type RegistrationController struct {
    registrationService services.RegistrationService
}

func NewRegistrationController(registrationService services.RegistrationService) *RegistrationController {
    return &RegistrationController{
        registrationService: registrationService,
    }
}

// RegisterPropertyManager handles POST /register/property_manager
// RegisterPropertyManager handles POST /register/property_manager
func (c *RegistrationController) RegisterPropertyManager(w http.ResponseWriter, r *http.Request) {
    var req dtos.RegisterPMRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    // Validate request
    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    // Validate TOTP token
    if !auth_utils.ValidateTOTPCode(req.TOTPSecret, req.TOTPToken) {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid TOTP token", nil)
        return
    }

    // Validate email format and MX records
    if err := c.registrationService.ValidateEmail(req.Email); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid email address", err)
        return
    }

    // Validate phone number if provided
    if req.PhoneNumber != nil {
        if err := c.registrationService.ValidatePhoneNumber(*req.PhoneNumber); err != nil {
            utils.RespondWithError(w, http.StatusBadRequest, "Invalid phone number", err)
            return
        }
    }

    // Proceed with registration
    ctx := r.Context()
    err := c.registrationService.RegisterPropertyManager(ctx, req)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Registration failed", err)
        return
    }

    utils.RespondWithJSON(w, http.StatusCreated, map[string]string{"message": "Property manager registered successfully"})
}

// RegisterWorker handles POST /register/worker
func (c *RegistrationController) RegisterWorker(w http.ResponseWriter, r *http.Request) {
    var req dtos.RegisterWorkerRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    // Validate request
    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    // Validate TOTP token
    if !auth_utils.ValidateTOTPCode(req.TOTPSecret, req.TOTPToken) {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid TOTP token", nil)
        return
    }

    // Validate email format and MX records
    if err := c.registrationService.ValidateEmail(req.Email); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid email address", err)
        return
    }

    // Validate phone number
    if err := c.registrationService.ValidatePhoneNumber(req.PhoneNumber); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid phone number", err)
        return
    }

    // Proceed with registration
    ctx := r.Context()
    err := c.registrationService.RegisterWorker(ctx, req)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Registration failed", err)
        return
    }

    utils.RespondWithJSON(w, http.StatusCreated, map[string]string{"message": "Worker registered successfully"})
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


## internal/controllers/auth_controller.go

package controllers

import (
	"encoding/json"
	"net/http"

	"github.com/go-playground/validator/v10"
	"github.com/gorilla/schema"
	"github.com/poofdev/poof-back-auth/config"
	"github.com/poofdev/poof-back-auth/internal/services"
	"github.com/poofdev/poof-back-utils"
	"github.com/poofdev/poof-back-auth/internal/dtos"
)

type AuthController struct {
	config       *config.Config
	authService  services.AuthService
	jwtService   services.JWTService
}

func NewAuthController(cfg *config.Config, authService services.AuthService, jwtService services.JWTService) *AuthController {
	return &AuthController{
		config:       cfg,
		authService:  authService,
		jwtService:   jwtService,
	}
}

var decoder = schema.NewDecoder()
var validate = validator.New()

// RefreshToken handles POST /auth/refresh_token
func (c *AuthController) RefreshToken(w http.ResponseWriter, r *http.Request) {
	var req struct {
		RefreshToken string `json:"refresh_token" validate:"required,len=64"`
	}
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
		return
	}

	if err := validate.Struct(req); err != nil {
		utils.RespondWithError(w, http.StatusBadRequest, "Invalid refresh token", err)
		return
	}

	ipAddress := utils.GetIPAddress(r)

	ctx := r.Context()
	accessToken, newRefreshToken, err := c.authService.RefreshToken(ctx, req.RefreshToken, ipAddress)
	if err != nil {
		utils.Logger.WithError(err).Error("Refresh token failed")
		utils.RespondWithError(w, http.StatusUnauthorized, "Invalid refresh token", err)
		return
	}

	response := map[string]string{
		"access_token":  accessToken,
		"refresh_token": newRefreshToken,
	}
	utils.RespondWithJSON(w, http.StatusOK, response)
}

// Logout handles POST /auth/logout
func (c *AuthController) Logout(w http.ResponseWriter, r *http.Request) {
	var req struct {
		RefreshToken string `json:"refresh_token" validate:"required,len=64"`
	}
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
		return
	}

	if err := validate.Struct(req); err != nil {
		utils.RespondWithError(w, http.StatusBadRequest, "Invalid refresh token", err)
		return
	}

	ctx := r.Context()
	err := c.authService.Logout(ctx, req.RefreshToken)
	if err != nil {
		utils.Logger.WithError(err).Error("Logout failed")
		utils.RespondWithError(w, http.StatusInternalServerError, "Failed to logout", err)
		return
	}

	utils.RespondWithJSON(w, http.StatusOK, map[string]string{"message": "Logged out successfully"})
}

// CheckEmailExists handles POST /auth/exists/email
func (c *AuthController) CheckEmailExists(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Email string `json:"email" validate:"required,email"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    // Validate input
    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    exists, err := c.authService.UserExistsByEmail(r.Context(), req.Email)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Internal error", err)
        return
    }

    // Return { "exists": true/false } in the response
    utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": exists})
}

// CheckPhoneNumberExists handles POST /auth/exists/phone
func (c *AuthController) CheckPhoneNumberExists(w http.ResponseWriter, r *http.Request) {
    var req struct {
        PhoneNumber string `json:"phone_number" validate:"required"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

    exists, err := c.authService.UserExistsByPhone(r.Context(), req.PhoneNumber)
    if err != nil {
        utils.RespondWithError(w, http.StatusInternalServerError, "Internal error", err)
        return
    }

    utils.RespondWithJSON(w, http.StatusOK, map[string]bool{"exists": exists})
}

// LoginPropertyManager handles POST /auth/login/pm
func (c *AuthController) LoginPropertyManager(w http.ResponseWriter, r *http.Request) {
    var req dtos.LoginPMRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    // Validate request
    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

	ipAddress := utils.GetIPAddress(r)

    // Proceed with authentication
    user, accessToken, refreshToken, err := c.authService.LoginPropertyManager(r.Context(), req, ipAddress)
    if err != nil {
        utils.RespondWithError(w, http.StatusUnauthorized, "Authentication failed", err)
        return
    }

    response := map[string]interface{}{
        "user":          user,
        "access_token":  accessToken,
        "refresh_token": refreshToken,
    }
    utils.RespondWithJSON(w, http.StatusOK, response)
}

// LoginWorker handles POST /auth/login/worker
func (c *AuthController) LoginWorker(w http.ResponseWriter, r *http.Request) {
    var req dtos.LoginWorkerRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Invalid request payload", err)
        return
    }

    // Validate request
    if err := validate.Struct(req); err != nil {
        utils.RespondWithError(w, http.StatusBadRequest, "Validation error", err)
        return
    }

	ipAddress := utils.GetIPAddress(r)

    // Proceed with authentication
    user, accessToken, refreshToken, err := c.authService.LoginWorker(r.Context(), req, ipAddress)
    if err != nil {
        utils.RespondWithError(w, http.StatusUnauthorized, "Authentication failed", err)
        return
    }

    response := map[string]interface{}{
        "user":          user,
        "access_token":  accessToken,
        "refresh_token": refreshToken,
    }
    utils.RespondWithJSON(w, http.StatusOK, response)
}

// Helper functions to generate state and nonce
func generateState() string {
	return utils.RandomString(16)
}

func generateNonce() string {
	return utils.RandomString(16)
}


## cmd/main.go

// main.go

package main

import (
    "net/http"

    "github.com/gorilla/mux"
    "github.com/rs/cors"
    "github.com/poofdev/poof-back-auth/config"
    "github.com/poofdev/poof-back-auth/internal/app"
    "github.com/poofdev/poof-back-auth/internal/controllers"
    "github.com/poofdev/poof-back-repositories"
    "github.com/poofdev/poof-back-auth/internal/services"
    "github.com/poofdev/poof-back-utils"
)

func main() {
    // Initialize logger
    utils.InitLogger()

    // Load configuration
    cfg := config.LoadConfig()

    // Initialize application components
    application, err := app.NewApp(cfg)
    if err != nil {
        utils.Logger.Fatal("Failed to initialize the application:", err)
    }
    defer application.Close()

    // Initialize repositories
    userRepo := repositories.NewUserRepository(application.DB, cfg.DBEncryptionKey)
    tokenRepo := repositories.NewTokenRepository(application.DB)
    smsRepo := repositories.NewSMSRepository(application.DB)
    emailRepo := repositories.NewEmailRepository(application.DB)
    propertyManagerRepo := repositories.NewPropertyManagerRepository(application.DB)
    workerRepo := repositories.NewWorkerRepository(application.DB)
    loginAttemptsRepo := repositories.NewLoginAttemptsRepository(application.DB)

    // Initialize services
    jwtService := services.NewJWTService(cfg, tokenRepo)
    smsService := services.NewSMSService(cfg, smsRepo)
    emailService := services.NewEmailService(cfg, emailRepo)
    if err != nil {
        utils.Logger.Fatal("Failed to initialize OpenID service:", err)
    }
    authService := services.NewAuthService(userRepo, tokenRepo, loginAttemptsRepo, jwtService, smsService, emailService, cfg)
    registrationService := services.NewRegistrationService(userRepo, propertyManagerRepo, workerRepo)

    // Initialize controllers
    authController := controllers.NewAuthController(cfg, authService, jwtService)
    healthController := controllers.NewHealthController(application)
    registrationController := controllers.NewRegistrationController(registrationService)
    verfiyController := controllers.NewVerificationController(smsService, emailService)

    // Set up router
    router := mux.NewRouter()

    // Health check route (add before other routes to catch /health early)
    router.HandleFunc("/health", healthController.HealthCheckHandler).Methods("GET")

    // Public routes
    authRouter := router.PathPrefix("/auth").Subrouter()
    authRouter.HandleFunc("/refresh_token", authController.RefreshToken).Methods("POST")
    authRouter.HandleFunc("/logout", authController.Logout).Methods("POST")
    authRouter.HandleFunc("/register/totp_secret", registrationController.GenerateTOTPSecret).Methods("POST")
    authRouter.HandleFunc("/verify/request_sms_code", verfiyController.RequestSMSCode).Methods("POST")
    authRouter.HandleFunc("/verify/check_sms_code", verfiyController.VerifySMSCode).Methods("POST")
    authRouter.HandleFunc("/verify/request_email_code", verfiyController.RequestEmailCode).Methods("POST")
    authRouter.HandleFunc("/verify/check_email_code", verfiyController.VerifyEmailCode).Methods("POST")
    authRouter.HandleFunc("/register/pm", registrationController.RegisterPropertyManager).Methods("POST")
    authRouter.HandleFunc("/register/worker", registrationController.RegisterWorker).Methods("POST")    
    authRouter.HandleFunc("/exists/email", authController.CheckEmailExists).Methods("POST")
    authRouter.HandleFunc("/exists/phone", authController.CheckPhoneNumberExists).Methods("POST")
    authRouter.HandleFunc("/login/pm", authController.LoginPropertyManager).Methods("POST")
    authRouter.HandleFunc("/login/worker", authController.LoginWorker).Methods("POST")

    // CORS configuration
    c := cors.New(cors.Options{
        AllowedOrigins:   []string{"https://poof.com"}, // Update with allowed origins
        AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
        AllowedHeaders:   []string{"Authorization", "Content-Type"},
        AllowCredentials: true,
    })

    handler := c.Handler(router)

    // Start server, fly takes care of TLS/SSL with a reverse proxy, so we can serve HTTP
    utils.Logger.Info("Starting server on port 8082")
    if err := http.ListenAndServe(":8082", handler); err != nil {
        utils.Logger.Fatal("Failed to start server:", err)
    }
}


## migrations/000001_initial.up.sql

-- Auth Users Table (authentication-related info only)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE,
    phone_number VARCHAR(20) UNIQUE,
    auth_provider VARCHAR(20) NOT NULL, -- e.g., "email", "sms_2fa"
    totp_secret VARCHAR(255),
    roles VARCHAR(50)[] NOT NULL DEFAULT ARRAY['tenant'],
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_phone_number ON users(phone_number);

-- User Profiles Table (account-related info managed by user microservice)
CREATE TABLE user_profiles (
    id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Property Managers Table (still references users(id))
CREATE TABLE property_managers (
    id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    business_name VARCHAR(255) NOT NULL,
    business_address VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(50) NOT NULL,
    zip_code VARCHAR(20) NOT NULL,
    persona_inquiry_id VARCHAR(255) NOT NULL
);

-- Properties Table
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

-- Property Buildings
CREATE TABLE property_buildings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID REFERENCES properties(id) ON DELETE CASCADE,
    building_name VARCHAR(100),
    address VARCHAR(255) NOT NULL,
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6)
);

CREATE INDEX idx_property_buildings_property_id ON property_buildings(property_id);

-- Units Table
CREATE TABLE units (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID REFERENCES properties(id) ON DELETE CASCADE,
    unit_number VARCHAR(50) NOT NULL,
    tenant_token VARCHAR(255) UNIQUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_units_tenant_token ON units(tenant_token);
CREATE INDEX idx_units_property_id ON units(property_id);

-- Trash Compactors Table
CREATE TABLE trash_compactors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    property_id UUID REFERENCES properties(id) ON DELETE CASCADE,
    latitude DECIMAL(9,6) NOT NULL,
    longitude DECIMAL(9,6) NOT NULL
);

CREATE INDEX idx_trash_compactors_property_id ON trash_compactors(property_id);

-- Workers Table (references users(id))
CREATE TABLE workers (
    id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    street_address VARCHAR(255) NOT NULL,
    apt_suite VARCHAR(50),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(50) NOT NULL,
    zip_code VARCHAR(20) NOT NULL,
    vehicle_year INT NOT NULL,
    vehicle_make VARCHAR(100) NOT NULL,
    vehicle_model VARCHAR(100) NOT NULL,
    persona_inquiry_id VARCHAR(255) NOT NULL
);

-- Refresh Tokens Table
CREATE TABLE refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    refresh_token VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    revoked BOOLEAN DEFAULT FALSE,
    ip_address VARCHAR(45)
);

CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);

-- SMS Verification Codes Table
CREATE TABLE sms_verification_codes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone_number VARCHAR(20) NOT NULL,
    verification_code VARCHAR(10) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    attempts INTEGER DEFAULT 0
);

CREATE INDEX idx_sms_codes_phone_number ON sms_verification_codes(phone_number);

-- Email Verification Codes Table
CREATE TABLE email_verification_codes (
    id UUID PRIMARY KEY,
    email TEXT NOT NULL,
    verification_code TEXT NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    attempts INT DEFAULT 0,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_email_verification_codes_email ON email_verification_codes(email);

-- Blacklisted Tokens Table
CREATE TABLE blacklisted_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    token_id VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_blacklisted_tokens_token_id ON blacklisted_tokens(token_id);

-- Track login attempts
CREATE TABLE login_attempts (
    user_id UUID PRIMARY KEY,
    attempt_count INT NOT NULL DEFAULT 0,
    locked_until TIMESTAMP WITH TIME ZONE,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


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

----------------------------------------------
####  Utils Package #####

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


---------------------------------------------
#### Repositories Package ####

## worker_repository.go

package repositories

import (
    "context"

    "github.com/poofdev/poof-back-models"
)

type WorkerRepository interface {
    Create(ctx context.Context, worker *models.Worker) error
}

type workerRepository struct {
    db DB
}

func NewWorkerRepository(db DB) WorkerRepository {
    return &workerRepository{db: db}
}

func (r *workerRepository) Create(ctx context.Context, worker *models.Worker) error {
    query := `
        INSERT INTO workers (
            id, first_name, last_name, street_address, apt_suite, city, state, zip_code,
            vehicle_year, vehicle_make, vehicle_model, persona_inquiry_id
        )
        VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12)
    `
    _, err := r.db.Exec(ctx, query,
        worker.ID,
        worker.FirstName,
        worker.LastName,
        worker.StreetAddress,
        worker.AptSuite,
        worker.City,
        worker.State,
        worker.ZipCode,
        worker.VehicleYear,
        worker.VehicleMake,
        worker.VehicleModel,
        worker.PersonaVerificationInquiryID,
    )
    return err
}


## login_attempts_repository.go

package repositories

import (
    "context"
    "time"

    "github.com/google/uuid"
)

type LoginAttempts struct {
    UserID       uuid.UUID
    AttemptCount int
    LockedUntil  *time.Time
    UpdatedAt    time.Time
    CreatedAt    time.Time
}

type LoginAttemptsRepository interface {
    GetOrCreate(ctx context.Context, userID uuid.UUID) (*LoginAttempts, error)
    Increment(ctx context.Context, userID uuid.UUID, lockDuration, window time.Duration, maxAttempts int) error
    Reset(ctx context.Context, userID uuid.UUID) error
    IsLocked(ctx context.Context, userID uuid.UUID) (bool, time.Time, error)
}

type loginAttemptsRepository struct {
    db DB
}

func NewLoginAttemptsRepository(db DB) LoginAttemptsRepository {
    return &loginAttemptsRepository{db: db}
}

func (r *loginAttemptsRepository) GetOrCreate(ctx context.Context, userID uuid.UUID) (*LoginAttempts, error) {
    // Attempt to get row
    query := `
        SELECT user_id, attempt_count, locked_until, updated_at, created_at
        FROM login_attempts
        WHERE user_id = $1
    `
    row := r.db.QueryRow(ctx, query, userID)
    
    la := &LoginAttempts{}
    err := row.Scan(
        &la.UserID,
        &la.AttemptCount,
        &la.LockedUntil,
        &la.UpdatedAt,
        &la.CreatedAt,
    )
    if err == nil {
        return la, nil
    }
    
    // If no record exists, insert a fresh one
    insert := `
        INSERT INTO login_attempts (user_id, attempt_count, locked_until, updated_at, created_at)
        VALUES ($1, 0, NULL, NOW(), NOW())
        RETURNING user_id, attempt_count, locked_until, updated_at, created_at
    `
    row = r.db.QueryRow(ctx, insert, userID)
    if err := row.Scan(
        &la.UserID,
        &la.AttemptCount,
        &la.LockedUntil,
        &la.UpdatedAt,
        &la.CreatedAt,
    ); err != nil {
        return nil, err
    }
    return la, nil
}

func (r *loginAttemptsRepository) Increment(
    ctx context.Context,
    userID uuid.UUID,
    lockDuration, window time.Duration,
    maxAttempts int,
) error {
    // We'll do a one-shot query approach to:
    // 1) if the user is locked, do nothing
    // 2) if not locked, increment attempts
    // 3) if attempts >= max within window => set locked_until
    query := `
WITH current AS (
    SELECT user_id,
           attempt_count,
           locked_until,
           updated_at
    FROM login_attempts
    WHERE user_id = $1
    FOR UPDATE
)
UPDATE login_attempts
SET 
  attempt_count = CASE 
    WHEN (
      locked_until IS NOT NULL 
      AND locked_until > NOW()
    ) 
    THEN attempt_count  -- still locked, do not increment 
    ELSE CASE
      WHEN (NOW() - current.updated_at) > $3 
        THEN 1 -- if outside the rolling window, reset to 1
      ELSE attempt_count + 1
    END
  END,
  locked_until = CASE 
    WHEN (
      locked_until IS NOT NULL 
      AND locked_until > NOW()
    ) 
    THEN locked_until  -- preserve existing lock
    ELSE CASE
      WHEN (
        (NOW() - current.updated_at) <= $3 
        AND (current.attempt_count + 1) >= $4
      )
      THEN NOW() + $2 -- lock out for lockDuration
      ELSE NULL
    END
  END,
  updated_at = NOW()
FROM current
WHERE login_attempts.user_id = current.user_id
RETURNING login_attempts.user_id
    `
    // Explanation of parameters:
    // $1 -> userID
    // $2 -> lockDuration (10 minutes)
    // $3 -> window (5 minutes)
    // $4 -> maxAttempts (10)
    _, err := r.db.Exec(ctx, query, userID, lockDuration, window, maxAttempts)
    return err
}

func (r *loginAttemptsRepository) Reset(ctx context.Context, userID uuid.UUID) error {
    query := `
UPDATE login_attempts
SET attempt_count = 0,
    locked_until = NULL,
    updated_at = NOW()
WHERE user_id = $1
    `
    _, err := r.db.Exec(ctx, query, userID)
    return err
}

func (r *loginAttemptsRepository) IsLocked(ctx context.Context, userID uuid.UUID) (bool, time.Time, error) {
    query := `
        SELECT locked_until
        FROM login_attempts
        WHERE user_id = $1
    `
    row := r.db.QueryRow(ctx, query, userID)
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



## .gitignore

codebase_content.txt


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
