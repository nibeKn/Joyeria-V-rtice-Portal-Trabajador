<script setup>
import { ref } from 'vue'

const isLoggedIn = ref(false)
const usernameInput = ref('')
const passwordInput = ref('')
const loginError = ref('')

const login = () => {
  if (usernameInput.value === 'Juan' && passwordInput.value === '123') {
    isLoggedIn.value = true
    loginError.value = ''
  } else {
    loginError.value = 'credenciales no validas'
  }
}

// Tab and Task State
const activeTab = ref('tareas')
const diaActivo = ref(true)
const nuevaTarea = ref('')
const tareas = ref([
  { nombre: 'Limpiar los mesones', completada: false },
  { nombre: 'Ordenar cajas nuevas', completada: false },
  { nombre: 'Revisar etiquetas nuevas en joyas', completada: false },
  { nombre: 'Actualizar inventario de mostrador', completada: false },
  { nombre: 'Limpiar vitrinas principales', completada: false },
  { nombre: 'Preparar empaques de regalo', completada: false },
  { nombre: 'Revisar correos y pedidos online', completada: false },
  { nombre: 'Realizar cuadratura de caja', completada: false }
])

// Drag and drop state
const draggedTaskIndex = ref(null)

const toggleDia = () => {
  diaActivo.value = !diaActivo.value
}

const agregarTarea = () => {
  if (nuevaTarea.value.trim() !== '') {
    tareas.value.push({
      nombre: nuevaTarea.value.trim(),
      completada: false
    })
    nuevaTarea.value = ''
  }
}

const toggleCompletada = (index) => {
  tareas.value[index].completada = !tareas.value[index].completada
}

const eliminarTarea = (index) => {
  tareas.value.splice(index, 1)
}

const onDragStart = (index, event) => {
  draggedTaskIndex.value = index
  event.dataTransfer.effectAllowed = 'move'
}

const onDrop = (index) => {
  if (draggedTaskIndex.value !== null && draggedTaskIndex.value !== index) {
    const draggedItem = tareas.value.splice(draggedTaskIndex.value, 1)[0]
    tareas.value.splice(index, 0, draggedItem)
  }
  draggedTaskIndex.value = null
}

const nombreEncargado = ref('Juan Pérez')
const tiendaAbierta = ref(true)

const nombreProducto = ref('')
const cantidadStock = ref(0)
const productos = ref([
  { nombre: 'Aro Donatello Oro 14', stock: 50 },
  { nombre: 'Cadena Plata Stefano 4mm', stock: 0 },
  { nombre: 'Pulsera de Plata Atelier 8mm', stock: 12 }
])

const toggleTienda = () => {
  tiendaAbierta.value = !tiendaAbierta.value
}

const agregarProducto = () => {
  if (nombreProducto.value.trim() === '' || cantidadStock.value === null || cantidadStock.value === '') {
    alert('Los campos nombre y stock no pueden estar vacíos.')
    return
  }
  
  productos.value.push({
    nombre: nombreProducto.value.trim(),
    stock: Number(cantidadStock.value)
  })
  
  // Limpiar campos
  nombreProducto.value = ''
  cantidadStock.value = 0
}

const disminuirStock = (index) => {
  if (productos.value[index].stock > 0) {
    productos.value[index].stock--
  }
}

const aumentarStock = (index) => {
  productos.value[index].stock++
}

const eliminarProducto = (index) => {
  productos.value.splice(index, 1)
}
</script>

<template>
  <div class="app-container">
    <!-- Login View -->
    <div v-if="!isLoggedIn" class="login-view">
      <header class="header">
        <h1 class="logo">VÉRTICE</h1>
      </header>

      <main class="main-content">
        <div class="login-card">
          <h3>Bienvenido</h3>
          <p class="login-prompt">Ingrese su nombre de usuario y contraseña para acceder a la plataforma de vértice</p>
          
          <div class="form-group">
            <label for="username">Usuario</label>
            <input id="username" type="text" v-model="usernameInput" class="input-field" placeholder="Ingrese su usuario">
          </div>
          <div class="form-group">
            <label for="password">Contraseña</label>
            <input id="password" type="password" v-model="passwordInput" class="input-field" placeholder="Ingrese su contraseña">
          </div>
          
          <p v-if="loginError" class="error-message">{{ loginError }}</p>
          
          <button @click="login" class="btn btn-primary">Ingresar</button>
        </div>
      </main>

      <footer class="footer">
        <p>VÉRTICE - Redefiniendo lo Esencial</p>
      </footer>
    </div>

    <!-- Main App View -->
    <div v-else class="app-view">
      <header class="header">
        <h1 class="logo">VÉRTICE</h1>
        <h2 class="subtitle">Bienvenido {{ nombreEncargado }}</h2>
        <div class="tabs-container">
          <button @click="activeTab = 'tareas'" class="tab-btn" :class="{ active: activeTab === 'tareas' }">Lista de Tareas</button>
          <button @click="activeTab = 'inventario'" class="tab-btn" :class="{ active: activeTab === 'inventario' }">Inventario</button>
        </div>
      </header>

      <main class="main-content">
        <!-- Task List Tab -->
        <section v-if="activeTab === 'tareas'" class="task-section">
          <div class="task-card">
            <h3>Panel de Tareas de {{ nombreEncargado }}</h3>
            
            <div class="status-indicator">
              <p v-if="diaActivo" class="status open">Día activo, ¡a trabajar!</p>
              <p v-else class="status closed">Día finalizado. Buen trabajo.</p>
              
              <button @click="toggleDia" class="btn btn-outline">
                {{ diaActivo ? 'Finalizar día' : 'Iniciar día' }}
              </button>
            </div>

            <div class="add-task-form" v-if="diaActivo">
              <form @submit.prevent="agregarTarea" class="form-row">
                <input type="text" v-model="nuevaTarea" placeholder="Añadir una nueva tarea..." class="input-field task-input">
                <button type="submit" class="btn btn-primary btn-add-task">Agregar</button>
              </form>
            </div>

            <div class="task-list-container">
              <p v-show="tareas.length === 0" class="empty-message">No hay tareas registradas</p>
              <ul class="task-list">
                <li v-for="(tarea, index) in tareas" 
                    :key="index" 
                    class="task-item"
                    :draggable="diaActivo"
                    @dragstart="diaActivo && onDragStart(index, $event)"
                    @drop="diaActivo && onDrop(index)"
                    @dragover.prevent
                    @dragenter.prevent>
                  <div v-if="diaActivo" class="drag-handle">≡</div>
                  <span class="task-content" :class="{ 'task-completed': tarea.completada }">
                    {{ index + 1 }}. {{ tarea.nombre }}
                  </span>
                  <div class="task-actions" v-if="diaActivo">
                    <button @click="toggleCompletada(index)" class="btn-check" :class="{ active: tarea.completada }">
                      ✓
                    </button>
                    <button @click="eliminarTarea(index)" class="btn-delete-task">
                      Borrar
                    </button>
                  </div>
                </li>
              </ul>
            </div>
          </div>
        </section>

        <!-- Inventory Tab -->
        <div v-else-if="activeTab === 'inventario'" class="inventory-management-wrapper">
          <section class="status-section">
            <div class="status-indicator">
              <p v-if="tiendaAbierta" class="status open">Tienda Operativa - Ingresando Stock</p>
              <p v-else class="status closed">Tienda Cerrada - Inventario Bloqueado</p>
              
              <button @click="toggleTienda" class="btn btn-outline">
                {{ tiendaAbierta ? 'Cerrar Tienda' : 'Abrir Tienda' }}
              </button>
            </div>
          </section>

          <section class="inventory-management">
            <div class="add-product-card">
              <h3>Ingresar Nuevo Producto</h3>
              <div class="form-group">
                <label for="nombre">Nombre Producto</label>
                <input id="nombre" type="text" v-model="nombreProducto" placeholder="Ej. Anillo Alma Onix" :disabled="!tiendaAbierta" class="input-field">
              </div>
              <div class="form-group">
                <label for="stock">Cantidad en Stock</label>
                <input id="stock" type="number" v-model="cantidadStock" placeholder="0" :disabled="!tiendaAbierta" class="input-field" min="0">
              </div>
              <button @click="agregarProducto" class="btn btn-primary" :disabled="!tiendaAbierta">Añadir al Inventario</button>
            </div>

            <div class="product-list-card">
              <h3>Lista de Productos</h3>
              <p v-show="productos.length === 0" class="empty-message">El inventario está vacío</p>
              
              <ul class="product-list">
                <li v-for="(producto, index) in productos" :key="index" class="product-item">
                  <span class="product-info" :class="{ 'out-of-stock': producto.stock === 0 || producto.stock === '0' }">
                    {{ index + 1 }}. {{ producto.nombre }} - Stock: {{ producto.stock }}
                  </span>
                  <div class="product-actions" v-if="tiendaAbierta">
                    <button @click="disminuirStock(index)" class="btn-action btn-decrease" :disabled="producto.stock <= 0">-</button>
                    <button @click="aumentarStock(index)" class="btn-action btn-increase">+</button>
                    <button @click="eliminarProducto(index)" class="btn-action btn-delete">Eliminar</button>
                  </div>
                </li>
              </ul>
            </div>
          </section>
        </div>
      </main>

      <footer class="footer">
        <p>VÉRTICE - Redefiniendo lo Esencial</p>
      </footer>
    </div>
  </div>
</template>
