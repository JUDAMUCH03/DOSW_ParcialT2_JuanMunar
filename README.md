# DOSW_ParcialT2_JuanMunar

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
- **Grupo:**  1
- **Profesor agregado al repositorio como colaborador:** Sí 

---

## 2. Descripción del proyecto

Microservicio backend desarrollado sobre Spring Boot 3 y Java 21 como base para el Parcial Práctico del Segundo Tercio de DOSW. El sistema implementa una arquitectura hexagonal por capas (MVC desacoplado), persistencia NoSQL mediante Spring Data MongoDB, autenticación sin estado preparada para JWT, documentación interactiva con OpenAPI/Swagger UI, empaquetado multi-etapa con Docker y automatización de integración y entrega continua (CI/CD) orientada a Azure App Services.

---

## 3. Despliegue

| Recurso | URL |
| :--- | :--- |
| **Servicio Base (Azure)** | `https://dosw-parcial-t2-jmunar-hsfmc4cwdfc7abh3.mexicocentral-01.azurewebsites.net/` |
| **Health Check Endpoint** | `https://dosw-parcial-t2-jmunar-hsfmc4cwdfc7abh3.mexicocentral-01.azurewebsites.net/api/v1/health` |
| **Documentación Swagger UI** | `https://dosw-parcial-t2-jmunar-hsfmc4cwdfc7abh3.mexicocentral-01.azurewebsites.net/swagger-ui/index.html` |

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
| `JWT_SECRET` | Clave secreta Base64 (mínimo 256 bits / 44 caracteres) | Definida por variable de entorno | `<CLAVE_BASE64_MIN_44_CARACTERES>` |
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
  -e JWT_SECRET=<TU_JWT_SECRET_BASE64> \
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
<img width="1236" height="463" alt="Captura de pantalla 2026-10-12 150356" src="https://github.com/user-attachments/assets/cc871b0c-0ad2-496e-a563-375f14c7c0d7" />


- **Reporte de cobertura JaCoCo:**
<img width="1051" height="168" alt="Captura de pantalla 2026-10-12 150446" src="https://github.com/user-attachments/assets/ddcb9be2-e736-4462-a458-aec7a405f095" />


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
<img width="1550" height="936" alt="image" src="https://github.com/user-attachments/assets/90dbbbd8-39d5-4739-a0f7-e67043eb5f4d" />
<img width="1058" height="303" alt="image" src="https://github.com/user-attachments/assets/5fcb9611-63da-4e51-a4d5-a450bb58014f" />

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
<img width="1893" height="449" alt="image" src="https://github.com/user-attachments/assets/68a85aeb-97da-4b9d-a8f2-89d4e9af9756" />

---

## 11. Herramientas de modelado y diseño (evidencias)

Prerrequisito completado con cuentas activas para el diseño de arquitectura y maquetación de interfaces del parcial:

- **Herramienta de Modelado (Drawio):**
<img width="1919" height="938" alt="image" src="https://github.com/user-attachments/assets/fb1b8f81-89b3-48c1-a0ff-a0efd64e37ea" />


- **Herramienta de Diseño de Interfaces (Figma):**
<img width="272" height="378" alt="image" src="https://github.com/user-attachments/assets/91f41c8f-459e-449f-98ab-8ca3eb49bc1a" />

