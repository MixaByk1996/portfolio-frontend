<template>
  <v-app>
    <v-card v-show="is_viewed" >
      <div class="d-flex align-center justify-center" style="height: 100vh">
        <v-sheet width="400" class="mx-auto">
          <v-form fast-fail  @submit.prevent="form">
            <h3 align="center">Войти в систему</h3>
            <v-text-field  variant="underlined" v-model="login" label="Логин"></v-text-field>

            <v-text-field variant="underlined" v-model="password" :type="show_password ? 'text' : 'password' " label="Пароль" :append-icon="show_password ?'mdi-eye':'mdi-eye-off'"  @click:append="show_password=!show_password" ></v-text-field>

            <v-btn type="submit" variant="outlined" color="primary" block class="mt-2">Войти</v-btn>

          </v-form>
        </v-sheet>
      </div>
    </v-card>
  </v-app>
</template>

<script>

import {axios} from 'axios';
export default {

  name: 'IndexPage',
  data() {
    return {
      is_viewed: false,
      login: '',
      password: '',
      show_password : false
    };
  },
  beforeMount() {
    this.$axios.get('/has-address')
      .then((response) =>{
        this.is_viewed = true
      })
      .catch((response) => {
        alert('Ваш IP аресс не внесен в список белых адрессов')
      })
  },
  methods: {
    async form() {
      // await this.$axios.get('/sanctum/csrf-cookie');
      var fm = new FormData();
      fm.append('login', this.login);
      fm.append('password', this.password);
      fm.append('device_name', 'browser');
      await this.$axios.post('/login', fm)
        .then((response) => {
          console.log(response.data);
          localStorage.setItem('token', response.data);
          this.$router.push('/dashboard')
        })
        .catch((error) => {
          console.log(error)
          alert('Неверный логин/пароль')
        })
    },
  },


}
</script>
