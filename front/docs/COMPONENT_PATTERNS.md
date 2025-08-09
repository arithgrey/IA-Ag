# Patrones de Componentes Vue

## 🎯 Principios SOLID en Vue

### 1. **Single Responsibility Principle**

Cada componente debe tener una sola responsabilidad. Ejemplo del proyecto real:

```vue
<!-- ✅ CORRECTO: ProductCard.vue - Solo maneja la visualización del producto -->
<template>
  <div class="product-card bg-white shadow-lg hover:shadow-xl transition-all duration-300">
    <div class="relative">
      <img 
        :src="product.image" 
        :alt="product.name"
        class="w-full h-48 object-cover"
        @click="trackProductView"
      />
      <div class="absolute top-2 right-2">
        <button 
          @click="addToCart"
          class="bg-blue-600 text-white p-2 rounded-full hover:bg-blue-700 transition-colors"
        >
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4m0 0L7 13m0 0l-2.5 5M7 13l2.5 5m6-5v6a2 2 0 01-2 2H9a2 2 0 01-2-2v-6m6 0V9a2 2 0 00-2-2H9a2 2 0 00-2 2v4.01"></path>
          </svg>
        </button>
      </div>
    </div>
    
    <div class="p-4">
      <h3 class="text-lg font-semibold text-gray-800 mb-2">{{ product.name }}</h3>
      <p class="text-gray-600 text-sm mb-3">{{ product.description }}</p>
      
      <div class="flex justify-between items-center">
        <span class="text-xl font-bold text-blue-600">${{ product.price }}</span>
        <button 
          @click="trackButtonClick('view_details')"
          class="text-blue-600 hover:text-blue-800 text-sm font-medium"
        >
          Ver detalles
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProductCard',
  props: {
    product: {
      type: Object,
      required: true
    }
  },
  methods: {
    trackProductView() {
      this.trackProductView(
        this.product.id,
        this.product.name,
        this.product.category
      )
    },
    
    addToCart() {
      this.trackAddToCart(
        this.product.id,
        this.product.name,
        1,
        this.product.price
      )
      this.$emit('add-to-cart', this.product)
    },
    
    trackButtonClick(buttonName) {
      this.trackButtonClick(buttonName)
    }
  }
}
</script>
```

### 2. **Open/Closed Principle**

Los componentes deben ser extensibles sin modificación. Usar props y slots:

```vue
<!-- ✅ CORRECTO: Componente extensible con slots -->
<template>
  <div class="bg-white rounded-lg shadow-md p-6">
    <div class="flex items-center justify-between mb-4">
      <h3 class="text-lg font-semibold text-gray-900">{{ title }}</h3>
      <slot name="header-actions"></slot>
    </div>
    
    <div class="space-y-4">
      <slot></slot>
    </div>
    
    <div class="mt-6">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CardContainer',
  props: {
    title: {
      type: String,
      required: true
    }
  }
}
</script>
```

### 3. **Liskov Substitution Principle**

Los componentes deben ser intercambiables. Ejemplo del proyecto:

```vue
<!-- ✅ CORRECTO: Componentes intercambiables para diferentes tipos de productos -->
<template>
  <div>
    <component 
      :is="current_product_video" 
      :product="product"
      @product-clicked="handleProductClick"
    />
  </div>
</template>

<script>
import ProductHalloween from '@/components/Video/ProductHalloween.vue'
import ProductChristmas from '@/components/Video/ProductChristmas.vue'
import ProductRunning from '@/components/Video/ProductRunning.vue'

export default {
  components: {
    ProductHalloween,
    ProductChristmas,
    ProductRunning
  },
  data() {
    return {
      current_product_video: 'ProductRunning',
    }
  },
  methods: {
    updateSeasonalContent() {
      const seasonalContent = getSeasonalContent()
      this.current_product_video = seasonalContent.product_offert
    }
  }
}
</script>
```

## 🔄 Patrones de Comunicación

### 1. **Props Down / Events Up**

```vue
<!-- ✅ CORRECTO: Comunicación padre-hijo -->
<template>
  <div>
    <ProductDetail 
      @open_shopping_cart_product="handle_open_shoping_cart"
      @open_config_product="handler_open_config_product"
    />
  </div>
</template>

<script>
import ProductDetail from '@/components/Products/ProductDetail.vue'

export default {
  components: {
    ProductDetail
  },
  methods: {
    handle_open_shoping_cart() {
      this.$emit("open_shopping_cart_product_list")
    },
    handler_open_config_product(product) {
      this.$emit("open_open_config_product", product)
    }
  }
}
</script>
```

### 2. **Provide/Inject**

Para comunicación entre componentes anidados profundamente:

```vue
<!-- ✅ CORRECTO: Provide en componente padre -->
<template>
  <div>
    <ProductList />
  </div>
</template>

<script>
import ProductList from '@/components/Products/ProductList.vue'

export default {
  components: {
    ProductList
  },
  provide() {
    return {
      storeId: this.$store.getters.storeId,
      user: this.$store.getters.user
    }
  }
}
</script>
```

```vue
<!-- ✅ CORRECTO: Inject en componente hijo -->
<template>
  <div>
    <p>Store ID: {{ storeId }}</p>
    <p>Usuario: {{ user?.name }}</p>
  </div>
</template>

<script>
export default {
  inject: ['storeId', 'user']
}
</script>
```

### 3. **Event Bus (Legacy - Solo para casos específicos)**

```javascript
// ✅ CORRECTO: Event Bus para comunicación global
// src/eventBus.js
import { ref } from 'vue'

const bus = ref(new Map())

export default {
  emit(event, ...args) {
    if (bus.value.has(event)) {
      bus.value.get(event).forEach(callback => callback(...args))
    }
  },
  
  on(event, callback) {
    if (!bus.value.has(event)) {
      bus.value.set(event, [])
    }
    bus.value.get(event).push(callback)
  },
  
  off(event, callback) {
    if (bus.value.has(event)) {
      const callbacks = bus.value.get(event)
      const index = callbacks.indexOf(callback)
      if (index > -1) {
        callbacks.splice(index, 1)
      }
    }
  }
}
```

## 🎨 Patrones de Estilos

### 1. **Tailwind CSS como Sistema Principal**

```vue
<!-- ✅ CORRECTO: Solo clases de Tailwind -->
<template>
  <div class="bg-white shadow-lg hover:shadow-xl transition-all duration-300 rounded-lg overflow-hidden">
    <div class="p-6">
      <h2 class="text-2xl font-bold text-gray-900 mb-4">Título del Componente</h2>
      <p class="text-gray-600 leading-relaxed mb-6">
        Descripción del componente usando solo clases de Tailwind.
      </p>
      <div class="flex space-x-4">
        <button class="bg-blue-600 hover:bg-blue-700 text-white font-medium py-2 px-4 rounded-md transition-colors duration-200">
          Botón Principal
        </button>
        <button class="bg-gray-200 hover:bg-gray-300 text-gray-800 font-medium py-2 px-4 rounded-md transition-colors duration-200">
          Botón Secundario
        </button>
      </div>
    </div>
  </div>
</template>
```

### 2. **CSS Personalizado Solo para Casos Específicos**

```vue
<!-- ✅ CORRECTO: CSS personalizado solo cuando es necesario -->
<template>
  <input 
    type="text" 
    class="input-cart"
    placeholder="Ingresa tu código postal"
  />
</template>

<style scoped>
/* Solo para casos muy específicos que Tailwind no puede manejar */
.input-cart {
  @apply text-black placeholder-gray-600 w-full px-4 py-2.5 mt-2 text-base transition duration-500 ease-in-out transform bg-gray-50 focus:bg-white focus:outline-none ring ring-offset-current ring-offset-2 ring-gray-300;
}
</style>
```

## 🔌 Patrones de Integración con APIs

### 1. **Uso de Axios Global**

```vue
<!-- ✅ CORRECTO: Usar la instancia global de Axios -->
<template>
  <div>
    <div v-if="loading" class="text-center py-8">
      <p class="text-gray-500">Cargando productos...</p>
    </div>
    <div v-else-if="error" class="text-center py-8">
      <p class="text-red-500">{{ error }}</p>
    </div>
    <div v-else>
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <ProductCard 
          v-for="product in products" 
          :key="product.id" 
          :product="product"
          @add-to-cart="handleAddToCart"
        />
      </div>
    </div>
  </div>
</template>

<script>
import ProductCard from '@/components/Products/ProductCard.vue'

export default {
  components: {
    ProductCard
  },
  data() {
    return {
      products: [],
      loading: false,
      error: null
    }
  },
  async mounted() {
    await this.fetchProducts()
  },
  methods: {
    async fetchProducts() {
      this.loading = true
      this.error = null
      
      try {
        // ✅ CORRECTO: Usar la instancia global de Axios
        const response = await this.$axios.get('/api/products/')
        this.products = response.data
      } catch (error) {
        this.error = 'Error al cargar los productos'
        console.error('Error fetching products:', error)
      } finally {
        this.loading = false
      }
    },
    
    handleAddToCart(product) {
      this.$store.commit('addToCart', product)
    }
  }
}
</script>
```

### 2. **Manejo de Estados de Carga y Error**

```vue
<!-- ✅ CORRECTO: Estados de carga y error bien manejados -->
<template>
  <div>
    <!-- Estado de carga -->
    <div v-if="loading" class="flex justify-center items-center py-12">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
    </div>
    
    <!-- Estado de error -->
    <div v-else-if="error" class="text-center py-8">
      <div class="bg-red-50 border border-red-200 rounded-md p-4">
        <svg class="mx-auto h-12 w-12 text-red-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L3.732 16.5c-.77.833.192 2.5 1.732 2.5z" />
        </svg>
        <h3 class="mt-2 text-sm font-medium text-red-800">Error al cargar datos</h3>
        <p class="mt-1 text-sm text-red-700">{{ error }}</p>
        <div class="mt-4">
          <button 
            @click="retry"
            class="bg-red-100 hover:bg-red-200 text-red-800 font-medium py-2 px-4 rounded-md transition-colors duration-200"
          >
            Reintentar
          </button>
        </div>
      </div>
    </div>
    
    <!-- Estado de éxito -->
    <div v-else>
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AsyncWrapper',
  props: {
    loading: {
      type: Boolean,
      default: false
    },
    error: {
      type: String,
      default: null
    }
  },
  emits: ['retry'],
  methods: {
    retry() {
      this.$emit('retry')
    }
  }
}
</script>
```

## 🧩 Patrones de Composición

### 1. **Composables para Lógica Reutilizable**

```javascript
// ✅ CORRECTO: Composable para manejo de productos
// src/composables/useProducts.js
import { ref, computed } from 'vue'
import { useStore } from 'vuex'

export function useProducts() {
  const store = useStore()
  const products = ref([])
  const loading = ref(false)
  const error = ref(null)

  const fetchProducts = async () => {
    loading.value = true
    error.value = null
    
    try {
      const response = await store.$axios.get('/api/products/')
      products.value = response.data
    } catch (err) {
      error.value = 'Error al cargar productos'
      console.error('Error fetching products:', err)
    } finally {
      loading.value = false
    }
  }

  const addToCart = (product) => {
    store.commit('addToCart', product)
  }

  const removeFromCart = (product) => {
    store.commit('removeFromCart', product)
  }

  const cartItems = computed(() => store.getters.getProductsFromCart)
  const totalItems = computed(() => store.getters.totalItemsInCart)

  return {
    products,
    loading,
    error,
    fetchProducts,
    addToCart,
    removeFromCart,
    cartItems,
    totalItems
  }
}
```

### 2. **Mixins para Funcionalidad Compartida**

```javascript
// ✅ CORRECTO: Mixin para analytics (patrón del proyecto real)
// src/helpers/AnalyticsMixin.js
export default {
  methods: {
    trackProductView(productId, productName, category) {
      if (this.$analyticsTracker) {
        this.$analyticsTracker.trackProductView(productId, productName, category)
      }
    },
    
    trackAddToCart(productId, productName, quantity, price) {
      if (this.$analyticsTracker) {
        this.$analyticsTracker.trackAddToCart(productId, productName, quantity, price)
      }
    },
    
    trackButtonClick(buttonName, section = null) {
      this.trackEvent('button_click', {
        button_name: buttonName,
        section: section || this.getCurrentSection(),
        page_url: this.$route.path
      })
    },
    
    trackEvent(eventType, data = {}) {
      if (this.$analyticsTracker) {
        this.$analyticsTracker.trackEvent(eventType, data)
      }
    },
    
    getCurrentSection() {
      const path = this.$route.path
      if (path.includes('/product-detail')) return 'product_detail'
      if (path.includes('/checkout')) return 'checkout'
      if (path.includes('/cart')) return 'shopping_cart'
      if (path.includes('/search')) return 'search_results'
      return 'other'
    }
  }
}
```

## 📱 Patrones de Responsive Design

### 1. **Grid Responsive con Tailwind**

```vue
<!-- ✅ CORRECTO: Grid responsive usando Tailwind -->
<template>
  <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4 md:gap-6 lg:gap-8">
    <ProductCard 
      v-for="product in products" 
      :key="product.id" 
      :product="product"
      class="w-full"
    />
  </div>
</template>
```

### 2. **Flexbox Responsive**

```vue
<!-- ✅ CORRECTO: Flexbox responsive -->
<template>
  <div class="flex flex-col md:flex-row lg:flex-col xl:flex-row gap-4">
    <div class="flex-1">
      <h2 class="text-xl md:text-2xl lg:text-xl xl:text-2xl font-bold text-gray-900">
        Título Responsive
      </h2>
    </div>
    <div class="flex-shrink-0">
      <button class="w-full md:w-auto bg-blue-600 text-white px-4 py-2 rounded-md">
        Acción
      </button>
    </div>
  </div>
</template>
```

## 🔒 Patrones de Seguridad

### 1. **Validación de Props**

```vue
<!-- ✅ CORRECTO: Validación robusta de props -->
<script>
export default {
  name: 'SecureComponent',
  props: {
    userId: {
      type: [String, Number],
      required: true,
      validator: function(value) {
        return value > 0 || (typeof value === 'string' && value.length > 0)
      }
    },
    permissions: {
      type: Array,
      default: () => [],
      validator: function(value) {
        return Array.isArray(value) && value.every(p => typeof p === 'string')
      }
    },
    config: {
      type: Object,
      default: () => ({}),
      validator: function(value) {
        return typeof value === 'object' && value !== null
      }
    }
  }
}
</script>
```

### 2. **Sanitización de Datos**

```vue
<!-- ✅ CORRECTO: Sanitización de datos del usuario -->
<template>
  <div>
    <h1 class="text-2xl font-bold text-gray-900">
      {{ sanitizedTitle }}
    </h1>
    <p class="text-gray-600 mt-2">
      {{ sanitizedDescription }}
    </p>
  </div>
</template>

<script>
export default {
  name: 'SanitizedComponent',
  props: {
    title: {
      type: String,
      default: ''
    },
    description: {
      type: String,
      default: ''
    }
  },
  computed: {
    sanitizedTitle() {
      return this.title.replace(/[<>]/g, '')
    },
    sanitizedDescription() {
      return this.description.replace(/[<>]/g, '')
    }
  }
}
</script>
```

## 📋 Checklist de Mejores Prácticas

### ✅ **Imports y Rutas**
- [ ] Usar exclusivamente `@/` para imports
- [ ] NUNCA usar rutas relativas (`./`, `../`)
- [ ] Importar componentes con nombres descriptivos

### ✅ **Estilos**
- [ ] Usar solo clases de Tailwind CSS
- [ ] CSS personalizado solo para casos muy específicos
- [ ] Aplicar clases responsive apropiadas

### ✅ **APIs y HTTP**
- [ ] Usar la instancia global de Axios (`this.$axios`)
- [ ] Manejar estados de carga y error
- [ ] Implementar retry logic cuando sea apropiado

### ✅ **Componentes**
- [ ] Seguir principio de responsabilidad única
- [ ] Usar props para datos de entrada
- [ ] Emitir eventos para comunicación hacia arriba
- [ ] Implementar validación de props

### ✅ **Testing**
- [ ] NO usar mocks
- [ ] Escribir tests de integración reales
- [ ] Seguir ciclo TDD: Red-Green-Refactor
- [ ] Mantener cobertura mínima del 90%

### ✅ **Performance**
- [ ] Usar lazy loading para componentes pesados
- [ ] Implementar virtual scrolling para listas largas
- [ ] Optimizar re-renders con `v-memo` cuando sea apropiado
- [ ] Usar `shallowRef` para objetos grandes que no necesitan reactividad profunda 