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

    <v-dialog v-model="modalBackupList" max-width="700">
      <v-card :loading="is_backup_load">
        <v-card-title>
          Список бекапов
        </v-card-title>
        <v-btn @click="createBackup">Создать бекап</v-btn>
        <v-data-table
          :items="list_backup"
          :headers="headersBackup"
          no-data-text="Данные отсувствуют"
          no-results-text="По запросу ничего не найдено"
        >
          <!--          <template v-slot:[`item.delete`]="{ item }">-->
          <!--            <v-btn small color="error" @click="deleteIp(item.id)" > Удалить пользователя </v-btn>-->
          <!--          </template>-->
        </v-data-table>
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

    <v-dialog v-model="modalUserUpdate" max-width="700">
      <v-card>
        <v-card-title>
          Обновить данные пользователя
        </v-card-title>
        <v-row>
          <v-col>
            <v-text-field
              v-model="form_update_user.login"
              label="Логин"
              single-line
              hide-details
            ></v-text-field>
          </v-col>
          <v-col>
            <v-text-field
              v-model="form_update_user.password"
              label="Новый пароль"
              single-line
              hide-details
            ></v-text-field>
          </v-col>

          <v-col>
            <v-switch
              v-model="form_update_user.rules"
              hide-details
              :label="form_update_user.rules ? 'Администратор' : 'Пользователь'"
            ></v-switch>
          </v-col>
          <v-col>
            <v-btn @click="updateUser">Обновить</v-btn>
          </v-col>
        </v-row>
      </v-card>
    </v-dialog>

    <v-dialog v-model="modalCompanyCreate" max-width="700">
      <v-card>
        <v-card-title>
          Добавить компанию
        </v-card-title>
<!--        form_create_company-->
        <v-text-field
          v-model="form_create_company.name"
          label="Наименование"
          single-line
          hide-details
        ></v-text-field>
        <v-text-field
          v-model="form_create_company.description"
          label="Описание"
          single-line
          hide-details
        ></v-text-field>
        <v-flex xs12 class="text-xs-center text-sm-center text-md-center text-lg-center">
          <!-- Here the image preview -->
          <img :src="imageUrl" height="150" v-if="imageUrl"/>
          <v-text-field label="Выберите фото" @click='pickFile' v-model='imageName' prepend-icon="mdi-file-image"></v-text-field>
          <input
            type="file"
            style="display: none"
            ref="image"
            accept="image/jpeg, image/jpg, image/png"
            @change="onFilePicked"
          >
        </v-flex>
        <v-card-actions>
          <v-btn @click="createCompany">Добавить</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>


    <v-dialog v-model="modalProjectCreate" max-width="700">
      <v-card>
        <v-card-title>
          Добавление проекта
        </v-card-title>
        <v-text-field
          v-model="form_create_project.name"
          label="Наименование"
          single-line
          hide-details
        ></v-text-field>
        <v-text-field
          v-model="form_create_project.description"
          label="Описание"
          single-line
          hide-details
        ></v-text-field>
        <v-select
          v-model="form_create_project.select_company"
          :items="this.list_companies"
          item-title="name"
          item-value="id"
          label="Выберите компанию"
          persistent-hint
          return-object
          single-line
        ></v-select>

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
          @click:row="getUserByUpdate"
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
        <v-btn width="240" height="100" @click="modalProjectCreate = !modalProjectCreate">Добавить проект в дерево</v-btn>
      </v-col>
      <v-col>
        <v-btn width="240"  height="100" @click="modalUsersList = !modalUsersList">Список пользователей</v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-btn width="240" height="100" @click="modalBackupList = !modalBackupList">Бекап</v-btn>
      </v-col>
      <v-col>
        <v-btn width="240"  height="100">Добавить тег/фильтр</v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-btn width="240"  height="100" @click="modalCompanyCreate = !modalCompanyCreate">Добавить компанию</v-btn>
      </v-col>
      <v-col>

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
    this.getBackups();
    this.getCompany();
  },
  data(){
    return{
      is_backup_load : false,
      modalAccessList : false,
      modalCreateUserForm: false,
      modalBackupList: false,
      ip_text: "",
      search: "",
      searchUser: "",
      access_list_array: null,
      list_users: null,
      list_backup: null,

      imageUrl: '',
      imageFile: null,
      imageName: '',

      list_companies: null,
      modalUsersList:false,
      modalUserUpdate: false,
      modalProjectCreate: false,

      model_switch_rules: false,
      is_hide_password: false,
      modalCompanyCreate: false,
      form_create_user:{
        login : "",
        password: "",
        rules: this.model_switch_rules ? "ADMIN" :"USER"
      },
      form_update_user:{
        id: 0,
        login : "",
        password: "",
        rules: ""
      },
      form_create_company:{
        name: "",
        photo_url: "",
        description: "",
      },
      form_create_project:{
        name: "",
        description: "",
        select_company: ""
      },

      headers: [
        {
          title: 'IP адрес',
          align: 'start',
          sortable: true,
          value: 'address',
        },
        {
          title: "Action",
          value: "delete",
          align: 'end',
          sortable: false,
        },
      ],
      headersBackup: [
        { title: 'Название файла',
          align: 'start',
          sortable: false,
          value: 'file_name',
        },
        { title: 'URL',
          align: 'end',
          sortable: false,
          value: 'file_url',
        }
        // {
        //   text: "",
        //   value: "delete",
        //   sortable: false,
        // },
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
          align: 'end',
          value: "rules",
          sortable: false,
        }
      ]
    }
  },
  methods : {


    pickFile() {
      this.$refs.image.click()
    },
    onFilePicked(e) {
      const files = e.target.files
      if(files[0] !== undefined) {
        this.imageName = files[0].name
        if (this.imageName.lastIndexOf('.') <= 0) {
          return
        }
        const fr = new FileReader()
        fr.readAsDataURL(files[0])
        fr.addEventListener('load', () => {
          this.imageUrl = fr.result
          this.imageFile = files[0]
        })
      } else {
        this.imageName = ''
        this.imageFile = ''
        this.imageUrl = ''
      }
    },

    createCompany(){
      let fm = new FormData();
      fm.append('name', this.form_create_company.name),
      fm.append('description', this.form_create_company.description),
      fm.append('image_data', this.imageFile)
      this.$axios.post('/company', fm)
        .then((response) => {
          alert('Успешно добавлена')
          this.modalCompanyCreate = false
          location.reload()
        })
    },
    createBackup(){
      this.is_backup_load = true;
      this.$axios.post('/backup', [])
        .then(response =>{
          alert('Бекап прошел успешно')
          this.is_backup_load = false;
          this.getBackups();
        })
        .catch(errors =>{
          alert(errors);
          this.is_backup_load = false;
        })
    },
    getUsers(){
      this.$axios.get('/users')
        .then(response => {
          this.list_users = response.data.data;
        })
    },
    getCompany(){
      this.$axios.get('/company')
        .then(response => {
          this.list_companies = response.data.data;
          console.log(this.list_companies);
        })
    },
    getUserByUpdate(value){
      console.log(value)
      this.modalUserUpdate = true
         this.$axios.get('/users/' + value.id)
        .then(response => {
          let obj = response.data.data
          this.form_update_user.id = obj.id
          this.form_update_user.login = obj.login
          this.form_update_user.rules = obj.rules !== 'Пользователь'

        })
    },
    updateUser(){
      var fm = new FormData();
      fm.append('login', this.form_update_user.login);
      fm.append('password', this.form_update_user.password);
      fm.append('rules', this.form_update_user.rules === true ? "ADMIN" : 'USER');
      this.$axios.put('/users/' + this.form_update_user.id, fm)
        .then((response) =>{
          alert('Данные обновленны успешно')
          this.form_update_user.id = 0
          this.form_update_user.login = ''
          this.form_update_user.rules = ''
          this.modalUserUpdate = false
        })
        .catch(error => {
          alert(error);
        })
    },
    getBackups(){
      this.$axios.get('/backup')
        .then(response => {
          this.list_backup = response.data.data;
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
