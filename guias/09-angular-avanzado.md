# Guía 09 — Angular Avanzado: Consumiendo el Backend del Triage

> **Materia:** Programación Avanzada | **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia | **Año:** 2026  
> **Guía número:** 09 de 12

---

## Tabla de Contenidos

1. [Objetivos de Aprendizaje](#1-objetivos-de-aprendizaje)
2. [Prerrequisitos](#2-prerrequisitos)
3. [Configurar HttpClient con la URL Base del Backend](#3-configurar-httpclient-con-la-url-base-del-backend)
4. [Interfaces TypeScript que Corresponden a los DTOs de Spring Boot](#4-interfaces-typescript-que-corresponden-a-los-dtos-de-spring-boot)
5. [Generar Cliente HTTP desde el Contrato OpenAPI](#5-generar-cliente-http-desde-el-contrato-openapi)
6. [Servicios Angular: SolicitudService, AuthService y UsuarioService](#6-servicios-angular-solicitudservice-authservice-y-usuarioservice)
7. [Interceptor JWT: Adjuntar Token Automáticamente](#7-interceptor-jwt-adjuntar-token-automáticamente)
8. [Interceptor de Errores: Manejo Global](#8-interceptor-de-errores-manejo-global)
9. [AuthGuard y RoleGuard con la Nueva API Funcional de Angular 17+](#9-authguard-y-roleguard-con-la-nueva-api-funcional-de-angular-17)
10. [Formularios Reactivos Completos](#10-formularios-reactivos-completos)
11. [Tabla de Solicitudes con Paginación y Filtros del Lado del Servidor](#11-tabla-de-solicitudes-con-paginación-y-filtros-del-lado-del-servidor)
12. [Componente de Historial: Timeline Visual](#12-componente-de-historial-timeline-visual)
13. [Manejo de Estado con BehaviorSubject](#13-manejo-de-estado-con-behaviorsubject)
14. [Notificaciones Toast y Loading States](#14-notificaciones-toast-y-loading-states)
15. [Antes vs Ahora: Resumen de Evolución Angular](#15-antes-vs-ahora-resumen-de-evolución-angular)
16. [Errores Comunes y Troubleshooting](#16-errores-comunes-y-troubleshooting)
17. [Ejercicios Prácticos](#17-ejercicios-prácticos)
18. [Resumen y Cheat Sheet](#18-resumen-y-cheat-sheet)
19. [Referencias y Recursos Adicionales](#19-referencias-y-recursos-adicionales)

---

## 1. Objetivos de Aprendizaje

Al finalizar esta guía serás capaz de:

- Configurar correctamente las variables de entorno en Angular para conectarte al backend en Spring Boot.
- Crear interfaces TypeScript que reflejen exactamente los DTOs del backend.
- Implementar servicios Angular que consuman todos los endpoints del sistema de Triage.
- Configurar interceptores HTTP para adjuntar el JWT automáticamente en cada petición.
- Manejar errores HTTP de forma global y mostrar mensajes útiles al usuario.
- Proteger rutas con guards funcionales basados en autenticación y roles.
- Construir formularios reactivos completos con validaciones síncronas y asíncronas.
- Implementar tablas con paginación y filtros del lado del servidor.
- Visualizar el historial de una solicitud como una línea de tiempo.
- Gestionar estado compartido entre componentes usando `BehaviorSubject`.
- Mostrar notificaciones y estados de carga (spinners, skeletons) de forma profesional.

---

## 2. Prerrequisitos

Antes de continuar, necesitas haber trabajado con:

- **Guía 08** — Angular desde Cero hasta Intermedio (componentes, servicios, routing, HttpClient, Reactive Forms).
- **Guía 06** — API REST con Spring Boot (sabes qué endpoints existen y qué datos reciben/devuelven).
- **Guía 07** — Spring Security + JWT (sabes qué token genera el backend y cómo funciona).
- Conocimiento de RxJS básico: `Observable`, `subscribe`, `pipe`, `map`, `catchError`.

Verificaciones rápidas antes de empezar:

```bash
# Verificar versiones instaladas
node --version    # Debe ser 20.x o superior
npm --version     # Debe ser 10.x o superior
ng version        # Debe mostrar Angular 19+

# El backend Spring Boot debe estar corriendo en:
# http://localhost:8080
```

---

## 3. Configurar HttpClient con la URL Base del Backend

### 3.1 ¿Por qué necesitas variables de entorno?

Imagina este escenario: durante el desarrollo, tu backend corre en `http://localhost:8080`. Pero cuando despliegas la aplicación en producción, el backend vive en `https://triage-api.railway.app`. Si tienes esa URL escrita directamente en tu código (lo que se llama *hardcodeado*), cada vez que cambies de entorno tendrías que buscar y reemplazar en todos tus archivos. Eso es propenso a errores y poco profesional.

Angular ofrece **archivos de entorno** que resuelven este problema: defines la URL una sola vez por entorno y Angular se encarga de usar la correcta al compilar.

### 3.2 Estructura de carpetas inicial del proyecto frontend

Antes de escribir una sola línea de código, verifica que tu proyecto tiene esta estructura. Si estás creando el proyecto desde cero:

```bash
ng new triage-frontend --standalone --routing --style=scss
cd triage-frontend
```

La estructura que vamos a construir a lo largo de esta guía es:

```
triage-frontend/
├── src/
│   ├── app/
│   │   ├── core/
│   │   │   ├── auth/
│   │   │   │   ├── auth.guard.ts
│   │   │   │   ├── role.guard.ts
│   │   │   │   └── auth.interceptor.ts
│   │   │   ├── interceptors/
│   │   │   │   └── error.interceptor.ts
│   │   │   ├── models/
│   │   │   │   ├── solicitud.model.ts
│   │   │   │   ├── usuario.model.ts
│   │   │   │   ├── historial.model.ts
│   │   │   │   └── page.model.ts
│   │   │   └── services/
│   │   │       ├── auth.service.ts
│   │   │       ├── solicitud.service.ts
│   │   │       ├── usuario.service.ts
│   │   │       └── notification.service.ts
│   │   ├── features/
│   │   │   ├── auth/
│   │   │   │   ├── login/
│   │   │   │   │   ├── login.component.ts
│   │   │   │   │   └── login.component.html
│   │   │   │   └── registro/
│   │   │   │       ├── registro.component.ts
│   │   │   │       └── registro.component.html
│   │   │   ├── solicitudes/
│   │   │   │   ├── lista-solicitudes/
│   │   │   │   │   ├── lista-solicitudes.component.ts
│   │   │   │   │   └── lista-solicitudes.component.html
│   │   │   │   ├── crear-solicitud/
│   │   │   │   │   ├── crear-solicitud.component.ts
│   │   │   │   │   └── crear-solicitud.component.html
│   │   │   │   ├── detalle-solicitud/
│   │   │   │   │   ├── detalle-solicitud.component.ts
│   │   │   │   │   └── detalle-solicitud.component.html
│   │   │   │   └── clasificar-solicitud/
│   │   │   │       ├── clasificar-solicitud.component.ts
│   │   │   │       └── clasificar-solicitud.component.html
│   │   │   └── dashboard/
│   │   │       ├── dashboard.component.ts
│   │   │       └── dashboard.component.html
│   │   ├── shared/
│   │   │   ├── components/
│   │   │   │   ├── historial-timeline/
│   │   │   │   │   ├── historial-timeline.component.ts
│   │   │   │   │   └── historial-timeline.component.html
│   │   │   │   ├── loading-spinner/
│   │   │   │   │   └── loading-spinner.component.ts
│   │   │   │   └── toast/
│   │   │   │       └── toast.component.ts
│   │   │   └── pipes/
│   │   │       └── estado-label.pipe.ts
│   │   ├── app.component.ts
│   │   ├── app.component.html
│   │   └── app.routes.ts
│   └── environments/
│       ├── environment.ts
│       └── environment.prod.ts
├── angular.json
├── package.json
└── tsconfig.json
```

### 3.3 Crear los archivos de entorno

En Angular 15+ los archivos de entorno ya no se generan automáticamente. Los creas manualmente:

```bash
# Crear la carpeta environments dentro de src/
mkdir src/environments
touch src/environments/environment.ts
touch src/environments/environment.prod.ts
```

**`src/environments/environment.ts`** — configuración para desarrollo:

```typescript
// src/environments/environment.ts

export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080/api/v1',
  appName: 'Sistema de Triage Académico',
  tokenKey: 'triage_auth_token',
  refreshTokenKey: 'triage_refresh_token',
  tokenExpiryKey: 'triage_token_expiry',
};
```

**`src/environments/environment.prod.ts`** — configuración para producción:

```typescript
// src/environments/environment.prod.ts
// Estas URLs vendrán de las variables de entorno del servidor de despliegue (Railway, Vercel, etc.)

export const environment = {
  production: true,
  apiUrl: 'https://triage-api.railway.app/api/v1',  // Cambia esto a tu URL real
  appName: 'Sistema de Triage Académico',
  tokenKey: 'triage_auth_token',
  refreshTokenKey: 'triage_refresh_token',
  tokenExpiryKey: 'triage_token_expiry',
};
```

### 3.4 Registrar los reemplazos de entorno en angular.json

Abre `angular.json` y busca la sección `"configurations"` dentro de `"build"`. Agrega el reemplazo de archivos para producción:

```json
"configurations": {
  "production": {
    "fileReplacements": [
      {
        "replace": "src/environments/environment.ts",
        "with": "src/environments/environment.prod.ts"
      }
    ]
  }
}
```

Esto le indica a Angular que cuando hagas `ng build --configuration production`, reemplace automáticamente `environment.ts` con `environment.prod.ts`. Durante desarrollo (`ng serve`), siempre usa `environment.ts`.

### 3.5 Configurar HttpClient con la URL base

En Angular 19+ la forma recomendada es usar `provideHttpClient()` con interceptores funcionales y configurar una URL base con `withInterceptors()`. Abre `app.config.ts` (o créalo si no existe):

```typescript
// src/app/app.config.ts

import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter, withComponentInputBinding } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { routes } from './app.routes';
import { authInterceptor } from './core/auth/auth.interceptor';
import { errorInterceptor } from './core/interceptors/error.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    // Mejora de rendimiento: usa señales zoneless (Angular 19+)
    provideZoneChangeDetection({ eventCoalescing: true }),

    // Router con soporte para pasar parámetros de ruta como @Input()
    provideRouter(routes, withComponentInputBinding()),

    // HttpClient con interceptores funcionales registrados en orden:
    // 1. authInterceptor: adjunta el JWT en cada petición
    // 2. errorInterceptor: maneja errores globalmente
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    ),
  ],
};
```

---

## 4. Interfaces TypeScript que Corresponden a los DTOs de Spring Boot

### 4.1 ¿Por qué TypeScript necesita saber la forma de los datos?

Cuando tu servicio Angular recibe datos del backend, JavaScript por defecto no sabe qué propiedades tiene ese objeto. TypeScript te permite *describir* la forma de los datos con interfaces. Esto te da autocompletado, detección de errores en tiempo de compilación, y documentación automática.

Piénsalo así: si el backend devuelve un JSON con una propiedad llamada `fechaRegistro` y en Angular la escribes como `fecharegistro` (sin mayúscula), TypeScript te avisa del error inmediatamente. Sin interfaces, ese error solo aparece en tiempo de ejecución — cuando el usuario ya está usando la app.

La regla de oro: **por cada DTO de Spring Boot, debe existir una interfaz TypeScript equivalente en el frontend**.

### 4.2 Crear los modelos del dominio

```
src/app/core/models/
├── solicitud.model.ts
├── usuario.model.ts
├── historial.model.ts
└── page.model.ts
```

**`src/app/core/models/page.model.ts`** — modelo genérico para respuestas paginadas de Spring:

```typescript
// src/app/core/models/page.model.ts
// Spring Boot devuelve la paginación con esta estructura cuando usas Page<T>
// Necesitamos un modelo genérico que funcione para cualquier tipo T

export interface Page<T> {
  content: T[];           // Los elementos de la página actual
  totalElements: number;  // Total de elementos en la BD (sin paginar)
  totalPages: number;     // Número total de páginas
  size: number;           // Elementos por página
  number: number;         // Número de página actual (0-based en Spring)
  first: boolean;         // ¿Es la primera página?
  last: boolean;          // ¿Es la última página?
  empty: boolean;         // ¿La página está vacía?
  numberOfElements: number; // Elementos en esta página
}

// Parámetros que Angular envía al backend para paginar
export interface PageRequest {
  page: number;
  size: number;
  sort?: string;        // Ejemplo: "fechaRegistro,desc"
}
```

**`src/app/core/models/usuario.model.ts`** — interfaces para usuarios y autenticación:

```typescript
// src/app/core/models/usuario.model.ts

// Roles que el sistema maneja — deben coincidir exactamente con el enum del backend
export type Rol = 'ESTUDIANTE' | 'FUNCIONARIO' | 'ADMIN';

// Datos del usuario autenticado (lo que va dentro del JWT)
export interface UsuarioAutenticado {
  id: number;
  nombre: string;
  email: string;
  rol: Rol;
}

// DTO de respuesta al hacer login — lo que devuelve POST /auth/login
export interface LoginResponse {
  accessToken: string;
  refreshToken: string;
  tokenType: string;        // Siempre "Bearer"
  expiresIn: number;        // Segundos hasta expiración
  usuario: UsuarioAutenticado;
}

// DTO de solicitud para hacer login — lo que enviamos a POST /auth/login
export interface LoginRequest {
  email: string;
  password: string;
}

// DTO de solicitud para registro — lo que enviamos a POST /auth/registro
export interface RegistroRequest {
  nombre: string;
  apellido: string;
  email: string;
  password: string;
  codigoEstudiantil?: string;  // Opcional: solo para rol ESTUDIANTE
}

// DTO de respuesta para consulta de usuario — lo que devuelve GET /usuarios/{id}
export interface UsuarioResponse {
  id: number;
  nombre: string;
  apellido: string;
  email: string;
  rol: Rol;
  activo: boolean;
  fechaCreacion: string;  // ISO 8601: "2026-03-15T10:30:00Z"
}
```

**`src/app/core/models/solicitud.model.ts`** — el núcleo del sistema de Triage:

```typescript
// src/app/core/models/solicitud.model.ts

// Enums del sistema — deben coincidir con los del backend
export type EstadoSolicitud =
  | 'REGISTRADA'
  | 'CLASIFICADA'
  | 'EN_ATENCION'
  | 'ATENDIDA'
  | 'CERRADA';

export type TipoSolicitud =
  | 'REGISTRO_ASIGNATURAS'
  | 'HOMOLOGACION'
  | 'CANCELACION_ASIGNATURAS'
  | 'SOLICITUD_CUPOS'
  | 'CONSULTA_ACADEMICA'
  | 'OTRO';

export type PrioridadSolicitud = 'BAJA' | 'MEDIA' | 'ALTA' | 'CRITICA';

export type CanalOrigen =
  | 'CSU'
  | 'CORREO'
  | 'SAC'
  | 'TELEFONICO'
  | 'PRESENCIAL';

// DTO que enviamos al backend para crear una solicitud (RF-01)
// Corresponde a CrearSolicitudRequest.java en el backend
export interface CrearSolicitudRequest {
  descripcion: string;
  canalOrigen: CanalOrigen;
}

// DTO que enviamos para clasificar una solicitud (RF-02, RF-03)
// Corresponde a ClasificarSolicitudRequest.java en el backend
export interface ClasificarSolicitudRequest {
  tipoSolicitud: TipoSolicitud;
  prioridad: PrioridadSolicitud;
  justificacionPrioridad: string;
}

// DTO que enviamos para asignar un responsable (RF-05)
export interface AsignarResponsableRequest {
  responsableId: number;
}

// DTO que enviamos para cambiar el estado (RF-04)
export interface CambiarEstadoRequest {
  nuevoEstado: EstadoSolicitud;
  observacion: string;
}

// DTO que enviamos para cerrar una solicitud (RF-08)
export interface CerrarSolicitudRequest {
  observacionCierre: string;
}

// DTO de respuesta — lo que el backend devuelve al consultar una solicitud
// Corresponde a SolicitudResponse.java en el backend
export interface SolicitudResponse {
  id: number;
  descripcion: string;
  estado: EstadoSolicitud;
  tipoSolicitud: TipoSolicitud | null;    // Null si aún no fue clasificada
  prioridad: PrioridadSolicitud | null;   // Null si aún no fue clasificada
  justificacionPrioridad: string | null;
  canalOrigen: CanalOrigen;
  fechaRegistro: string;                  // ISO 8601
  fechaUltimaActualizacion: string;
  solicitante: UsuarioResumenResponse;
  responsable: UsuarioResumenResponse | null;  // Null si no tiene responsable
}

// Versión resumida del usuario para incrustar en otras respuestas
export interface UsuarioResumenResponse {
  id: number;
  nombre: string;
  apellido: string;
  email: string;
  rol: string;
}

// Parámetros de filtro para consultar solicitudes (RF-07)
export interface FiltrosSolicitud {
  estado?: EstadoSolicitud;
  tipoSolicitud?: TipoSolicitud;
  prioridad?: PrioridadSolicitud;
  responsableId?: number;
  solicitanteId?: number;
  fechaDesde?: string;
  fechaHasta?: string;
}
```

**`src/app/core/models/historial.model.ts`** — historial auditable de acciones (RF-06):

```typescript
// src/app/core/models/historial.model.ts

// DTO de respuesta para cada entrada del historial
// Corresponde a HistorialSolicitudResponse.java en el backend
export interface HistorialSolicitudResponse {
  id: number;
  accion: string;                   // Descripción de la acción realizada
  estadoAnterior: string | null;    // Estado antes de la acción
  estadoNuevo: string | null;       // Estado después de la acción
  observaciones: string | null;
  fecha: string;                    // ISO 8601
  usuario: UsuarioResumenResponse;  // Quién realizó la acción
}

// Importamos aquí para no crear una dependencia circular
import { UsuarioResumenResponse } from './solicitud.model';
```

---

## 5. Generar Cliente HTTP desde el Contrato OpenAPI

### 5.1 ¿Qué es ng-openapi-gen y por qué usarlo?

En la Guía 06 configuraste Swagger/OpenAPI en Spring Boot. Eso genera un archivo `openapi.json` que describe todos tus endpoints, sus parámetros y sus respuestas.

`ng-openapi-gen` lee ese archivo y genera automáticamente todos los servicios y modelos TypeScript para consumir el backend. Esto evita tener que escribir el cliente HTTP manualmente y garantiza que el frontend siempre esté sincronizado con el backend.

> **Nota pedagógica:** En esta guía implementaremos los servicios manualmente para que entiendas exactamente qué hace cada línea. Pero en proyectos reales, `ng-openapi-gen` te ahorrará mucho tiempo.

```bash
# Instalar ng-openapi-gen como dependencia de desarrollo
npm install ng-openapi-gen --save-dev

# Crear el archivo de configuración
touch ng-openapi-gen.json
```

```json
// ng-openapi-gen.json
{
  "$schema": "node_modules/ng-openapi-gen/ng-openapi-gen-schema.json",
  "input": "http://localhost:8080/v3/api-docs",
  "output": "src/app/api",
  "module": false,
  "models": "src/app/api/models",
  "services": "src/app/api/services",
  "templates": false
}
```

```bash
# Generar el cliente (el backend debe estar corriendo)
npx ng-openapi-gen --config ng-openapi-gen.json
```

Para esta guía, **escribiremos los servicios manualmente** — así entiendes qué hace cada método. Cuando domines esto, puedes usar `ng-openapi-gen` para acelerar el proceso.

---

## 6. Servicios Angular: SolicitudService, AuthService y UsuarioService

### 6.1 AuthService — Gestión de autenticación y JWT

El `AuthService` es el más importante de todos. Maneja el login, el registro, el almacenamiento del token y el estado de autenticación de toda la aplicación.

```
src/app/core/services/auth.service.ts
```

```typescript
// src/app/core/services/auth.service.ts

import { Injectable, inject, signal, computed } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { Observable, tap, catchError, throwError } from 'rxjs';
import { environment } from '../../../environments/environment';
import {
  LoginRequest,
  LoginResponse,
  RegistroRequest,
  UsuarioAutenticado,
} from '../models/usuario.model';

@Injectable({
  providedIn: 'root',
})
export class AuthService {
  // inject() es la forma moderna de Angular 17+ para inyectar dependencias
  // Es equivalente al constructor injection pero más flexible
  private readonly http = inject(HttpClient);
  private readonly router = inject(Router);

  // Usamos signals para el estado de autenticación
  // Un signal es un valor reactivo que notifica automáticamente a los consumidores cuando cambia
  // Es la alternativa moderna a BehaviorSubject para estado local
  private readonly _usuarioActual = signal<UsuarioAutenticado | null>(
    this.cargarUsuarioDesdeStorage()
  );

  // computed() crea un signal derivado — se actualiza automáticamente cuando _usuarioActual cambia
  readonly usuarioActual = this._usuarioActual.asReadonly();
  readonly estaAutenticado = computed(() => this._usuarioActual() !== null);
  readonly rolActual = computed(() => this._usuarioActual()?.rol ?? null);

  private readonly apiUrl = `${environment.apiUrl}/auth`;

  // ─── Métodos públicos ────────────────────────────────────────────────────

  /**
   * Envía las credenciales al backend y procesa la respuesta.
   * RF-01 requiere que el usuario esté autenticado para crear solicitudes.
   */
  login(credenciales: LoginRequest): Observable<LoginResponse> {
    return this.http.post<LoginResponse>(`${this.apiUrl}/login`, credenciales).pipe(
      tap((respuesta) => {
        // tap() "escucha" el Observable sin modificar el valor
        // Aquí guardamos el token y actualizamos el estado
        this.guardarSesion(respuesta);
      }),
      catchError((error) => {
        // Propagamos el error para que el componente lo maneje
        return throwError(() => error);
      })
    );
  }

  /**
   * Registra un nuevo usuario en el sistema.
   */
  registro(datos: RegistroRequest): Observable<UsuarioAutenticado> {
    return this.http.post<UsuarioAutenticado>(`${this.apiUrl}/registro`, datos);
  }

  /**
   * Cierra la sesión del usuario: elimina el token y redirige al login.
   */
  logout(): void {
    localStorage.removeItem(environment.tokenKey);
    localStorage.removeItem(environment.refreshTokenKey);
    localStorage.removeItem(environment.tokenExpiryKey);
    localStorage.removeItem('usuario_actual');
    this._usuarioActual.set(null);
    this.router.navigate(['/auth/login']);
  }

  /**
   * Devuelve el token JWT almacenado o null si no existe.
   * Lo usa el interceptor para adjuntarlo a cada petición.
   */
  obtenerToken(): string | null {
    return localStorage.getItem(environment.tokenKey);
  }

  /**
   * Verifica si el token actual no ha expirado.
   */
  tokenEsValido(): boolean {
    const token = this.obtenerToken();
    if (!token) return false;

    const expiry = localStorage.getItem(environment.tokenExpiryKey);
    if (!expiry) return false;

    // Compara la fecha de expiración con la hora actual
    return new Date().getTime() < parseInt(expiry, 10);
  }

  /**
   * Verifica si el usuario tiene un rol específico.
   * Usado por el RoleGuard.
   */
  tieneRol(...roles: string[]): boolean {
    const rol = this.rolActual();
    return rol !== null && roles.includes(rol);
  }

  // ─── Métodos privados ────────────────────────────────────────────────────

  private guardarSesion(respuesta: LoginResponse): void {
    // Calcular timestamp de expiración
    const expiry = new Date().getTime() + respuesta.expiresIn * 1000;

    localStorage.setItem(environment.tokenKey, respuesta.accessToken);
    localStorage.setItem(environment.refreshTokenKey, respuesta.refreshToken);
    localStorage.setItem(environment.tokenExpiryKey, expiry.toString());
    localStorage.setItem('usuario_actual', JSON.stringify(respuesta.usuario));

    // Actualizar el signal con el nuevo usuario
    this._usuarioActual.set(respuesta.usuario);
  }

  private cargarUsuarioDesdeStorage(): UsuarioAutenticado | null {
    // Intentar recuperar el usuario al iniciar la aplicación
    // Esto mantiene la sesión activa si el usuario recarga la página
    try {
      const usuarioJson = localStorage.getItem('usuario_actual');
      if (!usuarioJson) return null;

      const usuario: UsuarioAutenticado = JSON.parse(usuarioJson);

      // Verificar que el token todavía sea válido
      const expiry = localStorage.getItem(environment.tokenExpiryKey);
      if (!expiry || new Date().getTime() >= parseInt(expiry, 10)) {
        // Token expirado — limpiar storage
        localStorage.clear();
        return null;
      }

      return usuario;
    } catch {
      return null;
    }
  }
}
```

### 6.2 SolicitudService — Consumir todos los endpoints del Triage

```typescript
// src/app/core/services/solicitud.service.ts

import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';
import {
  SolicitudResponse,
  CrearSolicitudRequest,
  ClasificarSolicitudRequest,
  AsignarResponsableRequest,
  CambiarEstadoRequest,
  CerrarSolicitudRequest,
  FiltrosSolicitud,
} from '../models/solicitud.model';
import { HistorialSolicitudResponse } from '../models/historial.model';
import { Page, PageRequest } from '../models/page.model';

@Injectable({
  providedIn: 'root',
})
export class SolicitudService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = `${environment.apiUrl}/solicitudes`;

  // ─── RF-01: Registro de solicitudes ─────────────────────────────────────

  /**
   * Crea una nueva solicitud académica.
   * El backend extrae el ID del solicitante del JWT automáticamente.
   */
  crear(request: CrearSolicitudRequest): Observable<SolicitudResponse> {
    return this.http.post<SolicitudResponse>(this.baseUrl, request);
  }

  // ─── RF-02 y RF-03: Clasificación y priorización ─────────────────────────

  /**
   * Clasifica una solicitud: asigna tipo y prioridad.
   * Solo FUNCIONARIO o ADMIN pueden hacer esto.
   */
  clasificar(
    id: number,
    request: ClasificarSolicitudRequest
  ): Observable<SolicitudResponse> {
    return this.http.patch<SolicitudResponse>(
      `${this.baseUrl}/${id}/clasificar`,
      request
    );
  }

  // ─── RF-04: Ciclo de vida ────────────────────────────────────────────────

  /**
   * Cambia el estado de una solicitud validando que la transición sea válida.
   */
  cambiarEstado(
    id: number,
    request: CambiarEstadoRequest
  ): Observable<SolicitudResponse> {
    return this.http.patch<SolicitudResponse>(
      `${this.baseUrl}/${id}/estado`,
      request
    );
  }

  // ─── RF-05: Asignación de responsables ───────────────────────────────────

  /**
   * Asigna un funcionario responsable a la solicitud.
   */
  asignarResponsable(
    id: number,
    request: AsignarResponsableRequest
  ): Observable<SolicitudResponse> {
    return this.http.patch<SolicitudResponse>(
      `${this.baseUrl}/${id}/responsable`,
      request
    );
  }

  // ─── RF-06: Historial ────────────────────────────────────────────────────

  /**
   * Obtiene el historial completo de acciones de una solicitud.
   */
  obtenerHistorial(id: number): Observable<HistorialSolicitudResponse[]> {
    return this.http.get<HistorialSolicitudResponse[]>(
      `${this.baseUrl}/${id}/historial`
    );
  }

  // ─── RF-07: Consulta de solicitudes ──────────────────────────────────────

  /**
   * Consulta solicitudes con filtros y paginación.
   * Construimos los HttpParams dinámicamente para no enviar parámetros vacíos.
   */
  consultar(
    filtros: FiltrosSolicitud,
    paginacion: PageRequest
  ): Observable<Page<SolicitudResponse>> {
    // HttpParams es la forma correcta de pasar query params en Angular
    // Es inmutable: cada llamada a .set() devuelve un nuevo objeto
    let params = new HttpParams()
      .set('page', paginacion.page.toString())
      .set('size', paginacion.size.toString());

    if (paginacion.sort) {
      params = params.set('sort', paginacion.sort);
    }

    // Solo agregar filtros que tienen valor (no undefined, no null, no string vacío)
    if (filtros.estado) params = params.set('estado', filtros.estado);
    if (filtros.tipoSolicitud) params = params.set('tipo', filtros.tipoSolicitud);
    if (filtros.prioridad) params = params.set('prioridad', filtros.prioridad);
    if (filtros.responsableId) params = params.set('responsableId', filtros.responsableId.toString());
    if (filtros.solicitanteId) params = params.set('solicitanteId', filtros.solicitanteId.toString());
    if (filtros.fechaDesde) params = params.set('fechaDesde', filtros.fechaDesde);
    if (filtros.fechaHasta) params = params.set('fechaHasta', filtros.fechaHasta);

    return this.http.get<Page<SolicitudResponse>>(this.baseUrl, { params });
  }

  /**
   * Obtiene una solicitud por su ID.
   */
  obtenerPorId(id: number): Observable<SolicitudResponse> {
    return this.http.get<SolicitudResponse>(`${this.baseUrl}/${id}`);
  }

  // ─── RF-08: Cierre de solicitudes ────────────────────────────────────────

  /**
   * Cierra definitivamente una solicitud. Después no puede modificarse.
   */
  cerrar(
    id: number,
    request: CerrarSolicitudRequest
  ): Observable<SolicitudResponse> {
    return this.http.patch<SolicitudResponse>(
      `${this.baseUrl}/${id}/cerrar`,
      request
    );
  }

  /**
   * Obtiene mis solicitudes (las del usuario autenticado).
   * El backend filtra automáticamente por el JWT.
   */
  misSolicitudes(paginacion: PageRequest): Observable<Page<SolicitudResponse>> {
    const params = new HttpParams()
      .set('page', paginacion.page.toString())
      .set('size', paginacion.size.toString());

    return this.http.get<Page<SolicitudResponse>>(
      `${this.baseUrl}/mis-solicitudes`,
      { params }
    );
  }
}
```

### 6.3 UsuarioService — Gestión de usuarios

```typescript
// src/app/core/services/usuario.service.ts

import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';
import { UsuarioResponse } from '../models/usuario.model';
import { Page, PageRequest } from '../models/page.model';

@Injectable({
  providedIn: 'root',
})
export class UsuarioService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = `${environment.apiUrl}/usuarios`;

  /**
   * Obtiene todos los funcionarios activos.
   * Usado en el formulario de asignación de responsable.
   */
  obtenerFuncionariosActivos(): Observable<UsuarioResponse[]> {
    return this.http.get<UsuarioResponse[]>(`${this.baseUrl}/funcionarios`);
  }

  /**
   * Obtiene un usuario por ID.
   */
  obtenerPorId(id: number): Observable<UsuarioResponse> {
    return this.http.get<UsuarioResponse>(`${this.baseUrl}/${id}`);
  }

  /**
   * Lista todos los usuarios (solo ADMIN).
   */
  listar(paginacion: PageRequest): Observable<Page<UsuarioResponse>> {
    return this.http.get<Page<UsuarioResponse>>(this.baseUrl);
  }
}
```

---

## 7. Interceptor JWT: Adjuntar Token Automáticamente

### 7.1 ¿Qué es un interceptor HTTP?

Un interceptor en Angular es como un punto de control en un aeropuerto. Cada petición HTTP que tu aplicación hace pasa por ese punto de control antes de salir, y puede ser modificada: agregar cabeceras, transformar el cuerpo, registrar en consola, etc. Las respuestas también pasan por el interceptor al regresar.

En nuestro caso, el interceptor JWT toma cada petición saliente, busca el token en `localStorage` y lo adjunta como cabecera `Authorization: Bearer <token>`. Así no tienes que acordarte de enviarlo en cada llamada manualmente.

```
src/app/core/auth/auth.interceptor.ts
```

```typescript
// src/app/core/auth/auth.interceptor.ts

import { HttpInterceptorFn, HttpRequest, HttpHandlerFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';
import { environment } from '../../../environments/environment';

// En Angular 17+, los interceptores son funciones puras (no clases)
// Esto se llama "interceptor funcional" y es la forma moderna
export const authInterceptor: HttpInterceptorFn = (
  req: HttpRequest<unknown>,
  next: HttpHandlerFn
) => {
  const authService = inject(AuthService);

  // No adjuntar token a los endpoints de autenticación
  // (login y registro no necesitan token porque son los que generan el token)
  const esEndpointPublico =
    req.url.includes('/auth/login') ||
    req.url.includes('/auth/registro') ||
    req.url.includes('/auth/refresh');

  if (esEndpointPublico) {
    // Dejar pasar la petición sin modificar
    return next(req);
  }

  const token = authService.obtenerToken();

  if (!token) {
    // No hay token — dejar pasar (el backend responderá 401 si el endpoint requiere auth)
    return next(req);
  }

  // HttpRequest es inmutable: clone() crea una copia con las modificaciones
  // Esto es una buena práctica para no mutar el objeto original
  const requestConToken = req.clone({
    setHeaders: {
      Authorization: `Bearer ${token}`,
    },
  });

  return next(requestConToken);
};
```

### 7.2 Comparativa: Interceptor de Clase vs Interceptor Funcional

| Aspecto | Clase (deprecada) | Función (moderna, Angular 17+) |
|---|---|---|
| Sintaxis | `class MyInterceptor implements HttpInterceptor` | `const myInterceptor: HttpInterceptorFn = (req, next) => ...` |
| Inyección de dependencias | Constructor injection | `inject()` dentro de la función |
| Registro | `HTTP_INTERCEPTORS` token en módulo | `withInterceptors([])` en `provideHttpClient()` |
| Tree-shaking | No funciona bien | Funciona correctamente |
| Orden de ejecución | Declarativo, frágil | Explícito por orden en el array |

---

## 8. Interceptor de Errores: Manejo Global

### 8.1 ¿Por qué manejar errores globalmente?

Sin un interceptor de errores, tendrías que escribir manejo de error en cada `subscribe()` de cada componente. Eso es repetitivo y fácil de olvidar. Con un interceptor global, defines una única vez qué hacer con cada tipo de error HTTP.

```
src/app/core/interceptors/error.interceptor.ts
```

```typescript
// src/app/core/interceptors/error.interceptor.ts

import {
  HttpInterceptorFn,
  HttpRequest,
  HttpHandlerFn,
  HttpErrorResponse,
} from '@angular/common/http';
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { catchError, throwError } from 'rxjs';
import { AuthService } from '../services/auth.service';
import { NotificationService } from '../services/notification.service';

export const errorInterceptor: HttpInterceptorFn = (
  req: HttpRequest<unknown>,
  next: HttpHandlerFn
) => {
  const router = inject(Router);
  const authService = inject(AuthService);
  const notificationService = inject(NotificationService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      // Analizar el código de estado HTTP y actuar en consecuencia
      switch (error.status) {
        case 0:
          // Error de red: no hay conexión con el servidor
          notificationService.mostrarError(
            'No se puede conectar con el servidor. Verifica tu conexión a internet.'
          );
          break;

        case 400:
          // Bad Request: datos de entrada inválidos
          // El backend debería devolver detalles del error (Problem Details RFC 7807)
          const mensajeValidacion =
            error.error?.detail ||
            error.error?.message ||
            'Los datos enviados no son válidos.';
          notificationService.mostrarError(mensajeValidacion);
          break;

        case 401:
          // No autenticado: el token expiró o no es válido
          // Limpiar sesión y redirigir al login
          notificationService.mostrarAdvertencia(
            'Tu sesión ha expirado. Por favor inicia sesión nuevamente.'
          );
          authService.logout();
          // El logout() ya redirige al login, pero por si acaso:
          router.navigate(['/auth/login']);
          break;

        case 403:
          // Prohibido: el usuario está autenticado pero no tiene permiso
          notificationService.mostrarError(
            'No tienes permiso para realizar esta acción.'
          );
          router.navigate(['/acceso-denegado']);
          break;

        case 404:
          // No encontrado: el recurso solicitado no existe
          notificationService.mostrarError(
            'El recurso solicitado no fue encontrado.'
          );
          break;

        case 409:
          // Conflicto: típicamente una transición de estado inválida en el Triage
          const mensajeConflicto =
            error.error?.detail || 'Operación no permitida en el estado actual.';
          notificationService.mostrarError(mensajeConflicto);
          break;

        case 422:
          // Entidad no procesable: errores de validación de negocio
          notificationService.mostrarError(
            error.error?.detail || 'Los datos no cumplen las reglas de negocio.'
          );
          break;

        case 500:
        case 502:
        case 503:
          // Errores del servidor
          notificationService.mostrarError(
            'Ocurrió un error interno en el servidor. Por favor intenta más tarde.'
          );
          break;

        default:
          notificationService.mostrarError(
            `Error inesperado (${error.status}). Por favor intenta nuevamente.`
          );
      }

      // Propagar el error para que los componentes puedan manejarlo también si lo necesitan
      return throwError(() => error);
    })
  );
};
```

### 8.2 NotificationService — Servicio de notificaciones

Este servicio lo implementaremos completamente en la sección 14. Por ahora, aquí está la definición básica que necesita el interceptor:

```typescript
// src/app/core/services/notification.service.ts

import { Injectable, signal } from '@angular/core';

export interface Toast {
  id: string;
  tipo: 'exito' | 'error' | 'advertencia' | 'info';
  mensaje: string;
  duracion: number;
}

@Injectable({
  providedIn: 'root',
})
export class NotificationService {
  // Lista de toasts activos — los componentes de toast escuchan este signal
  readonly toasts = signal<Toast[]>([]);

  mostrarExito(mensaje: string, duracion = 4000): void {
    this.agregar('exito', mensaje, duracion);
  }

  mostrarError(mensaje: string, duracion = 6000): void {
    this.agregar('error', mensaje, duracion);
  }

  mostrarAdvertencia(mensaje: string, duracion = 5000): void {
    this.agregar('advertencia', mensaje, duracion);
  }

  mostrarInfo(mensaje: string, duracion = 4000): void {
    this.agregar('info', mensaje, duracion);
  }

  eliminar(id: string): void {
    this.toasts.update((toasts) => toasts.filter((t) => t.id !== id));
  }

  private agregar(
    tipo: Toast['tipo'],
    mensaje: string,
    duracion: number
  ): void {
    const id = crypto.randomUUID();
    const toast: Toast = { id, tipo, mensaje, duracion };

    this.toasts.update((toasts) => [...toasts, toast]);

    // Eliminar automáticamente después de la duración
    setTimeout(() => this.eliminar(id), duracion);
  }
}
```

---

## 9. AuthGuard y RoleGuard con la Nueva API Funcional de Angular 17+

### 9.1 ¿Qué es un guard?

Un guard es una función que Angular ejecuta antes de activar una ruta. Si el guard devuelve `true`, la navegación continúa. Si devuelve `false` (o un `UrlTree` que es una redirección), Angular cancela la navegación.

Usamos dos guards en el Triage:
- `AuthGuard`: verifica que el usuario esté autenticado antes de entrar a cualquier ruta protegida.
- `RoleGuard`: verifica que el usuario tenga el rol correcto para acceder a una ruta específica.

### 9.2 Antes vs Ahora: Guards

| Aspecto | Antes (hasta Angular 14) | Ahora (Angular 15+) |
|---|---|---|
| Implementación | Clase que implementa `CanActivate` | Función pura |
| Registro | `providers: [MyGuard]` + `canActivate: [MyGuard]` | `canActivate: [myGuard]` directamente |
| Inyección | Constructor injection | `inject()` |
| Testabilidad | Requiere TestBed más elaborado | Función simple de testear |

```typescript
// src/app/core/auth/auth.guard.ts
// Guard funcional — Angular 15+ recomienda este enfoque

import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.estaAutenticado()) {
    return true;
  }

  // El usuario no está autenticado — redirigir al login
  // Guardamos la URL a la que intentaba acceder para redirigir después del login
  return router.createUrlTree(['/auth/login'], {
    queryParams: { returnUrl: state.url },
  });
};
```

```typescript
// src/app/core/auth/role.guard.ts
// Guard de roles — verifica que el usuario tenga el rol correcto

import { inject } from '@angular/core';
import { CanActivateFn, Router, ActivatedRouteSnapshot } from '@angular/router';
import { AuthService } from '../services/auth.service';

// Este guard espera que la ruta tenga data: { roles: ['ADMIN', 'FUNCIONARIO'] }
export const roleGuard: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state
) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  // Primero verificar que esté autenticado
  if (!authService.estaAutenticado()) {
    return router.createUrlTree(['/auth/login'], {
      queryParams: { returnUrl: state.url },
    });
  }

  // Obtener los roles permitidos de los datos de la ruta
  const rolesPermitidos: string[] = route.data?.['roles'] ?? [];

  if (rolesPermitidos.length === 0) {
    // Si no se especificaron roles, cualquier usuario autenticado puede acceder
    return true;
  }

  if (authService.tieneRol(...rolesPermitidos)) {
    return true;
  }

  // El usuario está autenticado pero no tiene el rol necesario
  return router.createUrlTree(['/acceso-denegado']);
};
```

### 9.3 Configurar las rutas con los guards

```typescript
// src/app/app.routes.ts

import { Routes } from '@angular/router';
import { authGuard } from './core/auth/auth.guard';
import { roleGuard } from './core/auth/role.guard';

export const routes: Routes = [
  // Rutas públicas — no requieren autenticación
  {
    path: 'auth',
    children: [
      {
        path: 'login',
        loadComponent: () =>
          import('./features/auth/login/login.component').then(
            (m) => m.LoginComponent
          ),
      },
      {
        path: 'registro',
        loadComponent: () =>
          import('./features/auth/registro/registro.component').then(
            (m) => m.RegistroComponent
          ),
      },
    ],
  },

  // Rutas protegidas — requieren estar autenticado
  {
    path: 'dashboard',
    canActivate: [authGuard],
    loadComponent: () =>
      import('./features/dashboard/dashboard.component').then(
        (m) => m.DashboardComponent
      ),
  },

  // Rutas de solicitudes — requieren autenticación
  {
    path: 'solicitudes',
    canActivate: [authGuard],
    children: [
      {
        path: '',
        loadComponent: () =>
          import('./features/solicitudes/lista-solicitudes/lista-solicitudes.component').then(
            (m) => m.ListaSolicitudesComponent
          ),
      },
      {
        path: 'nueva',
        // Solo ESTUDIANTE puede crear solicitudes
        canActivate: [roleGuard],
        data: { roles: ['ESTUDIANTE'] },
        loadComponent: () =>
          import('./features/solicitudes/crear-solicitud/crear-solicitud.component').then(
            (m) => m.CrearSolicitudComponent
          ),
      },
      {
        path: ':id',
        loadComponent: () =>
          import('./features/solicitudes/detalle-solicitud/detalle-solicitud.component').then(
            (m) => m.DetalleSolicitudComponent
          ),
      },
      {
        path: ':id/clasificar',
        // Solo FUNCIONARIO y ADMIN pueden clasificar
        canActivate: [roleGuard],
        data: { roles: ['FUNCIONARIO', 'ADMIN'] },
        loadComponent: () =>
          import('./features/solicitudes/clasificar-solicitud/clasificar-solicitud.component').then(
            (m) => m.ClasificarSolicitudComponent
          ),
      },
    ],
  },

  // Ruta de acceso denegado
  {
    path: 'acceso-denegado',
    loadComponent: () =>
      import('./features/acceso-denegado/acceso-denegado.component').then(
        (m) => m.AccesoDenegadoComponent
      ),
  },

  // Redireccionamiento por defecto
  { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
  { path: '**', redirectTo: 'dashboard' },
];
```

---

## 10. Formularios Reactivos Completos

### 10.1 Formulario de Login

```
src/app/features/auth/login/login.component.ts
```

```typescript
// src/app/features/auth/login/login.component.ts

import { Component, inject, signal } from '@angular/core';
import { CommonModule } from '@angular/common';
import {
  ReactiveFormsModule,
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';
import { Router, ActivatedRoute } from '@angular/router';
import { AuthService } from '../../../core/services/auth.service';

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  templateUrl: './login.component.html',
})
export class LoginComponent {
  private readonly fb = inject(FormBuilder);
  private readonly authService = inject(AuthService);
  private readonly router = inject(Router);
  private readonly route = inject(ActivatedRoute);

  // Signals para el estado del componente
  readonly cargando = signal(false);
  readonly mostrarPassword = signal(false);
  readonly errorLogin = signal<string | null>(null);

  // URL a la que redirigir después del login (si venía de una ruta protegida)
  private readonly returnUrl = this.route.snapshot.queryParams['returnUrl'] ?? '/dashboard';

  readonly formulario: FormGroup = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(6)]],
  });

  // Getters para acceder fácilmente a los controles en el template
  get email() { return this.formulario.get('email')!; }
  get password() { return this.formulario.get('password')!; }

  onSubmit(): void {
    if (this.formulario.invalid) {
      // Marcar todos los campos como "touched" para mostrar errores de validación
      this.formulario.markAllAsTouched();
      return;
    }

    this.cargando.set(true);
    this.errorLogin.set(null);

    this.authService.login(this.formulario.value).subscribe({
      next: () => {
        // El interceptor JWT se encargará de los futuros requests
        this.router.navigateByUrl(this.returnUrl);
      },
      error: (err) => {
        this.cargando.set(false);
        // 401 = credenciales incorrectas
        if (err.status === 401) {
          this.errorLogin.set('Email o contraseña incorrectos.');
        } else {
          this.errorLogin.set('Ocurrió un error. Intenta nuevamente.');
        }
      },
      complete: () => {
        this.cargando.set(false);
      },
    });
  }
}
```

```html
<!-- src/app/features/auth/login/login.component.html -->

<div class="login-container">
  <div class="login-card">
    <h1>Sistema de Triage Académico</h1>
    <h2>Iniciar Sesión</h2>

    @if (errorLogin()) {
      <div class="alert alert-error" role="alert">
        {{ errorLogin() }}
      </div>
    }

    <form [formGroup]="formulario" (ngSubmit)="onSubmit()" novalidate>

      <!-- Campo Email -->
      <div class="campo-formulario">
        <label for="email">Correo electrónico</label>
        <input
          id="email"
          type="email"
          formControlName="email"
          placeholder="correo@uniquindio.edu.co"
          [class.campo-invalido]="email.invalid && email.touched"
          autocomplete="email"
        />
        @if (email.invalid && email.touched) {
          <span class="error-mensaje">
            @if (email.errors?.['required']) {
              El correo es obligatorio.
            } @else if (email.errors?.['email']) {
              Ingresa un correo válido.
            }
          </span>
        }
      </div>

      <!-- Campo Contraseña -->
      <div class="campo-formulario">
        <label for="password">Contraseña</label>
        <div class="input-con-icono">
          <input
            id="password"
            [type]="mostrarPassword() ? 'text' : 'password'"
            formControlName="password"
            placeholder="Tu contraseña"
            [class.campo-invalido]="password.invalid && password.touched"
            autocomplete="current-password"
          />
          <button
            type="button"
            class="btn-toggle-password"
            (click)="mostrarPassword.set(!mostrarPassword())"
            [attr.aria-label]="mostrarPassword() ? 'Ocultar contraseña' : 'Mostrar contraseña'"
          >
            {{ mostrarPassword() ? '👁️' : '👁️‍🗨️' }}
          </button>
        </div>
        @if (password.invalid && password.touched) {
          <span class="error-mensaje">
            @if (password.errors?.['required']) {
              La contraseña es obligatoria.
            } @else if (password.errors?.['minlength']) {
              La contraseña debe tener al menos 6 caracteres.
            }
          </span>
        }
      </div>

      <!-- Botón de submit -->
      <button
        type="submit"
        class="btn btn-primario btn-completo"
        [disabled]="cargando()"
      >
        @if (cargando()) {
          <span class="spinner-pequeno"></span>
          Iniciando sesión...
        } @else {
          Iniciar Sesión
        }
      </button>

    </form>

    <p class="enlace-registro">
      ¿No tienes cuenta?
      <a routerLink="/auth/registro">Regístrate aquí</a>
    </p>
  </div>
</div>
```

### 10.2 Formulario de Creación de Solicitud (RF-01)

```typescript
// src/app/features/solicitudes/crear-solicitud/crear-solicitud.component.ts

import { Component, inject, signal } from '@angular/core';
import { CommonModule } from '@angular/common';
import {
  ReactiveFormsModule,
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';
import { Router } from '@angular/router';
import { SolicitudService } from '../../../core/services/solicitud.service';
import { NotificationService } from '../../../core/services/notification.service';
import { CanalOrigen } from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-crear-solicitud',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  templateUrl: './crear-solicitud.component.html',
})
export class CrearSolicitudComponent {
  private readonly fb = inject(FormBuilder);
  private readonly solicitudService = inject(SolicitudService);
  private readonly router = inject(Router);
  private readonly notificationService = inject(NotificationService);

  readonly cargando = signal(false);

  // Opciones para el select de canal de origen
  readonly canalesOrigen: { valor: CanalOrigen; etiqueta: string }[] = [
    { valor: 'CSU', etiqueta: 'Centro de Servicios Universitarios (CSU)' },
    { valor: 'CORREO', etiqueta: 'Correo electrónico' },
    { valor: 'SAC', etiqueta: 'Sistema de Atención al Cliente (SAC)' },
    { valor: 'TELEFONICO', etiqueta: 'Atención telefónica' },
    { valor: 'PRESENCIAL', etiqueta: 'Atención presencial' },
  ];

  readonly formulario: FormGroup = this.fb.group({
    descripcion: [
      '',
      [
        Validators.required,
        Validators.minLength(20),
        Validators.maxLength(1000),
      ],
    ],
    canalOrigen: ['', Validators.required],
  });

  get descripcion() { return this.formulario.get('descripcion')!; }
  get canalOrigen() { return this.formulario.get('canalOrigen')!; }

  get caracteresDescripcion(): number {
    return this.descripcion.value?.length ?? 0;
  }

  onSubmit(): void {
    if (this.formulario.invalid) {
      this.formulario.markAllAsTouched();
      return;
    }

    this.cargando.set(true);

    this.solicitudService.crear(this.formulario.value).subscribe({
      next: (solicitudCreada) => {
        this.notificationService.mostrarExito(
          `Solicitud #${solicitudCreada.id} creada exitosamente.`
        );
        // Navegar al detalle de la solicitud recién creada
        this.router.navigate(['/solicitudes', solicitudCreada.id]);
      },
      error: () => {
        // El errorInterceptor ya mostró el toast de error
        this.cargando.set(false);
      },
      complete: () => {
        this.cargando.set(false);
      },
    });
  }
}
```

```html
<!-- src/app/features/solicitudes/crear-solicitud/crear-solicitud.component.html -->

<div class="pagina-container">
  <div class="pagina-encabezado">
    <h1>Nueva Solicitud Académica</h1>
    <p>Describe tu situación con el mayor detalle posible para una atención más eficiente.</p>
  </div>

  <div class="card formulario-card">
    <form [formGroup]="formulario" (ngSubmit)="onSubmit()" novalidate>

      <!-- Canal de origen -->
      <div class="campo-formulario">
        <label for="canalOrigen">¿Por dónde realizas esta solicitud?</label>
        <select
          id="canalOrigen"
          formControlName="canalOrigen"
          [class.campo-invalido]="canalOrigen.invalid && canalOrigen.touched"
        >
          <option value="">-- Selecciona una opción --</option>
          @for (canal of canalesOrigen; track canal.valor) {
            <option [value]="canal.valor">{{ canal.etiqueta }}</option>
          }
        </select>
        @if (canalOrigen.invalid && canalOrigen.touched) {
          <span class="error-mensaje">Selecciona el canal de origen.</span>
        }
      </div>

      <!-- Descripción de la solicitud -->
      <div class="campo-formulario">
        <label for="descripcion">
          Descripción de la solicitud
          <span class="contador-caracteres">
            {{ caracteresDescripcion }}/1000
          </span>
        </label>
        <textarea
          id="descripcion"
          formControlName="descripcion"
          rows="6"
          placeholder="Explica detalladamente tu solicitud. Por ejemplo: 'Solicito homologación de la materia Cálculo I cursada en la Universidad XXXX con nota 4.5...'"
          [class.campo-invalido]="descripcion.invalid && descripcion.touched"
        ></textarea>
        @if (descripcion.invalid && descripcion.touched) {
          <span class="error-mensaje">
            @if (descripcion.errors?.['required']) {
              La descripción es obligatoria.
            } @else if (descripcion.errors?.['minlength']) {
              La descripción debe tener al menos 20 caracteres.
            } @else if (descripcion.errors?.['maxlength']) {
              La descripción no puede superar los 1000 caracteres.
            }
          </span>
        }
      </div>

      <!-- Botones -->
      <div class="acciones-formulario">
        <button
          type="button"
          class="btn btn-secundario"
          (click)="router.navigate(['/solicitudes'])"
          [disabled]="cargando()"
        >
          Cancelar
        </button>
        <button
          type="submit"
          class="btn btn-primario"
          [disabled]="cargando()"
        >
          @if (cargando()) {
            Enviando solicitud...
          } @else {
            Enviar Solicitud
          }
        </button>
      </div>

    </form>
  </div>
</div>
```

### 10.3 Formulario de Clasificación de Solicitud (RF-02, RF-03)

```typescript
// src/app/features/solicitudes/clasificar-solicitud/clasificar-solicitud.component.ts

import { Component, inject, signal, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import {
  ReactiveFormsModule,
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';
import { Router, ActivatedRoute } from '@angular/router';
import { SolicitudService } from '../../../core/services/solicitud.service';
import { NotificationService } from '../../../core/services/notification.service';
import {
  SolicitudResponse,
  TipoSolicitud,
  PrioridadSolicitud,
} from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-clasificar-solicitud',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  templateUrl: './clasificar-solicitud.component.html',
})
export class ClasificarSolicitudComponent implements OnInit {
  private readonly fb = inject(FormBuilder);
  private readonly solicitudService = inject(SolicitudService);
  private readonly router = inject(Router);
  private readonly route = inject(ActivatedRoute);
  private readonly notificationService = inject(NotificationService);

  readonly cargando = signal(false);
  readonly cargandoSolicitud = signal(true);
  readonly solicitud = signal<SolicitudResponse | null>(null);
  private solicitudId!: number;

  readonly tiposSolicitud: { valor: TipoSolicitud; etiqueta: string }[] = [
    { valor: 'REGISTRO_ASIGNATURAS', etiqueta: 'Registro de asignaturas' },
    { valor: 'HOMOLOGACION', etiqueta: 'Homologación' },
    { valor: 'CANCELACION_ASIGNATURAS', etiqueta: 'Cancelación de asignaturas' },
    { valor: 'SOLICITUD_CUPOS', etiqueta: 'Solicitud de cupos' },
    { valor: 'CONSULTA_ACADEMICA', etiqueta: 'Consulta académica' },
    { valor: 'OTRO', etiqueta: 'Otro' },
  ];

  readonly prioridades: { valor: PrioridadSolicitud; etiqueta: string; descripcion: string }[] = [
    { valor: 'BAJA', etiqueta: 'Baja', descripcion: 'Puede atenderse en los próximos 10 días hábiles' },
    { valor: 'MEDIA', etiqueta: 'Media', descripcion: 'Requiere atención en los próximos 5 días hábiles' },
    { valor: 'ALTA', etiqueta: 'Alta', descripcion: 'Requiere atención en los próximos 2 días hábiles' },
    { valor: 'CRITICA', etiqueta: 'Crítica', descripcion: 'Requiere atención inmediata — impacto académico grave' },
  ];

  readonly formulario: FormGroup = this.fb.group({
    tipoSolicitud: ['', Validators.required],
    prioridad: ['', Validators.required],
    justificacionPrioridad: ['', [Validators.required, Validators.minLength(10)]],
  });

  get tipoSolicitud() { return this.formulario.get('tipoSolicitud')!; }
  get prioridad() { return this.formulario.get('prioridad')!; }
  get justificacionPrioridad() { return this.formulario.get('justificacionPrioridad')!; }

  ngOnInit(): void {
    this.solicitudId = Number(this.route.snapshot.paramMap.get('id'));
    this.cargarSolicitud();
  }

  private cargarSolicitud(): void {
    this.solicitudService.obtenerPorId(this.solicitudId).subscribe({
      next: (solicitud) => {
        this.solicitud.set(solicitud);
        this.cargandoSolicitud.set(false);

        // Si ya tenía clasificación previa, pre-poblar el formulario
        if (solicitud.tipoSolicitud) {
          this.formulario.patchValue({
            tipoSolicitud: solicitud.tipoSolicitud,
            prioridad: solicitud.prioridad,
            justificacionPrioridad: solicitud.justificacionPrioridad,
          });
        }
      },
      error: () => {
        this.cargandoSolicitud.set(false);
        this.router.navigate(['/solicitudes']);
      },
    });
  }

  onSubmit(): void {
    if (this.formulario.invalid) {
      this.formulario.markAllAsTouched();
      return;
    }

    this.cargando.set(true);

    this.solicitudService
      .clasificar(this.solicitudId, this.formulario.value)
      .subscribe({
        next: () => {
          this.notificationService.mostrarExito(
            'Solicitud clasificada y priorizada exitosamente.'
          );
          this.router.navigate(['/solicitudes', this.solicitudId]);
        },
        error: () => {
          this.cargando.set(false);
        },
      });
  }
}
```

---

## 11. Tabla de Solicitudes con Paginación y Filtros del Lado del Servidor

### 11.1 ¿Por qué paginación del lado del servidor?

Si tienes 5.000 solicitudes en la base de datos y las traes todas de una vez, el backend envía un JSON enorme, el navegador lo parsea todo, y la tabla tarda en renderizar. La paginación del lado del servidor le dice al backend: "dame solo las primeras 10 solicitudes ordenadas por fecha". El backend aplica `LIMIT` y `OFFSET` en SQL y devuelve solo esas 10 filas.

```typescript
// src/app/features/solicitudes/lista-solicitudes/lista-solicitudes.component.ts

import { Component, inject, signal, OnInit, OnDestroy } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule, FormBuilder, FormGroup } from '@angular/forms';
import { RouterLink } from '@angular/router';
import { Subject, takeUntil, debounceTime, distinctUntilChanged } from 'rxjs';
import { SolicitudService } from '../../../core/services/solicitud.service';
import { AuthService } from '../../../core/services/auth.service';
import {
  SolicitudResponse,
  EstadoSolicitud,
  TipoSolicitud,
  PrioridadSolicitud,
  FiltrosSolicitud,
} from '../../../core/models/solicitud.model';
import { Page } from '../../../core/models/page.model';

@Component({
  selector: 'app-lista-solicitudes',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule, RouterLink],
  templateUrl: './lista-solicitudes.component.html',
})
export class ListaSolicitudesComponent implements OnInit, OnDestroy {
  private readonly solicitudService = inject(SolicitudService);
  readonly authService = inject(AuthService);
  private readonly fb = inject(FormBuilder);

  // Subject para cancelar suscripciones cuando el componente se destruye
  // Esto evita memory leaks
  private readonly destroy$ = new Subject<void>();

  // Estado del componente
  readonly cargando = signal(false);
  readonly paginaActual = signal<Page<SolicitudResponse> | null>(null);

  // Configuración de paginación
  readonly tamanioPagina = signal(10);
  readonly numeroPagina = signal(0);
  readonly ordenamiento = signal('fechaRegistro,desc');

  // Formulario de filtros
  readonly formularioFiltros: FormGroup = this.fb.group({
    estado: [''],
    tipoSolicitud: [''],
    prioridad: [''],
  });

  // Opciones para los filtros
  readonly estadosSolicitud: EstadoSolicitud[] = [
    'REGISTRADA', 'CLASIFICADA', 'EN_ATENCION', 'ATENDIDA', 'CERRADA',
  ];

  readonly tiposSolicitud: TipoSolicitud[] = [
    'REGISTRO_ASIGNATURAS', 'HOMOLOGACION', 'CANCELACION_ASIGNATURAS',
    'SOLICITUD_CUPOS', 'CONSULTA_ACADEMICA', 'OTRO',
  ];

  readonly prioridades: PrioridadSolicitud[] = ['BAJA', 'MEDIA', 'ALTA', 'CRITICA'];

  ngOnInit(): void {
    this.cargarSolicitudes();
    this.configurarEscuchaFiltros();
  }

  ngOnDestroy(): void {
    // Cancelar todas las suscripciones para evitar memory leaks
    this.destroy$.next();
    this.destroy$.complete();
  }

  /**
   * Escucha cambios en el formulario de filtros y recarga la tabla.
   * debounceTime(300): espera 300ms antes de disparar para no hacer una petición por cada tecla.
   * distinctUntilChanged(): no dispara si el valor es igual al anterior.
   */
  private configurarEscuchaFiltros(): void {
    this.formularioFiltros.valueChanges
      .pipe(
        debounceTime(300),
        distinctUntilChanged(
          (anterior, actual) => JSON.stringify(anterior) === JSON.stringify(actual)
        ),
        takeUntil(this.destroy$)
      )
      .subscribe(() => {
        // Al cambiar filtros, volver a la primera página
        this.numeroPagina.set(0);
        this.cargarSolicitudes();
      });
  }

  cargarSolicitudes(): void {
    this.cargando.set(true);

    const filtros: FiltrosSolicitud = {};
    const valores = this.formularioFiltros.value;

    if (valores.estado) filtros.estado = valores.estado;
    if (valores.tipoSolicitud) filtros.tipoSolicitud = valores.tipoSolicitud;
    if (valores.prioridad) filtros.prioridad = valores.prioridad;

    this.solicitudService
      .consultar(filtros, {
        page: this.numeroPagina(),
        size: this.tamanioPagina(),
        sort: this.ordenamiento(),
      })
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: (pagina) => {
          this.paginaActual.set(pagina);
          this.cargando.set(false);
        },
        error: () => {
          this.cargando.set(false);
        },
      });
  }

  irAPagina(numero: number): void {
    this.numeroPagina.set(numero);
    this.cargarSolicitudes();
  }

  paginaAnterior(): void {
    if (this.numeroPagina() > 0) {
      this.numeroPagina.update((n) => n - 1);
      this.cargarSolicitudes();
    }
  }

  paginaSiguiente(): void {
    const pagina = this.paginaActual();
    if (pagina && !pagina.last) {
      this.numeroPagina.update((n) => n + 1);
      this.cargarSolicitudes();
    }
  }

  cambiarOrdenamiento(campo: string): void {
    const ordenActual = this.ordenamiento();
    const [campoActual, direccion] = ordenActual.split(',');

    if (campoActual === campo) {
      // Invertir dirección
      this.ordenamiento.set(`${campo},${direccion === 'asc' ? 'desc' : 'asc'}`);
    } else {
      this.ordenamiento.set(`${campo},desc`);
    }
    this.cargarSolicitudes();
  }

  limpiarFiltros(): void {
    this.formularioFiltros.reset();
    this.numeroPagina.set(0);
  }

  // Helpers para el template
  get numeroPaginasArray(): number[] {
    const total = this.paginaActual()?.totalPages ?? 0;
    return Array.from({ length: total }, (_, i) => i);
  }

  obtenerEtiquetaEstado(estado: EstadoSolicitud): string {
    const etiquetas: Record<EstadoSolicitud, string> = {
      REGISTRADA: 'Registrada',
      CLASIFICADA: 'Clasificada',
      EN_ATENCION: 'En Atención',
      ATENDIDA: 'Atendida',
      CERRADA: 'Cerrada',
    };
    return etiquetas[estado];
  }

  obtenerClaseEstado(estado: EstadoSolicitud): string {
    const clases: Record<EstadoSolicitud, string> = {
      REGISTRADA: 'badge-gris',
      CLASIFICADA: 'badge-azul',
      EN_ATENCION: 'badge-naranja',
      ATENDIDA: 'badge-verde',
      CERRADA: 'badge-oscuro',
    };
    return clases[estado];
  }

  obtenerClasePrioridad(prioridad: PrioridadSolicitud | null): string {
    if (!prioridad) return '';
    const clases: Record<PrioridadSolicitud, string> = {
      BAJA: 'prioridad-baja',
      MEDIA: 'prioridad-media',
      ALTA: 'prioridad-alta',
      CRITICA: 'prioridad-critica',
    };
    return clases[prioridad];
  }
}
```

```html
<!-- src/app/features/solicitudes/lista-solicitudes/lista-solicitudes.component.html -->

<div class="pagina-container">
  <div class="pagina-encabezado">
    <h1>Solicitudes Académicas</h1>
    @if (authService.tieneRol('ESTUDIANTE')) {
      <a routerLink="/solicitudes/nueva" class="btn btn-primario">
        + Nueva Solicitud
      </a>
    }
  </div>

  <!-- Filtros -->
  <div class="card filtros-card">
    <form [formGroup]="formularioFiltros" class="filtros-grilla">
      <div class="campo-filtro">
        <label>Estado</label>
        <select formControlName="estado">
          <option value="">Todos</option>
          @for (estado of estadosSolicitud; track estado) {
            <option [value]="estado">{{ obtenerEtiquetaEstado(estado) }}</option>
          }
        </select>
      </div>

      <div class="campo-filtro">
        <label>Tipo de Solicitud</label>
        <select formControlName="tipoSolicitud">
          <option value="">Todos</option>
          @for (tipo of tiposSolicitud; track tipo) {
            <option [value]="tipo">{{ tipo | replace:'_':' ' }}</option>
          }
        </select>
      </div>

      <div class="campo-filtro">
        <label>Prioridad</label>
        <select formControlName="prioridad">
          <option value="">Todas</option>
          @for (prioridad of prioridades; track prioridad) {
            <option [value]="prioridad">{{ prioridad }}</option>
          }
        </select>
      </div>

      <button type="button" class="btn btn-texto" (click)="limpiarFiltros()">
        Limpiar filtros
      </button>
    </form>
  </div>

  <!-- Tabla -->
  <div class="card tabla-card">
    @if (cargando()) {
      <!-- Skeleton loader mientras carga -->
      <div class="skeleton-tabla">
        @for (fila of [1,2,3,4,5]; track fila) {
          <div class="skeleton-fila"></div>
        }
      </div>
    } @else if (paginaActual()?.empty) {
      <div class="estado-vacio">
        <p>No se encontraron solicitudes con los filtros seleccionados.</p>
        <button class="btn btn-texto" (click)="limpiarFiltros()">
          Ver todas las solicitudes
        </button>
      </div>
    } @else {
      <table class="tabla-datos">
        <thead>
          <tr>
            <th (click)="cambiarOrdenamiento('id')" class="columna-ordenable">
              ID ↕
            </th>
            <th>Descripción</th>
            <th (click)="cambiarOrdenamiento('estado')" class="columna-ordenable">
              Estado ↕
            </th>
            <th>Tipo</th>
            <th>Prioridad</th>
            <th (click)="cambiarOrdenamiento('fechaRegistro')" class="columna-ordenable">
              Fecha ↕
            </th>
            <th>Solicitante</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          @for (solicitud of paginaActual()?.content; track solicitud.id) {
            <tr>
              <td>#{{ solicitud.id }}</td>
              <td class="descripcion-truncada">
                {{ solicitud.descripcion | slice:0:60 }}...
              </td>
              <td>
                <span [class]="'badge ' + obtenerClaseEstado(solicitud.estado)">
                  {{ obtenerEtiquetaEstado(solicitud.estado) }}
                </span>
              </td>
              <td>{{ solicitud.tipoSolicitud ?? '—' }}</td>
              <td>
                @if (solicitud.prioridad) {
                  <span [class]="obtenerClasePrioridad(solicitud.prioridad)">
                    {{ solicitud.prioridad }}
                  </span>
                } @else {
                  <span class="texto-gris">—</span>
                }
              </td>
              <td>{{ solicitud.fechaRegistro | date:'dd/MM/yyyy HH:mm' }}</td>
              <td>{{ solicitud.solicitante.nombre }} {{ solicitud.solicitante.apellido }}</td>
              <td class="acciones-celda">
                <a [routerLink]="['/solicitudes', solicitud.id]" class="btn btn-mini">
                  Ver
                </a>
                @if (authService.tieneRol('FUNCIONARIO', 'ADMIN') && solicitud.estado === 'REGISTRADA') {
                  <a [routerLink]="['/solicitudes', solicitud.id, 'clasificar']" class="btn btn-mini btn-secundario">
                    Clasificar
                  </a>
                }
              </td>
            </tr>
          }
        </tbody>
      </table>

      <!-- Paginación -->
      @if ((paginaActual()?.totalPages ?? 0) > 1) {
        <div class="paginacion">
          <span class="info-paginacion">
            Mostrando {{ (paginaActual()?.numberOfElements ?? 0) }} de
            {{ paginaActual()?.totalElements ?? 0 }} solicitudes
          </span>

          <div class="controles-paginacion">
            <button
              class="btn btn-paginacion"
              [disabled]="paginaActual()?.first"
              (click)="paginaAnterior()"
            >
              ← Anterior
            </button>

            @for (numero of numeroPaginasArray; track numero) {
              <button
                class="btn btn-paginacion"
                [class.activa]="numero === numeroPagina()"
                (click)="irAPagina(numero)"
              >
                {{ numero + 1 }}
              </button>
            }

            <button
              class="btn btn-paginacion"
              [disabled]="paginaActual()?.last"
              (click)="paginaSiguiente()"
            >
              Siguiente →
            </button>
          </div>
        </div>
      }
    }
  </div>
</div>
```

---

## 12. Componente de Historial: Timeline Visual

El historial es uno de los requisitos más importantes del sistema (RF-06). Vamos a mostrarlo como una línea de tiempo vertical que permita ver claramente el recorrido de cada solicitud.

```
src/app/shared/components/historial-timeline/
├── historial-timeline.component.ts
└── historial-timeline.component.html
```

```typescript
// src/app/shared/components/historial-timeline/historial-timeline.component.ts

import { Component, input, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HistorialSolicitudResponse } from '../../../core/models/historial.model';

@Component({
  selector: 'app-historial-timeline',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './historial-timeline.component.html',
})
export class HistorialTimelineComponent {
  // input() con signal es la forma moderna de Angular 17+ para @Input()
  // Es reactivo y type-safe
  readonly entradas = input.required<HistorialSolicitudResponse[]>();

  // computed() derivado del input — se recalcula automáticamente
  readonly entradasOrdenadas = computed(() =>
    [...this.entradas()].sort(
      (a, b) => new Date(a.fecha).getTime() - new Date(b.fecha).getTime()
    )
  );

  obtenerIconoAccion(accion: string): string {
    if (accion.toLowerCase().includes('registr')) return '📝';
    if (accion.toLowerCase().includes('clasificad')) return '🏷️';
    if (accion.toLowerCase().includes('asignad')) return '👤';
    if (accion.toLowerCase().includes('atencion') || accion.toLowerCase().includes('atención')) return '⚙️';
    if (accion.toLowerCase().includes('atendid')) return '✅';
    if (accion.toLowerCase().includes('cerrad')) return '🔒';
    return '📌';
  }

  obtenerClaseEstado(estadoNuevo: string | null): string {
    if (!estadoNuevo) return 'evento-neutral';
    const clases: Record<string, string> = {
      REGISTRADA: 'evento-registrada',
      CLASIFICADA: 'evento-clasificada',
      EN_ATENCION: 'evento-atencion',
      ATENDIDA: 'evento-atendida',
      CERRADA: 'evento-cerrada',
    };
    return clases[estadoNuevo] ?? 'evento-neutral';
  }
}
```

```html
<!-- src/app/shared/components/historial-timeline/historial-timeline.component.html -->

<div class="historial-timeline">
  <h3 class="historial-titulo">Historial de la Solicitud</h3>

  @if (entradasOrdenadas().length === 0) {
    <p class="sin-historial">No hay registros en el historial.</p>
  } @else {
    <div class="timeline-contenedor">
      @for (entrada of entradasOrdenadas(); track entrada.id; let esUltimo = $last) {
        <div class="timeline-item" [class]="obtenerClaseEstado(entrada.estadoNuevo)">

          <!-- Línea conectora -->
          @if (!esUltimo) {
            <div class="linea-conectora"></div>
          }

          <!-- Punto en la línea de tiempo -->
          <div class="timeline-punto">
            <span class="icono-accion">{{ obtenerIconoAccion(entrada.accion) }}</span>
          </div>

          <!-- Contenido del evento -->
          <div class="timeline-contenido">
            <div class="timeline-encabezado">
              <strong class="accion-nombre">{{ entrada.accion }}</strong>
              <span class="timeline-fecha">
                {{ entrada.fecha | date:'dd/MM/yyyy HH:mm' }}
              </span>
            </div>

            <!-- Cambio de estado -->
            @if (entrada.estadoAnterior && entrada.estadoNuevo) {
              <div class="cambio-estado">
                <span class="estado-anterior">{{ entrada.estadoAnterior }}</span>
                <span class="flecha-estado">→</span>
                <span class="estado-nuevo">{{ entrada.estadoNuevo }}</span>
              </div>
            }

            <!-- Observaciones -->
            @if (entrada.observaciones) {
              <p class="observaciones">{{ entrada.observaciones }}</p>
            }

            <!-- Usuario responsable de la acción -->
            <div class="usuario-accion">
              <span class="usuario-icono">👤</span>
              {{ entrada.usuario.nombre }} {{ entrada.usuario.apellido }}
              <span class="usuario-rol">({{ entrada.usuario.rol }})</span>
            </div>
          </div>

        </div>
      }
    </div>
  }
</div>
```

### 12.1 Usar el componente de historial en el detalle de solicitud

```typescript
// src/app/features/solicitudes/detalle-solicitud/detalle-solicitud.component.ts

import { Component, inject, signal, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ActivatedRoute, RouterLink } from '@angular/router';
import { forkJoin } from 'rxjs';
import { SolicitudService } from '../../../core/services/solicitud.service';
import { AuthService } from '../../../core/services/auth.service';
import { SolicitudResponse } from '../../../core/models/solicitud.model';
import { HistorialSolicitudResponse } from '../../../core/models/historial.model';
import { HistorialTimelineComponent } from '../../../shared/components/historial-timeline/historial-timeline.component';

@Component({
  selector: 'app-detalle-solicitud',
  standalone: true,
  imports: [CommonModule, RouterLink, HistorialTimelineComponent],
  templateUrl: './detalle-solicitud.component.html',
})
export class DetalleSolicitudComponent implements OnInit {
  private readonly route = inject(ActivatedRoute);
  private readonly solicitudService = inject(SolicitudService);
  readonly authService = inject(AuthService);

  readonly cargando = signal(true);
  readonly solicitud = signal<SolicitudResponse | null>(null);
  readonly historial = signal<HistorialSolicitudResponse[]>([]);

  ngOnInit(): void {
    const id = Number(this.route.snapshot.paramMap.get('id'));

    // forkJoin: ejecuta ambas peticiones en paralelo y espera a que AMBAS terminen
    // Más eficiente que hacerlas secuencialmente
    forkJoin({
      solicitud: this.solicitudService.obtenerPorId(id),
      historial: this.solicitudService.obtenerHistorial(id),
    }).subscribe({
      next: ({ solicitud, historial }) => {
        this.solicitud.set(solicitud);
        this.historial.set(historial);
        this.cargando.set(false);
      },
      error: () => {
        this.cargando.set(false);
      },
    });
  }

  // Determinar qué acciones están disponibles según el estado actual y el rol
  get puedeClasificar(): boolean {
    return (
      this.solicitud()?.estado === 'REGISTRADA' &&
      this.authService.tieneRol('FUNCIONARIO', 'ADMIN')
    );
  }

  get puedeAsignarResponsable(): boolean {
    return (
      this.solicitud()?.estado === 'CLASIFICADA' &&
      this.authService.tieneRol('FUNCIONARIO', 'ADMIN')
    );
  }
}
```

---

## 13. Manejo de Estado con BehaviorSubject

### 13.1 ¿Cuándo usar BehaviorSubject?

Un `BehaviorSubject` es un tipo especial de Observable de RxJS que:
- Guarda el último valor emitido.
- Emite ese último valor inmediatamente a cualquier nuevo suscriptor.
- Permite emitir nuevos valores manualmente con `.next()`.

Es ideal para compartir estado entre componentes que no tienen relación directa padre-hijo (donde usarías `@Input`/`@Output`).

En el Triage, lo usamos para compartir el estado de la lista de solicitudes cuando múltiples componentes necesitan saber cuántas solicitudes hay pendientes (por ejemplo, un badge en el menú de navegación).

> **Nota:** Para estado de componente individual, usa `signal()` como hemos hecho. Para estado compartido entre múltiples componentes, `BehaviorSubject` sigue siendo la opción más común con RxJS, aunque los signals también pueden compartirse si el servicio está en `providedIn: 'root'`.

```typescript
// src/app/core/services/solicitud-estado.service.ts
// Servicio para compartir estado de solicitudes entre componentes

import { Injectable, signal, computed } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { SolicitudResponse, EstadoSolicitud } from '../models/solicitud.model';

@Injectable({
  providedIn: 'root',
})
export class SolicitudEstadoService {
  // BehaviorSubject para la lista de solicitudes
  // null indica "no cargado aún", [] indica "cargado pero vacío"
  private readonly _solicitudes$ = new BehaviorSubject<SolicitudResponse[] | null>(null);

  // Exponemos como Observable público (no permiten emitir desde afuera)
  readonly solicitudes$: Observable<SolicitudResponse[] | null> =
    this._solicitudes$.asObservable();

  // Signal para el conteo de solicitudes pendientes (para badges en menú)
  // Se calcula reactivamente desde el BehaviorSubject
  private readonly _totalPendientes = signal(0);
  readonly totalPendientes = this._totalPendientes.asReadonly();

  // Estado de carga
  private readonly _cargando = signal(false);
  readonly cargando = this._cargando.asReadonly();

  /**
   * Actualiza la lista completa de solicitudes.
   * Lo llama el componente de lista después de cargar del servidor.
   */
  actualizarSolicitudes(solicitudes: SolicitudResponse[]): void {
    this._solicitudes$.next(solicitudes);
    this.calcularPendientes(solicitudes);
  }

  /**
   * Actualiza una sola solicitud en la lista (ej: después de un cambio de estado).
   */
  actualizarSolicitud(solicitudActualizada: SolicitudResponse): void {
    const actual = this._solicitudes$.getValue();
    if (!actual) return;

    const nueva = actual.map((s) =>
      s.id === solicitudActualizada.id ? solicitudActualizada : s
    );
    this._solicitudes$.next(nueva);
    this.calcularPendientes(nueva);
  }

  /**
   * Agrega una nueva solicitud a la lista.
   */
  agregarSolicitud(nueva: SolicitudResponse): void {
    const actual = this._solicitudes$.getValue() ?? [];
    this._solicitudes$.next([nueva, ...actual]);
    this.calcularPendientes([nueva, ...actual]);
  }

  private calcularPendientes(solicitudes: SolicitudResponse[]): void {
    const estados: EstadoSolicitud[] = ['REGISTRADA', 'CLASIFICADA', 'EN_ATENCION'];
    const pendientes = solicitudes.filter((s) => estados.includes(s.estado)).length;
    this._totalPendientes.set(pendientes);
  }
}
```

### 13.2 Usar el estado compartido en el componente de navegación

```typescript
// src/app/shared/components/nav/nav.component.ts

import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink, RouterLinkActive } from '@angular/router';
import { AuthService } from '../../../core/services/auth.service';
import { SolicitudEstadoService } from '../../../core/services/solicitud-estado.service';

@Component({
  selector: 'app-nav',
  standalone: true,
  imports: [CommonModule, RouterLink, RouterLinkActive],
  template: `
    <nav class="barra-navegacion">
      <div class="nav-marca">
        <a routerLink="/dashboard">🎓 Triage Académico</a>
      </div>

      <ul class="nav-enlaces">
        <li>
          <a routerLink="/dashboard" routerLinkActive="activo">
            Panel
          </a>
        </li>
        <li>
          <a routerLink="/solicitudes" routerLinkActive="activo">
            Solicitudes
            @if (solicitudEstado.totalPendientes() > 0) {
              <span class="badge-contador">
                {{ solicitudEstado.totalPendientes() }}
              </span>
            }
          </a>
        </li>
      </ul>

      <div class="nav-usuario">
        <span>{{ authService.usuarioActual()?.nombre }}</span>
        <span class="rol-badge">{{ authService.rolActual() }}</span>
        <button (click)="authService.logout()" class="btn btn-texto">
          Cerrar sesión
        </button>
      </div>
    </nav>
  `,
})
export class NavComponent {
  readonly authService = inject(AuthService);
  readonly solicitudEstado = inject(SolicitudEstadoService);
}
```

---

## 14. Notificaciones Toast y Loading States

### 14.1 Componente Toast

```typescript
// src/app/shared/components/toast/toast.component.ts

import { Component, inject, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { NotificationService, Toast } from '../../../core/services/notification.service';

@Component({
  selector: 'app-toast',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="toast-contenedor" aria-live="polite" aria-atomic="false">
      @for (toast of notificationService.toasts(); track toast.id) {
        <div
          class="toast"
          [class]="'toast-' + toast.tipo"
          role="alert"
        >
          <div class="toast-icono">
            @switch (toast.tipo) {
              @case ('exito') { ✅ }
              @case ('error') { ❌ }
              @case ('advertencia') { ⚠️ }
              @case ('info') { ℹ️ }
            }
          </div>
          <p class="toast-mensaje">{{ toast.mensaje }}</p>
          <button
            class="toast-cerrar"
            (click)="notificationService.eliminar(toast.id)"
            aria-label="Cerrar notificación"
          >
            ✕
          </button>
        </div>
      }
    </div>
  `,
  styles: [`
    .toast-contenedor {
      position: fixed;
      top: 1rem;
      right: 1rem;
      z-index: 9999;
      display: flex;
      flex-direction: column;
      gap: 0.5rem;
      max-width: 400px;
    }

    .toast {
      display: flex;
      align-items: flex-start;
      gap: 0.75rem;
      padding: 1rem;
      border-radius: 8px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.15);
      animation: entrarToast 0.3s ease-out;
    }

    @keyframes entrarToast {
      from { transform: translateX(100%); opacity: 0; }
      to { transform: translateX(0); opacity: 1; }
    }

    .toast-exito { background: #d4edda; border-left: 4px solid #28a745; color: #155724; }
    .toast-error { background: #f8d7da; border-left: 4px solid #dc3545; color: #721c24; }
    .toast-advertencia { background: #fff3cd; border-left: 4px solid #ffc107; color: #856404; }
    .toast-info { background: #d1ecf1; border-left: 4px solid #17a2b8; color: #0c5460; }

    .toast-mensaje { flex: 1; margin: 0; font-size: 0.9rem; line-height: 1.4; }
    .toast-cerrar {
      background: none;
      border: none;
      cursor: pointer;
      font-size: 1rem;
      opacity: 0.6;
      padding: 0;
    }
    .toast-cerrar:hover { opacity: 1; }
  `],
})
export class ToastComponent {
  readonly notificationService = inject(NotificationService);
}
```

### 14.2 Componente de loading spinner

```typescript
// src/app/shared/components/loading-spinner/loading-spinner.component.ts

import { Component, input } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-loading-spinner',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div
      class="spinner-contenedor"
      [class.spinner-completo]="pantallaCopleta()"
      role="status"
      aria-label="Cargando..."
    >
      <div class="spinner" [class]="'spinner-' + tamano()"></div>
      @if (mensaje()) {
        <p class="spinner-mensaje">{{ mensaje() }}</p>
      }
    </div>
  `,
  styles: [`
    .spinner-contenedor {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 1rem;
      padding: 2rem;
    }

    .spinner-completo {
      position: fixed;
      inset: 0;
      background: rgba(255,255,255,0.8);
      z-index: 9998;
    }

    .spinner {
      border-radius: 50%;
      border: 3px solid #e0e0e0;
      border-top-color: #1976d2;
      animation: girar 0.8s linear infinite;
    }

    .spinner-sm { width: 24px; height: 24px; }
    .spinner-md { width: 40px; height: 40px; }
    .spinner-lg { width: 64px; height: 64px; }

    @keyframes girar {
      to { transform: rotate(360deg); }
    }
  `],
})
export class LoadingSpinnerComponent {
  readonly mensaje = input<string>('');
  readonly tamano = input<'sm' | 'md' | 'lg'>('md');
  readonly pantallaCopleta = input(false);
}
```

### 14.3 Registrar el ToastComponent en el AppComponent

```typescript
// src/app/app.component.ts

import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { ToastComponent } from './shared/components/toast/toast.component';
import { NavComponent } from './shared/components/nav/nav.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, ToastComponent, NavComponent],
  template: `
    <!-- Barra de navegación global -->
    <app-nav />

    <!-- Contenido de la ruta activa -->
    <main class="contenido-principal">
      <router-outlet />
    </main>

    <!-- Toast notifications — siempre presentes en el DOM -->
    <app-toast />
  `,
})
export class AppComponent {}
```

---

## 15. Antes vs Ahora: Resumen de Evolución Angular

Esta sección reúne todas las comparaciones importantes de esta guía en un solo lugar.

### 15.1 Tabla comparativa completa

| Concepto | Antes (Angular ≤ 14) | Ahora (Angular 17+) |
|---|---|---|
| **Módulos** | `NgModule` obligatorio en toda app | `standalone: true` por defecto |
| **Inyección** | Constructor: `constructor(private s: Service)` | `inject()`: `private s = inject(Service)` |
| **Inputs** | `@Input() valor: string` | `readonly valor = input<string>()` |
| **Estado local** | `BehaviorSubject` o variable simple | `signal()` — reactivo y eficiente |
| **Estado derivado** | Pipes en template o cálculo manual | `computed()` — se actualiza automáticamente |
| **Control flow template** | `*ngIf`, `*ngFor`, `*ngSwitch` (directivas) | `@if`, `@for`, `@switch` (sintaxis nativa) |
| **Guards** | Clase que implementa `CanActivate` | Función pura `CanActivateFn` |
| **Interceptores** | Clase que implementa `HttpInterceptor` | Función pura `HttpInterceptorFn` |
| **Lazy loading** | `loadChildren: () => import(...).then(m => m.Module)` | `loadComponent: () => import(...).then(m => m.Component)` |
| **Registro HttpClient** | `HttpClientModule` en imports del módulo | `provideHttpClient(withInterceptors([]))` |
| **Registro Router** | `RouterModule.forRoot(routes)` | `provideRouter(routes)` |
| **`@for` tracking** | `trackBy` función separada | `track` inline: `@for (item of items; track item.id)` |

### 15.2 Ejemplo concreto: Guard de clase vs Guard funcional

**Antes (Angular ≤ 14):**
```typescript
// Requería una clase con decorador, constructor y registro en providers
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
    if (this.authService.estaAutenticado()) return true;
    this.router.navigate(['/login']);
    return false;
  }
}
// En las rutas: canActivate: [AuthGuard]
```

**Ahora (Angular 17+):**
```typescript
// Una función pura — más simple, testeable, sin boilerplate
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  return authService.estaAutenticado()
    ? true
    : router.createUrlTree(['/auth/login']);
};
// En las rutas: canActivate: [authGuard]
```

---

## 16. Errores Comunes y Troubleshooting

### Error 1: `NullInjectorError: No provider for HttpClient`

**Síntoma:** La aplicación falla al iniciar con este error en consola.

**Causa:** Olvidaste registrar `provideHttpClient()` en `app.config.ts`.

**Solución:**
```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor, errorInterceptor])), // ← Esto faltaba
  ],
};
```

---

### Error 2: `CORS policy: No 'Access-Control-Allow-Origin' header`

**Síntoma:** La petición falla en el navegador con error CORS. En Postman funciona perfectamente.

**Causa:** El backend Spring Boot no tiene configurado CORS para aceptar peticiones desde `http://localhost:4200`.

**Solución en el backend** (recordando la Guía 06):
```java
@Configuration
public class CorsConfig {
  @Bean
  public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("http://localhost:4200"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
    config.setAllowedHeaders(List.of("*"));
    config.setAllowCredentials(true);
    // ...
  }
}
```

**Verificación rápida:** Abre las DevTools del navegador (F12) → pestaña Network → busca la petición fallida → mira la pestaña Headers → verifica si hay cabeceras `Access-Control-Allow-Origin` en la respuesta.

---

### Error 3: El interceptor JWT no adjunta el token

**Síntoma:** Las peticiones llegan al backend sin el header `Authorization`. El backend responde 401.

**Causas posibles:**

1. El interceptor no está registrado en `provideHttpClient()`.
2. El token se está guardando con una clave diferente a la que lee el interceptor.
3. La condición `esEndpointPublico` es demasiado amplia y coincide con rutas que no deberían ser públicas.

**Diagnóstico:**
```typescript
// Agrega temporalmente este log al interceptor para depurar
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.obtenerToken();

  console.log('Interceptor ejecutado para:', req.url);
  console.log('Token disponible:', token ? 'SÍ' : 'NO');

  // ... resto del interceptor
};
```

---

### Error 4: `NG0303: Can't bind to 'formGroup' since it isn't a known property`

**Síntoma:** Error en el template al usar `[formGroup]`.

**Causa:** Olvidaste importar `ReactiveFormsModule` en el componente standalone.

**Solución:**
```typescript
@Component({
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule], // ← Agregar ReactiveFormsModule
  // ...
})
```

---

### Error 5: La paginación muestra los números incorrectos

**Síntoma:** Spring Boot devuelve `number: 0` para la primera página, pero en el template muestras `number + 1` y el resultado se desincroniza.

**Causa:** Spring Boot usa páginas con índice base 0 (`page=0` es la primera página), pero al usuario le muestras "Página 1". Debes mantener esa conversión consistente.

**Regla de oro:**
- Al enviar al backend: usa el índice que guarda `numeroPagina` (0, 1, 2...).
- Al mostrar al usuario: suma 1 (`numeroPagina() + 1`).
- Al recibir del backend: `pagina.number` ya viene en base 0, úsalo directamente.

---

### Error 6: Memory leak — el componente sigue recibiendo eventos después de destruirse

**Síntoma:** En la consola aparece `WARNING: subscriptions after component destroyed`.

**Causa:** El `Observable` sigue activo después de que el componente se destruyó.

**Solución:** Usar el patrón `takeUntil(destroy$)`:
```typescript
export class MiComponente implements OnInit, OnDestroy {
  private readonly destroy$ = new Subject<void>();

  ngOnInit(): void {
    miServicio.datos$
      .pipe(takeUntil(this.destroy$)) // ← Cancela cuando destroy$ emite
      .subscribe(datos => { /* ... */ });
  }

  ngOnDestroy(): void {
    this.destroy$.next(); // Emitir señal de destrucción
    this.destroy$.complete();
  }
}
```

---

### Error 7: El guard redirige en bucle infinito

**Síntoma:** La aplicación se queda en un ciclo de redirecciones entre `/auth/login` y la ruta protegida.

**Causa:** La ruta `/auth/login` está protegida por el `authGuard`, que redirige a `/auth/login` si no hay sesión, creando un bucle.

**Solución:** Asegúrate de que las rutas de autenticación NO tengan el `authGuard`:
```typescript
// CORRECTO: las rutas /auth/* son públicas
{
  path: 'auth',
  // Sin canActivate: [authGuard] aquí
  children: [
    { path: 'login', component: LoginComponent },
    { path: 'registro', component: RegistroComponent },
  ],
},
```

---

## 17. Ejercicios Prácticos

Los siguientes retos están diseñados para que practiques y consolides lo aprendido en esta guía. Son progresivos: cada uno construye sobre el anterior.

### Reto 1 — Básico: Completar el formulario de registro

Implementa el componente `RegistroComponent` siguiendo el mismo patrón del `LoginComponent`. El formulario debe tener:
- Campos: nombre, apellido, email, password, confirmarPassword.
- Validación de que `password` y `confirmarPassword` sean iguales (validador personalizado a nivel de FormGroup).
- Mostrar el error de contraseñas no coincidentes debajo del segundo campo.
- Al registrarse exitosamente, mostrar un toast y redirigir al login.

### Reto 2 — Intermedio: Formulario de asignación de responsable

Implementa el componente `AsignarResponsableComponent` que:
- Carga la lista de funcionarios activos desde `UsuarioService.obtenerFuncionariosActivos()`.
- Muestra un select con los funcionarios disponibles.
- Al seleccionar y confirmar, llama a `SolicitudService.asignarResponsable()`.
- Muestra un spinner mientras carga los funcionarios.
- Muestra un toast de éxito o error según el resultado.

### Reto 3 — Intermedio: Cambio de estado con confirmación

En el componente de detalle de solicitud, agrega un botón "Cambiar Estado" que:
- Solo aparezca para FUNCIONARIO y ADMIN.
- Al hacer clic, muestre un modal de confirmación con:
  - El estado actual.
  - Un select con los estados válidos siguientes (según las reglas del Triage).
  - Un campo de observación obligatorio.
- Al confirmar, llame a `SolicitudService.cambiarEstado()` y actualice la vista.

### Reto 4 — Avanzado: Dashboard con estadísticas en tiempo real

Crea el componente `DashboardComponent` que muestre:
- Total de solicitudes por estado (tarjetas con números).
- Total de solicitudes por prioridad.
- Las 5 solicitudes más recientes.
- Usa `forkJoin` para cargar todos los datos en paralelo.
- Implementa un botón "Actualizar" que recargue los datos.
- Usa signals para todo el estado del componente.

### Reto 5 — Avanzado: Pipe personalizado para labels

Crea un pipe standalone `EstadoLabelPipe` que:
- Reciba un valor de `EstadoSolicitud`.
- Devuelva la etiqueta legible en español (REGISTRADA → "Registrada", EN_ATENCION → "En Atención", etc.).
- Sea reutilizable en todos los componentes que muestran estados.
- Agrega también un segundo argumento opcional `formato: 'corto' | 'completo'`.

---

## 18. Resumen y Cheat Sheet

### Cheat Sheet: Configuración esencial

```typescript
// 1. Registrar HttpClient con interceptores
provideHttpClient(withInterceptors([authInterceptor, errorInterceptor]))

// 2. Inyectar dependencias (forma moderna)
private readonly service = inject(MiService);

// 3. Estado reactivo local
readonly cargando = signal(false);
readonly datos = signal<MiTipo | null>(null);
readonly estaVacio = computed(() => this.datos()?.length === 0);

// 4. Input moderno en componentes
readonly titulo = input.required<string>();
readonly opcional = input<number>(0);

// 5. Guard funcional
export const miGuard: CanActivateFn = (route, state) => {
  return inject(AuthService).estaAutenticado() || router.createUrlTree(['/login']);
};

// 6. Interceptor funcional
export const miInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }));
};

// 7. Cancelar suscripciones
private readonly destroy$ = new Subject<void>();
miObservable$.pipe(takeUntil(this.destroy$)).subscribe(...);
ngOnDestroy() { this.destroy$.next(); this.destroy$.complete(); }

// 8. Peticiones en paralelo
forkJoin({ solicitud: service.get(id), historial: service.getHistorial(id) })
  .subscribe(({ solicitud, historial }) => { ... });

// 9. Filtros dinámicos con debounce
formGroup.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  takeUntil(this.destroy$)
).subscribe(() => cargar());

// 10. HttpParams para query strings
let params = new HttpParams().set('page', '0').set('size', '10');
if (filtro) params = params.set('estado', filtro);
```

### Cheat Sheet: Control flow moderno en templates

```html
<!-- Condicional -->
@if (condicion) {
  <div>Contenido si verdadero</div>
} @else if (otraCondicion) {
  <div>Alternativa</div>
} @else {
  <div>Por defecto</div>
}

<!-- Bucle con index, first, last -->
@for (item of lista; track item.id; let i = $index; let esPrimero = $first; let esUltimo = $last) {
  <div [class.primero]="esPrimero">{{ i + 1 }}. {{ item.nombre }}</div>
}

<!-- Vacío en bucle -->
@for (item of lista; track item.id) {
  <div>{{ item }}</div>
} @empty {
  <div>No hay elementos</div>
}

<!-- Switch -->
@switch (estado) {
  @case ('REGISTRADA') { <span class="gris">Registrada</span> }
  @case ('CERRADA') { <span class="oscuro">Cerrada</span> }
  @default { <span>{{ estado }}</span> }
}
```

### Cheat Sheet: Reactive Forms

```typescript
// Crear formulario
readonly form = this.fb.group({
  nombre: ['', [Validators.required, Validators.minLength(3)]],
  email: ['', [Validators.required, Validators.email]],
});

// Getter para acceso fácil en template
get nombre() { return this.form.get('nombre')!; }

// Verificar validez
nombre.invalid && nombre.touched  // Para mostrar error

// Acceder a errores específicos
nombre.errors?.['required']
nombre.errors?.['minlength']?.requiredLength

// Marcar todo como touched (para mostrar todos los errores al hacer submit)
this.form.markAllAsTouched();

// Pre-poblar el formulario
this.form.patchValue({ nombre: 'Juan', email: 'juan@test.com' });

// Deshabilitar campo
this.form.get('email')?.disable();

// Obtener valor (incluyendo campos deshabilitados)
this.form.getRawValue();
```

---

## 19. Referencias y Recursos Adicionales

- **Documentación oficial Angular 19:** https://angular.dev
- **Angular Signals:** https://angular.dev/guide/signals
- **RxJS Operators:** https://rxjs.dev/guide/operators
- **Angular Control Flow:** https://angular.dev/guide/templates/control-flow
- **Standalone Components:** https://angular.dev/guide/components/importing
- **Functional Guards:** https://angular.dev/guide/routing/common-router-tasks#preventing-unauthorized-access
- **Functional Interceptors:** https://angular.dev/guide/http/interceptors
- **HttpParams:** https://angular.dev/api/common/http/HttpParams
- **forkJoin:** https://rxjs.dev/api/index/function/forkJoin
- **debounceTime:** https://rxjs.dev/api/operators/debounceTime
- **BehaviorSubject:** https://rxjs.dev/api/index/class/BehaviorSubject

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
