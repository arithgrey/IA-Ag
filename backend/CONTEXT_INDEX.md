# Índice de Contexto para Agentes de IA - Backend

## 🎯 Propósito
Este índice sirve como punto de entrada para que los agentes de IA encuentren rápidamente la información necesaria para implementar soluciones **BACKEND** siguiendo nuestras mejores prácticas.

## 📚 Documentación Principal

### Principios Fundamentales
- **[PRINCIPLES.md](PRINCIPLES.md)** - Principios SOLID, DRY, TDD y stack tecnológico
- **[patterns/SOLID_PATTERNS.md](patterns/SOLID_PATTERNS.md)** - Ejemplos prácticos de SOLID en Django

### Guías Específicas
- **[guidelines/TDD_GUIDE.md](guidelines/TDD_GUIDE.md)** - Guía completa de TDD sin mocks
- **[guidelines/DOCKER_GUIDE.md](guidelines/DOCKER_GUIDE.md)** - Docker y microservicios con gunicorn + watchmedo + Redis
- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Arquitectura de microservicios

### Mapa de Microservicios Backend
- **[BACKEND_MICROSERVICES.md](BACKEND_MICROSERVICES.md)** - Mapa completo de microservicios **BACKEND** del sistema Enid

## 🔍 Búsqueda Rápida por Tema

### Para Nuevas Implementaciones Backend
1. **Revisar principios** → `PRINCIPLES.md`
2. **Aplicar SOLID** → `patterns/SOLID_PATTERNS.md`
3. **Implementar TDD** → `guidelines/TDD_GUIDE.md`
4. **Configurar Docker** → `guidelines/DOCKER_GUIDE.md`
5. **Identificar microservicio backend** → `BACKEND_MICROSERVICES.md`

### Para Testing Backend
- **TDD sin mocks** → `guidelines/TDD_GUIDE.md`
- **Pruebas de integración** → `guidelines/TDD_GUIDE.md#pruebas-de-integración`
- **Faker y datos de prueba** → `guidelines/TDD_GUIDE.md#comandos-django-con-faker`

### Para Docker Backend
- **Comandos esenciales** → `guidelines/DOCKER_GUIDE.md#comandos-docker-esenciales`
- **Testing en contenedores** → `guidelines/DOCKER_GUIDE.md#testing-en-contenedores`
- **Debugging** → `guidelines/DOCKER_GUIDE.md#debugging-y-desarrollo`
- **Entrypoint.sh** → `guidelines/DOCKER_GUIDE.md#entrypointsh`
- **Redis** → `guidelines/DOCKER_GUIDE.md#redis---criterios-de-uso`

### Para Arquitectura Backend
- **Estructura de microservicios** → `docs/ARCHITECTURE.md`
- **Stack tecnológico** → `docs/ARCHITECTURE.md#stack-tecnológico`
- **Base de datos** → `docs/ARCHITECTURE.md#base-de-datos`
- **Gunicorn + Watchmedo** → `docs/ARCHITECTURE.md#entrypointsh`
- **Redis criterios** → `docs/ARCHITECTURE.md#redis---criterios-de-uso-elegante`

### Para Microservicios Backend Específicos
- **service_leads** → `BACKEND_MICROSERVICES.md#1-service_leads---gestión-de-leads-y-analytics`
- **service-oauth** → `BACKEND_MICROSERVICES.md#2-service-oauth---autenticación-y-autorización`
- **service_stock** → `BACKEND_MICROSERVICES.md#3-service_stock---gestión-de-inventario`
- **service_asistence** → `BACKEND_MICROSERVICES.md#4-service_asistence---asistente-ia`
- **service-references** → `BACKEND_MICROSERVICES.md#5-service-references---gestión-de-referencias-e-imágenes`
- **service-faqs** → `BACKEND_MICROSERVICES.md#6-service-faqs---preguntas-frecuentes`

## 🚀 Checklist de Implementación Backend

### Antes de Empezar
- [ ] ¿Entendiste los principios SOLID?
- [ ] ¿Planeaste la solución más concisa?
- [ ] ¿Identificaste oportunidades para DRY?
- [ ] ¿Preparaste el entorno Docker?
- [ ] ¿Evaluaste si Redis es realmente necesario?
- [ ] ¿Identificaste el microservicio backend correcto?

### Durante el Desarrollo
- [ ] ¿Escribiste pruebas primero (TDD)?
- [ ] ¿Evitaste repetir código?
- [ ] ¿Usaste Serializers en lugar de Repositories?
- [ ] ¿Incluiste pruebas de integración?

### Antes de Finalizar
- [ ] ¿Ejecutaste tests en contenedores?
- [ ] ¿Verificaste que no hay mocks?
- [ ] ¿Documentaste solo lo esencial?
- [ ] ¿Revisaste logs del contenedor?
- [ ] ¿Configuraste Redis solo si es necesario?
- [ ] ¿Verificaste que no rompe otros servicios backend?

## 💡 Consejos Rápidos - Backend

### Código Backend
- **DRY**: Reutiliza, no repitas
- **SOLID**: Cada clase una responsabilidad
- **TDD**: Prueba primero, implementa después
- **Serializers**: Usa Django Serializers, no Repositories

### Testing Backend
- **Sin mocks**: Usa datos reales con Faker
- **Integración**: Prueba flujos completos
- **Contenedores**: Ejecuta tests dentro de Docker

### Docker Backend
- **Logs**: `docker-compose logs -f microservice_enid`
- **Tests**: `docker-compose exec microservice_enid python manage.py test`
- **Shell**: `docker-compose exec microservice_enid python manage.py shell`
- **Entrypoint**: Usa entrypoint.sh con watchmedo + gunicorn
- **Redis**: Solo cuando sea necesario para mejorar performance

### Microservicios Backend
- **Identifica el servicio correcto** antes de implementar
- **Verifica dependencias** con service-oauth
- **Usa el puerto correcto** (8086-8091)
- **Documenta endpoints** nuevos

## 🔗 Referencias Rápidas - Backend

### Comandos Docker Esenciales Backend
```bash
# Desarrollo
docker-compose up --build
docker-compose logs -f microservice_enid

# Testing
docker-compose exec microservice_enid python manage.py test
docker-compose exec microservice_enid python manage.py shell

# Debugging
docker-compose exec microservice_enid python manage.py createsuperuser
docker-compose exec microservice_enid python manage.py create_test_data

# Redis (solo si lo usas)
docker-compose exec redis redis-cli ping
```

### Estructura de Archivos Backend
```
microservice/
├── app/                    # Aplicación Django principal
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── [app1]/                # Apps Django específicas
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── tests/
├── manage.py
├── requirements.txt
├── Dockerfile             # Python 3.8-alpine
├── docker-compose.yml     # Con networks, health checks y Redis opcional
├── entrypoint.sh          # Con watchmedo + gunicorn
└── .env                   # Variables de entorno
```

### Stack Tecnológico Backend
- **Django 4.2.7**: Framework web
- **PostgreSQL**: Base de datos
- **Redis**: Cache y sesiones (solo cuando sea necesario)
- **Gunicorn**: Servidor WSGI
- **Watchmedo**: Auto-reload en desarrollo
- **Docker**: Containerización
- **Python 3.8-alpine**: Imagen base

### Puertos de Microservicios Backend
| Servicio | Puerto | Función |
|----------|--------|---------|
| service_leads | 8086 | Gestión de leads |
| service-oauth | 8087 | Autenticación |
| service_stock | 8088 | Inventario |
| service_asistence | 8089 | Asistente IA |
| service-references | 8090 | Imágenes |
| service-faqs | 8091 | FAQs |

## 🔴 Criterios para Redis en Backend

### ✅ USAR Redis en Backend cuando:
- **Cache de consultas frecuentes** que no cambian frecuentemente
- **Sesiones de usuario** que requieren alta disponibilidad
- **Rate limiting** para APIs
- **Colas de tareas** asíncronas
- **Datos temporales** que se acceden muy frecuentemente
- **Mejora de tiempos de respuesta** en endpoints críticos

### ❌ NO usar Redis en Backend cuando:
- Los datos cambian constantemente
- No hay ganancia real en performance
- Añade complejidad innecesaria
- Los datos son pequeños y simples
- No hay patrones de acceso frecuente

## 📞 Recordatorios Importantes - Backend

1. **SIEMPRE** aplica DRY - nunca repitas código
2. **SIEMPRE** usa TDD - escribe pruebas primero
3. **SIEMPRE** incluye pruebas de integración
4. **NUNCA** uses mocks en pruebas unitarias
5. **NUNCA** uses patrones Repository - usa Serializers
6. **SIEMPRE** ejecuta tests en contenedores
7. **SIEMPRE** revisa logs del contenedor para debugging
8. **MANTÉN** documentación limitada y esencial
9. **USA** entrypoint.sh con watchmedo + gunicorn
10. **CONFIGURA** networks y health checks en docker-compose
11. **EVALÚA** si Redis es realmente necesario antes de incluirlo
12. **USA** Redis solo para mejorar tiempos de respuesta cuando sea justificado
13. **IDENTIFICA** el microservicio backend correcto antes de implementar
14. **VERIFICA** dependencias con service-oauth
15. **USA** puertos 8086-8091 para servicios backend 