<template>
  <v-container>
    <h1 class="text-h4 mb-4">
      Inventario
    </h1>

    <!-- Barra de búsqueda y botón de agregar -->
    <v-row align="center" justify="space-between">
      <v-col cols="12" md="6">
        <v-text-field v-model="search" label="Buscar producto" append-icon="mdi-magnify" solo />
      </v-col>
      <v-col cols="12" md="3" class="text-right">
        <v-btn color="primary" @click="openDialog">
          <v-icon left>
            mdi-plus
          </v-icon>
          Agregar Producto
        </v-btn>
        <v-btn color="secondary" class="ml-2" @click="filtrarPorCategoria">
          <v-icon left>
            mdi-filter-variant
          </v-icon>
          Filtrar
        </v-btn>
      </v-col>
    </v-row>

    <!-- Tabla resumen del inventario -->
    <v-card class="mb-6">
      <v-card-title>Inventario General</v-card-title>
      <v-simple-table>
        <thead>
          <tr>
            <th>Categoría</th>
            <th>Total de productos</th>
            <th>Lo más vendido</th>
            <th>Existencias bajas</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="resumen in resumenInventario" :key="resumen.categoria">
            <td>{{ resumen.categoria }}</td>
            <td>{{ resumen.total }}</td>
            <td>{{ resumen.masVendido }}</td>
            <td>{{ resumen.bajoStock }}</td>
          </tr>
        </tbody>
      </v-simple-table>
    </v-card>

    <!-- Tabla de productos -->
    <v-data-table
      :headers="headers"
      :items="productosFiltrados"
      class="elevation-1"
    >
      <template #[`item.disponibilidad`]="{ item }">
        <v-chip :color="item.disponibilidad ? 'green' : 'red'" dark>
          {{ item.disponibilidad ? 'Disponible' : 'Agotado' }}
        </v-chip>
      </template>

      <template #[`item.actions`]="{ item }">
        <v-icon small class="mr-2" @click="editarProducto(item)">
          mdi-pencil
        </v-icon>
      </template>
    </v-data-table>

    <!-- Modal para agregar/editar producto -->
    <v-dialog v-model="dialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h6">{{ editMode ? 'Editar Producto' : 'Agregar Producto' }}</span>
        </v-card-title>
        <v-card-text>
          <v-form ref="form">
            <v-text-field v-model="productoActual.imagen" label="URL de Imagen" required />
            <v-text-field v-model="productoActual.nombre" label="Nombre del producto" required />
            <v-text-field v-model="productoActual.id" label="ID del producto" readonly />
            <v-select v-model="productoActual.categoria" :items="categorias" label="Categoría" required />
            <v-text-field v-model="productoActual.cantidadPorPaquete" label="Cantidad por paquete" type="number" required />
            <v-text-field v-model="productoActual.unidades" label="Unidades (paquetes)" type="number" required />
            <v-text-field v-model="productoActual.fechaCaducidad" label="Fecha de caducidad" type="date" required />
            <v-switch v-model="productoActual.disponibilidad" label="¿Disponible?" />
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="cerrarDialog">
            Cancelar
          </v-btn>
          <v-btn color="primary" @click="guardarProducto">
            {{ editMode ? 'Actualizar' : 'Agregar' }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  data () {
    return {
      search: '',
      dialog: false,
      editMode: false,
      categorias: ['Limpieza', 'Alimentos', 'Electrodomésticos'],
      resumenInventario: [
        { categoria: 'Limpieza', total: 20, masVendido: 'Cloro', bajoStock: 3 },
        { categoria: 'Alimentos', total: 50, masVendido: 'Arroz', bajoStock: 5 },
        { categoria: 'Electrodomésticos', total: 10, masVendido: 'Licuadora', bajoStock: 2 }
      ],
      headers: [
        { text: 'Producto', value: 'nombre' },
        { text: 'Precio', value: 'precio' },
        { text: 'Cantidad por paquete', value: 'cantidadPorPaquete' },
        { text: 'Límite por pedido', value: 'limitePedido' },
        { text: 'Caducidad', value: 'fechaCaducidad' },
        { text: 'Disponibilidad', value: 'disponibilidad' },
        { text: 'Acciones', value: 'actions', sortable: false }
      ],
      productos: [],
      productoActual: {
        id: '',
        nombre: '',
        imagen: '',
        categoria: '',
        cantidadPorPaquete: '',
        unidades: '',
        fechaCaducidad: '',
        disponibilidad: true,
        precio: '',
        limitePedido: ''
      }
    }
  },
  computed: {
    productosFiltrados () {
      if (!this.search) { return this.productos }
      return this.productos.filter(p => p.nombre.toLowerCase().includes(this.search.toLowerCase()))
    }
  },
  methods: {
    openDialog () {
      this.editMode = false
      this.dialog = true
      this.resetProductoActual()
    },
    cerrarDialog () {
      this.dialog = false
    },
    editarProducto (item) {
      this.editMode = true
      this.dialog = true
      this.productoActual = { ...item }
    },
    guardarProducto () {
      if (this.editMode) {
        const index = this.productos.findIndex(p => p.id === this.productoActual.id)
        if (index !== -1) { this.productos.splice(index, 1, { ...this.productoActual }) }
      } else {
        this.productoActual.id = Date.now().toString()
        this.productos.push({ ...this.productoActual })
      }
      this.cerrarDialog()
    },
    filtrarPorCategoria () {
      // lógica de filtrado opcional aquí
      alert('Filtrar por categoría (pendiente)')
    },
    resetProductoActual () {
      this.productoActual = {
        id: '',
        nombre: '',
        imagen: '',
        categoria: '',
        cantidadPorPaquete: '',
        unidades: '',
        fechaCaducidad: '',
        disponibilidad: true,
        precio: '',
        limitePedido: ''
      }
    }
  }
}
</script>

    <style scoped>
    .v-data-table {
      font-size: 14px;
    }
    </style>
