# Backend Reference Guide — TechCup & Spring Boot

Guía de referencia rápida para el stack backend del proyecto. Contiene comandos, dependencias y configuraciones esenciales.

---

## Tabla de contenido

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
14. [Git — flujo de trabajo](#git--flujo-de-trabajo)
15. [pom.xml — dependencias de referencia](#pomxml--dependencias-de-referencia)

---

## Maven

```bash
mvn compile                          # Compilar el proyecto
mvn test                             # Correr los tests
mvn clean test                       # Limpiar y correr tests
mvn clean verify                     # Limpiar, compilar, testear y verificar
mvn spring-boot:run                  # Levantar el servidor
mvn clean test jacoco:report         # Generar reporte de cobertura JaCoCo
mvn package -DskipTests              # Generar el JAR sin correr tests
mvn sonar:sonar                      # Correr análisis de SonarQube
```

> **Con perfil local (cuando se usan variables de entorno):**
> ```bash
> mvn spring-boot:run "-Dspring-boot.run.profiles=local"
> ```

---

## Spring Boot

### Levantar el proyecto con variables de entorno (PowerShell)

```powershell
$env:DB_URL="jdbc:postgresql://localhost:5432/techcup"
$env:DB_USERNAME="postgres"
$env:DB_PASSWORD="tu_password"
mvn spring-boot:run
```

### application.properties — configuración base

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

### application-local.properties — credenciales locales (nunca subir a Git)

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/techcup
spring.datasource.username=postgres
spring.datasource.password=tu_password_personal
```

### application-test.properties — H2 para pruebas

Ubicación: `src/test/resources/application-test.properties`

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

### Agregar al PATH (PowerShell — temporal por sesión)

```powershell
$env:PATH += ";C:\Program Files\PostgreSQL\18\bin"
```

### Comandos básicos

```bash
psql -U postgres                     # Conectarse a PostgreSQL
psql --version                       # Ver versión instalada
```

### Dentro de psql

```sql
CREATE DATABASE techcup;             -- Crear base de datos
\l                                   -- Listar bases de datos
\c techcup                           -- Conectarse a una base de datos
\dt                                  -- Listar tablas
\q                                   -- Salir
```

### Cambiar contraseña del usuario postgres

```sql
ALTER USER postgres WITH PASSWORD 'nueva_password';
```

---

## Docker

### Comandos esenciales

```bash
docker --version                     # Verificar instalación
docker ps                            # Ver contenedores corriendo
docker ps -a                         # Ver todos los contenedores
docker images                        # Ver imágenes descargadas
docker start nombre                  # Iniciar un contenedor detenido
docker stop nombre                   # Detener un contenedor
docker rm nombre                     # Eliminar un contenedor
docker logs nombre                   # Ver logs de un contenedor
```

### PostgreSQL con Docker

```bash
docker run --name postgres-techcup \
  -e POSTGRES_DB=techcup \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  -d postgres
```

### MongoDB con Docker

```bash
docker run --name mongo-techcup -p 27017:27017 -d mongo
```

### SonarQube con Docker

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
docker start sonarqube               # Reiniciar si se detuvo
```

> **Nota:** Docker Desktop debe estar abierto antes de correr cualquier comando docker.

---

## SonarQube

### Acceso

```
URL:      http://localhost:9000
Usuario:  admin
Password: (la que configuraste al primer login)
```

### Generar token

En SonarQube: `My Account → Security → Generate Token`
- Type: `User Token`
- Expiration: `No expiration`

### Configuración en pom.xml (dentro de `<properties>`)

```xml
<sonar.projectKey>tech_cup</sonar.projectKey>
<sonar.projectName>tech_cup</sonar.projectName>
<sonar.host.url>http://localhost:9000</sonar.host.url>
<sonar.login>TU_TOKEN_AQUI</sonar.login>
<sonar.coverage.jacoco.xmlReportPaths>
    target/site/jacoco/jacoco.xml
</sonar.coverage.jacoco.xmlReportPaths>
```

> **Importante:** Usar `sonar.login` (no `sonar.token`) para SonarQube versión 9.x

### Correr el análisis

```bash
# Si el token está en el pom.xml
mvn sonar:sonar

# Si el token NO está en el pom.xml (recomendado para equipos)
mvn sonar:sonar "-Dsonar.login=TU_TOKEN_AQUI"

# Análisis completo con tests y reporte
mvn clean verify sonar:sonar "-Dsonar.login=TU_TOKEN_AQUI"
```

### Plugin en pom.xml

```xml
<plugin>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>4.0.0.4121</version>
</plugin>
```

---

## JaCoCo

### Comandos

```bash
mvn clean test jacoco:report                    # Generar reporte HTML
mvn clean verify                                # Generar reporte + verificar mínimos
```

### Ver el reporte (PowerShell)

```powershell
Invoke-Item target/site/jacoco/index.html
```

### Plugin completo en pom.xml

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
                                <minimum>0.80</minimum>  <!-- Ajustar según el proyecto -->
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Dependencia H2 (para pruebas sin PostgreSQL)

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

---

## Swagger / SpringDoc

### Acceso

```
http://localhost:8080/swagger-ui.html
```

### Dependencia en pom.xml

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

### Configuración en application.properties

```properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.enabled=true
```

### Anotaciones en los controllers

```java
@Tag(name = "Torneos", description = "Operaciones relacionadas con torneos")
@RestController
@RequestMapping("/api/tournaments")
public class TournamentController {

    @Operation(summary = "Crear torneo", description = "Crea un nuevo torneo en estado Borrador")
    @ApiResponse(responseCode = "201", description = "Torneo creado exitosamente")
    @PostMapping
    public ResponseEntity<TournamentDto> create(@RequestBody CreateTournamentDto dto) { ... }
}
```

### Clase de configuración (opcional para personalizar título)

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("TechCup API")
                .version("1.0.0")
                .description("API REST para la gestión del torneo TechCup"));
    }
}
```

> **Si usas Spring Security:** permitir acceso a Swagger en SecurityConfig:
> ```java
> .requestMatchers("/v3/api-docs/**", "/swagger-ui/**", "/swagger-ui.html").permitAll()
> ```

---

## Postman

### Configuración del environment

| Variable | Value |
|----------|-------|
| `base_url` | `http://localhost:8080` |
| `token` | (se llena automáticamente con el script) |

### Script para guardar el token automáticamente

En el request de login → pestaña `Scripts` → `Post-response`:

```javascript
const response = pm.response.json();
pm.environment.set("token", response.accessToken);
```

### Flujo de autenticación

```
POST {{base_url}}/api/auth/login
Body → raw → JSON:
{
  "email": "usuario@escuelaing.edu.co",
  "password": "123456"
}
```

### Usar el token en requests protegidos

`Authorization → Bearer Token → {{token}}`

### Endpoints principales TechCup

```
# Autenticación
POST   {{base_url}}/api/auth/login
POST   {{base_url}}/api/auth/refresh

# Usuarios
POST   {{base_url}}/api/users               (público)
GET    {{base_url}}/api/users               (ADMINISTRATOR)
GET    {{base_url}}/api/users/{id}          (ADMINISTRATOR)
PUT    {{base_url}}/api/users/{id}          (ADMINISTRATOR)
PATCH  {{base_url}}/api/users/{id}/role     (ADMINISTRATOR)
PATCH  {{base_url}}/api/users/{id}/inactive (ADMINISTRATOR)

# Torneos
GET    {{base_url}}/api/tournaments         (público)
GET    {{base_url}}/api/tournaments/{id}    (público)
POST   {{base_url}}/api/tournaments         (ORGANIZER/ADMINISTRATOR)
PUT    {{base_url}}/api/tournaments/{id}    (ORGANIZER/ADMINISTRATOR)
DELETE {{base_url}}/api/tournaments/{id}    (ORGANIZER/ADMINISTRATOR)
PATCH  {{base_url}}/api/tournaments/{id}/start   (ORGANIZER/ADMINISTRATOR)
PATCH  {{base_url}}/api/tournaments/{id}/finish  (ORGANIZER/ADMINISTRATOR)
```

---

## MongoDB

### Agregar al PATH (PowerShell — temporal)

```powershell
$env:PATH += ";C:\Program Files\MongoDB\Server\8.2\bin"
```

### Verificar servicio

```powershell
Get-Service -Name MongoDB
```

### Dependencia en pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

### application.properties del microservicio de imágenes

```properties
spring.application.name=image-service
server.port=8081
spring.data.mongodb.uri=mongodb://localhost:27017/lab8images
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

### Endpoints del microservicio de imágenes

```
POST   http://localhost:8081/imagenes
       Body: form-data
         archivo: [File]
         referenciaExterna: "torneo-1"

GET    http://localhost:8081/imagenes
GET    http://localhost:8081/imagenes/{id}
GET    http://localhost:8081/imagenes/referencia/{referenciaExterna}
DELETE http://localhost:8081/imagenes/{id}
```

### Anotación de documento MongoDB

```java
@Document(collection = "imagenes")
public class ImagenDocument {
    @Id
    private String id;
    // campos...
}
```

---

## JWT & Spring Security

### Dependencias en pom.xml

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

### Configuración JWT en application.properties

```properties
jwt.secret=${JWT_SECRET:mi-clave-secreta-de-al-menos-32-caracteres-1234}
jwt.access-token-expiration-minutes=15
jwt.refresh-token-expiration-days=7
```

### Respuesta del login

```json
{
  "id": "uuid",
  "email": "usuario@escuelaing.edu.co",
  "name": "Nombre",
  "roles": ["PLAYER"],
  "accessToken": "eyJhbGci...",
  "refreshToken": "eyJhbGci...",
  "tokenType": "Bearer",
  "expiresIn": 900,
  "message": "Login successful"
}
```

> - `accessToken` expira en 15 minutos — úsalo en los requests
> - `refreshToken` expira en 7 días — úsalo para renovar el accessToken sin hacer login

---

## SLF4J Logger

### Uso en cualquier clase de servicio

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class TournamentService {

    private static final Logger log = LoggerFactory.getLogger(TournamentService.class);

    public TournamentDto create(CreateTournamentDto dto) {
        log.info("Creando torneo: {}", dto.getName());
        // lógica...
        log.debug("Torneo procesado correctamente");
        return resultado;
    }

    public TournamentDto getById(String id) {
        log.debug("Buscando torneo con id: {}", id);
        // si no existe:
        log.warn("Torneo no encontrado con id: {}", id);
        // si hay error:
        log.error("Error al buscar torneo: {}", e.getMessage());
    }
}
```

### Niveles de log

| Nivel | Cuándo usar |
|-------|-------------|
| `log.debug()` | Información detallada para desarrollo |
| `log.info()` | Acciones importantes del sistema |
| `log.warn()` | Algo inesperado pero no crítico |
| `log.error()` | Errores que deben atenderse |

### Ver logs en tiempo real (PowerShell)

```powershell
Get-Content logs\tech_cup.log -Wait
```

### Configuración en application.properties

```properties
logging.file.name=logs/tech_cup.log
```

---

## GitHub Actions CI/CD

### Ubicación del archivo

```
.github/workflows/ci-cd.yml
```

### Pipeline completo

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

### Secrets a configurar en GitHub

`Settings → Secrets and variables → Actions → New repository secret`

| Secret | Valor |
|--------|-------|
| `DB_URL` | `jdbc:postgresql://host:5432/techcup` |
| `DB_USERNAME` | usuario de PostgreSQL |
| `DB_PASSWORD` | contraseña de PostgreSQL |
| `JWT_SECRET` | clave secreta JWT |
| `AZURE_WEBAPP_NAME` | nombre del App Service en Azure |
| `AZURE_PUBLISH_PROFILE` | contenido del archivo .PublishSettings de Azure |

---

## Azure App Service

### Configuración recomendada (plan gratuito)

| Campo | Valor |
|-------|-------|
| Runtime stack | Java 17 |
| Java web server | Java SE |
| Operating System | Linux |
| Pricing plan | Free F1 |

### Cambiar puerto para Azure (en application.properties)

```properties
server.port=80
```

### Variables de entorno en Azure

En App Service: `Configuration → Application settings`

Agregar las mismas variables que en los secrets de GitHub:
- `DB_URL`
- `DB_USERNAME`
- `DB_PASSWORD`
- `JWT_SECRET`

### Ver logs en Azure

En App Service: `Log stream`

---

## Git — flujo de trabajo

### Comandos diarios

```bash
git status                           # Ver estado del repositorio
git branch                           # Ver ramas
git checkout develop                 # Cambiar a develop
git pull origin develop              # Traer cambios del equipo
git checkout -b feature/nombre       # Crear nueva rama
git add .                            # Agregar todos los cambios
git add archivo.java                 # Agregar un archivo específico
git commit -m "feat: descripción"    # Hacer commit
git push origin feature/nombre       # Subir la rama a GitHub
```

### Descartar cambios locales

```bash
git restore .                        # Descartar cambios no commiteados
git clean -fd                        # Eliminar archivos nuevos no trackeados
```

### Eliminar archivo subido por error

```bash
git rm '$null'                       # Eliminar archivo $null
git rm -r --cached logs/             # Dejar de trackear carpeta logs
git commit -m "remove unwanted files"
git push origin develop
```

### .gitignore recomendado

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

## pom.xml — dependencias de referencia

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

    <!-- H2 para pruebas -->
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

    <!-- Validaciones -->
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

## Comandos más importantes para memorizar

```bash
# Levantar el proyecto
mvn spring-boot:run "-Dspring-boot.run.profiles=local"

# Correr tests y ver cobertura
mvn clean test jacoco:report

# Análisis de calidad
mvn sonar:sonar "-Dsonar.login=TU_TOKEN"

# Análisis completo (tests + sonar)
mvn clean verify sonar:sonar "-Dsonar.login=TU_TOKEN"

# Levantar PostgreSQL con Docker
docker start postgres-techcup

# Levantar SonarQube con Docker
docker start sonarqube

# Ver logs en tiempo real
Get-Content logs\tech_cup.log -Wait    # PowerShell
```
