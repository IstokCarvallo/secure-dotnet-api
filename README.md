# Secure .NET API

Referencia de seguridad para APIs .NET, con autenticación, autorización, manejo seguro de credenciales, validación y protección de endpoints.

Este repositorio documenta una arquitectura de referencia para construir APIs seguras con ASP.NET Core. El foco está en decisiones prácticas y reutilizables: reducir superficie de ataque, proteger credenciales y datos, controlar acceso, validar entradas, observar el sistema sin filtrar información sensible y establecer límites operacionales razonables.

## Objetivos

- Diseñar autenticación y autorización de forma explícita.
- Separar identidad, permisos y lógica de negocio.
- Evitar secretos embebidos en código o configuración versionada.
- Validar toda entrada externa antes de procesarla.
- Aplicar controles de acceso con mínimo privilegio.
- Incorporar protección frente a abuso, errores y exposición accidental de datos.
- Mantener logging y observabilidad compatibles con requisitos de seguridad.
- Servir como referencia reutilizable para APIs empresariales .NET.

## Principios

1. **Deny by default**  
   Todo endpoint sensible requiere autorización explícita.

2. **Least privilege**  
   Usuarios, servicios y procesos reciben únicamente los permisos necesarios.

3. **Secrets fuera del repositorio**  
   Credenciales, claves y connection strings se suministran mediante variables de entorno, secret stores o servicios administrados.

4. **Validar antes de confiar**  
   Toda entrada externa debe tratarse como no confiable hasta ser validada.

5. **Errores sin filtración**  
   Las respuestas de error no deben revelar stack traces, credenciales, rutas internas ni detalles innecesarios de infraestructura.

6. **Observabilidad segura**  
   Los logs deben permitir investigar incidentes sin registrar tokens, contraseñas ni datos sensibles.

## Arquitectura de seguridad propuesta

```text
Client
  │
  ▼
HTTPS
  │
  ▼
ASP.NET Core API
  │
  ├── Rate limiting
  ├── CORS
  ├── Authentication
  ├── Authorization
  ├── Validation
  ├── Error handling
  └── Secure logging
  │
  ▼
Application
  │
  ▼
Domain
  │
  ▼
Infrastructure
  ├── Database
  ├── Identity provider
  ├── Secret store
  └── External services
```

## Controles incluidos en la referencia

### Autenticación

La referencia considerará autenticación basada en tokens para escenarios API:

- access tokens de vida limitada;
- refresh tokens cuando el escenario los requiera;
- validación de issuer, audience, firma y expiración;
- rechazo explícito de tokens inválidos o expirados;
- separación entre autenticación e información de autorización.

### Autorización

Los permisos no deben depender únicamente de que un usuario esté autenticado.

Se documentarán:

- roles cuando representen responsabilidades organizacionales estables;
- claims para atributos de identidad o contexto;
- policies para reglas de autorización expresivas y reutilizables;
- autorización por recurso cuando el acceso dependa de la entidad concreta solicitada.

### Manejo de secretos

No se versionarán secretos reales.

Ejemplo de configuración versionable:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": ""
  },
  "Jwt": {
    "Issuer": "",
    "Audience": "",
    "SigningKey": ""
  }
}
```

Los valores reales deberán obtenerse desde mecanismos externos al repositorio.

### Validación

La validación debe ocurrir en los límites del sistema.

Se cubrirán:

- formato;
- longitud;
- rangos;
- valores permitidos;
- identificadores;
- reglas de negocio cuando corresponda;
- límites de tamaño para payloads y archivos.

### Manejo de errores

La API utilizará respuestas consistentes y no revelará detalles internos.

```text
Exception
    │
    ▼
Global Exception Handler
    │
    ├── Log interno seguro
    │
    └── Respuesta HTTP controlada
```

### Rate limiting

Se incorporarán límites razonables para reducir abuso y proteger recursos costosos. La política dependerá del tipo de endpoint y no se aplicará una única regla indiscriminada a toda la API.

### CORS

CORS se configurará mediante orígenes explícitos. No se recomendará `AllowAnyOrigin` para escenarios autenticados de producción salvo justificación específica.

### Logging seguro

No registrar:

- contraseñas;
- access tokens;
- refresh tokens;
- claves API;
- secretos;
- connection strings;
- payloads completos cuando contengan datos sensibles.

Sí registrar, cuando corresponda:

- correlation ID;
- timestamp;
- endpoint;
- resultado;
- código HTTP;
- duración;
- identificadores no sensibles necesarios para diagnóstico.

## Estructura prevista

```text
secure-dotnet-api/
├── src/
│   ├── SecureApi.Api/
│   ├── SecureApi.Application/
│   ├── SecureApi.Domain/
│   └── SecureApi.Infrastructure/
│
├── tests/
│   ├── SecureApi.UnitTests/
│   └── SecureApi.IntegrationTests/
│
├── docs/
│   └── security-architecture.md
│
├── .gitignore
├── README.md
└── SecureApi.sln
```

La estructura se incorporará de forma incremental. No se agregarán capas, librerías o patrones sin una necesidad concreta.

## Alcance posterior

En una siguiente etapa se incorporarán ejemplos ejecutables de:

- JWT Bearer Authentication;
- authorization policies;
- refresh token rotation;
- rate limiting;
- global exception handling;
- request validation;
- secure configuration;
- integration tests de autenticación y autorización.

## Documentación

La arquitectura de seguridad se desarrolla en [`docs/security-architecture.md`](docs/security-architecture.md).

## Estado

Este repositorio está en construcción. La primera etapa está centrada en documentar los controles y decisiones de seguridad antes de incorporar código de referencia.

## Autor

**Istok Carvallo**  
Arquitectura de Software · Gestión de Proyectos TI · Desarrollo de Productos Digitales
