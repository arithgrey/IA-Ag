# Frontend Development Guidelines

## 🎯 Principios Fundamentales

### 1. **SOLID Principles**
- **Single Responsibility**: Cada componente debe tener una sola responsabilidad
- **Open/Closed**: Los componentes deben ser extensibles sin modificación
- **Liskov Substitution**: Los componentes deben ser intercambiables
- **Interface Segregation**: Interfaces específicas para cada propósito
- **Dependency Inversion**: Depender de abstracciones, no de implementaciones

### 2. **DRY (Don't Repeat Yourself)**
- Reutilizar componentes y lógica común
- Crear mixins y composables para funcionalidad compartida
- Evitar duplicación de código en componentes similares

### 3. **Test-Driven Development (TDD)**
- **NO MOCKS**: Estrictamente prohibido el uso de mocks en tests
- Todos los desarrollos deben seguir el ciclo Red-Green-Refactor
- Cobertura mínima de tests: 90%
- Tests de integración obligatorios para todos los componentes

### 4. **Reglas Específicas de Import y Estilos**
- **ALIAS @**: Usar exclusivamente `@/` para imports, NUNCA rutas relativas como `./` o `../`
- **Tailwind CSS**: Usar exclusivamente Tailwind CSS, NO CSS directo excepto casos muy específicos
- **Axios Global**: Consumir endpoints usando la instancia global de Axios configurada

## 🛠️ Stack Tecnológico

### Core Framework
- **Vue.js 3** con Composition API
- **Vite** como build tool
- **Vue Router 4** para navegación
- **Vuex 4** para estado global

### Styling & UI
- **Tailwind CSS** como sistema de estilos principal
- **@headlessui/vue** para componentes accesibles
- **@heroicons/vue** para iconografía
- **@tailwindcss/typography** para tipografía

### Testing & Quality
- **Jest** como framework de testing
- **@vue/test-utils** para testing de componentes
- **ESLint** para linting
- **Prettier** para formateo de código

### HTTP & APIs
- **Axios** configurado globalmente con interceptores
- **Reverse Proxy** para comunicación con backend
- Manejo automático de autenticación y tokens

### Containerización
- **Docker** para desarrollo y producción
- **Docker Compose** para orquestación
- **Nginx** para servidor de producción

## 📁 Estructura de Directorios

```
src/
├── components/          # Componentes reutilizables
│   ├── Products/       # Componentes específicos de productos
│   ├── Cart/          # Componentes del carrito
│   ├── Analytics/     # Componentes de analytics
│   └── ...            # Otros grupos de componentes
├── views/              # Páginas y vistas principales
├── layouts/            # Layouts de la aplicación
├── helpers/            # Mixins y helpers
├── composables/        # Composables de Vue 3
├── config/             # Configuraciones
├── assets/             # Recursos estáticos
├── store.js            # Store de Vuex
├── router.js           # Configuración de rutas
├── main.js             # Punto de entrada
├── axiosInstance.js    # Instancia global de Axios
└── style.css           # Estilos globales (solo Tailwind)
```

## 🔧 Configuración de Desarrollo

### Vite Configuration
```javascript
// vite.config.js
export default defineConfig({
  plugins: [Vue()],
  resolve: {
    alias: {
      '@': '/src',  // ALIAS OBLIGATORIO
    },
  },
});
```

### Tailwind Configuration
```javascript
// tailwind.config.js
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [
    require('@tailwindcss/typography'),
  ],
}
```

## 📚 Patrones de Componentes

### 1. **Imports con Alias @**
```javascript
// ✅ CORRECTO
import ProductCard from '@/components/Products/ProductCard.vue'
import { getSeasonalContent } from '@/helpers/SeasonalHelper.js'

// ❌ INCORRECTO
import ProductCard from './ProductCard.vue'
import { getSeasonalContent } from '../../helpers/SeasonalHelper.js'
```

### 2. **Uso de Tailwind CSS**
```vue
<template>
  <!-- ✅ CORRECTO: Solo clases de Tailwind -->
  <div class="bg-white shadow-lg hover:shadow-xl transition-all duration-300">
    <button class="bg-blue-600 text-white p-2 rounded-full hover:bg-blue-700">
      Agregar al carrito
    </button>
  </div>
</template>

<style scoped>
/* ❌ INCORRECTO: CSS directo (excepto casos muy específicos) */
.product-card {
  border-radius: 8px;
  overflow: hidden;
}
</style>
```

### 3. **Consumo de APIs con Axios Global**
```javascript
// ✅ CORRECTO: Usar la instancia global
export default {
  methods: {
    async fetchProducts() {
      try {
        const response = await this.$axios.get('/api/products/');
        return response.data;
      } catch (error) {
        console.error('Error fetching products:', error);
      }
    }
  }
}
```

## 🧪 Testing Strategy

### Regla Fundamental: NO MOCKS
- **Prohibido**: `jest.mock()`, `jest.fn()`, mocks de dependencias
- **Permitido**: Tests de integración real, tests de componentes reales
- **Alternativas**: Usar datos reales, componentes reales, store real

### Ejemplo de Test Sin Mocks
```javascript
// ✅ CORRECTO: Test sin mocks
import { mount } from '@vue/test-utils';
import { createStore } from 'vuex';
import ProductCard from '@/components/Products/ProductCard.vue';

describe('ProductCard.vue', () => {
  let store;
  
  beforeEach(() => {
    store = createStore({
      state: {
        cart: []
      },
      mutations: {
        addToCart: jest.fn()
      }
    });
  });

  it('emits add-to-cart event when button is clicked', async () => {
    const wrapper = mount(ProductCard, {
      global: {
        plugins: [store]
      },
      props: {
        product: {
          id: 1,
          name: 'Test Product',
          price: 100
        }
      }
    });

    await wrapper.find('button').trigger('click');
    expect(wrapper.emitted('add-to-cart')).toBeTruthy();
  });
});
```

## 🐳 Docker y Despliegue

### Dockerfile de Desarrollo
```dockerfile
FROM node:21.6.0-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --legacy-peer-deps
COPY . .
EXPOSE 5173
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

### Docker Compose
```yaml
version: '3'
services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5173:5173"
    volumes:
      - .:/app
    networks:
      - enid_service_network

networks:
  enid_service_network:
    external: true
```

## 📖 Documentación Adicional

Para información detallada sobre cada aspecto, consulta:
- [Patrones de Componentes](./docs/COMPONENT_PATTERNS.md)
- [Guía de TDD](./docs/TDD_GUIDE.md)
- [Guía de Docker](./docs/DOCKER_GUIDE.md)
- [Arquitectura](./docs/ARCHITECTURE.md)

## 🚀 Comandos de Desarrollo

```bash
# Instalar dependencias
npm install

# Desarrollo con hot reload
npm run dev

# Build de producción
npm run build

# Ejecutar tests
npm run test

# Ejecutar tests en modo watch
npm run test:watch

# Linting
npm run lint

# Formateo de código
npm run format
```

## ⚠️ Reglas Críticas a Recordar

1. **Siempre usar `@/` para imports** - NUNCA rutas relativas
2. **Solo Tailwind CSS** - NO CSS directo excepto casos muy específicos
3. **Usar Axios global** - NUNCA crear nuevas instancias de Axios
4. **NO MOCKS en tests** - Tests de integración reales
5. **TDD obligatorio** - Red-Green-Refactor para todo desarrollo
6. **Componentes reutilizables** - Seguir filosofía DRY de Vue 