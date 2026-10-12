# DOSW_ParcialT2_JuanMunar

Repositorio base y prerrequisitos técnicos para el **Parcial Práctico — Segundo Tercio (Parte 1.1)** de la asignatura **Desarrollo de Operaciones de Software (DOSW)** en la Escuela Colombiana de Ingeniería Julio Garavito.

---

## Índice
1. [Datos del estudiante](#1-datos-del-estudiante)
2. [Descripción del proyecto](#2-descripción-del-proyecto)
3. [Despliegue](#3-despliegue)
4. [Estructura del proyecto](#4-estructura-del-proyecto)
5. [Tecnologías y dependencias](#5-tecnologías-y-dependencias)
6. [Configuración y variables de entorno](#6-configuración-y-variables-de-entorno)
7. [Cómo ejecutar](#7-cómo-ejecutar)
8. [Pruebas](#8-pruebas)
9. [Docker](#9-docker)
10. [CI/CD](#10-cicd)
11. [Herramientas de modelado y diseño (evidencias)](#11-herramientas-de-modelado-y-diseño-evidencias)

---

## 1. Datos del estudiante

- **Nombre completo:** Juan David Munar  
- **Materia:** Desarrollo de Operaciones de Software (DOSW)  
- **Grupo:** [Tu Grupo aquí, ej. 1]  
- **Profesor agregado al repositorio como colaborador:** ✔ Sí (Acceso otorgado)

---

## 2. Descripción del proyecto

Microservicio backend desarrollado sobre Spring Boot 3 y Java 21 como base para el Parcial Práctico del Segundo Tercio de DOSW. El sistema implementa una arquitectura hexagonal por capas (MVC desacoplado), persistencia NoSQL mediante Spring Data MongoDB, autenticación sin estado preparada para JWT, documentación interactiva con OpenAPI/Swagger UI, empaquetado multi-etapa con Docker y automatización de integración y entrega continua (CI/CD) orientada a Azure App Services.

---

## 3. Despliegue

El servicio cuenta con despliegue automatizado en **Azure App Service (Linux Containers)**:

| Recurso | URL | Estado |
| :--- | :--- | :--- |
| **Servicio Base (Azure)** | `https://<tu-app-name>.azurewebsites.net/` | En línea |
| **Health Check Endpoint** | `https://<tu-app-name>.azurewebsites.net/api/v1/health` | 200 OK |
| **Documentación Swagger UI** | `https://<tu-app-name>.azurewebsites.net/swagger-ui/index.html` | Disponible |
| **OpenAPI Spec (JSON)** | `https://<tu-app-name>.azurewebsites.net/v3/api-docs` | Disponible |

> **Nota:** Reemplazar `<tu-app-name>` con el identificador asignado a tu App Service en Azure.

---

## 4. Estructura del proyecto

La arquitectura implementa separación estricta de responsabilidades bajo convención MVC desacoplada:

```text
DOSW_ParcialT2_JuanMunar/
│
├── docs/
│   ├── uml/                          # Diagramas fuente de arquitectura y modelos (draw.io, Lucidchart, Miro)
│   └── images/                       # Exportes de diagramas y evidencias del README
│
├── src/
│   ├── main/
│   │   ├── java/edu/dosw/parcial/
│   │   │   ├── controller/           # Capa de presentación REST
│   │   │   │   ├── dtos/             # Objetos de transferencia de datos
│   │   │   │   │   ├── request/      # Contratos de entrada de la API
│   │   │   │   │   └── response/     # Contratos de salida de la API
│   │   │   │   ├── mappers/          # Mapper IN: Conversión entre DTOs y Objetos de Dominio
│   │   │   │   ├── handlers/         # Manejadores globales de excepciones (@RestControllerAdvice)
│   │   │   │   └── HealthController.java  # Endpoint de diagnóstico y salud
│   │   │   │
│   │   │   ├── core/                 # Núcleo del negocio (Agnóstico a infraestructura)
│   │   │   │   ├── services/         # Interfaces e implementaciones de lógica de negocio
│   │   │   │   ├── models/           # Entidades puras y modelos de dominio
│   │   │   │   ├── validators/       # Reglas de validación de negocio complejas
│   │   │   │   └── utils/            # Utilidades transversales del dominio
│   │   │   │
│   │   │   ├── persistence/          # Capa de acceso a datos e infraestructura NoSQL
│   │   │   │   ├── entities/         # Modelos de persistencia anotados con @Document (MongoDB)
│   │   │   │   ├── repositories/     # Interfaces de persistencia extendiendo MongoRepository
│   │   │   │   └── mappers/          # Mapper OUT: Conversión entre Dominio y @Document
│   │   │   │
│   │   │   ├── config/               # Configuraciones del framework (Seguridad, OpenAPI, etc.)
│   │   │   └── main/                 # Clase principal ejecutable con @SpringBootApplication
│   │   │
│   │   └── resources/
│   │       └── application.yaml      # Parámetros y perfiles de configuración de la aplicación
│   │
│   └── test/                         # Pruebas automatizadas
│       ├── java/edu/dosw/parcial/
│       │   ├── controller/           # Pruebas unitarias de controladores con MockMvc
│       │   ├── core/                 # Pruebas unitarias de servicios de negocio y validadores
│       │   └── persistence/          # Pruebas de integración y persistencia
│       └── resources/                # Configuraciones para entorno de pruebas
│
├── Dockerfile                        # Definición multi-etapa para empaquetado optimizado
├── .dockerignore                     # Exclusiones de contexto para el daemon de Docker
├── .github/workflows/ci-cd.yml       # Pipeline automatizado de GitHub Actions
├── pom.xml                           # Descriptor Maven con plugins y dependencias
├── .gitignore                        # Exclusiones de control de versiones Git
└── README.md                         # Documentación del proyecto y evidencias
```

### Responsabilidad de los componentes clave:
- **`controller/mappers` (Mapper IN):** Transforma `RequestDTO` $\rightarrow$ `Domain Model` y `Domain Model` $\rightarrow$ `ResponseDTO`. El controlador nunca interactúa con documentos de persistencia.
- **`persistence/mappers` (Mapper OUT):** Transforma `Domain Model` $\leftrightarrow$ `@Document`. El servicio de negocio nunca interactúa con DTOs ni con clases atadas a la base de datos.
- **`persistence/entities`:** Contiene las entidades mapeadas directamente a colecciones de MongoDB mediante anotaciones `@Document`.
- **`main/ParcialApplication.java`:** Punto de entrada que define explícitamente `scanBasePackages = "edu.dosw.parcial"` para permitir el descubrimiento jerárquico de todos los componentes Spring.

---

## 5. Tecnologías y dependencias

| Tecnología / Dependencia | Versión | Propósito en el proyecto |
| :--- | :--- | :--- |
| **Java (Eclipse Temurin)** | `21` | Lenguaje y entorno de ejecución moderno (LTS). |
| **Spring Boot** | `3.2.5` | Framework base para creación de microservicios. |
| **Spring Boot Starter Web** | `3.2.5` | Creación de endpoints REST y servidor Tomcat embebido. |
| **Spring Boot Starter Data MongoDB** | `3.2.5` | Abstracción NoSQL y soporte para `MongoRepository`. |
| **Spring Boot Starter Validation** | `3.2.5` | Validación declarativa de modelos y DTOs (`@NotNull`, `@Size`, etc.). |
| **Spring Boot Starter Security** | `3.2.5` | Configuración de filtros de seguridad, CORS y control de acceso. |
| **JJWT (`jjwt-api`, `impl`, `jackson`)** | `0.12.3` | Generación, firmado criptográfico y validación de tokens JWT. |
| **SpringDoc OpenAPI WebMVC UI** | `2.5.0` | Generación automática de especificación OpenAPI 3 y UI Swagger. |
| **Lombok** | `1.18.30` | Reducción de código boilerplate (getters, setters, constructores, `@Slf4j`). |
| **MapStruct** | `1.5.5.Final` | Mapeo rápido de objetos DTO/Dominio/Document en tiempo de compilación. |
| **Lombok MapStruct Binding** | `0.2.0` | Integración ordenada de procesadores de anotaciones entre Lombok y MapStruct. |
| **Spring Boot Starter Test** | `3.2.5` | Framework de pruebas que incluye JUnit 5, Mockito y AssertJ. |
| **Spring Security Test** | `3.2.5` | Utilidades y soporte de testing para endpoints protegidos con Spring Security. |
| **JaCoCo Maven Plugin** | `0.8.12` | Generación de métricas e informes de cobertura de código. |
| **SonarQube Maven Plugin** | `3.10.0.2594` | Análisis estático de código, seguridad y deuda técnica. |

---

## 6. Configuración y variables de entorno

La configuración sigue el patrón de doce factores (*Twelve-Factor App*), permitiendo inyectar configuraciones por variables de entorno sin exponer credenciales en el código fuente:

| Variable | Descripción | Valor por defecto (Local) | Ejemplo para Producción |
| :--- | :--- | :--- | :--- |
| `MONGODB_URI` | Cadena de conexión al clúster MongoDB | `mongodb://localhost:27017/dosw_parcial` | `mongodb+srv://user:pass@cluster.mongodb.net/dosw_prod` |
| `PORT` | Puerto HTTP donde se enlaza el servidor | `8080` | `8080` |
| `JWT_SECRET` | Clave secreta Base64 (mínimo 256 bits / 44 caracteres) | Clave de desarrollo incluida por defecto | `c2VjdXJlS2V5TXVzdEJlQXRMZWFzdDQ0Q2hhcnNMb25nRm9ySFMyNTY=` |
| `WEBSITES_PORT` | Puerto de enrutamiento para Azure App Service | `8080` | `8080` |

---

## 7. Cómo ejecutar

### 7.1. Levantar MongoDB local con Docker
Para ejecutar localmente sin instalar MongoDB en el host:
```bash
docker run -d -p 27017:27017 --name mongo-parcial mongo:7
```

### 7.2. Ejecutar con Maven
Compilar y ejecutar la aplicación en el puerto `8080`:
```bash
# Compilar el proyecto
mvn clean package -DskipTests

# Ejecutar el servicio
mvn spring-boot:run
```

O utilizando el JAR empaquetado:
```bash
java -jar target/DOSW-ParcialT2-1.0.0-SNAPSHOT.jar
```

### 7.3. Ejecutar con Docker
Construir y correr la imagen del contenedor:
```bash
# Construir la imagen
docker build -t dosw-parcial:local .

# Ejecutar el contenedor
docker run -d -p 8080:8080 \
  -e MONGODB_URI=mongodb://host.docker.internal:27017/dosw_parcial \
  -e JWT_SECRET=dGhpc0lzQVNlY3JldEtleUZvckp3dFRva2VuR2VuZXJhdGlvbk1pbjQ0Q2hhcnM= \
  --name dosw-parcial-app dosw-parcial:local
```

Verificar el endpoint de salud:
```bash
curl http://localhost:8080/api/v1/health
```

---

## 8. Pruebas

Para ejecutar el suite de pruebas automatizadas y generar el reporte de cobertura de JaCoCo:

```bash
mvn clean verify
```

El reporte de JaCoCo se genera en:
`target/site/jacoco/index.html`

### Evidencias de pruebas:
- **Resultado de pruebas unitarias (`mvn test`):**
  > *(Adjuntar captura de pantalla de la terminal con las pruebas en verde en `docs/images/test-results.png`)*
  ![Pruebas Unitarias](docs/images/test-results.png)

- **Reporte de cobertura JaCoCo:**
  > *(Adjuntar captura de pantalla del reporte HTML generado en `docs/images/jacoco-report.png`)*
  ![Reporte JaCoCo](docs/images/jacoco-report.png)

---

## 9. Docker

La aplicación cuenta con una construcción multi-etapa (*multi-stage build*) para optimizar el tamaño de la imagen final utilizando `eclipse-temurin:21-jre-alpine`:

```bash
# Construcción
docker build -t dosw-parcial:local .

# Inspección y ejecución
docker run -p 8080:8080 dosw-parcial:local
```

### Evidencias de Docker:
- **Contenedor en ejecución y verificación de salud:**
  > *(Adjuntar captura de pantalla de `docker ps` y petición curl al endpoint `/api/v1/health` en `docs/images/docker-running.png`)*
  ![Docker Running](docs/images/docker-running.png)

---

## 10. CI/CD

El repositorio cuenta con un pipeline automatizado en **GitHub Actions** (`.github/workflows/ci-cd.yml`) configurado para ejecutarse en cada `push` a las ramas `main` y `develop`:

1. **Job `test`:** Descarga el código, configura JDK 21 Temurin con caché Maven y ejecuta `mvn verify --no-transfer-progress`.
2. **Job `docker`:** Se autentica en Docker Hub y construye y publica la imagen `${{ secrets.DOCKERHUB_USERNAME }}/dosw-parcial-t2:latest`.
3. **Job `deploy`:** Se autentica en Azure mediante Service Principal y despliega la imagen actualizada al App Service de Azure.

### Secrets requeridos en GitHub:
- `DOCKERHUB_USERNAME`: Usuario de Docker Hub.
- `DOCKERHUB_TOKEN`: Personal Access Token de Docker Hub con permisos de lectura/escritura.
- `AZURE_CREDENTIALS`: JSON del Service Principal generado para Azure.
- `AZURE_WEBAPP_NAME`: Nombre del recurso Azure App Service.
- `JWT_SECRET`: Llave criptográfica secreta Base64.

### Evidencias del pipeline:
- **Pipeline de GitHub Actions en verde:**
  > *(Adjuntar captura del workflow completado exitosamente en `docs/images/pipeline-success.png`)*
  ![GitHub Actions Pipeline](docs/images/pipeline-success.png)

---

## 11. Herramientas de modelado y diseño (evidencias)

Prerrequisito completado con cuentas activas para el diseño de arquitectura y maquetación de interfaces del parcial:

- **Herramienta de Modelado (Lucidchart / Draw.io / Miro):**
  > *(Adjuntar captura de pantalla de la cuenta activa en `docs/images/modeling-account.png`)*
  ![Cuenta de Modelado](docs/images/modeling-account.png)

- **Herramienta de Diseño de Interfaces (Figma):**
  > *(Adjuntar captura de pantalla de la cuenta activa en `docs/images/figma-account.png`)*
  ![Cuenta Figma](docs/images/figma-account.png)
