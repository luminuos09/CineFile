# CineFile — Documento de Alcance y Stack Tecnológico

> **Versión:** 1.0  
> **Estado:** Definición inicial (pre-desarrollo)  
> **Licencia propuesta:** MIT  
> **Repositorio:** GitHub (código abierto)

---

## 1. Descripción del Proyecto

**CineFile** es una plantilla de código abierto (boilerplate) para construir aplicaciones web de **seguimiento de películas y series**. No reproduce contenido audiovisual ni aloja archivos: es una herramienta de organización personal que permite al usuario registrar qué está viendo, qué quiere ver, qué ha terminado y qué ha abandonado, con puntuaciones, notas y progreso.

El proyecto está pensado para que futuros desarrolladores lo clonen, lo ejecuten y lo personalicen (por ejemplo, adaptándolo a un nicho concreto como el anime), o lo usen como base para añadir sus propias funcionalidades.

---

## 2. Alcance del Proyecto

### 2.1 Lo que SÍ incluye CineFile

| Módulo | Contenido |
| :--- | :--- |
| **Autenticación de usuarios** | Registro, inicio de sesión, cierre de sesión y gestión de sesiones mediante JWT. |
| **Base de datos** | Persistencia de usuarios y entradas de seguimiento en PostgreSQL (o H2 en desarrollo). |
| **Separación de datos** | Cada usuario solo puede ver y modificar sus propios datos. |
| **Integración con TMDB** | Obtención de metadatos (carátulas, sinopsis, año, reparto) a través de un proxy en el backend para proteger la API key. |
| **Seguimiento de medios** | Estados: **Viendo**, **Pendiente**, **Finalizado**, **Abandonado**. Puntuación (1–10), notas personales y progreso (episodio actual en series). |
| **Interfaz de usuario** | HTML semántico, CSS responsive (Flexbox/Grid) y JavaScript vanilla para manipulación del DOM y `fetch`. Modo oscuro/claro. |
| **Búsqueda** | Búsqueda de películas y series en tiempo real contra la API de TMDB (con debounce). |
| **Panel de estadísticas** | Resumen de actividad del usuario (títulos vistos, género favorito, etc.). |

### 2.2 Lo que NO incluye CineFile

| Exclusión | Motivo |
| :--- | :--- |
| **Reproducción de video** | CineFile es un *tracker*, no un *streamer*. Evita problemas legales y mantiene el foco del código. |
| **Alojamiento de contenido** | No se almacenan archivos audiovisuales en ningún servidor del proyecto. |
| **Sistema de planes Premium** | Si el dueño del código quiere monetizar, añadirá su propia pasarela de pago. |
| **Sincronización offline avanzada** | La plantilla asume conexión a internet para hablar con el backend. |

### 2.3 Filosofía del proyecto

- **Código abierto y educativo:** sirve como referencia de arquitectura cliente-servidor clásica.
- **Migrable:** el frontend en JS vanilla puede migrarse progresivamente a React sin tocar el backend.
- **Sin dependencias innecesarias:** el frontend no usa frameworks; el backend usa el ecosistema estándar de Spring.

---

## 3. Stack Tecnológico

### 3.1 Visión general

| Capa | Tecnología | Justificación |
| :--- | :--- | :--- |
| **Frontend** | HTML5 + CSS3 + **JavaScript Vanilla (ES6+)** | Sin frameworks. Ideal para una plantilla que enseña fundamentos. Se comunica con el backend vía `fetch`. |
| **Backend** | **Java 17+ + Spring Boot 3+** | Robusto, tipado, con ecosistema maduro para seguridad y persistencia. |
| **API** | **REST (JSON)** | Comunicación estándar entre frontend y backend. |
| **Autenticación** | **Spring Security + JWT** | Estándar para APIs. El token se envía en el header `Authorization: Bearer <token>`. |
| **Base de Datos** | **PostgreSQL** (recomendado) o **H2** (para desarrollo) | PostgreSQL es producción-ready; H2 simplifica el "clonar y ejecutar". |
| **ORM** | **Spring Data JPA (Hibernate)** | Integración nativa con Spring Boot, reduce boilerplate. |
| **Build Backend** | **Maven** | Estándar en proyectos Spring. |
| **Build Frontend** | **Ninguno** (archivos estáticos) | Se sirven directamente o desde Spring Boot como recursos estáticos. |
| **CORS** | **Configuración en Spring Boot** | Crítico en desarrollo: frontend y backend corren en puertos distintos. |

### 3.2 Notas sobre el despliegue

- **Desarrollo:** frontend servido con Live Server (ej. `localhost:5500`) y backend en `localhost:8080`. Requiere configuración de CORS.
- **Producción:** Spring Boot puede servir los archivos estáticos del frontend desde `src/main/resources/static/`, unificando todo en un solo puerto y evitando CORS.

---

## 4. Arquitectura y Estructura del Repositorio

### 4.1 Estructura de carpetas recomendada

```
cinefile/
├── backend/          (proyecto Maven Spring Boot)
│   ├── src/main/java/com/cinefile/
│   │   ├── controller/   (REST endpoints)
│   │   ├── service/      (lógica de negocio)
│   │   ├── repository/   (Spring Data JPA)
│   │   ├── model/        (entidades)
│   │   ├── dto/          (request/response)
│   │   └── config/       (CORS, Security, JWT)
│   └── src/main/resources/
│       └── application.yml
├── frontend/         (HTML/CSS/JS vanilla)
│   ├── index.html
│   ├── css/
│   ├── js/
│   └── assets/
└── README.md
```

### 4.2 Modelo de datos (entidades JPA)

| Entidad | Campos principales |
| :--- | :--- |
| **User** | `id`, `username`, `email`, `passwordHash`, `createdAt` |
| **WatchlistEntry** | `id`, `userId`, `tmdbId`, `mediaType` (movie/series), `status` (VIENDO/PENDIENTE/FINALIZADO/ABANDONADO), `rating` (1–10), `notes`, `currentEpisode`, `createdAt`, `updatedAt` |

### 4.3 Endpoints REST principales

| Método | Endpoint | Descripción |
| :--- | :--- | :--- |
| `POST` | `/api/auth/register` | Registro de usuario |
| `POST` | `/api/auth/login` | Login, devuelve JWT |
| `GET` | `/api/movies/search?q=` | Búsqueda proxy a TMDB |
| `GET` | `/api/watchlist` | Listar entradas del usuario autenticado |
| `POST` | `/api/watchlist` | Añadir entrada |
| `PUT` | `/api/watchlist/{id}` | Actualizar estado, puntuación, progreso |
| `DELETE` | `/api/watchlist/{id}` | Eliminar entrada |

---

## 5. Consideraciones Críticas

### 5.1 CORS en desarrollo
Con frontend y backend en puertos distintos, Spring Boot debe configurar `WebMvcConfigurer` para permitir el origen del frontend. Si se usa Spring Security, también debe llamarse a `.cors()` en la cadena de filtros, o los preflight `OPTIONS` serán bloqueados.

### 5.2 Gestión del token JWT en JS Vanilla
- **Opción A (simple):** guardar el JWT en `localStorage` y adjuntarlo en cada `fetch`. Vulnerable a XSS.
- **Opción B (segura):** el backend devuelve el JWT en una cookie `HttpOnly`; el navegador la envía automáticamente y JS no puede leerla.
- La plantilla debe documentar ambas opciones.

### 5.3 Migración futura a React
Estrategia recomendada: **strangler pattern** (migración progresiva).
1. Configurar Vite + React + TypeScript en una carpeta `frontend/`.
2. Migrar componente por componente (primero el layout, luego las páginas).
3. El backend Spring Boot **no cambia**; solo cambia quién consume la API.
4. En producción, el build de React (`dist/`) se copia a `src/main/resources/static/` de Spring Boot.

### 5.4 Variables de entorno
Quien clone el proyecto necesitará configurar: `DATABASE_URL`, `JWT_SECRET`, `TMDB_API_KEY`. Debe documentarse claramente en el README.

---

## 6. Resumen del Alcance para el README

> **CineFile** es una plantilla full-stack de código abierto para seguimiento de películas/series. **Backend:** Java Spring Boot + Spring Security + JWT + Spring Data JPA + PostgreSQL/H2. **Frontend:** HTML, CSS y JavaScript vanilla (sin frameworks). Incluye autenticación de usuarios, separación de datos por usuario, integración con TMDB vía proxy del backend, y gestión completa de estados de visionado (Viendo, Pendiente, Finalizado, Abandonado) con puntuaciones y progreso. **No incluye** reproducción de video. Diseñada para ser migrable a React en el futuro sin modificar el backend.

---

## 7. Roadmap sugerido

| Fase | Entregable |
| :--- | :--- |
| **Fase 1** | Configuración del backend: Spring Boot + Security + JWT + CORS + entidades JPA. |
| **Fase 2** | Endpoints REST de autenticación y CRUD de watchlist. |
| **Fase 3** | Frontend vanilla: login/registro, búsqueda, listas, detalle. |
| **Fase 4** | Integración TMDB (proxy) y panel de estadísticas. |
| **Fase 5** | Documentación (README, `.env.example`, guía de migración a React). |

---

*Documento generado para el repositorio de CineFile. Listo para copiar en `README.md` o en `docs/scope.md`.*
