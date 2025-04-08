<template>
  <v-app>
    <v-navigation-drawer
      permanent
      app
      color="#F8FAFC"
    >
      <v-list dense>
        <template v-for="(opcion, index) in opcionesMenu">
          <v-list-item
            v-if="!opcion.children && canAccess(opcion.roles)"
            :key="`opcion-${index}`"
            link
            @click="goTo(opcion.path)"
          >
            <v-list-item-icon>
              <v-icon color="blue">
                {{ opcion.icon }}
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>
                {{ opcion.title }}
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-list-group
            v-else-if="canAccess(opcion.roles)"
            :key="`group-${index}`"
            no-action
            prepend-icon="mdi-tools"
            append-icon="mdi-chevron-down"
          >
            <template #activator>
              <v-list-item-content>
                <v-list-item-title>
                  {{ opcion.title }}
                </v-list-item-title>
              </v-list-item-content>
            </template>
            <v-list-item
              v-for="(child, cIndex) in opcion.children"
              :key="`child-${index}-${cIndex}`"
              link
              @click="goTo(child.path)"
            >
              <v-list-item-icon>
                <v-icon color="blue darkeen-3">
                  {{ child.icon }}
                </v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title>
                  {{ child.title }}
                </v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list-group>
        </template>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-alert
        v-if="$store.state.alert.visible"
        :type="$store.state.alert.type"
        dismissible
        top
        right
        class="position-fixed"
      >
        {{ $store.state.alert.message }}
      </v-alert>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      opcionesMenu: [
        {
          title: 'Dashboard',
          icon: 'mdi-view-dashboard',
          path: '/principal',
          roles: ['admin']
        },
        {
          title: 'Inventario',
          icon: 'mdi-human-dolly',
          path: 'principal/configuracion/inventario',
          roles: ['admin', 'usuario']
        },
        {
          title: 'Reportes',
          icon: 'mdi-chart-box-multiple',
          path: '/',
          roles: ['admin', 'usuario']
        },
        {
          title: 'Proveedores',
          icon: 'mdi-account-circle-outline',
          path: '/',
          roles: ['admin', 'usuario']
        },
        {
          title: 'Ordenes',
          icon: 'mdi-package-variant-closed',
          path: '/',
          roles: ['admin', 'usuario']
        },
        {
          title: 'Gerente',
          icon: 'mdi-format-list-bulleted',
          path: '/',
          roles: ['admin', 'usuario']
        },
        {
          title: 'Configuracion',
          icon: 'mdi-tools',
          roles: ['admin'],
          children: [
            {
              title: 'Usuarios',
              icon: 'mdi-account-multiple',
              path: '/principal/configuracion/usuarios'
            },
            {
              title: 'Desbloquear',
              icon: 'mdi-account-lock-open',
              path: '/principal/configuracion/desbloquear'
            }
          ]
        },
        {
          title: 'Cerrar Sesión',
          icon: 'mdi-logout',
          path: '/',
          roles: ['admin', 'usuario']
        }
      ]
    }
  },
  methods: {
    canAccess (allowedRoles) {
      return allowedRoles.includes(this.$auth.user.rol)
    },
    goTo (route) {
      if (route === '/') {
        this.logout()
      } else {
        this.$router.push(route)
      }
    },
    async logout () {
      try {
        await this.$axios.post('/users/logout')
        window.location.href = '/'
      } catch (error) {
        this.$store.dispatch('alert/triggerAlert', {
          message: error.message,
          type: 'error'
        })
      }
    }
  }
}
</script>

  <style scoped>
  .position-fixed {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 2000;
  }
  </style>
