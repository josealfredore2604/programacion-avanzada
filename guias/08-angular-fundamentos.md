# Guía 08 — Angular desde Cero hasta Intermedio: Fundamentos Prácticos

> **Materia:** Programación Avanzada | **Núcleo temático:** 4 — Capa de Presentación  
> **Prerrequisitos:** Guías 01–07 (arquitecturas, Spring Boot, JPA, API REST, Spring Security + JWT)  
> **Tiempo estimado de estudio:** 10–14 horas  
> **Stack:** Angular 19+, TypeScript 5+, Node.js 22 LTS

---

## Tabla de Contenidos

1. [Objetivos de Aprendizaje](#1-objetivos-de-aprendizaje)
2. [Instalar Node.js, npm y Angular CLI](#2-instalar-nodejs-npm-y-angular-cli)
3. [TypeScript Esencial para Angular](#3-typescript-esencial-para-angular)
4. [Crear tu Primer Proyecto Angular 19+](#4-crear-tu-primer-proyecto-angular-19)
5. [Anatomía del Proyecto Angular](#5-anatomía-del-proyecto-angular)
6. [Componentes: El Corazón de Angular](#6-componentes-el-corazón-de-angular)
7. [Templates Modernos: Interpolación, Bindings y Eventos](#7-templates-modernos-interpolación-bindings-y-eventos)
8. [Control Flow Moderno: @if, @for, @switch](#8-control-flow-moderno-if-for-switch)
9. [Directivas de Atributo: ngClass y ngStyle](#9-directivas-de-atributo-ngclass-y-ngstyle)
10. [Pipes: Transformar Datos en la Vista](#10-pipes-transformar-datos-en-la-vista)
11. [Servicios e Inyección de Dependencias](#11-servicios-e-inyección-de-dependencias)
12. [HttpClient: Consumir APIs REST](#12-httpclient-consumir-apis-rest)
13. [Observables y RxJS Esencial](#13-observables-y-rxjs-esencial)
14. [Reactive Forms: Formularios Robustos](#14-reactive-forms-formularios-robustos)
15. [Routing: Navegación entre Vistas](#15-routing-navegación-entre-vistas)
16. [Estructura de Carpetas Profesional para el Triage](#16-estructura-de-carpetas-profesional-para-el-triage)
17. [Ejercicios Prácticos](#17-ejercicios-prácticos)
18. [Errores Comunes y Troubleshooting](#18-errores-comunes-y-troubleshooting)
19. [Resumen y Cheat Sheet](#19-resumen-y-cheat-sheet)
20. [Referencias y Recursos Adicionales](#20-referencias-y-recursos-adicionales)

---

## 1. Objetivos de Aprendizaje

Al finalizar esta guía serás capaz de:

- Instalar y configurar el entorno de desarrollo para Angular 19+.
- Entender TypeScript lo suficiente para trabajar fluidamente con Angular.
- Crear un proyecto Angular moderno con **Standalone Components** (sin NgModules).
- Construir componentes reutilizables con sus templates, estilos y lógica.
- Usar el **control flow moderno** (`@if`, `@for`, `@switch`) introducido en Angular 17.
- Crear **servicios** e inyectar dependencias con `inject()`.
- Consumir una API REST desde Angular con `HttpClient`.
- Manejar flujos asíncronos con **Observables** y operadores **RxJS** esenciales.
- Construir formularios robustos con validaciones usando **Reactive Forms**.
- Configurar **routing** con rutas protegidas, lazy loading y functional guards.
- Organizar el proyecto frontend del **Sistema de Triage** de forma profesional.

---

## 2. Instalar Node.js, npm y Angular CLI

### ¿Por qué necesitas Node.js si Angular corre en el navegador?

Esta es una pregunta muy válida. Angular es un framework que corre en el **navegador**, pero su cadena de herramientas de desarrollo (compilación, servidor de desarrollo, transpilación de TypeScript) corre sobre **Node.js**. Piénsalo así: Node.js es como el "taller" donde construyes el carro; el navegador es la calle donde el carro corre.

### Paso 1: Instalar Node.js 22 LTS

Visita [https://nodejs.org](https://nodejs.org) y descarga la versión **LTS (Long Term Support)** — en 2026 esa es la versión 22.x.

**Verificar la instalación:**

```bash
node --version   # Debe mostrar algo como: v22.x.x
npm --version    # Debe mostrar algo como: 10.x.x
```

> **Nota para Windows:** Se recomienda instalar Node.js a través de **nvm-windows** (Node Version Manager para Windows). Esto te permite tener múltiples versiones de Node y cambiar entre ellas fácilmente. Descárgalo desde: https://github.com/coreybutler/nvm-windows

> **Nota para macOS/Linux:** Usa **nvm** (Node Version Manager):
> ```bash
> curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
> nvm install 22
> nvm use 22
> ```

### Paso 2: Instalar Angular CLI

**Angular CLI** es la herramienta de línea de comandos oficial que te permite crear proyectos, generar componentes, servicios, construir y servir la aplicación.

```bash
npm install -g @angular/cli
```

**Verificar la instalación:**

```bash
ng version
```

Deberías ver algo así:

```
     _                      _                 ____ _     ___
    / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
   / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
  / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
 /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                |___/

Angular CLI: 19.x.x
Node: 22.x.x
Package Manager: npm 10.x.x
```

### Paso 3: Configurar el editor

Se recomienda **Visual Studio Code** con las siguientes extensiones:

| Extensión | Propósito |
|-----------|-----------|
| Angular Language Service | Autocompletado en templates HTML |
| ESLint | Análisis estático de código |
| Prettier | Formateo de código |
| Angular Snippets | Snippets de código Angular |
| GitLens | Integración con Git |

---

## 3. TypeScript Esencial para Angular

Angular está escrito en **TypeScript**, un superconjunto de JavaScript que añade **tipado estático**. Si sabes Java, vas a sentirte muy cómodo porque TypeScript tiene conceptos muy similares a los que ya conoces.

### 3.1 Tipos Primitivos y Variables

```typescript
// En TypeScript declaras el tipo con dos puntos (:)
let nombre: string = "Juan Pérez";
let edad: number = 22;
let activo: boolean = true;

// TypeScript puede inferir el tipo (como `var` con tipo en Java)
let semestre = 6;  // TypeScript infiere: number

// Constantes
const MAX_SOLICITUDES = 100;

// Array tipado — similar a List<String> en Java
let estados: string[] = ["REGISTRADA", "CLASIFICADA", "EN_ATENCION"];

// Alternativa: Array<tipo>
let prioridades: Array<number> = [1, 2, 3];
```

### 3.2 Interfaces: Definir la Forma de los Objetos

En TypeScript, las **interfaces** definen la estructura de un objeto. Son parecidas a las interfaces de Java, pero aquí se usan principalmente para describir la forma de los datos.

```typescript
// Esta interface describe cómo se ve una Solicitud que viene del backend
export interface Solicitud {
  id: number;
  descripcion: string;
  estado: EstadoSolicitud;
  prioridad: Prioridad;
  tipoSolicitud: TipoSolicitud;
  fechaRegistro: string;    // Las fechas del backend llegan como string ISO 8601
  solicitante: UsuarioResumen;
  responsable?: UsuarioResumen;  // El ? indica que es OPCIONAL (puede ser null)
}

// Interfaces auxiliares
export interface UsuarioResumen {
  id: number;
  nombre: string;
  correo: string;
}

// Enums — igual que en Java
export enum EstadoSolicitud {
  REGISTRADA = 'REGISTRADA',
  CLASIFICADA = 'CLASIFICADA',
  EN_ATENCION = 'EN_ATENCION',
  ATENDIDA = 'ATENDIDA',
  CERRADA = 'CERRADA'
}

export enum Prioridad {
  BAJA = 'BAJA',
  MEDIA = 'MEDIA',
  ALTA = 'ALTA',
  CRITICA = 'CRITICA'
}

export enum TipoSolicitud {
  REGISTRO_ASIGNATURAS = 'REGISTRO_ASIGNATURAS',
  HOMOLOGACION = 'HOMOLOGACION',
  CANCELACION_ASIGNATURAS = 'CANCELACION_ASIGNATURAS',
  SOLICITUD_CUPOS = 'SOLICITUD_CUPOS',
  CONSULTA_ACADEMICA = 'CONSULTA_ACADEMICA'
}
```

### 3.3 Clases en TypeScript

```typescript
// Las clases de TypeScript son muy similares a Java
export class SolicitudService {
  private baseUrl: string;

  // Constructor con visibilidad
  constructor(private http: HttpClient) {
    this.baseUrl = '/api/v1/solicitudes';
  }

  // Método tipado
  obtenerTodas(): Observable<Solicitud[]> {
    return this.http.get<Solicitud[]>(this.baseUrl);
  }
}
```

### 3.4 Genéricos — Como los de Java

```typescript
// Tipo genérico para respuestas paginadas del backend
export interface Page<T> {
  content: T[];
  totalElements: number;
  totalPages: number;
  number: number;     // página actual (0-indexed)
  size: number;
  first: boolean;
  last: boolean;
}

// Uso:
// Page<Solicitud> — una página de solicitudes
// Page<Usuario>   — una página de usuarios
```

### 3.5 Async/Await y Promises vs Observables

Angular usa **Observables** (de la librería RxJS) en lugar de Promises para manejo asíncrono. En la sección 13 lo explicamos a fondo, pero aquí la idea básica:

```typescript
// Promise (JavaScript clásico) — resuelve UNA VEZ
function cargarDatosConPromise(): Promise<string> {
  return fetch('/api/datos').then(res => res.json());
}

// Observable (RxJS / Angular) — puede emitir MÚLTIPLES valores a lo largo del tiempo
import { Observable, of } from 'rxjs';

function cargarDatosConObservable(): Observable<string> {
  return this.http.get<string>('/api/datos');  // HttpClient devuelve Observables
}
```

### 3.6 Decoradores

Los **decoradores** en TypeScript son como las **anotaciones** en Java (`@Override`, `@Service`, etc.). Angular los usa extensivamente:

```typescript
// @Component es un decorador — le dice a Angular que esta clase ES un componente
@Component({
  selector: 'app-solicitud',
  templateUrl: './solicitud.component.html'
})
export class SolicitudComponent {
  // ...
}
```

### 3.7 Tabla Comparativa: Java vs TypeScript

| Concepto | Java | TypeScript |
|----------|------|------------|
| Tipo de variable | `String nombre = "Juan"` | `let nombre: string = "Juan"` |
| Constante | `final int MAX = 10` | `const MAX = 10` |
| Arreglo | `String[] nombres` | `string[]` o `Array<string>` |
| Nulabilidad | `Optional<T>` | `T \| null` o `T?` |
| Interface (datos) | No existe directamente | `interface Solicitud { ... }` |
| Clase abstracta | `abstract class` | `abstract class` (igual) |
| Genéricos | `List<T>` | `Array<T>` o `T[]` |
| Anotaciones | `@Service` | `@Injectable()` (decoradores) |
| Lambda | `x -> x * 2` | `x => x * 2` |
| Async | `CompletableFuture<T>` | `Promise<T>` u `Observable<T>` |

---

## 4. Crear tu Primer Proyecto Angular 19+

### NgModules vs Standalone Components: El Gran Cambio

Antes de crear el proyecto, necesitas entender el cambio más importante de Angular en los últimos años:

**Antes (Angular < 14): NgModules obligatorios**

```
AppModule
├── declarations: [AppComponent, SolicitudComponent, UsuarioComponent]
├── imports: [BrowserModule, HttpClientModule, RouterModule, ...]
├── providers: [SolicitudService]
└── bootstrap: [AppComponent]
```

Los NgModules eran contenedores obligatorios que agrupaban componentes, directivas, pipes y servicios. Esto generaba mucho código *boilerplate* y archivos adicionales difíciles de mantener.

**Ahora (Angular 17+): Standalone Components por defecto**

```typescript
// Cada componente declara sus propias dependencias — sin módulo intermediario
@Component({
  standalone: true,           // <-- Este componente es autónomo
  imports: [CommonModule, RouterLink, NgClass],  // Importa solo lo que necesita
  template: `...`
})
export class SolicitudComponent { }
```

| Característica | NgModules (anterior) | Standalone (actual) |
|----------------|----------------------|---------------------|
| Boilerplate | Alto — archivo de módulo por feature | Mínimo — todo en el componente |
| Lazy loading | Requería módulos | Directo con `loadComponent()` |
| Testing | Complejo — hay que configurar el módulo | Más simple |
| Tree shaking | Menos eficiente | Más eficiente |
| Recomendado por Google | No (legacy) | Sí (desde Angular 17) |

### Crear el Proyecto

```bash
ng new triage-frontend
```

Angular CLI te hará algunas preguntas:

```
? Which stylesheet format would you like to use?
  ❯ CSS
    SCSS   [https://sass-lang.com/documentation/syntax#scss]
    Sass   [https://sass-lang.com/documentation/syntax#the-indented-syntax]
    Less   [http://lesscss.org]

→ Selecciona SCSS (te da variables, anidamiento y más potencia)

? Do you want to enable Server-Side Rendering (SSR) and Static Site Generation (SSG/Prerendering)?
→ No (para este curso usamos SPA — Single Page Application)
```

Angular CLI generará el proyecto y ejecutará `npm install` automáticamente.

**Iniciar el servidor de desarrollo:**

```bash
cd triage-frontend
ng serve
```

Abre el navegador en `http://localhost:4200`. Verás la página de bienvenida de Angular. ¡Tu proyecto está corriendo!

> **Dato importante:** El servidor de desarrollo tiene **hot reload** — cada vez que guardas un archivo, el navegador se actualiza automáticamente. No necesitas reiniciar el servidor.

---

## 5. Anatomía del Proyecto Angular

```
triage-frontend/
├── src/
│   ├── app/
│   │   ├── app.component.ts        ← Componente raíz
│   │   ├── app.component.html      ← Template del componente raíz
│   │   ├── app.component.scss      ← Estilos del componente raíz
│   │   ├── app.component.spec.ts   ← Tests del componente raíz
│   │   └── app.routes.ts           ← Configuración de rutas
│   ├── assets/                     ← Imágenes, fuentes, archivos estáticos
│   ├── index.html                  ← HTML principal (solo un div #app-root)
│   ├── main.ts                     ← Punto de entrada de la aplicación
│   └── styles.scss                 ← Estilos globales
├── angular.json                    ← Configuración del proyecto Angular
├── package.json                    ← Dependencias npm
├── tsconfig.json                   ← Configuración de TypeScript
└── tsconfig.app.json               ← Config TypeScript específica del app
```

### Archivo por archivo

**`src/index.html`** — El HTML de toda la aplicación. Angular inyecta la app aquí:

```html
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>Sistema de Triage Académico</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <!-- Angular reemplaza este selector con el AppComponent -->
  <app-root></app-root>
</body>
</html>
```

**`src/main.ts`** — Punto de entrada. Le dice a Angular cómo arrancar:

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

// Arranca la aplicación con el componente raíz y la configuración
bootstrapApplication(AppComponent, appConfig)
  .catch((err) => console.error(err));
```

**`src/app/app.config.ts`** — Configuración central de la aplicación (reemplaza a AppModule):

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    // Change detection optimizado
    provideZoneChangeDetection({ eventCoalescing: true }),
    // Configura el router con las rutas definidas
    provideRouter(routes),
    // Configura HttpClient (equivalente a importar HttpClientModule)
    provideHttpClient(),
    // Habilita animaciones de Angular Material
    provideAnimationsAsync(),
  ]
};
```

**`src/app/app.component.ts`** — El componente raíz:

```typescript
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',        // El tag HTML que usará este componente
  standalone: true,
  imports: [RouterOutlet],     // Necesita RouterOutlet para mostrar rutas
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'triage-frontend';
}
```

**`src/app/app.routes.ts`** — Definición de rutas:

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  // Aquí definiremos nuestras rutas — lo veremos en la sección 15
];
```

---

## 6. Componentes: El Corazón de Angular

Un **componente** en Angular es la unidad básica de la interfaz de usuario. Cada componente tiene tres partes:

1. **Clase TypeScript (`.ts`)** — La lógica: datos, métodos, propiedades.
2. **Template HTML (`.html`)** — La vista: lo que el usuario ve.
3. **Estilos CSS/SCSS (`.scss`)** — El aspecto visual.

Puedes pensar en un componente como una **pantalla** o una **sección** de la interfaz: el formulario de crear solicitud es un componente, la tabla de solicitudes es otro, el menú de navegación es otro.

### 6.1 Generar un Componente con Angular CLI

```bash
ng generate component features/solicitudes/lista-solicitudes
# Atajo:
ng g c features/solicitudes/lista-solicitudes
```

Esto crea automáticamente:

```
src/app/features/solicitudes/lista-solicitudes/
├── lista-solicitudes.component.ts
├── lista-solicitudes.component.html
├── lista-solicitudes.component.scss
└── lista-solicitudes.component.spec.ts
```

### 6.2 Anatomía de un Componente Completo

Vamos a crear el componente para listar solicitudes del sistema de Triage. Primero te explico qué haremos y por qué:

> **Contexto:** Necesitamos una vista que muestre todas las solicitudes del sistema. Esta vista tendrá un listado de solicitudes con su estado y prioridad. El componente tendrá datos hardcodeados por ahora; en la Guía 09 lo conectaremos al backend real.

```typescript
// src/app/features/solicitudes/lista-solicitudes/lista-solicitudes.component.ts

import { Component, OnInit, OnDestroy, signal, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink } from '@angular/router';
import { Solicitud, EstadoSolicitud, Prioridad } from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-lista-solicitudes',   // Cómo se usa en otros templates: <app-lista-solicitudes>
  standalone: true,                     // Es un componente autónomo (sin módulos)
  imports: [
    CommonModule,    // Provee ngClass, ngStyle, AsyncPipe, DatePipe, etc.
    RouterLink,      // Permite usar [routerLink] en el template
  ],
  templateUrl: './lista-solicitudes.component.html',
  styleUrl: './lista-solicitudes.component.scss'
})
export class ListaSolicitudesComponent implements OnInit, OnDestroy {

  // ---------------------------------------------------------
  // SIGNALS: La forma moderna de manejar estado reactivo
  // (explicados abajo)
  // ---------------------------------------------------------

  /** Lista de solicitudes cargadas */
  solicitudes = signal<Solicitud[]>([]);

  /** Estado del filtro activo */
  filtroEstado = signal<EstadoSolicitud | null>(null);

  /** Si la pantalla está cargando datos */
  cargando = signal<boolean>(false);

  /** Mensaje de error si algo falla */
  error = signal<string | null>(null);

  // Computed: se recalcula automáticamente cuando cambia solicitudes o filtroEstado
  solicitudesFiltradas = computed(() => {
    const estado = this.filtroEstado();
    if (!estado) return this.solicitudes();
    return this.solicitudes().filter(s => s.estado === estado);
  });

  // Valores del enum para usar en el template
  readonly EstadoSolicitud = EstadoSolicitud;
  readonly Prioridad = Prioridad;

  // ---------------------------------------------------------
  // CICLO DE VIDA
  // ---------------------------------------------------------

  ngOnInit(): void {
    // ngOnInit se ejecuta UNA VEZ cuando el componente se inicializa
    // Es el lugar correcto para cargar datos iniciales
    console.log('ListaSolicitudesComponent inicializado');
    this.cargarDatosDePrueba();
  }

  ngOnDestroy(): void {
    // ngOnDestroy se ejecuta cuando el componente se destruye (navegar a otra ruta)
    // Aquí debes cancelar suscripciones para evitar memory leaks
    console.log('ListaSolicitudesComponent destruido');
  }

  // ---------------------------------------------------------
  // MÉTODOS
  // ---------------------------------------------------------

  /** Filtra las solicitudes por estado */
  aplicarFiltro(estado: EstadoSolicitud | null): void {
    this.filtroEstado.set(estado);
  }

  /** Retorna una clase CSS según la prioridad */
  clasePrioridad(prioridad: Prioridad): string {
    const clases: Record<Prioridad, string> = {
      [Prioridad.BAJA]: 'badge-baja',
      [Prioridad.MEDIA]: 'badge-media',
      [Prioridad.ALTA]: 'badge-alta',
      [Prioridad.CRITICA]: 'badge-critica',
    };
    return clases[prioridad] ?? 'badge-baja';
  }

  /** Datos de prueba mientras no tenemos el backend conectado */
  private cargarDatosDePrueba(): void {
    this.cargando.set(true);

    // Simulamos un retardo de red
    setTimeout(() => {
      this.solicitudes.set([
        {
          id: 1,
          descripcion: 'Solicitud de homologación de Cálculo I',
          estado: EstadoSolicitud.REGISTRADA,
          prioridad: Prioridad.ALTA,
          tipoSolicitud: 'HOMOLOGACION' as any,
          fechaRegistro: '2026-02-15T10:30:00',
          solicitante: { id: 1, nombre: 'Carlos Gómez', correo: 'cgomez@uq.edu.co' },
        },
        {
          id: 2,
          descripcion: 'Cancelación de asignatura Programación Avanzada',
          estado: EstadoSolicitud.CLASIFICADA,
          prioridad: Prioridad.MEDIA,
          tipoSolicitud: 'CANCELACION_ASIGNATURAS' as any,
          fechaRegistro: '2026-02-16T08:00:00',
          solicitante: { id: 2, nombre: 'Laura Torres', correo: 'ltorres@uq.edu.co' },
        },
        {
          id: 3,
          descripcion: 'Solicitud de cupo en Bases de Datos II',
          estado: EstadoSolicitud.EN_ATENCION,
          prioridad: Prioridad.CRITICA,
          tipoSolicitud: 'SOLICITUD_CUPOS' as any,
          fechaRegistro: '2026-02-17T14:00:00',
          solicitante: { id: 3, nombre: 'Andrés Muñoz', correo: 'amunoz@uq.edu.co' },
        },
      ]);
      this.cargando.set(false);
    }, 800);
  }
}
```

### 6.3 Signals: El Nuevo Sistema de Reactividad

Angular 17+ introduce **Signals** como la forma moderna de manejar estado reactivo. Es mucho más simple que la detección de cambios anterior.

```typescript
import { signal, computed, effect } from '@angular/core';

// signal() crea un valor reactivo
const contador = signal(0);

// Leer el valor: se llama como función
console.log(contador());  // → 0

// Modificar el valor
contador.set(5);           // Reemplaza el valor
contador.update(v => v + 1);  // Modifica basándose en el valor actual

// computed() crea un valor que se deriva de otros signals
// Se actualiza automáticamente cuando cambian sus dependencias
const doble = computed(() => contador() * 2);

// effect() ejecuta código cuando cambian los signals
effect(() => {
  console.log('El contador cambió a:', contador());
});
```

### 6.4 Ciclo de Vida Completo

| Hook | Cuándo se ejecuta | Uso típico |
|------|-------------------|------------|
| `ngOnChanges` | Cuando cambian los `@Input()` | Reaccionar a cambios de datos del padre |
| `ngOnInit` | Una vez al inicializar | Cargar datos iniciales, iniciar observables |
| `ngDoCheck` | En cada ciclo de detección | Detección manual de cambios |
| `ngAfterContentInit` | Después de proyectar contenido | Acceder a `@ContentChild` |
| `ngAfterViewInit` | Después de renderizar la vista | Acceder a elementos DOM con `@ViewChild` |
| `ngOnDestroy` | Al destruir el componente | Cancelar suscripciones, limpiar timers |

### 6.5 Inputs y Outputs: Comunicación entre Componentes

```typescript
// Componente HIJO: recibe datos del padre con @Input()
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { Solicitud } from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-tarjeta-solicitud',
  standalone: true,
  template: `
    <div class="tarjeta">
      <h3>{{ solicitud.descripcion }}</h3>
      <p>Estado: {{ solicitud.estado }}</p>
      <button (click)="verDetalle()">Ver detalle</button>
    </div>
  `
})
export class TarjetaSolicitudComponent {
  // @Input: datos que vienen del componente padre
  @Input({ required: true }) solicitud!: Solicitud;

  // @Output: eventos que el hijo emite al padre
  @Output() solicitudSeleccionada = new EventEmitter<number>();

  verDetalle(): void {
    // Emite el id al componente padre
    this.solicitudSeleccionada.emit(this.solicitud.id);
  }
}
```

```typescript
// Componente PADRE: envía datos y escucha eventos del hijo
@Component({
  selector: 'app-lista-solicitudes',
  standalone: true,
  imports: [TarjetaSolicitudComponent],
  template: `
    @for (s of solicitudes(); track s.id) {
      <!-- Envía cada solicitud al hijo con property binding -->
      <!-- Escucha el evento solicitudSeleccionada del hijo -->
      <app-tarjeta-solicitud
        [solicitud]="s"
        (solicitudSeleccionada)="onSolicitudSeleccionada($event)"
      />
    }
  `
})
export class ListaSolicitudesComponent {
  solicitudes = signal<Solicitud[]>([]);

  onSolicitudSeleccionada(id: number): void {
    console.log('El usuario seleccionó la solicitud con id:', id);
    // Navegar al detalle...
  }
}
```

---

## 7. Templates Modernos: Interpolación, Bindings y Eventos

Los templates de Angular son HTML enriquecido con sintaxis especial para conectar la vista con la lógica del componente.

### 7.1 Interpolación `{{ }}`

Muestra el valor de una expresión TypeScript en el HTML:

```html
<!-- Muestra el valor de la propiedad 'titulo' del componente -->
<h1>{{ titulo }}</h1>

<!-- Expresiones TypeScript válidas -->
<p>Total: {{ solicitudes().length }} solicitudes</p>
<p>Hoy es: {{ hoy | date:'dd/MM/yyyy' }}</p>
<p>{{ estaActivo ? 'Activo' : 'Inactivo' }}</p>
```

### 7.2 Property Binding `[propiedad]="expresion"`

Vincula una propiedad del elemento HTML con una expresión del componente:

```html
<!-- Deshabilita el botón según la condición del componente -->
<button [disabled]="cargando()">Guardar</button>

<!-- Asigna una clase CSS dinámicamente -->
<div [class]="clasePrioridad(solicitud.prioridad)">...</div>

<!-- Asigna el src de una imagen -->
<img [src]="usuario.fotoPerfil" [alt]="usuario.nombre">

<!-- Pasa datos a un componente hijo (Input) -->
<app-tarjeta-solicitud [solicitud]="solicitudActual" />
```

### 7.3 Event Binding `(evento)="metodo()"`

Escucha eventos del DOM y ejecuta métodos del componente:

```html
<!-- Click del botón llama al método 'guardar()' del componente -->
<button (click)="guardar()">Guardar</button>

<!-- Tecla Enter en un input -->
<input (keyup.enter)="buscar()">

<!-- Pasar el evento completo con $event -->
<input (input)="onInput($event)">

<!-- Evento personalizado de un componente hijo -->
<app-tarjeta-solicitud (solicitudSeleccionada)="onSolicitudSeleccionada($event)" />
```

### 7.4 Two-Way Binding `[(ngModel)]`

Vinculación bidireccional: el input muestra el valor del componente y lo actualiza automáticamente cuando el usuario escribe.

```typescript
// En el componente
busqueda = signal('');
```

```html
<!-- Requiere FormsModule importado en el componente -->
<input [(ngModel)]="busqueda" placeholder="Buscar solicitud...">
<p>Buscando: {{ busqueda() }}</p>
```

> **Nota:** Para Reactive Forms (la forma recomendada para formularios complejos) no usamos `ngModel`. Ver sección 14.

### 7.5 Template Reference Variables `#variable`

Permiten hacer referencia a un elemento DOM desde el template o el componente:

```html
<!-- #inputBusqueda es una referencia al elemento <input> -->
<input #inputBusqueda type="text" placeholder="Buscar...">
<button (click)="buscar(inputBusqueda.value)">Buscar</button>
```

---

## 8. Control Flow Moderno: @if, @for, @switch

Angular 17 introdujo una sintaxis nueva y más legible para el control de flujo en los templates. **Esta es la forma recomendada en 2026.** La sintaxis antigua con `*ngIf` y `*ngFor` sigue funcionando pero está en camino de ser deprecada.

### Comparativa: Antes vs Ahora

| Concepto | Sintaxis antigua (NgIf/NgFor) | Sintaxis moderna (@if/@for) |
|----------|-------------------------------|------------------------------|
| Condicional | `*ngIf="condicion"` | `@if (condicion) { }` |
| Else | `ng-template #else` | `@else { }` |
| Iteración | `*ngFor="let item of items"` | `@for (item of items; track item.id) { }` |
| Switch | `[ngSwitch]` + `*ngSwitchCase` | `@switch (valor) { @case (x) { } }` |
| Vacío | Manejo manual | `@empty { }` |
| Import necesario | `CommonModule` | Nada — es sintaxis nativa |

### 8.1 @if — Condicionales

```html
<!-- Mostrar loading mientras cargan los datos -->
@if (cargando()) {
  <div class="loading-spinner">
    <span>Cargando solicitudes...</span>
  </div>
} @else if (error()) {
  <div class="error-message">
    <p>{{ error() }}</p>
    <button (click)="reintentar()">Reintentar</button>
  </div>
} @else {
  <!-- Contenido principal cuando no hay error ni carga -->
  <div class="lista-container">
    <!-- ... -->
  </div>
}
```

### 8.2 @for — Iteración

```html
<!-- Iterar sobre solicitudes — track es OBLIGATORIO y mejora el rendimiento -->
<!-- track le dice a Angular cómo identificar cada elemento de la lista -->

@for (solicitud of solicitudesFiltradas(); track solicitud.id) {
  <div class="solicitud-card">
    <span class="id">#{{ solicitud.id }}</span>
    <h3>{{ solicitud.descripcion }}</h3>
    <span [class]="clasePrioridad(solicitud.prioridad)">
      {{ solicitud.prioridad }}
    </span>
    <span class="estado">{{ solicitud.estado }}</span>
    <small>
      Registrada el {{ solicitud.fechaRegistro | date:'dd/MM/yyyy HH:mm' }}
    </small>
  </div>
} @empty {
  <!-- Se muestra cuando la lista está vacía -->
  <div class="empty-state">
    <p>No hay solicitudes que coincidan con el filtro.</p>
    <button (click)="aplicarFiltro(null)">Ver todas</button>
  </div>
}
```

**Variables especiales dentro de @for:**

```html
@for (solicitud of solicitudes(); track solicitud.id; let idx = $index; let ultimo = $last) {
  <div [class.ultimo-item]="ultimo">
    {{ idx + 1 }}. {{ solicitud.descripcion }}
  </div>
}
```

| Variable | Descripción |
|----------|-------------|
| `$index` | Índice actual (0-based) |
| `$first` | `true` si es el primer elemento |
| `$last` | `true` si es el último elemento |
| `$even` | `true` si el índice es par |
| `$odd` | `true` si el índice es impar |
| `$count` | Total de elementos en la colección |

### 8.3 @switch — Switch / Case

```html
<!-- Mostrar un ícono diferente según el estado -->
@switch (solicitud.estado) {
  @case (EstadoSolicitud.REGISTRADA) {
    <span class="estado-badge registrada">📋 Registrada</span>
  }
  @case (EstadoSolicitud.CLASIFICADA) {
    <span class="estado-badge clasificada">🏷️ Clasificada</span>
  }
  @case (EstadoSolicitud.EN_ATENCION) {
    <span class="estado-badge en-atencion">⚙️ En Atención</span>
  }
  @case (EstadoSolicitud.ATENDIDA) {
    <span class="estado-badge atendida">✅ Atendida</span>
  }
  @case (EstadoSolicitud.CERRADA) {
    <span class="estado-badge cerrada">🔒 Cerrada</span>
  }
  @default {
    <span class="estado-badge">❓ Desconocido</span>
  }
}
```

### 8.4 Template Completo del Componente Lista de Solicitudes

```html
<!-- src/app/features/solicitudes/lista-solicitudes/lista-solicitudes.component.html -->

<div class="lista-solicitudes-container">
  <!-- Encabezado -->
  <div class="header">
    <h2>Solicitudes Académicas</h2>
    <a routerLink="/solicitudes/nueva" class="btn btn-primary">
      + Nueva Solicitud
    </a>
  </div>

  <!-- Filtros de estado -->
  <div class="filtros">
    <button
      [class.activo]="filtroEstado() === null"
      (click)="aplicarFiltro(null)">
      Todas
    </button>

    @for (estado of [EstadoSolicitud.REGISTRADA, EstadoSolicitud.CLASIFICADA,
                     EstadoSolicitud.EN_ATENCION, EstadoSolicitud.ATENDIDA,
                     EstadoSolicitud.CERRADA]; track estado) {
      <button
        [class.activo]="filtroEstado() === estado"
        (click)="aplicarFiltro(estado)">
        {{ estado }}
      </button>
    }
  </div>

  <!-- Contenido principal -->
  @if (cargando()) {
    <div class="loading">
      <p>Cargando solicitudes...</p>
    </div>
  } @else if (error()) {
    <div class="error">
      <p>Error: {{ error() }}</p>
    </div>
  } @else {
    <div class="total-info">
      Mostrando {{ solicitudesFiltradas().length }} solicitudes
    </div>

    @for (solicitud of solicitudesFiltradas(); track solicitud.id) {
      <div class="solicitud-card">
        <div class="solicitud-header">
          <span class="solicitud-id">#{{ solicitud.id }}</span>
          <span [class]="'badge prioridad-' + solicitud.prioridad.toLowerCase()">
            {{ solicitud.prioridad }}
          </span>

          @switch (solicitud.estado) {
            @case (EstadoSolicitud.REGISTRADA) {
              <span class="estado registrada">Registrada</span>
            }
            @case (EstadoSolicitud.CLASIFICADA) {
              <span class="estado clasificada">Clasificada</span>
            }
            @case (EstadoSolicitud.EN_ATENCION) {
              <span class="estado en-atencion">En Atención</span>
            }
            @case (EstadoSolicitud.ATENDIDA) {
              <span class="estado atendida">Atendida</span>
            }
            @case (EstadoSolicitud.CERRADA) {
              <span class="estado cerrada">Cerrada</span>
            }
          }
        </div>

        <p class="solicitud-descripcion">{{ solicitud.descripcion }}</p>

        <div class="solicitud-footer">
          <span>{{ solicitud.solicitante.nombre }}</span>
          <small>{{ solicitud.fechaRegistro | date:'dd/MM/yyyy' }}</small>
          <a [routerLink]="['/solicitudes', solicitud.id]">Ver detalle →</a>
        </div>
      </div>
    } @empty {
      <div class="empty-state">
        <p>No hay solicitudes con el filtro seleccionado.</p>
        <button (click)="aplicarFiltro(null)">Ver todas</button>
      </div>
    }
  }
</div>
```

---

## 9. Directivas de Atributo: ngClass y ngStyle

Las directivas de atributo modifican el aspecto o comportamiento de elementos existentes (a diferencia de las directivas estructurales `@if`, `@for` que cambian la estructura del DOM).

### 9.1 [ngClass] — Clases CSS Dinámicas

```html
<!-- Forma 1: Un string con la clase -->
<span [ngClass]="clasePrioridad(solicitud.prioridad)">{{ solicitud.prioridad }}</span>

<!-- Forma 2: Un objeto — la clase se aplica si el valor es true -->
<div [ngClass]="{
  'cargando': cargando(),
  'con-error': error() !== null,
  'lista-vacia': solicitudes().length === 0
}">
  Contenido
</div>

<!-- Forma 3: Un array de clases -->
<button [ngClass]="['btn', 'btn-primary', esAdmin ? 'btn-admin' : '']">
  Acción
</button>
```

### 9.2 [ngStyle] — Estilos Inline Dinámicos

```html
<!-- Pinta el fondo según la prioridad (útil para colores calculados dinámicamente) -->
<div [ngStyle]="{
  'background-color': prioridadColor(solicitud.prioridad),
  'font-size': tamañoFuente + 'px'
}">
  Contenido
</div>
```

> **Recomendación:** Prefiere `[ngClass]` sobre `[ngStyle]`. Es más mantenible definir las clases en el SCSS que los estilos inline en el template.

---

## 10. Pipes: Transformar Datos en la Vista

Los **Pipes** transforman valores en los templates. Son como filtros que puedes encadenar con el operador `|`.

### 10.1 Pipes Integrados de Angular

```html
<!-- DatePipe: formatear fechas -->
<p>{{ solicitud.fechaRegistro | date:'dd/MM/yyyy' }}</p>
<p>{{ solicitud.fechaRegistro | date:'dd/MM/yyyy HH:mm' }}</p>
<p>{{ solicitud.fechaRegistro | date:'long' }}</p>
<!-- Salida: "15 de febrero de 2026, 10:30:00 a. m." -->

<!-- CurrencyPipe: moneda -->
<span>{{ costo | currency:'COP':'symbol':'1.0-0' }}</span>
<!-- Salida: "$1.500.000" -->

<!-- UpperCasePipe y LowerCasePipe -->
<span>{{ solicitud.estado | uppercase }}</span>
<span>{{ solicitud.tipoSolicitud | lowercase }}</span>

<!-- DecimalPipe: números -->
<span>{{ promedio | number:'1.2-2' }}</span>
<!-- "1.2-2" = mínimo 1 dígito entero, entre 2 y 2 decimales -->

<!-- SlicePipe: recortar arrays o strings -->
<span>{{ descripcionLarga | slice:0:100 }}...</span>

<!-- JsonPipe: útil para debug -->
<pre>{{ solicitud | json }}</pre>

<!-- AsyncPipe: desarrollar con Observables directamente en el template -->
<!-- Lo veremos en detalle en la sección 13 -->
<div>{{ solicitudes$ | async }}</div>
```

### 10.2 Crear un Pipe Personalizado

Vamos a crear un pipe que traduzca los estados del Triage al español:

```bash
ng generate pipe shared/pipes/estado-label
```

```typescript
// src/app/shared/pipes/estado-label.pipe.ts

import { Pipe, PipeTransform } from '@angular/core';
import { EstadoSolicitud } from '../../core/models/solicitud.model';

@Pipe({
  name: 'estadoLabel',   // Nombre que usarás en el template: {{ valor | estadoLabel }}
  standalone: true,       // Pipe autónomo — se importa directamente en el componente
  pure: true,             // Pure = se recalcula solo cuando el input cambia (por defecto)
})
export class EstadoLabelPipe implements PipeTransform {

  private readonly etiquetas: Record<EstadoSolicitud, string> = {
    [EstadoSolicitud.REGISTRADA]: '📋 Registrada',
    [EstadoSolicitud.CLASIFICADA]: '🏷️ Clasificada',
    [EstadoSolicitud.EN_ATENCION]: '⚙️ En Atención',
    [EstadoSolicitud.ATENDIDA]: '✅ Atendida',
    [EstadoSolicitud.CERRADA]: '🔒 Cerrada',
  };

  // transform() es el método que se llama cuando el pipe procesa un valor
  transform(value: EstadoSolicitud | null | undefined): string {
    if (!value) return 'Sin estado';
    return this.etiquetas[value] ?? value;
  }
}
```

**Uso en el template:**

```typescript
// Primero importar el pipe en el componente
@Component({
  standalone: true,
  imports: [EstadoLabelPipe],
  template: `
    <!-- Ahora puedes usar el pipe -->
    <span>{{ solicitud.estado | estadoLabel }}</span>
    <!-- Salida: "⚙️ En Atención" -->
  `
})
```

---

## 11. Servicios e Inyección de Dependencias

Los **servicios** en Angular encapsulan lógica que puede ser compartida entre múltiples componentes. Son la forma de:

- Llamar a APIs REST.
- Compartir estado entre componentes.
- Encapsular lógica de negocio del frontend.

> **Analogía:** Si los componentes son las pantallas (lo que el usuario ve), los servicios son los "ayudantes invisibles" que hacen el trabajo pesado: buscar datos, procesarlos, comunicarse con el servidor.

### 11.1 Crear un Servicio

```bash
ng generate service core/services/solicitud
```

```typescript
// src/app/core/services/solicitud.service.ts

import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';
import { Solicitud, EstadoSolicitud } from '../models/solicitud.model';
import { Page } from '../models/page.model';

// @Injectable marca esta clase como un servicio que Angular puede inyectar
// providedIn: 'root' significa que hay UNA SOLA instancia en toda la aplicación (Singleton)
// No necesitas declararlo en ningún módulo ni app.config
@Injectable({
  providedIn: 'root'
})
export class SolicitudService {

  // inject() es la forma MODERNA de obtener dependencias (Angular 14+)
  // Reemplaza la inyección por constructor en muchos casos
  private readonly http = inject(HttpClient);

  // URL base del backend (configurada en environment.ts)
  private readonly baseUrl = `${environment.apiUrl}/solicitudes`;

  /** Obtiene solicitudes paginadas con filtros opcionales */
  obtenerSolicitudes(
    pagina: number = 0,
    tamaño: number = 10,
    estado?: EstadoSolicitud
  ): Observable<Page<Solicitud>> {
    let params = new HttpParams()
      .set('page', pagina)
      .set('size', tamaño);

    if (estado) {
      params = params.set('estado', estado);
    }

    return this.http.get<Page<Solicitud>>(this.baseUrl, { params });
  }

  /** Obtiene una solicitud por ID */
  obtenerPorId(id: number): Observable<Solicitud> {
    return this.http.get<Solicitud>(`${this.baseUrl}/${id}`);
  }

  /** Crea una nueva solicitud */
  crear(solicitud: CrearSolicitudRequest): Observable<Solicitud> {
    return this.http.post<Solicitud>(this.baseUrl, solicitud);
  }

  /** Cambia el estado de una solicitud */
  cambiarEstado(id: number, nuevoEstado: EstadoSolicitud, observacion: string): Observable<Solicitud> {
    return this.http.patch<Solicitud>(`${this.baseUrl}/${id}/estado`, {
      nuevoEstado,
      observacion
    });
  }
}

// DTOs de request
export interface CrearSolicitudRequest {
  descripcion: string;
  tipoSolicitud: string;
  canalOrigen: string;
}
```

### 11.2 inject() vs Constructor Injection

| Aspecto | Constructor Injection (clásico) | inject() (moderno) |
|---------|---------------------------------|---------------------|
| Sintaxis | `constructor(private http: HttpClient)` | `private http = inject(HttpClient)` |
| Disponible | Siempre | Angular 14+ |
| Uso fuera de constructores | No | Sí (funciones, factories) |
| Functional Guards | No aplica | Necesario |
| Legibilidad | Verboso con muchas deps | Más limpio |
| Recomendado en 2026 | Legacy | ✅ Sí |

```typescript
// Antes (constructor injection):
@Injectable({ providedIn: 'root' })
export class MiServicio {
  constructor(
    private http: HttpClient,
    private router: Router,
    private authService: AuthService,
    private snackBar: MatSnackBar
  ) { }
}

// Ahora (inject function):
@Injectable({ providedIn: 'root' })
export class MiServicio {
  private readonly http = inject(HttpClient);
  private readonly router = inject(Router);
  private readonly authService = inject(AuthService);
  private readonly snackBar = inject(MatSnackBar);
}
```

### 11.3 Usar un Servicio en un Componente

```typescript
// src/app/features/solicitudes/lista-solicitudes/lista-solicitudes.component.ts

import { Component, OnInit, signal, inject } from '@angular/core';
import { SolicitudService } from '../../../core/services/solicitud.service';
import { Solicitud } from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-lista-solicitudes',
  standalone: true,
  imports: [/* ... */],
  templateUrl: './lista-solicitudes.component.html',
})
export class ListaSolicitudesComponent implements OnInit {

  // Inyectar el servicio con inject()
  private readonly solicitudService = inject(SolicitudService);

  solicitudes = signal<Solicitud[]>([]);
  cargando = signal(false);
  error = signal<string | null>(null);

  ngOnInit(): void {
    this.cargarSolicitudes();
  }

  private cargarSolicitudes(): void {
    this.cargando.set(true);
    this.error.set(null);

    // Suscribirse al Observable que devuelve el servicio
    this.solicitudService.obtenerSolicitudes().subscribe({
      next: (pagina) => {
        this.solicitudes.set(pagina.content);
        this.cargando.set(false);
      },
      error: (err) => {
        this.error.set('Error al cargar las solicitudes. Intenta de nuevo.');
        this.cargando.set(false);
        console.error(err);
      }
    });
  }
}
```

---

## 12. HttpClient: Consumir APIs REST

`HttpClient` es el servicio oficial de Angular para hacer peticiones HTTP. Está integrado en el framework y devuelve **Observables**.

### 12.1 Configurar HttpClient

Ya lo viste en `app.config.ts`, pero aquí está con detalle:

```typescript
// src/app/app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      // withInterceptors([authInterceptor, errorInterceptor])  // Los agregamos en Guía 09
    ),
  ]
};
```

### 12.2 Variables de Entorno

```typescript
// src/environments/environment.ts  (desarrollo)
export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080/api/v1',
};

// src/environments/environment.prod.ts  (producción)
export const environment = {
  production: true,
  apiUrl: 'https://tu-backend.railway.app/api/v1',
};
```

### 12.3 Todos los Verbos HTTP

```typescript
// src/app/core/services/solicitud.service.ts

@Injectable({ providedIn: 'root' })
export class SolicitudService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = `${environment.apiUrl}/solicitudes`;

  // GET — Obtener datos
  obtenerTodas(): Observable<Solicitud[]> {
    return this.http.get<Solicitud[]>(this.baseUrl);
  }

  // GET con parámetros de query
  buscar(estado: string, pagina: number): Observable<Page<Solicitud>> {
    const params = new HttpParams()
      .set('estado', estado)
      .set('page', pagina)
      .set('size', 10);
    return this.http.get<Page<Solicitud>>(this.baseUrl, { params });
  }

  // GET por ID — path variable
  obtenerPorId(id: number): Observable<Solicitud> {
    return this.http.get<Solicitud>(`${this.baseUrl}/${id}`);
  }

  // POST — Crear
  crear(datos: CrearSolicitudRequest): Observable<Solicitud> {
    return this.http.post<Solicitud>(this.baseUrl, datos);
  }

  // PUT — Reemplazar completamente
  reemplazar(id: number, datos: ActualizarSolicitudRequest): Observable<Solicitud> {
    return this.http.put<Solicitud>(`${this.baseUrl}/${id}`, datos);
  }

  // PATCH — Actualizar parcialmente
  cambiarEstado(id: number, request: CambiarEstadoRequest): Observable<Solicitud> {
    return this.http.patch<Solicitud>(`${this.baseUrl}/${id}/estado`, request);
  }

  // DELETE
  eliminar(id: number): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`);
  }
}
```

---

## 13. Observables y RxJS Esencial

Los **Observables** son el corazón de la programación reactiva en Angular. `HttpClient` los usa internamente. Entiéndelos bien y Angular se vuelve mucho más poderoso.

### 13.1 ¿Qué es un Observable?

Piensa en un Observable como un **periódico de suscripción**:

- El periódico (Observable) emite ediciones (valores) a lo largo del tiempo.
- Tú te suscribes (`subscribe`) para recibirlas.
- Puedes cancelar la suscripción (`unsubscribe`) cuando ya no te interesa.

A diferencia de una `Promise` (que resuelve **una vez**), un Observable puede emitir **múltiples valores** a lo largo del tiempo.

```typescript
import { Observable, of, from, interval } from 'rxjs';

// Observable que emite un solo valor y completa (como una Promise)
const simple$ = of('Hola Triage');
simple$.subscribe(valor => console.log(valor));  // "Hola Triage"

// Observable que emite un array como valores individuales
const lista$ = from(['REGISTRADA', 'CLASIFICADA', 'ATENDIDA']);
lista$.subscribe(estado => console.log(estado));

// Observable que emite cada segundo (infinito hasta que cancelas)
const temporizador$ = interval(1000);
const suscripcion = temporizador$.subscribe(n => console.log(n));
setTimeout(() => suscripcion.unsubscribe(), 5000);  // Cancela a los 5 segundos
```

> **Convención importante:** En Angular se usa el sufijo `$` para nombrar variables que son Observables. `solicitudes$`, `usuario$`, `error$`. Esto te ayuda a saber de un vistazo qué es reactivo.

### 13.2 Operadores RxJS Esenciales

Los operadores de RxJS son funciones que transforman, filtran o combinan Observables. Se usan dentro del método `pipe()`.

```typescript
import { map, filter, catchError, tap, switchMap, forkJoin, debounceTime, distinctUntilChanged } from 'rxjs/operators';
import { EMPTY, throwError } from 'rxjs';
```

**`map` — Transformar valores** (como `Stream.map()` en Java)

```typescript
// El backend devuelve Page<Solicitud>, pero solo queremos el array
this.solicitudService.obtenerSolicitudes().pipe(
  map(pagina => pagina.content)   // Transforma Page<Solicitud> → Solicitud[]
).subscribe(solicitudes => {
  this.solicitudes.set(solicitudes);
});
```

**`filter` — Filtrar valores**

```typescript
this.solicitudService.obtenerSolicitudes().pipe(
  map(p => p.content),
  // Solo queremos solicitudes de alta prioridad
  map(lista => lista.filter(s => s.prioridad === Prioridad.ALTA))
).subscribe(/* ... */);
```

**`tap` — Efectos secundarios sin modificar el valor**

```typescript
this.solicitudService.obtenerSolicitudes().pipe(
  tap(() => this.cargando.set(true)),    // Activa loading ANTES de la respuesta
  tap(p => console.log('Respuesta raw:', p)),  // Debug sin modificar el flujo
).subscribe(/* ... */);
```

**`catchError` — Capturar errores**

```typescript
import { catchError, EMPTY } from 'rxjs';

this.solicitudService.obtenerSolicitudes().pipe(
  catchError(error => {
    this.error.set('No se pudieron cargar las solicitudes.');
    console.error(error);
    return EMPTY;  // Termina el Observable sin emitir más valores
  })
).subscribe(/* ... */);
```

**`switchMap` — Cancelar petición anterior y hacer una nueva**

```typescript
// Caso de uso: buscador en tiempo real
// Si el usuario escribe rápido, cancela las búsquedas anteriores y solo hace la última

this.termino$.pipe(
  debounceTime(300),           // Espera 300ms de pausa antes de emitir
  distinctUntilChanged(),      // Solo emite si el valor cambió
  switchMap(termino =>         // Cancela la búsqueda anterior, inicia nueva
    this.solicitudService.buscarPorTermino(termino)
  )
).subscribe(resultados => this.solicitudes.set(resultados));
```

**`forkJoin` — Esperar múltiples Observables** (como `Promise.all()`)

```typescript
// Cargar solicitudes Y usuario actual en PARALELO
forkJoin({
  solicitudes: this.solicitudService.obtenerTodas(),
  usuario: this.authService.obtenerPerfil()
}).subscribe(({ solicitudes, usuario }) => {
  this.solicitudes.set(solicitudes);
  this.usuario.set(usuario);
});
```

### 13.3 AsyncPipe: El Mejor Amigo del Template

El `AsyncPipe` te permite suscribirte a un Observable directamente en el template, **sin llamar a `.subscribe()` manualmente**. Se desuscribe automáticamente cuando el componente se destruye, evitando memory leaks.

```typescript
@Component({
  standalone: true,
  imports: [AsyncPipe, CommonModule],
  template: `
    <!-- El async pipe maneja todo: subscribe, unsubscribe, loading states -->
    @if (solicitudes$ | async; as solicitudes) {
      @for (s of solicitudes; track s.id) {
        <div>{{ s.descripcion }}</div>
      }
    } @else {
      <p>Cargando...</p>
    }
  `
})
export class ListaComponent {
  // Observable público — el template lo consume con async pipe
  solicitudes$: Observable<Solicitud[]>;

  constructor() {
    this.solicitudes$ = inject(SolicitudService).obtenerTodas();
  }
}
```

### 13.4 Evitar Memory Leaks: Cancelar Suscripciones

Si usas `.subscribe()` manualmente, **debes cancelar la suscripción** en `ngOnDestroy`.

```typescript
import { Component, OnInit, OnDestroy, inject } from '@angular/core';
import { Subject, takeUntil } from 'rxjs';

@Component({ /* ... */ })
export class MiComponente implements OnInit, OnDestroy {
  // Subject que emite cuando el componente se destruye
  private readonly destroy$ = new Subject<void>();

  ngOnInit(): void {
    this.solicitudService.obtenerTodas().pipe(
      takeUntil(this.destroy$)  // Se desuscribe automáticamente cuando destroy$ emite
    ).subscribe(solicitudes => {
      this.solicitudes.set(solicitudes);
    });
  }

  ngOnDestroy(): void {
    this.destroy$.next();   // Emite una señal
    this.destroy$.complete(); // Cierra el Subject
  }
}
```

---

## 14. Reactive Forms: Formularios Robustos

Angular tiene dos formas de manejar formularios:

| Característica | Template-Driven Forms | Reactive Forms |
|----------------|-----------------------|----------------|
| Configuración | En el HTML con ngModel | En el TypeScript |
| Validaciones | Directivas HTML | Código TypeScript |
| Testing | Difícil | Fácil |
| Escalabilidad | Para formularios simples | Para formularios complejos |
| Recomendado para el Triage | No | ✅ Sí |

### 14.1 Configurar Reactive Forms

```typescript
import { ReactiveFormsModule, FormGroup, FormControl, Validators, FormBuilder } from '@angular/forms';
```

### 14.2 Formulario de Crear Solicitud

> **Contexto:** Vamos a implementar el formulario para que un estudiante registre una nueva solicitud académica (RF-01). El formulario debe capturar la descripción, tipo de solicitud y canal de origen.

```typescript
// src/app/features/solicitudes/crear-solicitud/crear-solicitud.component.ts

import { Component, inject } from '@angular/core';
import { ReactiveFormsModule, FormBuilder, Validators, AbstractControl } from '@angular/forms';
import { Router } from '@angular/router';
import { CommonModule } from '@angular/common';
import { SolicitudService, CrearSolicitudRequest } from '../../../core/services/solicitud.service';
import { TipoSolicitud } from '../../../core/models/solicitud.model';

@Component({
  selector: 'app-crear-solicitud',
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule],
  templateUrl: './crear-solicitud.component.html',
})
export class CrearSolicitudComponent {

  // FormBuilder simplifica la creación de FormGroups
  private readonly fb = inject(FormBuilder);
  private readonly solicitudService = inject(SolicitudService);
  private readonly router = inject(Router);

  // Opciones para los selects
  readonly tiposSolicitud = Object.values(TipoSolicitud);
  readonly canalesOrigen = ['PRESENCIAL', 'CORREO', 'SAC', 'TELEFONICO'];

  enviando = false;
  errorEnvio: string | null = null;

  // Definir el formulario con sus controles y validaciones
  formulario = this.fb.group({
    descripcion: ['', [
      Validators.required,
      Validators.minLength(20),
      Validators.maxLength(500),
    ]],
    tipoSolicitud: ['', [
      Validators.required,
    ]],
    canalOrigen: ['PRESENCIAL', [
      Validators.required,
    ]],
  });

  // Getters para acceder fácilmente a los controles en el template
  get descripcionControl() { return this.formulario.get('descripcion')!; }
  get tipoSolicitudControl() { return this.formulario.get('tipoSolicitud')!; }
  get canalOrigenControl() { return this.formulario.get('canalOrigen')!; }

  /** Verifica si un control tiene un error específico y fue tocado */
  tieneError(nombreControl: string, tipoError: string): boolean {
    const control = this.formulario.get(nombreControl);
    return !!(control?.hasError(tipoError) && (control.dirty || control.touched));
  }

  /** Enviar el formulario */
  onSubmit(): void {
    // Marcar todos los controles como tocados para mostrar errores
    if (this.formulario.invalid) {
      this.formulario.markAllAsTouched();
      return;
    }

    this.enviando = true;
    this.errorEnvio = null;

    const request: CrearSolicitudRequest = {
      descripcion: this.formulario.value.descripcion!,
      tipoSolicitud: this.formulario.value.tipoSolicitud!,
      canalOrigen: this.formulario.value.canalOrigen!,
    };

    this.solicitudService.crear(request).subscribe({
      next: (solicitudCreada) => {
        this.enviando = false;
        // Navegar al detalle de la solicitud recién creada
        this.router.navigate(['/solicitudes', solicitudCreada.id]);
      },
      error: (err) => {
        this.enviando = false;
        this.errorEnvio = 'No se pudo crear la solicitud. Intenta de nuevo.';
        console.error(err);
      }
    });
  }
}
```

**Template del formulario:**

```html
<!-- src/app/features/solicitudes/crear-solicitud/crear-solicitud.component.html -->

<div class="formulario-container">
  <h2>Nueva Solicitud Académica</h2>

  @if (errorEnvio) {
    <div class="alert alert-error">
      {{ errorEnvio }}
    </div>
  }

  <!-- novalidate: deshabilita la validación nativa del navegador -->
  <form [formGroup]="formulario" (ngSubmit)="onSubmit()" novalidate>

    <!-- Descripción -->
    <div class="form-group">
      <label for="descripcion">Descripción de la solicitud *</label>
      <textarea
        id="descripcion"
        formControlName="descripcion"
        rows="4"
        placeholder="Describe detalladamente tu solicitud (mínimo 20 caracteres)..."
        [class.campo-invalido]="tieneError('descripcion', 'required') || tieneError('descripcion', 'minlength')"
      ></textarea>

      <!-- Mensajes de error por tipo -->
      @if (tieneError('descripcion', 'required')) {
        <span class="error-msg">La descripción es obligatoria.</span>
      }
      @if (tieneError('descripcion', 'minlength')) {
        <span class="error-msg">
          La descripción debe tener al menos 20 caracteres.
          Tienes {{ descripcionControl.value?.length ?? 0 }}/20.
        </span>
      }
      @if (tieneError('descripcion', 'maxlength')) {
        <span class="error-msg">La descripción no puede superar 500 caracteres.</span>
      }
    </div>

    <!-- Tipo de Solicitud -->
    <div class="form-group">
      <label for="tipoSolicitud">Tipo de Solicitud *</label>
      <select id="tipoSolicitud" formControlName="tipoSolicitud"
              [class.campo-invalido]="tieneError('tipoSolicitud', 'required')">
        <option value="">-- Selecciona un tipo --</option>
        @for (tipo of tiposSolicitud; track tipo) {
          <option [value]="tipo">{{ tipo | lowercase | titlecase }}</option>
        }
      </select>
      @if (tieneError('tipoSolicitud', 'required')) {
        <span class="error-msg">Debes seleccionar el tipo de solicitud.</span>
      }
    </div>

    <!-- Canal de Origen -->
    <div class="form-group">
      <label>Canal de Origen *</label>
      <div class="radio-group">
        @for (canal of canalesOrigen; track canal) {
          <label class="radio-label">
            <input
              type="radio"
              formControlName="canalOrigen"
              [value]="canal"
            >
            {{ canal }}
          </label>
        }
      </div>
    </div>

    <!-- Botones -->
    <div class="form-actions">
      <button type="button" routerLink="/solicitudes" class="btn btn-secondary">
        Cancelar
      </button>
      <button
        type="submit"
        class="btn btn-primary"
        [disabled]="enviando">
        @if (enviando) {
          Enviando...
        } @else {
          Registrar Solicitud
        }
      </button>
    </div>

  </form>
</div>
```

### 14.3 Validadores Integrados

| Validador | Descripción | Uso |
|-----------|-------------|-----|
| `Validators.required` | Campo obligatorio | `Validators.required` |
| `Validators.minLength(n)` | Mínimo n caracteres | `Validators.minLength(20)` |
| `Validators.maxLength(n)` | Máximo n caracteres | `Validators.maxLength(500)` |
| `Validators.email` | Formato de email válido | `Validators.email` |
| `Validators.pattern(regex)` | Patrón regex | `Validators.pattern(/^\d{10}$/)` |
| `Validators.min(n)` | Valor numérico mínimo | `Validators.min(0)` |
| `Validators.max(n)` | Valor numérico máximo | `Validators.max(100)` |

### 14.4 Validador Personalizado

```typescript
// src/app/shared/validators/no-palabras-prohibidas.validator.ts

import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Un validador es simplemente una función que retorna null (válido) o un objeto de error
export function noPalabrasProhibidas(palabras: string[]): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const valor = control.value as string;
    if (!valor) return null;  // Si está vacío, deja que 'required' lo maneje

    const encontrada = palabras.find(p => valor.toLowerCase().includes(p.toLowerCase()));
    if (encontrada) {
      // Retorna un objeto — la clave es el nombre del error, el valor es info extra
      return { palabraProhibida: { palabra: encontrada } };
    }

    return null;  // null = válido
  };
}

// Uso en el componente:
descripcion: ['', [
  Validators.required,
  noPalabrasProhibidas(['spam', 'prueba', 'test']),
]],
```

---

## 15. Routing: Navegación entre Vistas

El **Router** de Angular permite crear aplicaciones de una sola página (SPA) con múltiples vistas sin recargar el navegador.

### 15.1 Configurar Rutas del Triage

```typescript
// src/app/app.routes.ts

import { Routes } from '@angular/router';

export const routes: Routes = [
  // Redirección por defecto
  {
    path: '',
    redirectTo: '/solicitudes',
    pathMatch: 'full'
  },

  // Rutas de autenticación — carga inmediata
  {
    path: 'login',
    loadComponent: () =>
      import('./features/auth/login/login.component')
        .then(m => m.LoginComponent)
  },
  {
    path: 'registro',
    loadComponent: () =>
      import('./features/auth/registro/registro.component')
        .then(m => m.RegistroComponent)
  },

  // Rutas protegidas — requieren autenticación
  {
    path: 'solicitudes',
    // canActivate: [authGuard],   // Lo implementamos en la Guía 09
    children: [
      {
        path: '',
        loadComponent: () =>
          import('./features/solicitudes/lista-solicitudes/lista-solicitudes.component')
            .then(m => m.ListaSolicitudesComponent)
      },
      {
        path: 'nueva',
        loadComponent: () =>
          import('./features/solicitudes/crear-solicitud/crear-solicitud.component')
            .then(m => m.CrearSolicitudComponent)
      },
      {
        path: ':id',
        loadComponent: () =>
          import('./features/solicitudes/detalle-solicitud/detalle-solicitud.component')
            .then(m => m.DetalleSolicitudComponent)
      },
    ]
  },

  // Ruta 404
  {
    path: '**',
    loadComponent: () =>
      import('./shared/components/not-found/not-found.component')
        .then(m => m.NotFoundComponent)
  }
];
```

### 15.2 RouterOutlet: Dónde se Renderiza el Componente

```html
<!-- src/app/app.component.html -->
<nav class="navbar">
  <a routerLink="/solicitudes" routerLinkActive="active">Solicitudes</a>
  <a routerLink="/solicitudes/nueva" routerLinkActive="active">Nueva Solicitud</a>
</nav>

<!-- Angular renderiza aquí el componente de la ruta activa -->
<main>
  <router-outlet />
</main>
```

### 15.3 Navegar Programáticamente

```typescript
import { Router, ActivatedRoute } from '@angular/router';

@Component({ /* ... */ })
export class DetalleComponent implements OnInit {
  private readonly router = inject(Router);
  private readonly route = inject(ActivatedRoute);

  solicitudId!: number;

  ngOnInit(): void {
    // Obtener el parámetro :id de la URL
    this.solicitudId = Number(this.route.snapshot.paramMap.get('id'));
  }

  irALista(): void {
    // Navegar programáticamente
    this.router.navigate(['/solicitudes']);
  }

  irAEditar(): void {
    // Navegar con parámetros
    this.router.navigate(['/solicitudes', this.solicitudId, 'editar']);
  }

  irConQuery(): void {
    // Navegar con query params: /solicitudes?estado=REGISTRADA&pagina=2
    this.router.navigate(['/solicitudes'], {
      queryParams: { estado: 'REGISTRADA', pagina: 2 }
    });
  }
}
```

### 15.4 Functional Guards: Proteger Rutas

Los **guards** son funciones que deciden si una ruta puede activarse. Los functional guards (Angular 15+) son mucho más simples que los guards de clase.

```typescript
// src/app/core/guards/auth.guard.ts

import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

// Un functional guard es simplemente una función
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  // Si el usuario está autenticado, permite el acceso
  if (authService.estaAutenticado()) {
    return true;
  }

  // Si no, redirige al login guardando la URL original
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// Guard para roles específicos
export const rolGuard = (rolesPermitidos: string[]): CanActivateFn => {
  return (route, state) => {
    const authService = inject(AuthService);
    const router = inject(Router);

    const rol = authService.obtenerRolActual();
    if (rolesPermitidos.includes(rol)) {
      return true;
    }

    return router.createUrlTree(['/acceso-denegado']);
  };
};
```

**Usar los guards en las rutas:**

```typescript
// src/app/app.routes.ts
import { authGuard, rolGuard } from './core/guards/auth.guard';

export const routes: Routes = [
  {
    path: 'solicitudes',
    canActivate: [authGuard],   // Solo usuarios autenticados
    children: [/* ... */]
  },
  {
    path: 'admin',
    canActivate: [authGuard, rolGuard(['ADMIN'])],  // Solo ADMIN
    loadComponent: () => import('./features/admin/admin.component').then(m => m.AdminComponent)
  }
];
```

---

## 16. Estructura de Carpetas Profesional para el Triage

Esta es la estructura que seguirás en el proyecto. Cada carpeta tiene una responsabilidad clara:

```
triage-frontend/
├── src/
│   ├── app/
│   │   │
│   │   ├── core/                          ← Singleton services, guards, interceptors
│   │   │   ├── guards/
│   │   │   │   ├── auth.guard.ts
│   │   │   │   └── rol.guard.ts
│   │   │   ├── interceptors/
│   │   │   │   ├── auth.interceptor.ts    ← Añade JWT a cada petición
│   │   │   │   └── error.interceptor.ts   ← Manejo global de errores HTTP
│   │   │   ├── models/                    ← Interfaces TypeScript (DTOs)
│   │   │   │   ├── solicitud.model.ts
│   │   │   │   ├── usuario.model.ts
│   │   │   │   ├── historial.model.ts
│   │   │   │   └── page.model.ts
│   │   │   └── services/                  ← Servicios singleton
│   │   │       ├── auth.service.ts
│   │   │       ├── solicitud.service.ts
│   │   │       └── usuario.service.ts
│   │   │
│   │   ├── features/                      ← Un directorio por feature/módulo de negocio
│   │   │   ├── auth/
│   │   │   │   ├── login/
│   │   │   │   │   ├── login.component.ts
│   │   │   │   │   ├── login.component.html
│   │   │   │   │   └── login.component.scss
│   │   │   │   └── registro/
│   │   │   │       ├── registro.component.ts
│   │   │   │       └── ...
│   │   │   │
│   │   │   └── solicitudes/
│   │   │       ├── lista-solicitudes/
│   │   │       │   ├── lista-solicitudes.component.ts
│   │   │       │   ├── lista-solicitudes.component.html
│   │   │       │   └── lista-solicitudes.component.scss
│   │   │       ├── crear-solicitud/
│   │   │       │   └── ...
│   │   │       ├── detalle-solicitud/
│   │   │       │   └── ...
│   │   │       └── historial-solicitud/
│   │   │           └── ...
│   │   │
│   │   ├── shared/                        ← Componentes, pipes y directivas reutilizables
│   │   │   ├── components/
│   │   │   │   ├── loading-spinner/
│   │   │   │   ├── empty-state/
│   │   │   │   ├── badge-estado/
│   │   │   │   ├── badge-prioridad/
│   │   │   │   └── not-found/
│   │   │   ├── pipes/
│   │   │   │   ├── estado-label.pipe.ts
│   │   │   │   └── prioridad-label.pipe.ts
│   │   │   └── validators/
│   │   │       └── no-palabras-prohibidas.validator.ts
│   │   │
│   │   ├── app.component.ts
│   │   ├── app.component.html
│   │   ├── app.component.scss
│   │   ├── app.config.ts
│   │   └── app.routes.ts
│   │
│   ├── environments/
│   │   ├── environment.ts              ← URL del backend en desarrollo
│   │   └── environment.prod.ts         ← URL del backend en producción
│   │
│   ├── assets/
│   │   ├── images/
│   │   └── icons/
│   │
│   ├── index.html
│   ├── main.ts
│   └── styles.scss                     ← Variables SCSS globales, reset CSS
│
├── angular.json
├── package.json
└── tsconfig.json
```

### Reglas de la Estructura

- **`core/`** — Solo servicios que deben ser Singleton en toda la app. Un servicio de `core` NO importa cosas de `features`.
- **`features/`** — Cada feature es independiente. Un feature NO importa de otro feature directamente.
- **`shared/`** — Componentes reutilizables que no dependen de ningún feature. Un `shared` NO importa de `core` ni de `features`.

Esta separación te permite trabajar en equipo sin conflictos y escalar el proyecto fácilmente.

### Modelos del Triage

```typescript
// src/app/core/models/solicitud.model.ts

export interface Solicitud {
  id: number;
  descripcion: string;
  estado: EstadoSolicitud;
  prioridad: Prioridad;
  tipoSolicitud: TipoSolicitud;
  canalOrigen: CanalOrigen;
  justificacionPrioridad?: string;
  fechaRegistro: string;
  fechaActualizacion: string;
  solicitante: UsuarioResumen;
  responsable?: UsuarioResumen;
  historial?: HistorialItem[];
}

export interface UsuarioResumen {
  id: number;
  nombre: string;
  correo: string;
  rol: Rol;
}

export interface HistorialItem {
  id: number;
  fechaAccion: string;
  accion: string;
  observacion?: string;
  usuarioResponsable: UsuarioResumen;
}

export enum EstadoSolicitud {
  REGISTRADA   = 'REGISTRADA',
  CLASIFICADA  = 'CLASIFICADA',
  EN_ATENCION  = 'EN_ATENCION',
  ATENDIDA     = 'ATENDIDA',
  CERRADA      = 'CERRADA',
}

export enum Prioridad {
  BAJA    = 'BAJA',
  MEDIA   = 'MEDIA',
  ALTA    = 'ALTA',
  CRITICA = 'CRITICA',
}

export enum TipoSolicitud {
  REGISTRO_ASIGNATURAS    = 'REGISTRO_ASIGNATURAS',
  HOMOLOGACION            = 'HOMOLOGACION',
  CANCELACION_ASIGNATURAS = 'CANCELACION_ASIGNATURAS',
  SOLICITUD_CUPOS         = 'SOLICITUD_CUPOS',
  CONSULTA_ACADEMICA      = 'CONSULTA_ACADEMICA',
}

export enum CanalOrigen {
  CSU        = 'CSU',
  CORREO     = 'CORREO',
  SAC        = 'SAC',
  TELEFONICO = 'TELEFONICO',
}

export enum Rol {
  ESTUDIANTE  = 'ESTUDIANTE',
  FUNCIONARIO = 'FUNCIONARIO',
  ADMIN       = 'ADMIN',
}
```

```typescript
// src/app/core/models/page.model.ts

export interface Page<T> {
  content: T[];
  totalElements: number;
  totalPages: number;
  number: number;
  size: number;
  numberOfElements: number;
  first: boolean;
  last: boolean;
  empty: boolean;
}
```

---

## 17. Ejercicios Prácticos

Los siguientes retos te ayudarán a consolidar lo aprendido en esta guía. Están ordenados de menor a mayor dificultad.

### Reto 1 — Componente Badge de Estado (Básico)

Crea un componente `BadgeEstadoComponent` en `shared/components/badge-estado/` que:

- Reciba un `@Input({ required: true }) estado: EstadoSolicitud`.
- Muestre el texto del estado con el emoji correspondiente.
- Aplique una clase CSS diferente según el estado (usa `@switch` y `[ngClass]`).
- Sea reutilizable: debe poder usarse en cualquier parte del app solo importándolo.

### Reto 2 — Pipe de Prioridad (Básico-Intermedio)

Crea un `PrioridadLabelPipe` en `shared/pipes/` que:

- Reciba un valor de tipo `Prioridad | null | undefined`.
- Devuelva una etiqueta en español con emoji: `CRITICA` → `🔴 Crítica`, `ALTA` → `🟠 Alta`, etc.
- Si el valor es `null` o `undefined`, devuelva `'Sin prioridad'`.

### Reto 3 — Formulario de Clasificación (Intermedio)

Crea un componente `ClasificarSolicitudComponent` con un Reactive Form que permita:

- Seleccionar el `TipoSolicitud` de la solicitud.
- Seleccionar la `Prioridad`.
- Ingresar una `justificacion` obligatoria (mínimo 10 caracteres).
- Mostrar errores de validación por campo.
- Al hacer submit, mostrar en consola los valores del formulario (simulando el envío).

### Reto 4 — Lista con Filtros y Paginación (Intermedio-Avanzado)

Crea un componente `ListaAvanzadaSolicitudesComponent` que:

- Muestre una lista de solicitudes hardcodeadas (al menos 10 items).
- Tenga un botón de filtro por cada `EstadoSolicitud`.
- Implemente paginación simple: mostrar 3 solicitudes por página con botones "Anterior" y "Siguiente".
- Use `computed()` para calcular las solicitudes filtradas y paginadas.
- El contador de total debe actualizarse según el filtro activo.

### Reto 5 — Servicio de Auth Simulado (Avanzado)

Crea un `AuthService` en `core/services/` que:

- Tenga un `signal<boolean>(false)` para indicar si el usuario está autenticado.
- Tenga un método `login(correo: string, contraseña: string): void` que, si las credenciales son `admin@uq.edu.co` / `admin123`, setee el signal en `true`.
- Tenga un método `logout(): void` que setee el signal en `false`.
- Crea un `authGuard` funcional que use este servicio para proteger la ruta `/solicitudes`.
- Crea un `LoginComponent` con un formulario reactivo que use este servicio.

---

## 18. Errores Comunes y Troubleshooting

### ❌ Error: `NullInjectorError: No provider for HttpClient`

**Causa:** No configuraste `provideHttpClient()` en `app.config.ts`.

**Solución:**
```typescript
// app.config.ts
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),  // ← Agregar esto
  ]
};
```

---

### ❌ Error: `Can't bind to 'formGroup' since it isn't a known property`

**Causa:** No importaste `ReactiveFormsModule` en el componente.

**Solución:**
```typescript
@Component({
  standalone: true,
  imports: [ReactiveFormsModule],  // ← Agregar esto
  // ...
})
```

---

### ❌ Error: `RouterLink is not a known attribute`

**Causa:** No importaste `RouterLink` en el componente standalone.

**Solución:**
```typescript
import { RouterLink, RouterLinkActive } from '@angular/router';

@Component({
  standalone: true,
  imports: [RouterLink, RouterLinkActive],  // ← Agregar esto
  // ...
})
```

---

### ❌ El formulario no muestra errores aunque el campo esté vacío

**Causa:** Los errores solo se muestran cuando el campo está `touched` (el usuario lo tocó y salió) o `dirty` (el usuario escribió algo). Si nunca fue interactuado, no se muestra.

**Solución al hacer submit:**
```typescript
onSubmit(): void {
  if (this.formulario.invalid) {
    this.formulario.markAllAsTouched();  // ← Fuerza mostrar todos los errores
    return;
  }
  // ...
}
```

---

### ❌ Memory Leak: el componente sigue recibiendo datos después de ser destruido

**Causa:** Suscribiste a un Observable pero no cancelaste la suscripción en `ngOnDestroy`.

**Solución:**
```typescript
private readonly destroy$ = new Subject<void>();

ngOnInit(): void {
  this.miServicio.datos$.pipe(
    takeUntil(this.destroy$)  // ← Esto lo soluciona
  ).subscribe(/* ... */);
}

ngOnDestroy(): void {
  this.destroy$.next();
  this.destroy$.complete();
}
```

---

### ❌ `ExpressionChangedAfterItHasBeenCheckedError`

**Causa:** Estás modificando el estado del componente desde el ciclo de vida `ngAfterViewInit`, después de que la detección de cambios ya terminó.

**Solución:** Usa `setTimeout(() => { ... }, 0)` o `queueMicrotask()` para diferir el cambio, o mejor aún, usa Signals que manejan esto automáticamente.

---

### ❌ El lazy loading no funciona: el componente no se carga

**Causa:** El `loadComponent` usa una ruta de importación incorrecta o el componente no es el export default.

**Solución:** Verifica que la ruta sea correcta y que el componente sea exportado correctamente:
```typescript
// En la ruta:
loadComponent: () =>
  import('./features/solicitudes/lista-solicitudes/lista-solicitudes.component')
  .then(m => m.ListaSolicitudesComponent)   // ← El nombre debe coincidir exactamente

// En el archivo del componente:
export class ListaSolicitudesComponent { }  // ← debe estar exportado
```

---

### ❌ CORS al consumir el backend local

**Causa:** Angular corre en `localhost:4200` y el backend en `localhost:8080`. Los navegadores bloquean esto por seguridad (CORS).

**Solución (en desarrollo):** Configura un proxy en Angular:

```json
// proxy.conf.json — en la raíz del proyecto
{
  "/api": {
    "target": "http://localhost:8080",
    "secure": false,
    "changeOrigin": true
  }
}
```

```json
// angular.json — en la sección "serve" > "options"
{
  "serve": {
    "builder": "@angular-devkit/build-angular:dev-server",
    "options": {
      "proxyConfig": "proxy.conf.json"
    }
  }
}
```

```typescript
// environment.ts — apunta al proxy, no directamente al backend
export const environment = {
  apiUrl: '/api/v1',   // ← Sin http://localhost:8080
};
```

---

## 19. Resumen y Cheat Sheet

### Anatomía del Decorador @Component

```typescript
@Component({
  selector: 'app-mi-componente',     // Tag HTML
  standalone: true,                   // Componente autónomo (Angular 17+)
  imports: [CommonModule, RouterLink], // Dependencias del template
  templateUrl: './mi.component.html', // Archivo de template externo
  // template: `<div>inline</div>`,  // O template inline
  styleUrl: './mi.component.scss',    // Archivo de estilos
})
export class MiComponente implements OnInit, OnDestroy {
  @Input() dato!: string;            // Recibe datos del padre
  @Output() evento = new EventEmitter<string>();  // Emite al padre

  estado = signal('');              // Estado reactivo
  derivado = computed(() => '');    // Computed del signal

  ngOnInit() { /* inicialización */ }
  ngOnDestroy() { /* limpieza */ }
}
```

### Control Flow

```html
@if (condicion) { ... } @else if (otra) { ... } @else { ... }
@for (item of lista; track item.id) { ... } @empty { ... }
@switch (valor) { @case (x) { ... } @default { ... } }
```

### Bindings

```html
{{ expresion }}          <!-- Interpolación -->
[propiedad]="expr"       <!-- Property Binding -->
(evento)="metodo()"      <!-- Event Binding -->
[(ngModel)]="campo"      <!-- Two-Way Binding -->
```

### Reactive Form

```typescript
formulario = this.fb.group({
  campo: ['valorInicial', [Validators.required, Validators.minLength(5)]],
});
// Acceder: this.formulario.get('campo')?.value
// Errors: this.formulario.get('campo')?.hasError('required')
// Submit: this.formulario.markAllAsTouched()
```

### Observable rápido

```typescript
this.servicio.obtener().pipe(
  map(data => data.content),
  catchError(err => { this.error.set('Ups'); return EMPTY; }),
  takeUntil(this.destroy$)
).subscribe(items => this.items.set(items));
```

### inject() en 2026

```typescript
private readonly http    = inject(HttpClient);
private readonly router  = inject(Router);
private readonly service = inject(MiService);
```

### Functional Guard

```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn() ? true : router.createUrlTree(['/login']);
};
```

### Lazy Loading

```typescript
{
  path: 'ruta',
  loadComponent: () =>
    import('./features/mi/mi.component').then(m => m.MiComponent)
}
```

---

## 20. Referencias y Recursos Adicionales

- **Documentación oficial de Angular:** https://angular.dev (La nueva documentación oficial, actualizada para Angular 17+)
- **Angular DevTools:** Extensión de Chrome para depurar aplicaciones Angular en el navegador.
- **Tour of Heroes:** Tutorial oficial de Angular — https://angular.dev/tutorials/learn-angular
- **RxJS:** https://rxjs.dev — Documentación de todos los operadores
- **TypeScript Deep Dive** (libro gratuito): https://basarat.gitbook.io/typescript/
- **Angular Material:** Librería oficial de componentes UI — https://material.angular.io
- **PrimeNG:** Alternativa popular de componentes UI — https://primeng.org

---

> **Autor:** José Alfredo Ramírez Espinosa  
> **Materia:** Programación Avanzada  
> **Programa:** Ingeniería de Sistemas y Computación  
> **Universidad del Quindío** | Armenia, Colombia  
> **Año:** 2026  
> *Material de uso académico. Todos los derechos reservados al autor.*
