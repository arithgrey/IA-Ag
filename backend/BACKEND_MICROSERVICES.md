# Mapa de Microservicios Backend - Sistema Enid

## 🎯 Propósito
Este mapa proporciona una visión completa de todos los microservicios **BACKEND** del sistema Enid, sus funcionalidades, tecnologías y ubicaciones para que los agentes de IA puedan entender rápidamente dónde implementar nuevas funcionalidades backend.

## 📋 Resumen Ejecutivo

### Stack Tecnológico Backend
- **Framework**: Django 4.2.7 + Django REST Framework
- **Base de Datos**: PostgreSQL
- **Cache**: Redis (solo cuando sea necesario)
- **Servidor**: Gunicorn + Watchmedo
- **Containerización**: Docker + Docker Compose
- **Python**: 3.8-alpine

### Estructura de Red
- **Red Principal**: `enid_service_network`
- **Red Backend**: `backend`
- **Puertos**: 8086-8091 (cada servicio en puerto diferente)

## 🏗️ Microservicios Backend (Django)

### 1. **service_leads** - Gestión de Leads y Analytics
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service_leads/`
**🔧 Tecnologías**: Django 4.2.7, PostgreSQL, Redis
**🚀 Puerto**: 8086

#### Funcionalidades:
- **Gestión de Leads**: CRUD completo de leads
- **Métricas de Leads**: Analytics y reportes
- **Page Analytics**: Tracking de páginas y user journey
- **Lead Search**: Búsqueda avanzada de leads
- **Lead Types**: Tipos de leads categorizados

#### Apps Django:
- `lead/` - Gestión principal de leads
- `lead_metrics/` - Métricas y analytics
- `lead_type/` - Tipos de leads
- `lead_search/` - Búsqueda de leads
- `page_analytics/` - Analytics de páginas
- `expose/` - APIs expuestas

#### Endpoints Principales:
- `GET /lead/` - Listar leads
- `GET /lead-metrics/overview/` - Resumen de métricas
- `GET /page-analytics/page-access/` - Analytics de páginas
- `GET /lead-search/?q=query` - Búsqueda de leads

#### Comandos Útiles:
```bash
# Generar datos de prueba
docker-compose exec microservice_enid python manage.py generate_realistic_leads
docker-compose exec microservice_enid python manage.py generate_page_analytics_data

# Ejecutar tests
docker-compose exec microservice_enid python manage.py test
```

---

### 2. **service-oauth** - Autenticación y Autorización
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service-oauth/`
**🔧 Tecnologías**: Django 4.2.7, PostgreSQL, JWT
**🚀 Puerto**: 8087

#### Funcionalidades:
- **Autenticación**: Login/logout de usuarios
- **Autorización**: Gestión de permisos y roles
- **JWT Tokens**: Manejo de tokens de sesión
- **User Management**: Gestión de usuarios
- **Store Integration**: Integración con tiendas

#### Apps Django:
- `user/` - Gestión de usuarios
- `login/` - Autenticación
- `logout/` - Cierre de sesión
- `store/` - Integración con tiendas
- `feature/` - Características de autorización
- `share_test/` - Testing compartido

#### Endpoints Principales:
- `POST /login/` - Autenticación
- `POST /logout/` - Cierre de sesión
- `GET /user/profile/` - Perfil de usuario
- `GET /store/` - Información de tiendas

---

### 3. **service_stock** - Gestión de Inventario
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service_stock/`
**🔧 Tecnologías**: Django 4.2.7, PostgreSQL
**🚀 Puerto**: 8088

#### Funcionalidades:
- **Gestión de Almacenes**: CRUD de warehouses
- **Movimientos de Stock**: Tracking de movimientos
- **Inventario**: Control de inventario
- **Reportes de Stock**: Analytics de inventario

#### Apps Django:
- `warehouses/` - Gestión de almacenes
- `movements/` - Movimientos de stock
- `app/` - Configuración principal

#### Endpoints Principales:
- `GET /warehouses/` - Listar almacenes
- `GET /movements/` - Movimientos de stock
- `POST /warehouses/` - Crear almacén

---

### 4. **service_asistence** - Asistente IA
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service_asistence/`
**🔧 Tecnologías**: Django 4.2.7, PostgreSQL, Redis, Google AI
**🚀 Puerto**: 8089

#### Funcionalidades:
- **Chatbot IA**: Asistente conversacional
- **Conversaciones**: Historial de chats
- **Mensajes**: Gestión de mensajes
- **Cache de IA**: Redis para respuestas rápidas
- **Google AI Integration**: Integración con Google Generative AI

#### Apps Django:
- `assistant/` - Lógica del asistente IA
- `conversation/` - Gestión de conversaciones
- `message/` - Gestión de mensajes
- `cache/` - Cache de respuestas

#### Dependencias Especiales:
- `google-generativeai==0.8.4`
- `redis==5.0.1`
- `django-redis==5.4.0`

#### Endpoints Principales:
- `POST /assistant/chat/` - Enviar mensaje al asistente
- `GET /conversation/` - Historial de conversaciones
- `GET /message/` - Mensajes de conversación

---

### 5. **service-references** - Gestión de Referencias e Imágenes
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service-references/`
**🔧 Tecnologías**: Django 5.0.4, PostgreSQL, Redis
**🚀 Puerto**: 8090

#### Funcionalidades:
- **Gestión de Imágenes**: Upload y gestión de imágenes
- **Referencias de Negocio**: Información de empresas
- **Cache de Imágenes**: Redis para imágenes frecuentes
- **Deployment**: Scripts de Kubernetes

#### Apps Django:
- `image/` - Gestión de imágenes
- `business/` - Referencias de negocio
- `app/` - Configuración principal

#### Dependencias Especiales:
- `Django==5.0.4` (versión más nueva)
- `redis==5.0.3`
- `django-redis==5.4.0`

#### Scripts Especiales:
- `initial_images.py` - Carga inicial de imágenes
- `deployment_to_kubernets.py` - Deployment a Kubernetes

#### Endpoints Principales:
- `POST /image/upload/` - Subir imagen
- `GET /business/` - Referencias de negocio
- `GET /image/` - Listar imágenes

---

### 6. **service-faqs** - Preguntas Frecuentes
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service-faqs/`
**🔧 Tecnologías**: Django 4.2.7, PostgreSQL
**🚀 Puerto**: 8091

#### Funcionalidades:
- **Gestión de FAQs**: CRUD de preguntas frecuentes
- **Categorías**: Organización por categorías
- **Returns**: Gestión de devoluciones
- **Búsqueda**: Búsqueda en FAQs

#### Apps Django:
- `faqs/` - Gestión de FAQs
- `returns/` - Gestión de devoluciones
- `app/` - Configuración principal

#### Endpoints Principales:
- `GET /faqs/` - Listar FAQs
- `GET /returns/` - Información de devoluciones
- `POST /faqs/` - Crear FAQ

---

## 🔗 Comunicación Entre Servicios Backend

### Dependencias Backend:
```
service_leads ← service-oauth (autenticación)
service_stock ← service-oauth (usuarios)
service_asistence ← service-oauth (usuarios)
service-references ← service-oauth (usuarios)
service-faqs ← service-oauth (usuarios)
```

### Redes Docker:
- **enid_service_network**: Red principal para comunicación entre servicios
- **backend**: Red interna para servicios backend

## 📊 Puertos y URLs Backend

| Servicio | Puerto | URL Desarrollo | Función |
|----------|--------|----------------|---------|
| service_leads | 8086 | http://localhost:8086 | Gestión de leads |
| service-oauth | 8087 | http://localhost:8087 | Autenticación |
| service_stock | 8088 | http://localhost:8088 | Inventario |
| service_asistence | 8089 | http://localhost:8089 | Asistente IA |
| service-references | 8090 | http://localhost:8090 | Imágenes |
| service-faqs | 8091 | http://localhost:8091 | FAQs |

## 🚀 Comandos de Desarrollo Backend

### Levantar Servicios Backend:
```bash
# Desde cada directorio de microservicio
docker-compose up --build

# Ver logs de un servicio específico
docker-compose logs -f microservice_enid
```

### Testing Backend:
```bash
# Ejecutar tests de Django
docker-compose exec microservice_enid python manage.py test

# Ejecutar tests específicos
docker-compose exec microservice_enid python manage.py test [app_name]

# Ejecutar tests con coverage
docker-compose exec microservice_enid coverage run --source='.' manage.py test
```

### Debugging Backend:
```bash
# Acceder al shell de Django
docker-compose exec microservice_enid python manage.py shell

# Ver logs en tiempo real
docker-compose logs -f [service_name]

# Crear superusuario
docker-compose exec microservice_enid python manage.py createsuperuser
```

## 📋 Checklist para Agentes de IA - Backend

### Para Nuevas Funcionalidades Backend:
- [ ] ¿Identificaste el microservicio backend correcto?
- [ ] ¿Entendiste las dependencias con service-oauth?
- [ ] ¿Configuraste el puerto correcto (8086-8091)?
- [ ] ¿Incluiste tests para la nueva funcionalidad?
- [ ] ¿Documentaste los endpoints nuevos?
- [ ] ¿Evaluaste si necesitas Redis?

### Para Modificaciones Backend:
- [ ] ¿Verificaste que no rompe otros servicios backend?
- [ ] ¿Actualizaste la documentación?
- [ ] ¿Ejecutaste tests en contenedores?
- [ ] ¿Revisaste logs del servicio?
- [ ] ¿Verificaste autenticación si es necesario?

### Para Deployment Backend:
- [ ] ¿Probaste en desarrollo primero?
- [ ] ¿Verificaste migraciones de base de datos?
- [ ] ¿Documentaste los cambios?
- [ ] ¿Actualizaste requirements.txt si es necesario?

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

## 💡 Consejos Rápidos - Backend

### Desarrollo:
- **Usa Django 4.2.7** para consistencia
- **Implementa TDD** en todos los servicios
- **Usa Serializers** en lugar de Repositories
- **Incluye tests de integración** obligatorios

### Testing:
- **Sin mocks** en pruebas unitarias
- **Usa Faker** para datos de prueba
- **Ejecuta tests en contenedores**
- **Mantén cobertura alta**

### Docker:
- **Usa Python 3.8-alpine** como base
- **Configura entrypoint.sh** con watchmedo + gunicorn
- **Incluye health checks** para PostgreSQL
- **Usa networks** correctamente

### Microservicios:
- **Identifica el servicio correcto** antes de implementar
- **Verifica dependencias** con service-oauth
- **Usa el puerto correcto** (8086-8091)
- **Documenta endpoints** nuevos 