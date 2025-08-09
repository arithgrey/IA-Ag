# Mapa de Microservicios - Sistema Enid

## 🎯 Propósito
Este mapa proporciona una visión completa de todos los microservicios del sistema Enid, sus funcionalidades, tecnologías y ubicaciones para que los agentes de IA puedan entender rápidamente dónde encontrar cada cosa.

## 📋 Resumen Ejecutivo

### Stack Tecnológico General
- **Backend**: Django 4.2.7 + Django REST Framework
- **Base de Datos**: PostgreSQL
- **Cache**: Redis (solo cuando sea necesario)
- **Servidor**: Gunicorn + Watchmedo
- **Containerización**: Docker + Docker Compose
- **Frontend**: Vue.js 3 + Vite (service-store)
- **Proxy**: Nginx (service_web)

### Estructura de Red
- **Red Principal**: `enid_service_network`
- **Red Backend**: `backend`
- **Puertos**: 8080-8090 (cada servicio en puerto diferente)

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

## 🌐 Microservicios Frontend y Web

### 7. **service-store** - Frontend Vue.js
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service-store/`
**🔧 Tecnologías**: Vue.js 3, Vite, Tailwind CSS, Stripe
**🚀 Puerto**: 5173

#### Funcionalidades:
- **Tienda Online**: Frontend de la tienda
- **Carrito de Compras**: Gestión de carrito
- **Pagos**: Integración con Stripe
- **UI/UX**: Interfaz moderna con Tailwind

#### Tecnologías Frontend:
- `vue@^3.2.36`
- `vite@^5.4.2`
- `tailwindcss@^3.3.5`
- `@stripe/stripe-js@^2.3.0`
- `vue-router@^4.2.5`
- `vuex@^4.1.0`

#### Comandos Útiles:
```bash
# Desarrollo
npm run dev

# Build
npm run build

# Tests
npm run test
```

---

### 8. **service_web** - Proxy Nginx
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service_web/`
**🔧 Tecnologías**: Nginx, SSL/TLS
**🚀 Puerto**: 80/443

#### Funcionalidades:
- **Reverse Proxy**: Redirección de requests
- **SSL/TLS**: Certificados HTTPS
- **Load Balancing**: Distribución de carga
- **Static Files**: Servir archivos estáticos

#### Configuración:
- `nginx.conf` - Configuración principal
- `nginx.conf.example` - Configuración de ejemplo
- Certbot para renovación de certificados

#### Comandos Útiles:
```bash
# Renovar certificados
certbot renew --force-renewal

# Reiniciar nginx
sudo systemctl restart nginx
```

---

### 9. **enid-store** - Backend Principal de Tienda
**📍 Ubicación**: `/home/arithgrey/enid_service/services/enid-store/`
**🔧 Tecnologías**: Django (presumiblemente)
**🚀 Puerto**: N/A (probablemente 8092)

#### Funcionalidades:
- **Backend de Tienda**: Lógica principal de e-commerce
- **Gestión de Productos**: CRUD de productos
- **Órdenes**: Procesamiento de órdenes
- **Integración**: Conecta con otros microservicios

---

## 🛠️ Servicios de Infraestructura

### 10. **service-scripts-deployment** - Scripts de Deployment
**📍 Ubicación**: `/home/arithgrey/enid_service/services/service-scripts-deployment/`
**🔧 Tecnologías**: Make, Docker, Shell Scripts

#### Funcionalidades:
- **Deployment Automation**: Scripts de despliegue
- **Status Monitoring**: Verificación de estado de servicios
- **Proxy Updates**: Actualización de configuración nginx
- **Container Management**: Gestión de contenedores

#### Comandos Principales:
```bash
# Ver estado de todos los servicios
make status_enid_service

# Parar todos los contenedores
make docker_stop

# Actualizar proxy
make update-local-proxy
```

---

## 🔗 Comunicación Entre Servicios

### Redes Docker:
- **enid_service_network**: Red principal para comunicación entre servicios
- **backend**: Red interna para servicios backend

### Dependencias:
```
service_leads ← service-oauth (autenticación)
service-store ← service_leads (datos de leads)
service-store ← service_stock (inventario)
service-store ← service-references (imágenes)
service_asistence ← service-oauth (usuarios)
```

## 📊 Puertos y URLs

| Servicio | Puerto | URL Desarrollo | Función |
|----------|--------|----------------|---------|
| service_leads | 8086 | http://localhost:8086 | Gestión de leads |
| service-oauth | 8087 | http://localhost:8087 | Autenticación |
| service_stock | 8088 | http://localhost:8088 | Inventario |
| service_asistence | 8089 | http://localhost:8089 | Asistente IA |
| service-references | 8090 | http://localhost:8090 | Imágenes |
| service-faqs | 8091 | http://localhost:8091 | FAQs |
| service-store | 5173 | http://localhost:5173 | Frontend |
| service_web | 80/443 | https://domain.com | Proxy |

## 🚀 Comandos de Desarrollo

### Levantar Todos los Servicios:
```bash
# Desde cada directorio de microservicio
docker-compose up --build

# Ver logs de un servicio específico
docker-compose logs -f microservice_enid
```

### Testing:
```bash
# Backend (Django)
docker-compose exec microservice_enid python manage.py test

# Frontend (Vue.js)
npm run test
```

### Debugging:
```bash
# Acceder al shell de Django
docker-compose exec microservice_enid python manage.py shell

# Ver logs en tiempo real
docker-compose logs -f [service_name]
```

## 📋 Checklist para Agentes de IA

### Para Nuevas Funcionalidades:
- [ ] ¿Identificaste el microservicio correcto?
- [ ] ¿Entendiste las dependencias entre servicios?
- [ ] ¿Configuraste el puerto correcto?
- [ ] ¿Incluiste tests para la nueva funcionalidad?
- [ ] ¿Documentaste los endpoints nuevos?

### Para Modificaciones:
- [ ] ¿Verificaste que no rompe otros servicios?
- [ ] ¿Actualizaste la documentación?
- [ ] ¿Ejecutaste tests en contenedores?
- [ ] ¿Revisaste logs del servicio?

### Para Deployment:
- [ ] ¿Actualizaste el proxy si es necesario?
- [ ] ¿Verificaste certificados SSL?
- [ ] ¿Probaste en desarrollo primero?
- [ ] ¿Documentaste los cambios? 