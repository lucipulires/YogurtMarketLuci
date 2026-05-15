# 🥛 YogurtMarket — API de Gestión de Producción de Yogurt

<div align="center">

![Java](https://img.shields.io/badge/Java-17-orange?logo=java)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.x-brightgreen?logo=springboot)
![Maven](https://img.shields.io/badge/Maven-3.9+-red?logo=apachemaven)
![H2](https://img.shields.io/badge/H2-In--Memory-blue?logo=h2)
![Swagger](https://img.shields.io/badge/Swagger-OpenAPI%203-green?logo=swagger)
![License](https://img.shields.io/badge/License-MIT-yellow)

**API REST para el control y seguimiento del proceso de producción de yogurt artesanal.**

</div>

---

## 📋 Tabla de Contenido

- [Descripción](#-descripción)
- [Tecnologías](#-tecnologías)
- [Arquitectura](#-arquitectura)
- [Endpoints](#-endpoints-principales)
- [Instalación y ejecución](#-instalación-y-ejecución)
- [Manual de usuario](#-manual-de-usuario)
- [Estructura del proyecto](#-estructura-del-proyecto)
- [Ramas del repositorio](#-ramas-del-repositorio)
- [Autor](#-autor)
- [Licencia](#-licencia)

---

## 📖 Descripción

**YogurtMarket** es una API REST desarrollada con **Spring Boot** que gestiona el ciclo de vida completo de la producción de yogurt artesanal. Permite:

- Registrar y administrar **recetas** con ingredientes y parámetros de producción.
- Controlar **lotes de producción** a través de sus etapas: preparación → calentamiento → inoculación → incubación → refrigeración → completado.
- Monitorear en tiempo real la **temperatura** de cada lote.
- Consultar **estadísticas y métricas** del proceso de producción.

---

## 🛠 Tecnologías

| Tecnología | Versión | Uso |
|---|---|---|
| Java | 17 | Lenguaje principal |
| Spring Boot | 3.5.x | Framework backend |
| Spring Data JPA | — | Persistencia de datos |
| H2 Database | — | Base de datos en memoria |
| Lombok | 1.18.34 | Reducción de boilerplate |
| SpringDoc OpenAPI | 2.8.8 | Documentación Swagger |
| Spring Actuator | — | Métricas y monitoreo |
| Maven | 3.9+ | Gestión de dependencias |

---

## 🏗 Arquitectura

El proyecto sigue una arquitectura en capas (Layered Architecture):

```
src/main/java/com/danieldev87/demo/
│
├── domain/
│   ├── controller/        # Controladores REST (capa de presentación)
│   ├── model/             # Entidades JPA (capa de dominio)
│   ├── repository/        # Interfaces de acceso a datos
│   └── service/           # Lógica de negocio (capa de aplicación)
│
├── dto/                   # Data Transfer Objects
├── exception/             # Manejo global de excepciones
└── DemoApplication.java   # Punto de entrada
```

### Entidades principales

- **Recipe** — Receta de yogurt con parámetros de temperatura e incubación.
- **YogurtBatch** — Lote de producción con su ciclo de vida completo.
- **Ingredient** — Ingrediente asociado a una receta.
- **TemperatureLog** — Registro histórico de temperaturas por lote.

### Estados de un lote (`BatchStatus`)

```
PREPARING → HEATING → COOLING → INOCULATING → INCUBATING → REFRIGERATING → COMPLETED
                                                                           ↘ FAILED
```

---

## 🌐 Endpoints Principales

### Recetas (`/api/recipes`)

| Método | Ruta | Descripción |
|---|---|---|
| `POST` | `/api/recipes` | Crear nueva receta |
| `GET` | `/api/recipes` | Listar todas las recetas activas |
| `GET` | `/api/recipes/{id}` | Obtener receta por ID |
| `PUT` | `/api/recipes/{id}` | Actualizar receta |
| `GET` | `/api/recipes/search?keyword=` | Buscar recetas |
| `PATCH` | `/api/recipes/{id}/activate` | Activar receta |
| `PATCH` | `/api/recipes/{id}/deactivate` | Desactivar receta |

### Lotes de Producción (`/api/batches`)

| Método | Ruta | Descripción |
|---|---|---|
| `POST` | `/api/batches` | Iniciar nuevo lote |
| `GET` | `/api/batches` | Listar lotes (filtrable por estado) |
| `GET` | `/api/batches/{id}` | Obtener lote por ID |
| `POST` | `/api/batches/{id}/heating` | Iniciar calentamiento |
| `POST` | `/api/batches/{id}/inoculating` | Iniciar inoculación |
| `POST` | `/api/batches/{id}/incubation` | Iniciar incubación |
| `POST` | `/api/batches/{id}/refrigeration` | Iniciar refrigeración |
| `POST` | `/api/batches/{id}/complete` | Completar lote |
| `POST` | `/api/batches/{id}/fail` | Marcar lote como fallido |
| `POST` | `/api/batches/{id}/temperature` | Registrar temperatura |

### Monitoreo (`/api/monitoring`)

| Método | Ruta | Descripción |
|---|---|---|
| `GET` | `/api/monitoring/active` | Ver lotes activos |
| `GET` | `/api/monitoring/stats` | Estadísticas generales |

---

## 🚀 Instalación y Ejecución

### Prerrequisitos

- **Java 17** o superior
- **Maven 3.9+** (o usar el wrapper incluido `./mvnw`)
- Git

### Pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/lucipulires/YogurtMarketLuci.git
cd YogurtMarketLuci

# 2. Compilar el proyecto
./mvnw clean install

# 3. Ejecutar la aplicación
./mvnw spring-boot:run
```

La API estará disponible en: `http://localhost:8080`

### Accesos rápidos

| Recurso | URL |
|---|---|
| Swagger UI | http://localhost:8080/swagger-ui.html |
| H2 Console | http://localhost:8080/h2-console |
| Actuator | http://localhost:8080/actuator |

> **H2 Console:** JDBC URL: `jdbc:h2:mem:yogurtdb`, Usuario: `sa`, Contraseña: *(vacía)*

---

## 📘 Manual de Usuario

Ver [`docs/MANUAL.md`](docs/MANUAL.md) para el manual completo con ejemplos de peticiones, respuestas y flujo de uso.

---

## 📁 Estructura del Proyecto

```
YogurtMarketLuci/
│
├── src/
│   ├── main/
│   │   ├── java/com/danieldev87/demo/
│   │   │   ├── domain/
│   │   │   │   ├── controller/
│   │   │   │   │   ├── MonitoringController.java
│   │   │   │   │   ├── RecipeController.java
│   │   │   │   │   └── YogurtBatchController.java
│   │   │   │   ├── model/
│   │   │   │   │   ├── Ingredient.java
│   │   │   │   │   ├── Recipe.java
│   │   │   │   │   ├── TemperatureLog.java
│   │   │   │   │   └── YogurtBatch.java
│   │   │   │   ├── repository/
│   │   │   │   └── service/
│   │   │   ├── dto/
│   │   │   ├── exception/
│   │   │   └── DemoApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│
├── docs/
│   ├── MANUAL.md
│   └── evidencias/
│
├── .gitignore
├── LICENSE
├── pom.xml
└── README.md
```

---

## 🌿 Ramas del Repositorio

| Rama | Descripción |
|---|---|
| `main` | Versión estable y funcional |
| `develop` | Integración de nuevas funcionalidades |
| `feature/recipes` | Módulo de gestión de recetas |
| `feature/batches` | Módulo de control de lotes |
| `feature/monitoring` | Módulo de monitoreo y temperatura |
| `docs` | Documentación, manual y evidencias |

---

## 👤 Autor

**danieldev87**

- GitHub: [@lucipulires](https://github.com/lucipulires)

---

## 📄 Licencia

Este proyecto está bajo la licencia **MIT**. Ver el archivo [`LICENSE`](LICENSE) para más detalles.
