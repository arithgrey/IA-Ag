# Arquitectura del Frontend

## 🎯 Visión General

Esta documentación describe la arquitectura del frontend basada en Vue.js 3, siguiendo los principios SOLID, DRY y TDD. La arquitectura está diseñada para ser escalable, mantenible y fácil de probar.

## 🏗️ Arquitectura de Alto Nivel

### 1. Capas de la Aplicación
```
┌─────────────────────────────────────┐
│           Presentation Layer        │ ← Componentes Vue
├─────────────────────────────────────┤
│           Business Logic Layer      │ ← Composables y Stores
├─────────────────────────────────────┤
│           Data Access Layer         │ ← APIs y Servicios
├─────────────────────────────────────┤
│           Infrastructure Layer      │ ← Configuración y Utilidades
└─────────────────────────────────────┘
```

### 2. Flujo de Datos
```
User Interaction → Component → Composable → Store → API → Backend
     ↑                                                      ↓
     └────────────── Response ← State Update ←──────────────┘
```

## 🧩 Patrones Arquitectónicos

### 1. Arquitectura por Capas
- **Separación de Responsabilidades**: Cada capa tiene una responsabilidad específica
- **Dependencias Unidireccionales**: Las dependencias fluyen de arriba hacia abajo
- **Inversión de Dependencias**: Las capas superiores dependen de abstracciones

### 2. Arquitectura de Componentes
- **Componentes Atómicos**: Componentes base reutilizables
- **Componentes Moleculares**: Combinaciones de componentes atómicos
- **Componentes Organismos**: Secciones completas de la interfaz
- **Templates**: Páginas que combinan organismos

### 3. Arquitectura de Estado
- **Estado Global**: Vuex para estado compartido entre componentes
- **Estado Local**: Estado específico de cada componente
- **Estado del Servidor**: Estado sincronizado con el backend

## 📁 Estructura de Directorios

### 1. Organización por Feature
```
src/
├── components/                    # Componentes reutilizables
│   ├── UI/                      # Componentes base (Button, Input, Modal)
│   │   ├── BaseButton.vue
│   │   ├── BaseInput.vue
│   │   └── BaseModal.vue
│   ├── Layout/                  # Componentes de layout
│   │   ├── Header.vue
│   │   ├── Footer.vue
│   │   └── Sidebar.vue
│   └── Feature/                 # Componentes específicos de funcionalidad
│       ├── Products/
│       │   ├── ProductCard.vue
│       │   ├── ProductList.vue
│       │   └── ProductDetail.vue
│       ├── Cart/
│       │   ├── CartItem.vue
│       │   └── CartSummary.vue
│       └── Auth/
│           ├── LoginForm.vue
│           └── RegisterForm.vue
├── composables/                  # Lógica reutilizable
│   ├── useProduct.js
│   ├── useCart.js
│   ├── useAuth.js
│   └── useAnalytics.js
├── stores/                       # Estado global (Vuex)
│   ├── index.js
│   ├── modules/
│   │   ├── product.js
│   │   ├── cart.js
│   │   └── auth.js
│   └── plugins/
├── views/                        # Páginas y vistas
│   ├── Home.vue
│   ├── Products.vue
│   ├── Cart.vue
│   └── Checkout.vue
├── layouts/                      # Layouts de página
│   ├── DefaultLayout.vue
│   ├── AuthLayout.vue
│   └── DashboardLayout.vue
├── router/                       # Configuración de rutas
│   ├── index.js
│   ├── routes/
│   └── guards/
├── services/                     # Servicios de API
│   ├── api.js
│   ├── productService.js
│   ├── cartService.js
│   └── authService.js
├── utils/                        # Utilidades y helpers
│   ├── validators.js
│   ├── formatters.js
│   └── constants.js
├── assets/                       # Recursos estáticos
│   ├── images/
│   ├── icons/
│   └── styles/
└── tests/                        # Pruebas
    ├── unit/
    ├── integration/
    └── helpers/
```

## 🔄 Patrones de Comunicación

### 1. Props Down, Events Up
```vue
<!-- Componente padre -->
<template>
  <ProductList 
    :products="products" 
    @product-selected="handleProductSelection"
  />
</template>

<!-- Componente hijo -->
<template>
  <div @click="$emit('product-selected', product)">
    {{ product.name }}
  </div>
</template>
```

### 2. Provide/Inject para Datos Profundos
```vue
<!-- Componente ancestro -->
<script>
import { provide } from 'vue'
import { useProductStore } from '@/composables/useProductStore'

export default {
  setup() {
    const productStore = useProductStore()
    
    provide('productStore', productStore)
    provide('user', computed(() => productStore.user))
  }
}
</script>

<!-- Componente descendiente -->
<script>
import { inject } from 'vue'

export default {
  setup() {
    const productStore = inject('productStore')
    const user = inject('user')
    
    return { productStore, user }
  }
}
</script>
```

### 3. Event Bus (Legacy)
```javascript
// eventBus.js
import mitt from 'mitt'
export default mitt()

// Uso en componentes
import eventBus from '@/eventBus'

// Emitir evento
eventBus.emit('product-added', product)

// Escuchar evento
eventBus.on('product-added', (product) => {
  console.log('Product added:', product)
})
```

## 🎨 Patrones de Estilo

### 1. Sistema de Diseño
```css
/* tokens.css */
:root {
  /* Colores */
  --primary-color: #3b82f6;
  --secondary-color: #64748b;
  --success-color: #10b981;
  --warning-color: #f59e0b;
  --danger-color: #ef4444;
  
  /* Tipografía */
  --font-family-base: 'Inter', sans-serif;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  
  /* Espaciado */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  /* Bordes */
  --border-radius-sm: 0.25rem;
  --border-radius-md: 0.5rem;
  --border-radius-lg: 0.75rem;
  --border-radius-xl: 1rem;
}
```

### 2. Componentes de Diseño
```vue
<!-- BaseCard.vue -->
<template>
  <div class="card" :class="cardClass">
    <slot name="header">
      <h3 class="card-title">{{ title }}</h3>
    </slot>
    
    <div class="card-body">
      <slot>{{ content }}</slot>
    </div>
    
    <slot name="footer"></slot>
  </div>
</template>

<style scoped>
.card {
  background: var(--card-background, white);
  border-radius: var(--border-radius-lg);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: var(--spacing-lg);
}

.card-title {
  font-size: var(--font-size-lg);
  font-weight: 600;
  margin-bottom: var(--spacing-md);
}
</style>
```

## 🧪 Arquitectura de Testing

### 1. Pirámide de Testing
```
        /\
       /  \      E2E Tests (Pocos)
      /____\     
     /      \    Integration Tests (Algunos)
    /________\   
   /          \  Unit Tests (Muchos)
  /____________\
```

### 2. Estrategia de Testing
- **Unit Tests**: Componentes individuales, composables, utilidades
- **Integration Tests**: Interacción entre componentes, flujos de usuario
- **E2E Tests**: Flujos completos de la aplicación

### 3. Testing sin Mocks
```javascript
// tests/unit/composables/useProduct.spec.js
import { useProduct } from '@/composables/useProduct'
import { createTestStore } from '@/tests/helpers/createTestStore'

describe('useProduct', () => {
  let store
  
  beforeEach(() => {
    store = createTestStore()
  })
  
  it('fetches product successfully', async () => {
    const { product, loading, error, fetchProduct } = useProduct(1)
    
    expect(loading.value).toBe(false)
    
    await fetchProduct()
    
    expect(loading.value).toBe(false)
    expect(product.value).toBeTruthy()
    expect(error.value).toBe(null)
  })
})
```

## 🔧 Configuración y Build

### 1. Vite Configuration
```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  server: {
    host: '0.0.0.0',
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true
      }
    }
  },
  build: {
    target: 'es2015',
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: true
  }
})
```

### 2. TypeScript Configuration
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## 🚀 Patrones de Despliegue

### 1. Multi-Stage Builds
```dockerfile
# Dockerfile.prod
FROM node:21.6.0-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Environment Configuration
```javascript
// config/environment.js
export const config = {
  development: {
    apiUrl: 'http://localhost:3000',
    debug: true,
    logLevel: 'debug'
  },
  staging: {
    apiUrl: 'https://staging-api.example.com',
    debug: false,
    logLevel: 'info'
  },
  production: {
    apiUrl: 'https://api.example.com',
    debug: false,
    logLevel: 'error'
  }
}

export const currentConfig = config[import.meta.env.MODE] || config.development
```

## 📊 Monitoreo y Observabilidad

### 1. Logging
```javascript
// utils/logger.js
export class Logger {
  static log(level, message, data = {}) {
    const timestamp = new Date().toISOString()
    const logEntry = {
      timestamp,
      level,
      message,
      data,
      component: this.getComponentName()
    }
    
    console.log(JSON.stringify(logEntry))
  }
  
  static info(message, data) {
    this.log('info', message, data)
  }
  
  static error(message, error) {
    this.log('error', message, { error: error.message, stack: error.stack })
  }
}
```

### 2. Performance Monitoring
```javascript
// composables/usePerformance.js
import { onMounted, onUnmounted } from 'vue'

export function usePerformance() {
  const startTime = performance.now()
  
  onMounted(() => {
    const loadTime = performance.now() - startTime
    console.log(`Component loaded in ${loadTime}ms`)
  })
  
  onUnmounted(() => {
    const totalTime = performance.now() - startTime
    console.log(`Component total lifetime: ${totalTime}ms`)
  })
}
```

## 🔒 Seguridad

### 1. Content Security Policy
```html
<!-- index.html -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline' 'unsafe-eval'; 
               style-src 'self' 'unsafe-inline'; 
               img-src 'self' data: https:;">
```

### 2. Input Validation
```javascript
// utils/validators.js
export const validators = {
  email: (value) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    return emailRegex.test(value)
  },
  
  password: (value) => {
    return value.length >= 8 && 
           /[A-Z]/.test(value) && 
           /[a-z]/.test(value) && 
           /[0-9]/.test(value)
  }
}
```

## 📋 Checklist de Arquitectura

Antes de implementar una nueva funcionalidad:

- [ ] ¿Sigue los principios SOLID?
- [ ] ¿Está respaldada por pruebas TDD?
- [ ] ¿Usa componentes reutilizables?
- [ ] ¿Sigue el patrón de comunicación establecido?
- [ ] ¿Está documentada adecuadamente?
- [ ] ¿Incluye manejo de errores?
- [ ] ¿Es accesible?
- [ ] ¿Está optimizada para rendimiento?
- [ ] ¿Sigue las convenciones de nomenclatura?

## 🔗 Enlaces Relacionados

- [Vue.js Architecture Patterns](https://vuejs.org/guide/extras/ways-of-using-vue.html)
- [Vuex State Management](https://vuex.vuejs.org/)
- [Vue Composition API](https://vuejs.org/guide/extras/composition-api-faq.html)
- [Vue Testing Best Practices](https://vuejs.org/guide/best-practices/testing.html)
- [Frontend Architecture Patterns](https://martinfowler.com/articles/micro-frontends.html) 