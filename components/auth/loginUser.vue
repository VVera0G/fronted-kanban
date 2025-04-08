<template>
  <v-card max-width="400" class="mx-auto" color="#FFF2F2" elevation="0">
    <v-card-title>Iniciar Sesión</v-card-title>
    <v-card-text>
      <v-form ref="form">
        <v-text-field
          v-model="user.usuario"
          label="Usuario"
          outlined
          required
        />
        <v-text-field
          v-model="user.password"
          label="Password"
          outlined
          required
          type="password"
        />
      </v-form>
    </v-card-text>
    <v-card-actions>
      <v-btn type="submit" color="#2D336B" @click="login">
        <div style="color: white !important; text-transform: none;">
          Ingresar
          <v-icon>
            mdi-login
          </v-icon>
        </div>
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
export default {
  data () {
    return {
      user: {
        usuario: '',
        password: ''
      }
    }
  },
  methods: {
    async login () {
      try {
        await this.$auth.loginWith('local', { data: this.user })
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
  </style>
