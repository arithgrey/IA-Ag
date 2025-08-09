# Frontend Development Guidelines

## 🎯 Visión General

Este directorio contiene las reglas, patrones y mejores prácticas para el desarrollo frontend de nuestro proyecto. Estas directrices están diseñadas para ser consumidas por agentes de IA para asegurar la consistencia y calidad en el desarrollo.

## 🏗️ Principios Fundamentales

### 1. Principios SOLID y DRY
- **S**ingle Responsibility: Cada componente debe tener una sola responsabilidad
- **O**pen/Closed: Los componentes deben ser extensibles sin modificación
- **L**iskov Substitution: Los componentes deben ser intercambiables
- **I**nterface Segregation: Interfaces específicas para cada necesidad
- **D**ependency Inversion: Depender de abstracciones, no de implementaciones
- **DRY**: Don't Repeat Yourself - Reutilizar código y componentes

### 2. TDD (Test-Driven Development)
- **NO MOCKS**: Prohibido el uso de mocks en pruebas
- Todas las funcionalidades deben estar respaldadas por pruebas unitarias
- Escribir pruebas ANTES de implementar funcionalidad
- Cobertura mínima del 90% en componentes críticos

### 3. Pruebas de Integración
- Todos los componentes deben tener pruebas de integración
- Las pruebas deben simular el comportamiento real del usuario
- Usar `@vue/test-utils` para pruebas de componentes Vue

## 🚀 Stack Tecnológico

### Framework Principal
- **Vue.js 3** con Composition API
- **Vite** como bundler y dev server
- **Vue Router 4** para navegación
- **Vuex 4** para gestión de estado

### Herramientas de Desarrollo
- **TypeScript** para tipado estático
- **Tailwind CSS** para estilos utilitarios
- **Jest** para testing
- **ESLint** + **Prettier** para calidad de código

### Librerías de UI
- **@headlessui/vue** para componentes accesibles
- **@heroicons/vue** para iconografía
- **@tailwindcss/typography** para tipografía

## 📁 Estructura de Directorios

```
src/
├── components/          # Componentes reutilizables
│   ├── UI/             # Componentes base (Button, Input, Modal)
│   ├── Layout/         # Componentes de layout (Header, Footer, Sidebar)
│   └── Feature/        # Componentes específicos de funcionalidad
├── composables/         # Lógica reutilizable (hooks de Vue 3)
├── helpers/             # Utilidades y mixins
├── views/               # Páginas y vistas
├── layouts/             # Layouts de página
├── store/               # Estado global (Vuex)
├── router/              # Configuración de rutas
├── config/              # Configuraciones
├── assets/              # Recursos estáticos
└── tests/               # Pruebas
    ├── unit/            # Pruebas unitarias
    └── integration/     # Pruebas de integración
```

## 🧩 Patrones de Componentes

### 1. Componentes Base (UI)
- **Reutilizables**: Cada componente debe poder usarse en múltiples contextos
- **Configurables**: Props para personalización sin modificar el componente
- **Accesibles**: Seguir estándares WCAG 2.1
- **Responsivos**: Mobile-first design

### 2. Componentes de Feature
- **Específicos**: Resolver un problema específico del dominio
- **Composables**: Usar composables para lógica reutilizable
- **Eventos**: Emitir eventos para comunicación padre-hijo

### 3. Estructura de Componente
```vue
<template>
  <!-- Template con clases Tailwind -->
</template>

<script>
export default {
  name: 'ComponentName',
  props: {
    // Props tipadas y validadas
  },
  emits: ['event-name'],
  setup(props, { emit }) {
    // Lógica del componente
    return {
      // Variables y métodos expuestos
    }
  }
}
</script>

<style scoped>
/* Estilos específicos del componente */
</style>
```

## 🧪 Testing Strategy

### 1. Pruebas Unitarias
- **Sin Mocks**: Usar implementaciones reales
- **Componentes**: Probar props, eventos y comportamiento
- **Composables**: Probar lógica de negocio aislada
- **Helpers**: Probar utilidades y funciones puras

### 2. Pruebas de Integración
- **Flujos de Usuario**: Simular interacciones reales
- **Comunicación**: Probar comunicación entre componentes
- **Estado**: Verificar cambios de estado correctos

### 3. Ejemplo de Prueba
```javascript
import { mount } from '@vue/test-utils'
import ProductCard from '@/components/Products/ProductCard.vue'

describe('ProductCard', () => {
  it('emits add-to-cart event when add to cart button is clicked', async () => {
    const wrapper = mount(ProductCard, {
      props: {
        product: {
          id: 1,
          name: 'Test Product',
          price: 99.99,
          image: 'test.jpg',
          description: 'Test description'
        }
      }
    })
    
    await wrapper.find('[data-test="add-to-cart"]').trigger('click')
    
    expect(wrapper.emitted('add-to-cart')).toBeTruthy()
    expect(wrapper.emitted('add-to-cart')[0]).toEqual([wrapper.props('product')])
  })
})
```

## 🐳 Docker y Despliegue

### 1. Dockerfile
```dockerfile
FROM node:21.6.0-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --legacy-peer-deps
COPY . .
EXPOSE 5173
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

### 2. Docker Compose
```yaml
version: '3'
services:
  frontend:
    build: .
    ports:
      - "5173:5173"
    volumes:
      - .:/app
    networks:
      - app_network
```

## 📚 Composición y Reutilización

### 1. Composables
- **Lógica de Negocio**: Extraer lógica compleja a composables
- **Estado Compartido**: Compartir estado entre componentes
- **API Calls**: Centralizar llamadas a APIs
- **Validaciones**: Reutilizar lógica de validación

### 2. Mixins (Legacy)
- **Analytics**: Tracking de eventos y métricas
- **Helpers**: Utilidades comunes
- **Validaciones**: Reglas de validación reutilizables

### 3. Ejemplo de Composable
```javascript
// useProduct.js
import { ref, computed } from 'vue'
import { useStore } from 'vuex'

export function useProduct(productId) {
  const store = useStore()
  const product = ref(null)
  const loading = ref(false)
  const error = ref(null)

  const fetchProduct = async () => {
    loading.value = true
    try {
      product.value = await store.dispatch('products/fetchProduct', productId)
    } catch (err) {
      error.value = err.message
    } finally {
      loading.value = false
    }
  }

  const addToCart = () => {
    store.dispatch('cart/addItem', product.value)
  }

  return {
    product: computed(() => product.value),
    loading: computed(() => loading.value),
    error: computed(() => error.value),
    fetchProduct,
    addToCart
  }
}
```

## 🎨 Estilos y Diseño

### 1. Tailwind CSS
- **Utility-First**: Usar clases utilitarias de Tailwind
- **Componentes**: Crear componentes reutilizables con clases consistentes
- **Responsive**: Mobile-first approach
- **Dark Mode**: Soporte para modo oscuro

### 2. Diseño de Sistema
- **Tokens**: Colores, tipografía y espaciado consistentes
- **Componentes**: Biblioteca de componentes base
- **Layouts**: Grids y layouts reutilizables

## 🔧 Configuración y Build

### 1. Vite
- **HMR**: Hot Module Replacement para desarrollo
- **Optimización**: Build optimizado para producción
- **Plugins**: Vue, JSX y TypeScript

### 2. TypeScript
- **Configuración**: `tsconfig.json` estricto
- **Tipos**: Definir interfaces para props y eventos
- **Validación**: Type checking en tiempo de compilación

## 📖 Documentación

### 1. Storybook
- **Componentes**: Documentar todos los componentes
- **Props**: Ejemplos de uso y variaciones
- **Interacciones**: Demostrar comportamiento

### 2. JSDoc
- **Comentarios**: Documentar funciones y métodos
- **Tipos**: Especificar tipos de parámetros
- **Ejemplos**: Proporcionar ejemplos de uso

## 🚫 Anti-Patrones

### 1. NO Hacer
- ❌ Usar mocks en pruebas
- ❌ Duplicar código entre componentes
- ❌ Crear componentes monolíticos
- ❌ Ignorar la accesibilidad
- ❌ Escribir código sin pruebas

### 2. SÍ Hacer
- ✅ Escribir pruebas antes del código
- ✅ Crear componentes reutilizables
- ✅ Seguir principios SOLID
- ✅ Implementar pruebas de integración
- ✅ Usar TypeScript para tipado

## 🔗 Enlaces Útiles

- [Backend Guidelines](../backend/) - Reglas del backend
- [Vue.js 3 Documentation](https://vuejs.org/)
- [Vite Documentation](https://vitejs.dev/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Jest Testing Framework](https://jestjs.io/)

## 📝 Notas de Implementación

Esta documentación debe ser actualizada cada vez que se introduzcan nuevos patrones o se modifiquen las reglas existentes. Los agentes de IA deben seguir estas directrices para mantener la consistencia en el desarrollo del frontend. 