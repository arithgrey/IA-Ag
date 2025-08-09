# IA-Ag: Contexto de Mejores Prácticas para Agentes de IA

Este repositorio contiene el contexto y las mejores prácticas que deben seguir los agentes de IA para el desarrollo de microservicios en nuestro ecosistema.

## 🎯 Propósito

Proporcionar un contexto estructurado y claro para que los agentes de IA puedan:
- Entender los principios y patrones de desarrollo establecidos
- Implementar soluciones que sigan las mejores prácticas del equipo
- Mantener consistencia en el código y arquitectura
- Aplicar TDD y testing de manera efectiva

## 📁 Estructura del Repositorio

```
IA-Ag/
└── backend/                    # TODO el contexto aquí
    ├── PRINCIPLES.md           # Principios fundamentales
    ├── CONTEXT_INDEX.md        # Índice de navegación
    ├── docs/                   # Documentación técnica
    ├── patterns/               # Patrones de diseño
    └── guidelines/             # Guías de desarrollo
```

## 🚀 Uso para Agentes de IA

Para usar este contexto con un agente de IA:

```bash
# Referenciar todo el directorio backend como contexto
@backend
```

## 📋 Principios Fundamentales

- **SOLID**: Todos los desarrollos siguen los principios SOLID
- **DRY**: Nunca repetir código
- **TDD**: Desarrollo dirigido por pruebas (sin mocks)
- **Testing**: Pruebas de integración obligatorias
- **Docker**: Microservicios dockerizados con docker-compose
- **Django + PostgreSQL**: Stack tecnológico principal

## 🔍 Navegación Rápida

- **Principios**: `backend/PRINCIPLES.md`
- **Índice**: `backend/CONTEXT_INDEX.md`
- **Arquitectura**: `backend/docs/ARCHITECTURE.md`
- **Patrones SOLID**: `backend/patterns/SOLID_PATTERNS.md`
- **Guía TDD**: `backend/guidelines/TDD_GUIDE.md`
- **Guía Docker**: `backend/guidelines/DOCKER_GUIDE.md` 