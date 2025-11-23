# Docker Compose: PostgreSQL, pgAdmin, and Keycloak

**⚠️ Do not use in production**  
The configuration is not secure and is intended for local development use only.

This setup runs Keycloak using Docker Compose alongside PostgreSQL and pgAdmin.  
[DockerComposeWithPgAdminPostgresqlKeycloak.yml](DockerComposeWithPgAdminPostgresqlKeycloak.yml)

## Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Keycloak Configuration](#keycloak-configuration)
- [Spring Boot Configuration](#spring-boot-configuration)

## Features
- **PostgreSQL 16**: Database for both Keycloak and the application.
- **pgAdmin**: Web-based UI for managing PostgreSQL.
- **Keycloak**: Identity and Access Management server.

## Prerequisites

### 1. PFX Certificate
Keycloak in this setup requires a PFX (PKCS#12) file for HTTPS/TLS.

1. **Generate the file** (example using OpenSSL):
   ```bash
   # Generate certificate and key
   openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem -subj "/CN=localhost"
   # Export to PFX (password must be 'password')
   openssl pkcs12 -export -out keycloak.pfx -inkey key.pem -in certificate.pem -passout pass:password
   ```
2. **Place the file**: Copy the generated `keycloak.pfx` to the same directory as the `DockerComposeWithPgAdminPostgresqlKeycloak.yml` file.

### 2. Docker Network
The Docker Compose file uses an external network named `local-development-network`. Create it if it doesn't exist:
```bash
docker network create local-development-network
```

## Getting Started

1. **Start the containers**:
   ```bash
   docker compose -f DockerComposeWithPgAdminPostgresqlKeycloak.yml up -d
   ```

2. **Create Databases**:
   Connect to the PostgreSQL server using [pgAdmin](http://localhost:15432) or a CLI.
    - **pgAdmin Credentials**: `admin@development.com` / `password`
    - **DB Credentials**: `postgres` / `password`

   Run the following SQL commands:
   ```sql
   create database keycloak with owner postgres;
   create database spring_keycloak with owner postgres;
   ```

3. **Restart Keycloak**:
   Keycloak may fail to start initially if the database was missing. Restart it:
   ```bash
   docker compose -f DockerComposeWithPgAdminPostgresqlKeycloak.yml restart keycloak
   ```

## Keycloak Configuration

1. Access **[Keycloak](http://localhost:9000)** and login with `admin` / `password`.
2. Go to the **Master** realm dropdown (top-left) and click **Create Realm**:
    - Realm name: `development`
    - Click **Create**.
3. Go to **Clients** (left menu):
    - **Create `api-client`**:
        - Click **Create client**.
        - Client ID: `api-client`
        - Click **Next** until saved.
    - **Create `spa-client`**:
        - Click **Create client**.
        - Client ID: `spa-client`
        - Click **Next**.
        - Ensure **Standard flow** and **Direct access grants** are ON.
        - Click **Save**.
        - In **Settings** > **Access settings**:
            - Valid redirect URIs: `/*`
            - Web origins: `/*`
        - Click **Save**.
4. Go to **Users** (left menu):
    - **Create Admin User**:
        - Click **Add user**.
        - Username: `admin`
        - Email: `admin@development.com`
        - Email verified: **Yes**.
        - Click **Create**.
        - Go to **Credentials** tab -> **Set Password** -> `password` (Temporary: **Off**).
    - **Create API User**:
        - Click **Add user**.
        - Username: `api.user`
        - Email: `api.user@development.com`
        - Email verified: **Yes**.
        - Click **Create**.
        - Go to **Credentials** tab -> **Set Password** -> `password` (Temporary: **Off**).

## Spring Boot Configuration

Use the following environment variables to configure the `spring-keycloak` application:

```properties
KEYCLOAK_AUTH_SERVER_URL=http://localhost:9000
KEYCLOAK_REALM=development
SERVER_PORT=8081
SPRING_APPLICATION_NAME=spring-keycloak
SPRING_DATASOURCE_PASSWORD=password
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/spring_keycloak
SPRING_DATASOURCE_USERNAME=postgres
SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_ISSUER_URI=http://localhost:9000/realms/development
SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK_SET_URI=http://localhost:9000/realms/development/protocol/openid-connect/certs
```

**Good Luck!**