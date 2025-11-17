## README.md

**Do not use in production**
The configuration is not secure and is intended for local development use only.

Keycloak is a required dependency. You can run Keycloak using Docker Compose along with Postgresql and pgAdmin.
[DockerComposeWithPgAdminPostgresqlKeycloak.yml](DockerComposeWithPgAdminPostgresqlKeycloak.yml)

### PFX file

A PFX file, also known as a PKCS#12 file, is a binary format commonly used to store cryptographic objects like
certificates and private keys in a single, password-protected file. PFX files can be created using tools like OpenSSL or
graphical utilities like Keystore Explorer.

1. Generate a PFX file using openssl, Keystore Explorer, etc.
2. Use _password_ as the keystore password.
3. Rename the file to _keycloak.pfx_.
4. Copy the file to the _${projectRoot}/setup_ directory.

### docker compose

1. [DockerComposeWithPgAdminPostgresqlKeycloak](DockerComposeWithPgAdminPostgresqlKeycloak.yml)
   `docker compose -f DockerComposeWithPgAdminPostgresqlKeycloak.yml up`
2. Connect to the Postgresql server. [pgAdmin](http://localhost:15432)
3. Create *keycloak* database. `create database keycloak with owner postgres;`
4. Create *spring-keycloak* database. `create database spring-keycloak with owner postgres;`
5. Restart docker compose. `docker compose -f DockerComposeWithPgAdminPostgresqlKeycloak.yml up --detach`

### Keycloak

1. Login to [Keycloak](http://localhost:8080) with credentials admin:password. 
2. Go to _Manage realms_.
3. _Create realm_, Realm name: *development*, Enabled = On. 
4. Go to _Clients_.
5. _Create_, Client ID: *api-client*.
6. _Create_, Client ID: *spa-client*. 
7. Set Valid redirect URIs and Web Origins to _/*_
8. Go to _Users_.
9. _Add user_, Username: admin, Email: _admin@development.com_, Email verified: On, on the Credentials tab, Select
    _Password_ and set the password to password. Select _Temporary_ Off.
10. _Add user_, Username: api.user, Email: _api.user@development.com_, Email verified: On, on the Credentials tab,
    Select _Password_ and set the password to password. Select _Temporary_ Off. 
11. Login to [Security Admin Console](http://localhost:9000/admin/development/console/) with both users.

### spring-keycloak

The env variables necessary to run the application are listed below.

`
KEYCLOAK_AUTH_SERVER_URL=http://localhost:9000;KEYCLOAK_REALM=development;SERVER_PORT=8081;SPRING_APPLICATION_NAME=spring-keycloak;SPRING_DATASOURCE_PASSWORD=password;SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/spring_keycloak;SPRING_DATASOURCE_USERNAME=postgres;SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_ISSUER-URI=http://localhost:9000/realms/development;SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_JWK-SET-URI=http://localhost:9000/realms/development/protocol/openid-connect/certs
`

**Good Luck!**
