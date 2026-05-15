# 📘 Manual de Usuario — YogurtMarket API

**Versión:** 0.0.1-SNAPSHOT  
**Última actualización:** Mayo 2026  
**Autor:** danieldev87

---

## Tabla de Contenido

1. [Introducción](#1-introducción)
2. [Requisitos previos](#2-requisitos-previos)
3. [Instalación](#3-instalación)
4. [Configuración](#4-configuración)
5. [Uso de la API](#5-uso-de-la-api)
   - 5.1 [Gestión de Recetas](#51-gestión-de-recetas)
   - 5.2 [Control de Lotes](#52-control-de-lotes)
   - 5.3 [Monitoreo y Temperatura](#53-monitoreo-y-temperatura)
6. [Flujo de producción completo](#6-flujo-de-producción-completo)
7. [Swagger UI](#7-swagger-ui)
8. [Consola H2](#8-consola-h2)
9. [Solución de problemas](#9-solución-de-problemas)

---

## 1. Introducción

**YogurtMarket** es una API REST diseñada para gestionar el proceso completo de producción de yogurt artesanal. A través de sus endpoints es posible:

- Crear y administrar recetas con sus respectivos ingredientes y parámetros técnicos.
- Crear lotes de producción vinculados a recetas.
- Avanzar cada lote por sus etapas de producción.
- Registrar lecturas de temperatura en tiempo real.
- Consultar estadísticas de los lotes activos.

La aplicación se ejecuta de forma local y utiliza una base de datos en memoria (H2), por lo que **no requiere instalación de base de datos externa**.

---

## 2. Requisitos Previos

| Herramienta | Versión mínima | Descarga |
|---|---|---|
| Java (JDK) | 17 | https://adoptium.net |
| Maven | 3.9+ | https://maven.apache.org |
| Git | 2.x | https://git-scm.com |
| Cliente HTTP | cualquiera | Postman / Insomnia / curl |

> Si usas el Maven Wrapper incluido (`./mvnw`), no necesitas instalar Maven por separado.

---

## 3. Instalación

### 3.1 Clonar el repositorio

```bash
git clone https://github.com/lucipulires/YogurtMarketLuci.git
cd YogurtMarketLuci
```

### 3.2 Compilar el proyecto

```bash
# Con Maven Wrapper (recomendado)
./mvnw clean install

# En Windows
mvnw.cmd clean install
```

### 3.3 Ejecutar la aplicación

```bash
./mvnw spring-boot:run
```

Deberías ver en consola:

```
Tomcat started on port(s): 8080 (http)
Started DemoApplication in X.XXX seconds
```

La API queda disponible en: **http://localhost:8080**

---

## 4. Configuración

El archivo `src/main/resources/application.properties` contiene la configuración principal:

```properties
# Puerto del servidor
server.port=8080

# Base de datos H2 en memoria
spring.datasource.url=jdbc:h2:mem:yogurtdb
spring.datasource.username=sa
spring.datasource.password=

# Consola H2 habilitada
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

> Para cambiar el puerto, modifica `server.port` y reinicia la aplicación.

---

## 5. Uso de la API

Todos los endpoints usan `Content-Type: application/json`.  
Base URL: `http://localhost:8080`

---

### 5.1 Gestión de Recetas

#### ✅ Crear una receta

**`POST /api/recipes`**

```json
{
  "name": "Yogurt Natural Clásico",
  "description": "Receta tradicional de yogurt natural",
  "type": "NATURAL",
  "defaultMilkVolume": 5.0,
  "defaultStarterAmount": 2.5,
  "heatingTemperature": 85.0,
  "inoculationTemperature": 43.0,
  "minIncubationTime": 6,
  "maxIncubationTime": 8,
  "refrigerationTemperature": 4.0
}
```

**Respuesta exitosa (201 Created):**

```json
{
  "id": 1,
  "name": "Yogurt Natural Clásico",
  "type": "NATURAL",
  "active": true,
  "defaultMilkVolume": 5.0,
  "inoculationTemperature": 43.0,
  "minIncubationTime": 6
}
```

---

#### 📋 Listar todas las recetas activas

**`GET /api/recipes`**

```bash
curl http://localhost:8080/api/recipes
```

---

#### 🔍 Buscar recetas por palabra clave

**`GET /api/recipes/search?keyword=natural`**

```bash
curl "http://localhost:8080/api/recipes/search?keyword=natural"
```

---

#### 🔄 Actualizar receta

**`PUT /api/recipes/{id}`**

```json
{
  "name": "Yogurt Natural Clásico v2",
  "defaultMilkVolume": 6.0
}
```

---

#### ⛔ Desactivar receta

**`PATCH /api/recipes/1/deactivate`**

```bash
curl -X PATCH http://localhost:8080/api/recipes/1/deactivate
```

---

### 5.2 Control de Lotes

#### 🆕 Iniciar un nuevo lote

**`POST /api/batches`**

```json
{
  "recipeId": 1,
  "customMilkVolume": 5.0,
  "customStarterAmount": 2.5
}
```

**Respuesta (201 Created):**

```json
{
  "id": 1,
  "batchCode": "YB-1716789012345",
  "status": "PREPARING",
  "milkVolume": 5.0,
  "starterAmount": 2.5,
  "targetTemperature": 43.0,
  "incubationTime": 6,
  "startTime": "2026-05-15T10:00:00"
}
```

---

#### 🔥 Avanzar etapas del lote

Cada endpoint cambia el estado del lote:

| Acción | Endpoint | Estado resultante |
|---|---|---|
| Iniciar calentamiento | `POST /api/batches/1/heating` | `HEATING` |
| Iniciar inoculación | `POST /api/batches/1/inoculating` | `INOCULATING` |
| Iniciar incubación | `POST /api/batches/1/incubation` | `INCUBATING` |
| Iniciar refrigeración | `POST /api/batches/1/refrigeration` | `REFRIGERATING` |
| Completar lote | `POST /api/batches/1/complete` | `COMPLETED` |

```bash
# Ejemplo: iniciar calentamiento del lote 1
curl -X POST http://localhost:8080/api/batches/1/heating
```

---

#### ❌ Marcar lote como fallido

**`POST /api/batches/{id}/fail`**

```json
{
  "reason": "Temperatura excedió el límite permitido"
}
```

---

#### 📋 Listar lotes (con filtro opcional por estado)

```bash
# Todos los lotes
curl http://localhost:8080/api/batches

# Solo lotes en incubación
curl "http://localhost:8080/api/batches?status=INCUBATING"
```

Estados disponibles: `PREPARING`, `HEATING`, `COOLING`, `INOCULATING`, `INCUBATING`, `REFRIGERATING`, `COMPLETED`, `FAILED`

---

### 5.3 Monitoreo y Temperatura

#### 🌡️ Registrar temperatura

**`POST /api/batches/{id}/temperature`**

```json
{
  "temperature": 43.5,
  "type": "INOCULATION"
}
```

---

#### 📊 Ver lotes activos

**`GET /api/monitoring/active`**

```bash
curl http://localhost:8080/api/monitoring/active
```

---

#### 📈 Estadísticas generales

**`GET /api/monitoring/stats`**

```bash
curl http://localhost:8080/api/monitoring/stats
```

---

## 6. Flujo de Producción Completo

Este es el flujo recomendado para un lote exitoso:

```
1. Crear receta        → POST /api/recipes
2. Iniciar lote        → POST /api/batches  (estado: PREPARING)
3. Calentar leche      → POST /api/batches/1/heating  (estado: HEATING)
4. Registrar temp.     → POST /api/batches/1/temperature  {"temperature": 85}
5. Inocular            → POST /api/batches/1/inoculating  (estado: INOCULATING)
6. Iniciar incubación  → POST /api/batches/1/incubation  (estado: INCUBATING)
7. Refrigerar          → POST /api/batches/1/refrigeration  (estado: REFRIGERATING)
8. Completar           → POST /api/batches/1/complete  (estado: COMPLETED)
```

---

## 7. Swagger UI

Spring Boot con SpringDoc genera automáticamente la documentación interactiva.

**URL:** http://localhost:8080/swagger-ui.html

Desde ahí puedes:
- Explorar todos los endpoints disponibles.
- Probar peticiones directamente desde el navegador.
- Ver los modelos de datos (schemas).

---

## 8. Consola H2

La base de datos en memoria H2 tiene una consola web integrada.

**URL:** http://localhost:8080/h2-console

**Configuración de conexión:**

| Campo | Valor |
|---|---|
| JDBC URL | `jdbc:h2:mem:yogurtdb` |
| User Name | `sa` |
| Password | *(dejar vacío)* |

Desde aquí puedes ejecutar consultas SQL directamente:

```sql
SELECT * FROM YOGURT_BATCHES;
SELECT * FROM RECIPE;
SELECT * FROM TEMPERATURE_LOG WHERE BATCH_ID = 1;
```

---

## 9. Solución de Problemas

### Puerto 8080 en uso

```bash
# Linux/Mac: encontrar proceso usando el puerto
lsof -i :8080
kill -9 <PID>

# Windows
netstat -ano | findstr :8080
taskkill /PID <PID> /F
```

O cambiar el puerto en `application.properties`:
```properties
server.port=8081
```

### Error de compilación con Java

Verificar la versión instalada:
```bash
java -version
# Debe mostrar: openjdk version "17.x.x"
```

### La aplicación inicia pero los endpoints dan 404

Verificar que estás usando el prefijo `/api/` en todas las rutas. Por ejemplo:
- ✅ `http://localhost:8080/api/recipes`
- ❌ `http://localhost:8080/recipes`

---

*Para reportar errores o sugerencias, abrir un Issue en el repositorio de GitHub.*
