<template>
  <v-container fluid>
    <v-row>
      <v-col cols="12">
        <v-text-field
          v-model="busqueda"
          label="Buscar producto"
          prepend-icon="mdi-magnify"
          solo
          clearable
          @input="filtrarProductos"
        />
      </v-col>

      <v-col cols="12">
        <v-card>
          <v-card-title>Inventario General</v-card-title>
          <v-simple-table>
            <thead>
              <tr>
                <th>Categoría</th>
                <th>Total de Productos</th>
                <th>Más Vendido</th>
                <th>Existencias Bajas</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>{{ resumen.categorias.join(', ') }}</td>
                <td>{{ resumen.total }}</td>
                <td>{{ resumen.masVendido }}</td>
                <td>{{ resumen.existenciasBajas }}</td>
              </tr>
            </tbody>
          </v-simple-table>
        </v-card>
      </v-col>

      <v-col cols="12">
        <v-card>
          <v-card-title class="d-flex justify-space-between">
            <span>Productos</span>
            <div>
              <v-btn color="primary" @click="abrirDialogo">
                Agregar Producto
              </v-btn>
              <v-btn @click="filtrarPorCategoria">
                Filtrar por Categoría
              </v-btn>
            </div>
          </v-card-title>

          <v-data-table
            :headers="headers"
            :items="productosFiltrados"
            item-value="id"
            class="elevation-1"
          >
            <!-- Imagen del producto -->
            <template #item.imagen="{ item }">
              <v-img :src="item.imagen" max-width="75" max-height="75" contain />
            </template>

            <!-- Disponibilidad con color -->
            <template #item.disponibilidad="{ item }">
              <v-chip :color="item.disponibilidad === 'Disponible' ? 'green' : 'red'" dark>
                {{ item.disponibilidad }}
              </v-chip>
            </template>

            <!-- Botones de acción -->
            <template #item.acciones="{ item }">
              <v-btn icon @click="editarProducto(item)">
                <v-icon>mdi-pencil</v-icon>
              </v-btn>
            </template>
          </v-data-table>
        </v-card>
      </v-col>

      <v-dialog v-model="dialogo" max-width="600px">
        <v-card>
          <v-card-title>{{ editando ? 'Editar Producto' : 'Agregar Producto' }}</v-card-title>
          <v-card-text>
            <v-form>
              <v-text-field v-model="producto.nombre" label="Nombre del Producto" />
              <v-text-field v-model="producto.imagen" label="URL de la Imagen" />
              <v-img :src="producto.imagen" max-height="150" max-width="150" contain class="my-2" />
              <v-text-field v-model="producto.id" label="ID del Producto" />
              <v-text-field v-model="producto.categoria" label="Categoría" />
              <v-text-field v-model="producto.precio" label="Precio" type="number" />
              <v-text-field v-model="producto.cantidadPorPaquete" label="Cantidad por Paquete" type="number" />
              <v-text-field v-model="producto.unidades" label="Unidades" type="number" />
              <v-text-field v-model="producto.fechaCaducidad" label="Fecha de Caducidad" type="date" />
              <v-btn-toggle v-model="producto.disponibilidad" mandatory class="mt-4">
                <v-btn value="Disponible" color="green" dark>
                  Disponible
                </v-btn>
                <v-btn value="No disponible" color="red" dark>
                  No disponible
                </v-btn>
              </v-btn-toggle>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer />
            <v-btn color="blue darken-1" text @click="guardarProducto">
              Guardar
            </v-btn>
            <v-btn color="red darken-1" text @click="cerrarDialogo">
              Cancelar
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-row>
  </v-container>
</template>

<script>
export default {
  layout: 'default',
  data () {
    return {
      busqueda: '',
      dialogo: false,
      editando: false,
      productos: [],
      producto: {
        nombre: '',
        imagen: '',
        id: '',
        categoria: '',
        precio: '',
        cantidadPorPaquete: '',
        unidades: '',
        fechaCaducidad: '',
        disponibilidad: 'Disponible'
      },
      resumen: {
        categorias: ['Alimentos', 'Limpieza'],
        total: 0,
        masVendido: 'Producto X',
        existenciasBajas: 0
      },
      headers: [
        { text: 'Imagen', value: 'imagen' },
        { text: 'Nombre', value: 'nombre' },
        { text: 'Precio', value: 'precio' },
        { text: 'Cantidad por Paquete', value: 'cantidadPorPaquete' },
        { text: 'Unidades', value: 'unidades' },
        { text: 'Fecha de Caducidad', value: 'fechaCaducidad' },
        { text: 'Disponibilidad', value: 'disponibilidad' },
        { text: 'Acciones', value: 'acciones', sortable: false }
      ]
    }
  },
  computed: {
    productosFiltrados () {
      return this.productos.filter(p =>
        p.nombre.toLowerCase().includes(this.busqueda.toLowerCase())
      )
    }
  },
  mounted () {
    this.obtenerProductos()
  },
  methods: {
    abrirDialogo () {
      this.dialogo = true
      this.editando = false
      this.producto = {
        nombre: '', imagen: '', id: '', categoria: '', precio: '', cantidadPorPaquete: '', unidades: '', fechaCaducidad: '', disponibilidad: 'Disponible'
      }
    },
    cerrarDialogo () {
      this.dialogo = false
    },
    async guardarProducto () {
      try {
        const productoParaEnviar = {
          id: this.producto.id,
          name: this.producto.nombre, // <- asegúrate que en el frontend es `nombre`
          imageUrl: this.producto.imagen, // <- asegúrate que así lo tienes
          category: this.producto.categoria,
          price: Number(this.producto.precio),
          packageQuantity: Number(this.producto.cantidadPorPaquete),
          units: Number(this.producto.unidades),
          expirationDate: this.producto.fechaCaducidad,
          availability: this.producto.disponible
        }

        const response = await this.$axios.post('http://localhost:3000/api/products/create', productoParaEnviar)
        console.log('Producto guardado:', response.data)
        this.obtenerProductos() // recarga los productos
      } catch (error) {
        console.error('Error al guardar producto:', error)
      }
    },
    editarProducto (item) {
      this.producto = { ...item }
      this.dialogo = true
      this.editando = true
    },
    filtrarProductos () {
      // Esto ya está integrado con computed
    },
    filtrarPorCategoria () {
      // Lógica de filtrado futura
    },
    mounted () {
      this.obtenerProductos()
    },
    async obtenerProductos () {
      try {
        const response = await this.$axios.get('http://localhost:3000/api/products')
        this.productos = response.data
        this.resumen.total = this.productos.length
        this.resumen.categorias = [...new Set(this.productos.map(p => p.categoria))]
        this.resumen.masVendido = this.productos[0]?.nombre || 'N/A'
        this.resumen.existenciasBajas = this.productos.filter(p => p.unidades < 5).length
      } catch (error) {
        console.error('Error al obtener productos:', error)
      }
    }
  }
}
</script>
