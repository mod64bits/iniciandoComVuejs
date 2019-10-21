<template>
  <v-container fill-height>
    <v-layout
      justify-center
      align-center
    >
      <v-flex
        xs12
        sm6
        md4
        lg3
        x13
      >
        <v-card class="elevation-15">
          <v-toolbar
            color="primary"
            dark
          >
            <v-toolbar-title>
              {{ texts.toolbar }}
            </v-toolbar-title>
          </v-toolbar>
          <v-card-text>
            <v-form @submit="salvar">
              <v-text-field
                v-if="!isLogin"
                prepend-icon="person"
                name="first_name"
                label="Primeiro Nome"
                type="text"
                :error-messages="first_nameErros"
                :success="!$v.user.first_name.$invalid"
                v-model.trim="$v.user.first_name.$model"
              ></v-text-field>

              <v-text-field
                v-if="!isLogin"
                prepend-icon="person"
                name="last_name"
                label="Último Nome"
                type="text"
                :error-messages="last_nameErros"
                :success="!$v.user.last_name.$invalid"
                v-model.trim="$v.user.last_name.$model"
              ></v-text-field>

              <v-text-field
                v-if="!isLogin"
                prepend-icon="email"
                name="email"
                label="Email"
                type="email"
                :error-messages="emailErros"
                :success="!$v.user.email.$invalid"
                v-model.trim="$v.user.email.$model"
              ></v-text-field>

              <v-text-field
                prepend-icon="person"
                name="username"
                label="Nome de Usúario"
                type="text"
                :error-messages="usernameErros"
                :success="!$v.user.username.$invalid"
                v-model.trim="$v.user.username.$model"
              ></v-text-field>

              <v-text-field
                prepend-icon="lock"
                name="password"
                label="Senha"
                type="password"
                :error-messages="passwordErros"
                :success="!$v.user.password.$invalid"
                v-model.trim="$v.user.password.$model"
              ></v-text-field>
            </v-form>
            <v-btn
              block
              depressed
              color="secondary"
              @click="isLogin = !isLogin"
            >
              {{ texts.button }}
            </v-btn>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              :disabled="$v.$invalid"
              color="primary"
              large
              @click="salvar"
            >{{texts.toolbar}}</v-btn>
          </v-card-actions>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import { required, email, minLength } from 'vuelidate/lib/validators'
import confUrl from '../../../config/config'
import axios from 'axios'
export default {
  name: 'Login',
  data: () => ({
    isLogin: true,
    user: {
      first_name: '',
      last_name: '',
      username: '',
      email: '',
      password: ''
    }
  }),
  validations () {
    const validations = {
      user: {
        username: {
          required,
          minLength: minLength(3)
        },
        password: {
          required,
          minLength: minLength(6)
        }
      }
    }
    if (this.isLogin) { return validations }

    return {
      user: {
        ...validations.user,
        first_name: {
          required,
          minLength: minLength(3)
        },
        last_name: {
          required,
          minLength: minLength(3)
        },
        email: {
          required,
          email,
          minLength: minLength(3)
        }

      }
    }
  },
  computed: {
    texts () {
      return this.isLogin
        ? { toolbar: 'Entrar', button: 'Criar conta' }
        : { toolbar: 'Criar Conta', button: 'Já Tenho uma conta' }
    },
    first_nameErros () {
      const errors = []
      const firstName = this.$v.user.first_name
      if (!firstName.$dirty) { return errors }
      !firstName.required && errors.push('O Sobrenome é Obrigatorio')
      !firstName.minLength && errors.push(`Insira Pelomenos ${firstName.$params.minLength.min} caracteres!`)

      return errors
    },
    last_nameErros () {
      const errors = []
      const lastName = this.$v.user.last_name
      if (!lastName.$dirty) { return errors }
      !lastName.required && errors.push('O Sobrenome é Obrigatorio')
      !lastName.minLength && errors.push(`Insira Pelomenos ${lastName.$params.minLength.min} caracteres!`)

      return errors
    },
    usernameErros () {
      const errors = []
      const username = this.$v.user.username
      if (!username.$dirty) { return errors }
      !username.required && errors.push('O Nome de Usúario é Obrigatório')
      !username.minLength && errors.push(`Insira Pelomenos ${username.$params.minLength.min} caracteres!`)

      return errors
    },
    emailErros () {
      const errors = []
      const email = this.$v.user.email
      if (!email.$dirty) { return errors }
      !email.required && errors.push('Email é Obrigatório')
      !email.email && errors.push('Insira um Email válido!')

      return errors
    },
    passwordErros () {
      const errors = []
      const password = this.$v.user.password
      if (!password.$dirty) { return errors }
      !password.required && errors.push('Senha é Obrigatório')
      !password.minLength && errors.push(`Insira Pelomenos ${password.$params.minLength.min} caracteres!`)

      return errors
    }
  },
  methods: {
    log () {
      console.log('Vuelidate', this.$v)
    },
    submit () {
      console.log('User: ', this.user)
    },
    async  getUser () {
      let dataUser = await axios.get('http://localhost:8000/user/users/')
      console.log(dataUser)
    },
    salvar (event) {
      event.preventDefault()
      let currentObj = this
      axios.post(`${confUrl.apiUrl}user/users/`, {
        username: this.user.username,
        first_name: this.user.first_name,
        last_name: this.user.last_name,
        email: this.user.email,
        password: this.user.password

      })
        .then((response) => {
          console.log('Post Novo User', response)
          if (response.status === 201) {
            this.user.first_name = ''
            this.user.last_name = ''
            this.user.email = ''
            this.isLogin = !this.isLogin
          }
          currentObj.output = response.data
        })
        .catch((error) => {
          currentObj.output = error
        })
    }
  }

}
</script>
