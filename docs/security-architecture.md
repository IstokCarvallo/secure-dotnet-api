# Arquitectura de seguridad

Este documento describe la arquitectura de seguridad propuesta para `secure-dotnet-api`.

El objetivo no es acumular controles, sino establecer una defensa coherente en capas, con responsabilidades claras y decisiones explícitas.

## Flujo principal

```text
Cliente
  │
  ▼
HTTPS
  │
  ▼
API Gateway / Reverse Proxy (opcional)
  │
  ▼
ASP.NET Core Pipeline
  │
  ├── Correlation ID
  ├── Rate Limiting
  ├── CORS
  ├── Authentication
  ├── Authorization
  ├── Validation
  ├── Endpoint / Controller
  └── Exception Handling
  │
  ▼
Application
  │
  ▼
Domain
  │
  ▼
Infrastructure
```

## Autenticación

La autenticación responde a una única pregunta: quién realiza la solicitud.

Para APIs HTTP se utilizarán tokens firmados y validados completamente. La API no debe aceptar un token sólo porque pueda decodificarse.

Validaciones mínimas:

- firma;
- issuer;
- audience;
- expiración;
- algoritmo esperado;
- claims obligatorios.

## Access tokens y refresh tokens

Los access tokens deben ser de corta duración. Los refresh tokens, cuando existan, se tratarán como credenciales sensibles persistentes.

Flujo conceptual:

```text
Login
  │
  ▼
Access Token + Refresh Token
  │
  ├── Access Token → llamadas API
  │
  └── Refresh Token → renovación controlada
                           │
                           ▼
                     Rotación / revocación
```

La implementación de referencia futura considerará rotación de refresh tokens y detección de reutilización cuando el escenario lo justifique.

## Autorización

La autorización se aplica después de establecer la identidad.

```text
Authenticated User
        │
        ▼
Claims / Roles
        │
        ▼
Authorization Policy
        │
        ▼
Resource Check
        │
        ▼
Allow / Deny
```

Se evitará distribuir reglas de acceso arbitrariamente dentro de controllers y servicios.

## Validación de entrada

La validación se ubica en los límites de confianza.

```text
External Input
      │
      ▼
Syntactic Validation
      │
      ▼
Application Validation
      │
      ▼
Domain Rules
```

Esto separa errores de formato de violaciones reales de reglas de negocio.

## Manejo de excepciones

Las excepciones internas no deben traducirse directamente a respuestas HTTP.

```text
Internal Exception
       │
       ▼
Exception Handler
       │
       ├── Secure structured log
       │
       └── Controlled Problem Details response
```

La respuesta pública debe contener únicamente la información necesaria para que el consumidor comprenda el error.

## Secretos y configuración

La configuración se divide conceptualmente entre valores no sensibles y secretos.

```text
appsettings.json
    │
    ├── parámetros no sensibles
    │
Environment / Secret Store
    │
    └── secretos
```

En desarrollo pueden utilizarse mecanismos locales seguros. En entornos desplegados deben preferirse variables de entorno o servicios administrados de secretos.

## Persistencia

La cuenta utilizada por la API para acceder a la base de datos debe tener únicamente los privilegios requeridos.

Cuando sea posible:

- evitar cuentas administrativas;
- restringir operaciones por esquema o rol;
- usar conexiones cifradas;
- rotar credenciales;
- no exponer connection strings en logs.

## Rate limiting

Los límites deben diseñarse según costo y riesgo del endpoint.

Ejemplos conceptuales:

```text
/auth/login       → límite restrictivo
/auth/refresh     → límite restrictivo
/search           → límite moderado
/reference-data   → límite más permisivo
```

El objetivo es proteger el servicio sin degradar innecesariamente el uso legítimo.

## CORS

CORS no sustituye autenticación ni autorización.

La configuración recomendada para aplicaciones web conocidas es permitir explícitamente los orígenes requeridos y limitar métodos y headers según necesidad.

## Logging y trazabilidad

Se utilizará logging estructurado con identificadores de correlación.

Ejemplo conceptual:

```text
Request
  │
  ├── CorrelationId
  ├── User/Subject ID no sensible
  ├── Endpoint
  ├── StatusCode
  └── Duration
```

Nunca debe utilizarse el logging como almacenamiento accidental de secretos o datos personales completos.

## Pruebas de seguridad

La implementación futura incluirá pruebas automáticas para verificar al menos:

- endpoint protegido rechaza solicitudes anónimas;
- token inválido es rechazado;
- token expirado es rechazado;
- usuario autenticado sin permiso recibe `403`;
- usuario autorizado obtiene acceso;
- validaciones producen respuestas consistentes;
- errores internos no filtran detalles técnicos.

## Evolución

La arquitectura se implementará progresivamente. Cada control debe justificarse por un riesgo o requisito concreto, evitando convertir la referencia en una colección indiscriminada de librerías de seguridad.
