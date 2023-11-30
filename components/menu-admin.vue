<template>
  <v-container>
    <v-dialog v-model="modalAccessList" max-width="700">
<v-card >
  <v-card >
    <v-card-title>Добавить ip</v-card-title>
    <v-text-field
      v-model="ip_text"
      placeholder="Введите ip адрес"
    ></v-text-field>
    <v-btn @click="addAcessList">Добавить адрес</v-btn>
  </v-card>
  <v-card>
    <v-card-title>
      Список ip
      <v-spacer></v-spacer>
      <v-text-field
        v-model="search"
        append-icon="mdi-magnify"
        label="Поиск"
        single-line
        hide-details
      ></v-text-field>
    </v-card-title>
    <v-data-table
      :items="access_list_array"
      :headers="headers"
      :search="search"
      no-data-text="Данные отсувствуют"
      no-results-text="По запросу ничего не найдено"
    >
      <template v-slot:[`item.delete`]="{ item }">
        <v-btn small color="error" @click="deleteIp(item.id)" > Удалить </v-btn>
      </template>
    </v-data-table>
  </v-card>
  <v-spacer></v-spacer>
  <v-card>
    <v-row>
      <v-col></v-col>
      <v-col></v-col>
      <v-col></v-col>
      <v-col><v-btn align="center" @click="modalAccessList = false">Выйти</v-btn></v-col>
    </v-row>
  </v-card>
</v-card>
    </v-dialog>

    <v-dialog v-model="modalCreateUserForm" max-width="700">
      <v-card>
        <v-card-title>Добавить пользователя</v-card-title>
          <v-row>
            <v-col>
              <v-text-field
                v-model="form_create_user.login"
                label="Логин"
                single-line
                hide-details
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field
                v-model="form_create_user.password"
                label="Пароль"
                :type="is_hide_password ? 'text' : 'password' "
                :append-icon="is_hide_password ?'mdi-eye':'mdi-eye-off'"
                @click:append="is_hide_password=!is_hide_password"
                single-line
                hide-details
              ></v-text-field>
            </v-col>
            <v-col>
              <v-switch
                v-model="model_switch_rules"
                hide-details
                :label="model_switch_rules ? 'Администратор' : 'Пользователь'"
              ></v-switch>
            </v-col>
        </v-row>
        <v-row></v-row>
        <v-spacer></v-spacer>
        <v-card-actions>
          <v-btn @click="createUser()">Добавить</v-btn>
          <v-btn @click="modalCreateUserForm = false">Выйти</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="modalUsersList" max-width="700">
      <v-card>
        <v-card-title>
          Список пользователей
          <v-spacer></v-spacer>
          <v-text-field
            v-model="searchUser"
            append-icon="mdi-magnify"
            label="Поиск"
            single-line
            hide-details
          ></v-text-field>
        </v-card-title>
        <v-data-table
          :items="list_users"
          :headers="headersUsersTable"
          :search="searchUser"
          no-data-text="Данные отсувствуют"
          no-results-text="По запросу ничего не найдено"
        >
<!--          <template v-slot:[`item.delete`]="{ item }">-->
<!--            <v-btn small color="error" @click="deleteIp(item.id)" > Удалить пользователя </v-btn>-->
<!--          </template>-->
        </v-data-table>
      </v-card>
    </v-dialog>

    <v-row>
      <v-col>
          <v-btn width="240" height="100" @click="modalCreateUserForm = !modalCreateUserForm">Создать пользователя</v-btn>
      </v-col>
      <v-col>
        <v-btn width="240"  height="100" @click="getModalAccessList()">Список белых ip</v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-btn width="240" height="100">Добавить проект в дерево</v-btn>
      </v-col>
      <v-col>
        <v-btn width="240"  height="100" @click="modalUsersList = !modalUsersList">Список пользователей</v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-btn width="240" height="100">Бекап</v-btn>
      </v-col>
      <v-col>
        <v-btn width="240"  height="100">Добавить тег/фильтр</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import {axios} from 'axios';
export default {
  beforeMount() {
    this.getIPs();
    this.getUsers();
  },
  data(){
    return{
      modalAccessList : false,
      modalCreateUserForm: false,
      ip_text: "",
      search: "",
      searchUser: "",
      access_list_array: null,
      list_users: null,
      modalUsersList:false,
      model_switch_rules: false,
      is_hide_password: false,
      form_create_user:{
        login : "",
        password: "",
        rules: this.model_switch_rules ? "ADMIN" :"USER"
      },
      headers: [
        { title: 'IP адрес',
          align: 'start',
          sortable: false,
          value: 'address',
        },
        {
          text: "",
          value: "delete",
          sortable: false,
        },
      ],
      headersUsersTable: [
        {
          title: "Логин",
          align: 'start',
          value: "login",
          sortable: true,
        },
        {
          title: "Тип",
          value: "rules",
          sortable: false,
        }
      ]
    }
  },
  methods : {
    getUsers(){
      this.$axios.get('/users')
        .then(response => {
          this.list_users = response.data.data;
        })
    },
    getIPs(){
      this.$axios.get('/access-list')
        .then(response =>{
          this.access_list_array = response.data.data;
        })
    },
    createUser(){
      this.$axios.post('/register', this.form_create_user)
        .then(response =>{
          alert('Пользователь успешно добавлен')
          this.modalCreateUserForm = false
        })
        .catch(errors =>{
          alert(errors);
        })
    },
    addAcessList(){
      let fm = new FormData();
      fm.append('address', this.ip_text);
      this.$axios.post('/access-list', fm)
        .then(response =>{
          alert(response.data.message)
          this.ip_text = ''
          this.getIPs()
        })
    },
    deleteIp(id){
      this.$axios.get('access-list/delete/' + id)
        .then(response =>{
          alert('Удален успешно');
          this.getIPs()
        })
    },
    getModalAccessList(){
      this.modalAccessList = !this.modalAccessList;
    }
  }
}
</script>
