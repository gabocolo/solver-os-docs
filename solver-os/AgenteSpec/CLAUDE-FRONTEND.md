# Spec Agent Portal — Frontend (Angular 21)

Este archivo define las reglas, convenciones y contexto obligatorio para cualquier generacion de codigo asistida por IA en el repositorio frontend del Spec Agent Portal.

> **Regla fundamental**: Primero especificar, luego invocar, despues generar y validar.

---

## 1. Stack tecnologico

| Componente | Tecnologia |
|------------|------------|
| Framework | Angular 21 |
| Lenguaje | TypeScript (strict mode habilitado) |
| UI Components | Angular Material |
| Utilidades CSS | Tailwind CSS |
| Editor de codigo | Monaco Editor (ngx-monaco-editor) |
| Graficos | ngx-charts |

---

## 2. Arquitectura del proyecto

```
src/app/
  core/
    interceptors/      -- HTTP interceptors (auth, correlationId, error handling)
    guards/            -- Route guards por rol
    services/          -- Servicios globales (auth, WebSocket, notification)
    websocket/         -- Conexion WebSocket para eventos en tiempo real
  shared/
    components/        -- Componentes compartidos (layout, skeleton loaders, dialogs)
    pipes/             -- Pipes reutilizables
    directives/        -- Directivas compartidas
    models/            -- Interfaces y tipos compartidos
  features/
    projects/          -- Gestion de proyectos (lazy-loaded)
    specs/             -- Gestion de especificaciones L1/L2/L3 (lazy-loaded)
    reviews/           -- Flujo de revision y aprobacion (lazy-loaded)
    prompts/           -- Generacion y gestion de prompts (lazy-loaded)
    metrics/           -- Dashboard de metricas (lazy-loaded)
    skills/            -- Catalogo y gestion de skills (lazy-loaded)
  auth/
    login/             -- Componentes de login
    guards/            -- Guards de autenticacion y autorizacion
    services/          -- OAuth, JWT, token refresh
    models/            -- Interfaces de auth (roles, permisos)
```

### Reglas de arquitectura

- Cada feature module se carga con **lazy loading** — sin excepciones
- Los servicios globales viven en `core/` y se proveen en root
- Los componentes compartidos viven en `shared/` — no duplicar logica entre features
- La comunicacion con el backend es SOLO via `HttpClient` con interceptors

---

## 3. ADRs relevantes

| ADR | Nombre | Impacto en este repo |
|-----|--------|---------------------|
| ADR-001 | Ports & Adapters | Frontend consume los puertos del backend via HTTP — no accede a infraestructura directamente |
| ADR-003 | Trazabilidad | correlationId inyectado en cada request via HTTP interceptor |
| ADR-006 | DLP | Datos sinteticos siempre, no PII real en codigo ni tests |

---

## 4. Convenciones de codigo

### Standalone components

- **Todos los componentes son standalone** — no usar NgModules para componentes
- Importar dependencias directamente en el decorator `@Component`

### Angular Signals

- **Usar Signals para estado reactivo** — no usar BehaviorSubject para estado local del componente
- `signal()` para estado local
- `computed()` para estado derivado
- `effect()` para side effects reactivos

### Reactive Forms

- **Formularios reactivos obligatorios** — no usar template-driven forms
- Validadores tipados con `FormControl<T>`

### HTTP

- **HttpClient exclusivamente** — NO usar axios ni fetch directamente
- Endpoints definidos en archivos de environment
- Interceptors para auth (JWT), correlationId y error handling

### Lazy loading

- **Obligatorio para todos los feature modules**
- Cada feature tiene su propio archivo de rutas

### Naming

- **Componentes**: kebab-case para selector (`app-spec-editor`), PascalCase para clase (`SpecEditorComponent`)
- **Servicios**: PascalCase con sufijo `Service` (`SpecService`, `AuthService`)
- **Interfaces/Tipos**: PascalCase con prefijo `I` para interfaces de contrato (`ISpecResponse`)
- **Archivos**: kebab-case (`spec-editor.component.ts`, `spec.service.ts`)

---

## 5. Consumo de API

### Configuracion de endpoints

Todos los endpoints se definen en los archivos de environment:

```typescript
export const environment = {
  production: false,
  apiBaseUrl: 'http://localhost:5000/api',
  wsUrl: 'ws://localhost:5000/ws',
};
```

### Interceptors obligatorios

- **AuthInterceptor**: agrega token JWT en header `Authorization`
- **CorrelationIdInterceptor**: genera y propaga `X-Correlation-Id` en cada request (ADR-003)
- **ErrorInterceptor**: manejo centralizado de errores HTTP, muestra snackbar con mensaje

### Contrato de referencia

El frontend consume el contrato OpenAPI generado por el skill SK-003. Los tipos TypeScript se generan a partir de este contrato.

---

## 6. Reglas DLP y proteccion de datos — ADR-006

- **Nunca PII real** en codigo, tests, fixtures ni mocks
- **Datos sinteticos siempre** — usar datos generados para pruebas y desarrollo
- **No almacenar PII en localStorage/sessionStorage** sin cifrado
- **No loguear PII en console.log** — usar el servicio de logging con filtros
- **Tokens JWT**: almacenar en memoria o httpOnly cookies, nunca en localStorage sin proteccion

### Ejemplo de datos sinteticos en tests

```typescript
// CORRECTO
const mockUser = { id: 'synthetic-001', name: 'Test User', email: 'test@synthetic.local' };

// INCORRECTO — nunca hacer esto
const mockUser = { id: '12345', name: 'Juan Perez', email: 'juan.perez@empresa.com' };
```

---

## 7. Testing

### Unit tests

- Framework: **Jasmine/Karma**
- Datos sinteticos obligatorios
- Cubrir escenarios de exito y error definidos en la spec

### Atributos para E2E

- Usar **`data-testid`** en elementos interactivos para facilitar pruebas E2E
- No depender de clases CSS ni estructura del DOM para selectores de test

```html
<button data-testid="create-spec-button" (click)="onCreate()">
  Crear especificacion
</button>
```

### Estructura de tests

```typescript
describe('SpecEditorComponent', () => {
  it('should display spec content when loaded successfully', () => {
    // Given
    const mockSpec = { id: 'synthetic-spec-001', title: 'Test Spec', version: '1.0.0' };

    // When
    component.spec.set(mockSpec);
    fixture.detectChanges();

    // Then
    const title = fixture.debugElement.query(By.css('[data-testid="spec-title"]'));
    expect(title.nativeElement.textContent).toContain('Test Spec');
  });
});
```

---

## 8. Patrones de UI

### Componentes de Angular Material

- Usar el tema de Angular Material configurado para el proyecto
- **Tailwind CSS** para utilidades de layout y espaciado — no para reemplazar Material
- Combinar Material (componentes) + Tailwind (utilidades) sin conflictos

### UX obligatorio

- **Skeleton loaders** durante carga de datos — no spinners genericos
- **Snackbar** (MatSnackBar) para notificaciones de error y exito
- **ARIA labels** en todos los elementos interactivos para accesibilidad
- **Loading states** explicitos en botones durante operaciones async

### Ejemplo de skeleton loader

```html
@if (isLoading()) {
  <div class="animate-pulse space-y-4">
    <div class="h-4 bg-gray-200 rounded w-3/4"></div>
    <div class="h-4 bg-gray-200 rounded w-1/2"></div>
  </div>
} @else {
  <app-spec-detail [spec]="spec()" />
}
```

---

## 9. Referencia a wireframes

Los wireframes de la interfaz estan documentados en:

```
solver-os/AgenteSpec/specs/05-UI-WIREFRAMES.md
```

Cualquier componente nuevo debe alinearse con los wireframes aprobados. Si el wireframe no existe para un componente, debe crearse y aprobarse antes de implementar.

---

## 10. Ubicacion de especificaciones

Las especificaciones que gobiernan este repositorio se encuentran en:

```
solver-os/AgenteSpec/specs/
```

### Regla de invocacion

El prompt NO contiene las reglas de negocio. El prompt INVOCA la especificacion:

```
Invocacion: Aplica la especificacion [nombre] v[X.Y.Z]
```

Las reglas, dependencias autorizadas, criterios de aceptacion y salidas esperadas provienen EXCLUSIVAMENTE del spec versionado. No inventar componentes, servicios ni dependencias fuera del spec.

---

## 11. Quality gates antes de merge

- [ ] Spec versionada y aprobada (Gate 1)
- [ ] Tests unitarios pasan
- [ ] Lint sin errores (`ng lint`)
- [ ] DLP Scan — sin PII en codigo ni tests
- [ ] PR menor a 400 lineas
- [ ] Revision de senior obligatoria (Gate 3)
- [ ] data-testid en elementos interactivos nuevos
- [ ] ARIA labels en componentes interactivos
- [ ] Lazy loading en nuevos feature modules
- [ ] Datos sinteticos en todos los tests y mocks
