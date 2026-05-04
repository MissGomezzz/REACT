# Backend Reference Guide — TechCup & Spring Boot

Quick reference guide for the backend stack. Contains commands, dependencies, and essential configurations.

---

## Table of Contents

1. [Maven](#maven)
2. [Spring Boot](#spring-boot)
3. [PostgreSQL](#postgresql)
4. [Docker](#docker)
5. [SonarQube](#sonarqube)
6. [JaCoCo](#jacoco)
7. [Swagger / SpringDoc](#swagger--springdoc)
8. [Postman](#postman)
9. [MongoDB](#mongodb)
10. [JWT & Spring Security](#jwt--spring-security)
11. [SLF4J Logger](#slf4j-logger)
12. [GitHub Actions CI/CD](#github-actions-cicd)
13. [Azure App Service](#azure-app-service)
14. [Git — Workflow](#git--workflow)
15. [pom.xml — Dependency Reference](#pomxml--dependency-reference)

---

## Maven

```bash
mvn compile                          # Compile the project
mvn test                             # Run tests
mvn clean test                       # Clean and run tests
mvn clean verify                     # Clean, compile, test and verify
mvn spring-boot:run                  # Start the server
mvn clean test jacoco:report         # Generate JaCoCo coverage report
mvn package -DskipTests              # Generate JAR without running tests
mvn sonar:sonar                      # Run SonarQube analysis
```

> **With local profile (when using environment variables):**
> ```bash
> mvn spring-boot:run "-Dspring-boot.run.profiles=local"
> ```

---

## Spring Boot

### Start the project with environment variables (PowerShell)

```powershell
$env:DB_URL="jdbc:postgresql://localhost:5432/techcup"
$env:DB_USERNAME="postgres"
$env:DB_PASSWORD="your_password"
mvn spring-boot:run
```

### application.properties — base configuration

```properties
logging.file.name=logs/tech_cup.log

# PostgreSQL
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect

# Spring Security
spring.security.user.name=techcupadmin
spring.security.user.password=Tc2026Secure

# JWT
jwt.secret=${JWT_SECRET:my-super-secret-key-that-should-be-overridden-in-production-1234567890}
jwt.access-token-expiration-minutes=15
jwt.refresh-token-expiration-days=7
```

### application-local.properties — local credentials (never push to Git)

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/techcup
spring.datasource.username=postgres
spring.datasource.password=your_personal_password
```

### application-test.properties — H2 for tests

Location: `src/test/resources/application-test.properties`

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
```

---

## PostgreSQL

### Add to PATH (PowerShell — temporary for current session)

```powershell
$env:PATH += ";C:\Program Files\PostgreSQL\18\bin"
```

### Basic commands

```bash
psql -U postgres                     # Connect to PostgreSQL
psql --version                       # Check installed version
```

### Inside psql

```sql
CREATE DATABASE techcup;             -- Create database
\l                                   -- List databases
\c techcup                           -- Connect to a database
\dt                                  -- List tables
\q                                   -- Exit
```

### Change postgres user password

```sql
ALTER USER postgres WITH PASSWORD 'new_password';
```

---

## Docker

### Essential commands

```bash
docker --version                     # Check installation
docker ps                            # Show running containers
docker ps -a                         # Show all containers
docker images                        # Show downloaded images
docker start name                    # Start a stopped container
docker stop name                     # Stop a container
docker rm name                       # Remove a container
docker logs name                     # View container logs
```

### PostgreSQL with Docker

```bash
docker run --name postgres-techcup \
  -e POSTGRES_DB=techcup \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  -d postgres
```

### MongoDB with Docker

```bash
docker run --name mongo-techcup -p 27017:27017 -d mongo
```

### SonarQube with Docker

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
docker start sonarqube               # Restart if stopped
```

> **Note:** Docker Desktop must be running before using any docker command.

---

## SonarQube

### Access

```
URL:      http://localhost:9000
User:     admin
Password: (the one you set on first login)
```

### Generate token

In SonarQube: `My Account → Security → Generate Token`
- Type: `User Token`
- Expiration: `No expiration`

### Configuration in pom.xml (inside `<properties>`)

```xml
<sonar.projectKey>tech_cup</sonar.projectKey>
<sonar.projectName>tech_cup</sonar.projectName>
<sonar.host.url>http://localhost:9000</sonar.host.url>
<sonar.login>YOUR_TOKEN_HERE</sonar.login>
<sonar.coverage.jacoco.xmlReportPaths>
    target/site/jacoco/jacoco.xml
</sonar.coverage.jacoco.xmlReportPaths>
```

> **Important:** Use `sonar.login` (not `sonar.token`) for SonarQube version 9.x

### Run the analysis

```bash
# If the token is in pom.xml
mvn sonar:sonar

# If the token is NOT in pom.xml (recommended for teams)
mvn sonar:sonar "-Dsonar.login=YOUR_TOKEN_HERE"

# Full analysis with tests and report
mvn clean verify sonar:sonar "-Dsonar.login=YOUR_TOKEN_HERE"
```

### Plugin in pom.xml

```xml
<plugin>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>4.0.0.4121</version>
</plugin>
```

---

## JaCoCo

### Commands

```bash
mvn clean test jacoco:report                    # Generate HTML report
mvn clean verify                                # Generate report + verify minimums
```

### Open the report (PowerShell)

```powershell
Invoke-Item target/site/jacoco/index.html
```

### Full plugin in pom.xml

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals><goal>prepare-agent</goal></goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>verify</phase>
            <goals><goal>report</goal></goals>
        </execution>
        <execution>
            <id>jacoco-check</id>
            <goals><goal>check</goal></goals>
            <configuration>
                <rules>
                    <rule>
                        <element>PACKAGE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>  <!-- Adjust per project -->
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### H2 dependency (for tests without PostgreSQL)

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

---

## Swagger / SpringDoc

### Access

```
http://localhost:8080/swagger-ui.html
```

### Dependency in pom.xml

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

### Configuration in application.properties

```properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.enabled=true
```

### Annotations in controllers

```java
@Tag(name = "Tournaments", description = "Operations related to tournament management")
@RestController
@RequestMapping("/api/tournaments")
public class TournamentController {

    @Operation(summary = "Create tournament", description = "Creates a new tournament in Draft status")
    @ApiResponse(responseCode = "201", description = "Tournament created successfully")
    @PostMapping
    public ResponseEntity<TournamentDto> create(@RequestBody CreateTournamentDto dto) { ... }
}
```

### Configuration class (optional, to customize title)

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("TechCup API")
                .version("1.0.0")
                .description("REST API for TechCup tournament management"));
    }
}
```

> **If using Spring Security:** allow Swagger access in SecurityConfig:
> ```java
> .requestMatchers("/v3/api-docs/**", "/swagger-ui/**", "/swagger-ui.html").permitAll()
> ```

---

## Postman

### Environment setup

| Variable | Value |
|----------|-------|
| `base_url` | `http://localhost:8080` |
| `token` | (filled automatically by script) |

### Script to save token automatically

In login request → `Scripts` tab → `Post-response`:

```javascript
const response = pm.response.json();
pm.environment.set("token", response.accessToken);
```

### Authentication flow

```
POST {{base_url}}/api/auth/login
Body → raw → JSON:
{
  "email": "user@escuelaing.edu.co",
  "password": "123456"
}
```

### Use the token in protected requests

`Authorization → Bearer Token → {{token}}`

### Main TechCup endpoints

```
# Authentication
POST   {{base_url}}/api/auth/login
POST   {{base_url}}/api/auth/refresh

# Users
POST   {{base_url}}/api/users               (public)
GET    {{base_url}}/api/users               (ADMINISTRATOR)
GET    {{base_url}}/api/users/{id}          (ADMINISTRATOR)
PUT    {{base_url}}/api/users/{id}          (ADMINISTRATOR)
PATCH  {{base_url}}/api/users/{id}/role     (ADMINISTRATOR)
PATCH  {{base_url}}/api/users/{id}/inactive (ADMINISTRATOR)

# Tournaments
GET    {{base_url}}/api/tournaments         (public)
GET    {{base_url}}/api/tournaments/{id}    (public)
POST   {{base_url}}/api/tournaments         (ORGANIZER/ADMINISTRATOR)
PUT    {{base_url}}/api/tournaments/{id}    (ORGANIZER/ADMINISTRATOR)
DELETE {{base_url}}/api/tournaments/{id}    (ORGANIZER/ADMINISTRATOR)
PATCH  {{base_url}}/api/tournaments/{id}/start   (ORGANIZER/ADMINISTRATOR)
PATCH  {{base_url}}/api/tournaments/{id}/finish  (ORGANIZER/ADMINISTRATOR)
```

---

## MongoDB

### Add to PATH (PowerShell — temporary)

```powershell
$env:PATH += ";C:\Program Files\MongoDB\Server\8.2\bin"
```

### Check service status

```powershell
Get-Service -Name MongoDB
```

### Dependency in pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

### application.properties for image microservice

```properties
spring.application.name=image-service
server.port=8081
spring.data.mongodb.uri=mongodb://localhost:27017/lab8images
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

### Image microservice endpoints

```
POST   http://localhost:8081/imagenes
       Body: form-data
         archivo: [File]
         referenciaExterna: "tournament-1"

GET    http://localhost:8081/imagenes
GET    http://localhost:8081/imagenes/{id}
GET    http://localhost:8081/imagenes/referencia/{referenciaExterna}
DELETE http://localhost:8081/imagenes/{id}
```

### MongoDB document annotation

```java
@Document(collection = "imagenes")
public class ImagenDocument {
    @Id
    private String id;
    // fields...
}
```

---

## JWT & Spring Security

### Dependencies in pom.xml

```xml
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- JWT -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.13.0</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>
```

### JWT configuration in application.properties

```properties
jwt.secret=${JWT_SECRET:my-secret-key-at-least-32-characters-1234}
jwt.access-token-expiration-minutes=15
jwt.refresh-token-expiration-days=7
```

### Login response

```json
{
  "id": "uuid",
  "email": "user@escuelaing.edu.co",
  "name": "Name",
  "roles": ["PLAYER"],
  "accessToken": "eyJhbGci...",
  "refreshToken": "eyJhbGci...",
  "tokenType": "Bearer",
  "expiresIn": 900,
  "message": "Login successful"
}
```

> - `accessToken` expires in 15 minutes — use it in requests
> - `refreshToken` expires in 7 days — use it to renew the accessToken without logging in again

---

## SLF4J Logger

### Usage in any service class

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class TournamentService {

    private static final Logger log = LoggerFactory.getLogger(TournamentService.class);

    public TournamentDto create(CreateTournamentDto dto) {
        log.info("Creating tournament: {}", dto.getName());
        // logic...
        log.debug("Tournament processed successfully");
        return result;
    }

    public TournamentDto getById(String id) {
        log.debug("Looking for tournament with id: {}", id);
        // if not found:
        log.warn("Tournament not found with id: {}", id);
        // if error:
        log.error("Error looking for tournament: {}", e.getMessage());
    }
}
```

### Log levels

| Level | When to use |
|-------|-------------|
| `log.debug()` | Detailed info for development |
| `log.info()` | Important system actions |
| `log.warn()` | Something unexpected but not critical |
| `log.error()` | Errors that need attention |

### Watch logs in real time (PowerShell)

```powershell
Get-Content logs\tech_cup.log -Wait
```

### Configuration in application.properties

```properties
logging.file.name=logs/tech_cup.log
```

---

## GitHub Actions CI/CD

### File location

```
.github/workflows/ci-cd.yml
```

### Full pipeline

```yaml
name: CI/CD Pipeline TechCup

on:
  pull_request:
    branches: [ develop, main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Compile
        run: mvn compile

  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests and verify
        run: mvn verify
        env:
          DB_URL: ${{ secrets.DB_URL }}
          DB_USERNAME: ${{ secrets.DB_USERNAME }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}

  deploy:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Build JAR
        run: mvn package -DskipTests
        env:
          DB_URL: ${{ secrets.DB_URL }}
          DB_USERNAME: ${{ secrets.DB_USERNAME }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
      - name: Deploy to Azure
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_PUBLISH_PROFILE }}
          package: target/*.jar
```

### Secrets to configure in GitHub

`Settings → Secrets and variables → Actions → New repository secret`

| Secret | Value |
|--------|-------|
| `DB_URL` | `jdbc:postgresql://host:5432/techcup` |
| `DB_USERNAME` | PostgreSQL username |
| `DB_PASSWORD` | PostgreSQL password |
| `JWT_SECRET` | JWT secret key |
| `AZURE_WEBAPP_NAME` | Azure App Service name |
| `AZURE_PUBLISH_PROFILE` | Contents of the .PublishSettings file from Azure |

---

## Azure App Service

### Recommended configuration (free plan)

| Field | Value |
|-------|-------|
| Runtime stack | Java 17 |
| Java web server | Java SE |
| Operating System | Linux |
| Pricing plan | Free F1 |

### Change port for Azure (in application.properties)

```properties
server.port=80
```

### Environment variables in Azure

In App Service: `Configuration → Application settings`

Add the same variables as the GitHub secrets:
- `DB_URL`
- `DB_USERNAME`
- `DB_PASSWORD`
- `JWT_SECRET`

### View logs in Azure

In App Service: `Log stream`

---

## Git — Workflow

### Daily commands

```bash
git status                           # Check repository status
git branch                           # List branches
git checkout develop                 # Switch to develop
git pull origin develop              # Pull team changes
git checkout -b feature/name         # Create new branch
git add .                            # Stage all changes
git add File.java                    # Stage a specific file
git commit -m "feat: description"    # Commit
git push origin feature/name         # Push branch to GitHub
```

### Discard local changes

```bash
git restore .                        # Discard uncommitted changes
git clean -fd                        # Remove new untracked files
```

### Remove a file pushed by mistake

```bash
git rm '$null'                       # Remove $null file
git rm -r --cached logs/             # Stop tracking logs folder
git commit -m "remove unwanted files"
git push origin develop
```

### Recommended .gitignore

```
target/
.env
logs/
application-local.properties
*-local.properties
*.iml
.idea/
.vscode/
$null
```

---

## pom.xml — Dependency Reference

```xml
<dependencies>
    <!-- Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- H2 for tests -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- MongoDB -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>

    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- Swagger -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.5.0</version>
    </dependency>

    <!-- JWT -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.13.0</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.13.0</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.13.0</version>
        <scope>runtime</scope>
    </dependency>

    <!-- Validations -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## Most Important Commands to Memorize

```bash
# Start the project
mvn spring-boot:run "-Dspring-boot.run.profiles=local"

# Run tests and check coverage
mvn clean test jacoco:report

# Quality analysis
mvn sonar:sonar "-Dsonar.login=YOUR_TOKEN"

# Full analysis (tests + sonar)
mvn clean verify sonar:sonar "-Dsonar.login=YOUR_TOKEN"

# Start PostgreSQL with Docker
docker start postgres-techcup

# Start SonarQube with Docker
docker start sonarqube

# Watch logs in real time
Get-Content logs\tech_cup.log -Wait    # PowerShell
```
