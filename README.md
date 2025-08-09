# IA-Ag: Contexto de Mejores Prácticas para Agentes de IA - Backend

Este repositorio contiene el contexto y las mejores prácticas que deben seguir los agentes de IA para el desarrollo de **microservicios backend** en nuestro ecosistema.

## 🎯 Propósito

Proporcionar un contexto estructurado y claro para que los agentes de IA puedan:
- Entender los principios y patrones de desarrollo backend establecidos
- Implementar soluciones backend que sigan las mejores prácticas del equipo
- Mantener consistencia en el código y arquitectura backend
- Aplicar TDD y testing de manera efectiva en servicios backend
- Documentar APIs con Swagger/OpenAPI de manera consistente
- Realizar pruebas de integración completas con todos los microservicios

## 📁 Estructura del Repositorio

```
IA-Ag/
└── backend/                    # TODO el contexto BACKEND aquí
    ├── PRINCIPLES.md           # Principios fundamentales
    ├── CONTEXT_INDEX.md        # Índice de navegación backend
    ├── BACKEND_MICROSERVICES.md # Mapa de microservicios backend
    ├── docs/                   # Documentación técnica backend
    ├── patterns/               # Patrones de diseño backend
    └── guidelines/             # Guías de desarrollo backend
        ├── TDD_GUIDE.md        # TDD sin mocks
        ├── DOCKER_GUIDE.md     # Docker y microservicios
        └── API_DOCUMENTATION.md # Documentación de APIs con Swagger
```

## 🚀 Uso para Agentes de IA

Para usar este contexto con un agente de IA para desarrollo **BACKEND**:

```bash
# Referenciar todo el directorio backend como contexto
@backend
```

## 📋 Principios Fundamentales - Backend

- **SOLID**: Todos los desarrollos backend siguen los principios SOLID
- **DRY**: Nunca repetir código
- **TDD**: Desarrollo dirigido por pruebas (sin mocks)
- **Testing**: Pruebas de integración obligatorias
- **Docker**: Microservicios backend dockerizados con docker-compose
- **Django + PostgreSQL**: Stack tecnológico principal backend
- **Swagger/OpenAPI**: Todas las APIs deben estar bien documentadas
- **Integración Completa**: Pruebas con todos los microservicios

## 🔍 Navegación Rápida - Backend

- **Principios**: `backend/PRINCIPLES.md`
- **Índice**: `backend/CONTEXT_INDEX.md`
- **Mapa Backend**: `backend/BACKEND_MICROSERVICES.md`
- **Arquitectura**: `backend/docs/ARCHITECTURE.md`
- **Patrones SOLID**: `backend/patterns/SOLID_PATTERNS.md`
- **Guía TDD**: `backend/guidelines/TDD_GUIDE.md`
- **Guía Docker**: `backend/guidelines/DOCKER_GUIDE.md`
- **Documentación APIs**: `backend/guidelines/API_DOCUMENTATION.md`

## 🏗️ Microservicios Backend Incluidos

### Puertos 8086-8091:
- **service_leads** (8086) - Gestión de leads y analytics
- **service-oauth** (8087) - Autenticación y autorización
- **service_stock** (8088) - Gestión de inventario
- **service_asistence** (8089) - Asistente IA
- **service-references** (8090) - Gestión de imágenes
- **service-faqs** (8091) - Preguntas frecuentes

## 🚀 Integración Completa para Pruebas

### Alias local_up
Para levantar todos los microservicios de una sola vez (incluyendo frontend) para pruebas de integración:

```bash
# Levantar todo el ecosistema
local_up

# Verificar estado de todos los servicios
docker ps

# Ver logs de todos los servicios
docker logs $(docker ps -q)
```

### URLs de Integración Completa
| Servicio | URL | Función |
|----------|-----|---------|
| Frontend | http://localhost:5173 | Vue.js frontend |
| service_leads | http://localhost:8086 | Gestión de leads |
| service-oauth | http://localhost:8087 | Autenticación |
| service_stock | http://localhost:8088 | Inventario |
| service_asistence | http://localhost:8089 | Asistente IA |
| service-references | http://localhost:8090 | Imágenes |
| service-faqs | http://localhost:8091 | FAQs |
| Swagger UI | http://localhost:8086/swagger/ | Documentación APIs |

### Comandos de Verificación
```bash
# Verificar conectividad entre servicios
curl http://localhost:8086/health/
curl http://localhost:8087/health/
curl http://localhost:8088/health/

# Verificar Swagger
curl http://localhost:8086/swagger.json
```

## 📚 Documentación de APIs

### Swagger/OpenAPI
- **TODAS las APIs** deben estar documentadas con Swagger
- **drf-yasg** para documentación automática
- **Ejemplos claros** de request/response
- **Códigos de respuesta** documentados
- **Parámetros explicados** detalladamente

### URLs de Documentación
Cada microservicio incluye:
- `/swagger/` - Interfaz Swagger UI
- `/redoc/` - Documentación ReDoc
- `/swagger.json` - Especificación OpenAPI

### Ejemplo de Acceso:
```bash
# Swagger UI
http://localhost:8086/swagger/

# JSON de Swagger
curl http://localhost:8086/swagger.json
```

## 💡 Nota Importante

Este directorio está **específicamente diseñado para desarrollo backend**. Para desarrollo frontend, se creará un directorio separado con contexto específico para Vue.js, React, o tecnologías frontend.

### Stack Backend Documentado:
- **Django 4.2.7** + Django REST Framework
- **PostgreSQL** como base de datos
- **Redis** (solo cuando sea necesario)
- **Gunicorn + Watchmedo** como servidor
- **Docker + Docker Compose** para containerización
- **Python 3.8-alpine** como imagen base
- **drf-yasg** para documentación de APIs
- **local_up** para integración completa 