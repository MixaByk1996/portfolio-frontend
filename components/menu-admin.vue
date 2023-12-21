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
    <template v-if="ip_text.length > 0">
      <v-btn @click="addAcessList">Добавить адрес</v-btn>
    </template>

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
                    <template v-slot:[`item.delete`]="{ item }">
                      <v-btn small color="error" @click="deleteBackup(item.id)" >Удалить </v-btn>
                    </template>
        </v-data-table>
      </v-card>
    </v-dialog>


    <v-dialog v-model="modalCompaniesList" max-width="700">
      <v-card >
        <v-card-title>
          Список компаний
        </v-card-title>
        <v-data-table
          :items="list_companies"
          :headers="headersCompanies"
          no-data-text="Данные отсувствуют"
          no-results-text="По запросу ничего не найдено"
        >
          <template v-slot:[`item.delete`]="{ item }">
            <v-btn small color="error" @click="deleteCompany (item.id)" >Удалить </v-btn>
          </template>
        </v-data-table>
      </v-card>
    </v-dialog>


    <v-dialog v-model="modalCreateUserForm" max-width="700">
      <v-form ref="formCreateUser" @submit.prevent="createUser">
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
            <v-btn type="submit">Добавить</v-btn>
          </v-card-actions>
        </v-card>
      </v-form>

    </v-dialog>

    <v-dialog v-model="modalUserUpdate" max-width="700">
      <v-card>
        <v-card-title>
          Обновить / Удалить данные пользователя
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
        <v-row>
          <v-col>
            <v-btn color="error" @click="deleteUser(form_update_user.id)" > Удалить </v-btn>
          </v-col>
        </v-row>
      </v-card>
    </v-dialog>

    <v-dialog v-model="modalCompanyCreate" max-width="700">
      <v-form ref="formCreateCompany" @submit.prevent="createCompany">
        <v-card>
          <v-card-title>
            Добавить компанию
          </v-card-title>
          <!--        form_create_company-->
          <v-text-field
            v-model="form_create_company.name"
            label="Наименование"
            :rules="rules"
            single-line
            hide-details
          ></v-text-field>
          <v-text-field
            v-model="form_create_company.description"
            label="Описание"
            :rules="rules"
            single-line
            hide-details
          ></v-text-field>
          <v-flex xs12 class="text-xs-center text-sm-center text-md-center text-lg-center"
                  :rules="rules"
          >
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
            <v-btn type="submit">Добавить</v-btn>
          </v-card-actions>
        </v-card>

      </v-form>

    </v-dialog>

    <v-dialog v-model="modalCreateSubproject" max-width="700">
      <v-form ref="formSubjectCreate" v-scroll:#scroll-target="onScroll" @submit.prevent="createSubproject">
        <v-card>
          <v-card-title>
            Добавление листа
          </v-card-title>
          <v-text-field
            v-model="form_create_sub_project.name"
            label="Наименование"
            single-line
            :rules="rules"
            hide-details
          ></v-text-field>
          <v-select
            v-model="form_create_sub_project.select_project"
            item-text="name"
            item-value='id'
            :rules="rules"
            :items="this.list_projects_for_select"
            label="Выберите проект"
            persistent-hint
            return-object
            single-line
          ></v-select>

          <p>Описание листа</p>
          <v-select
            label="Выберите шаблон(необязательно)"
          :items="templates"
          item-text="name"
          item-value="text"
            @change="getShowItemTemplate"
          return-object
          >
          </v-select>
          <v-btn @click="show_variable = true">Найти и заменить</v-btn>
          <template v-if="show_variable === true">
            <v-dialog v-model="show_variable" max-width="600">
              <v-form ref="replace_form" ><

                <v-card >
                  <v-card-title>Замена</v-card-title>
                  <v-text-field
                    v-model="search_text_template"
                    :rules="rules"
                    label="Слово на замену"
                  ></v-text-field>
                  <v-text-field
                    v-model="replace_text_template"
                    :rules="rules"
                    label="Новое слово"
                  >
                  </v-text-field>
                </v-card>
                <v-btn type="submit"></v-btn>
              </v-form>

            </v-dialog>
          </template>
          <vue-editor  @keyup="setPosition"  :editor-options="editorSettings" v-model="form_create_sub_project.description"></vue-editor>
<!--          <v-text-field-->
<!--            v-model="form_create_sub_project.description"-->
<!--            label="Описание"-->
<!--            single-line-->
<!--            :rules="rules"-->
<!--            hide-details-->
<!--          ></v-text-field>-->

          <v-card-actions>
            <v-btn type="submit">Добавить</v-btn>
          </v-card-actions>
        </v-card>
      </v-form>
    </v-dialog>

    <v-dialog v-model="modalProjectCreate" max-width="700">
      <v-form ref="formCreateProject" @submit.prevent="createProject">
        <v-card>
          <v-card-title>
            Добавление проекта
          </v-card-title>
          <v-text-field
            v-model="form_create_project.name"
            label="Наименование"
            single-line
            :rules="rules"
            hide-details
          ></v-text-field>
          <v-text-field
            v-model="form_create_project.description"
            label="Описание"
            single-line
            :rules="rules"
            hide-details
          ></v-text-field>
          <v-select
            v-model="form_create_project.select_company"
            item-text="name"
            item-value='id'
            :items="this.list_companies_for_select"
            label="Выберите компанию"
            persistent-hint
            return-object
            single-line
          ></v-select>
          <v-card-actions>
            <v-btn type="submit">Добавить</v-btn>
          </v-card-actions>
        </v-card>
      </v-form>
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
        </v-data-table>
      </v-card>
    </v-dialog>

    <v-dialog v-model="modalTagsCreate" max-width="700">
      <v-form ref="formCreateTag" @submit.prevent="sendDataToTagsCreate">
        <v-card>
          <v-card-title>
            Добавить тег/фильтр
          </v-card-title>
          <v-switch
            v-model="model_switch_in_tags"
            :label="model_switch_in_tags ? 'Листы' : 'Проекты'"
            @change="getSwitchInTagsChanged"
          >
          </v-switch>
          <v-select
            v-model="select_in_tags"
            item-text="name"
            item-value='id'
            :rules="rules"
            :items="this.list_general"
            :label="model_switch_in_tags ? 'Выберите лист' : 'Выберите проект'"
            persistent-hint
            return-object
            single-line
          ></v-select>
          <v-text-field
            v-model="name_tag"
            :rules="rules"
            label="Введите название тега/фильтра"
            single-line
            hide-details
          ></v-text-field>
          <v-card-actions>
            <v-btn type="submit">Добавить</v-btn>
          </v-card-actions>
        </v-card>
      </v-form>

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
        <v-btn width="240"  height="100" @click="modalTagsCreate = !modalTagsCreate">Добавить тег/фильтр</v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-btn width="240"  height="100" @click="modalCompanyCreate = !modalCompanyCreate">Добавить компанию</v-btn>
      </v-col>
      <v-col>
          <v-btn width="240"  height="100" @click="modalCreateSubproject = !modalCreateSubproject">Добавить лист</v-btn>

      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-btn width="240"  height="100" @click="modalCompaniesList = !modalCompaniesList">Список компаний</v-btn>
      </v-col>
      <v-col>

      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import {axios} from 'axios';
import { VueEditor, Quill } from "vue2-editor";
import ImageResize from 'quill-image-resize-vue';
import { ImageDrop } from 'quill-image-drop-module';

Quill.register("modules/imageDrop", ImageDrop);
Quill.register("modules/imageResize", ImageResize);
export default {
  components:{VueEditor},
  beforeMount() {
    this.getIPs();
    this.getUsers();
    this.getBackups();
    this.getCompany();
    this.getTemplates();
    // this.getProjects();
    // this.getSubprojects();
    this.getSwitchInTagsChanged();
  },
  data(){
    return{
      template_obj: null,
      editorSettings: {
        modules: {
          imageDrop: true,
          imageResize: {},
        }
      },
      select_template:null,
      is_backup_load : false,
      offsetTop: 0,
      modalAccessList : false,
      modalCreateUserForm: false,
      modalBackupList: false,
      modalCompaniesList:false,
      modalCreateSubproject: false,
      templates: null,
      ip_text: "",
      search: "",
      searchUser: "",
      access_list_array: null,
      list_users: null,
      list_backup: null,
      project_files: null,
      subproject_files: null,
      rules: [
        v => !!v || 'Поле обязательно для заполнения!',
      ],
      imageUrl: '',
      imageFile: null,
      imageName: '',

      list_companies: null,
      list_companies_for_select: null,
      list_projects_for_select: null,
      list_subprojects_for_select: null,
      list_general: null,

      modalTagsCreate: false,
      modalUsersList:false,
      modalUserUpdate: false,
      modalProjectCreate: false,

      model_switch_in_tags: false,
      model_switch_rules: false,
      search_text_template: '',
      replace_text_template: '',
      is_hide_password: false,
      modalCompanyCreate: false,
      select_in_tags: null,
      name_tag: "",
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
        select_company: null
      },
      form_create_sub_project:{
        name: "",
        description: "",
        select_project: null
      },

      headers: [
        {
          text: 'IP адрес',
          align: 'start',
          sortable: true,
          value: 'address',
        },
        {
          text: "Action",
          value: "delete",
          align: 'end',
          sortable: false,
        },
      ],
      show_variable: false,
      headersBackup: [
        { text: 'Название файла',
          align: 'start',
          sortable: false,
          value: 'file_name',
        },
        { text: 'URL',
          align: 'end',
          sortable: false,
          value: 'file_url',
        },
        {
          text: "Action",
          value: "delete",
          align: 'end',
          sortable: false,
        },
      ],
      headersCompanies: [
        { text: 'Название',
          align: 'start',
          sortable: false,
          value: 'name',
        },
        {
          text: "Action",
          value: "delete",
          align: 'end',
          sortable: false,
        },
      ],
      headersUsersTable: [
        {
          text: "Логин",
          align: 'start',
          value: "login",
          sortable: true,
        },
        {
          text: "Тип",
          align: 'end',
          value: "rules",
          sortable: false,
        },
      ]
    }
  },
  methods : {
    replaceTextInTemplate(){
      if(this.$refs.replace_form.validate()){
        this.form_create_sub_project.description.replace(this.search_text_template, this.replace_text_template);
        this.show_variable = false;
      }
    },
    getShowItemTemplate(item){
      console.log(item)
      this.form_create_sub_project.description = item.text
    },
    getTemplates(){
      this.$axios.get('/templates')
        .then((response) =>{
          this.templates = response.data.data
        })
        .catch((error) => {
          console.log(error)
        })
    },
    onScroll (e) {
      this.offsetTop = e.target.scrollTop
    },
    setPosition(e){
      if(e.keyCode === 86 && e.ctrlKey){
        console.log(this.offsetTop)
        this.$refs.formSubjectCreate.target.offsetTop = this.offsetTop
      }
    },
    pickFile() {
      this.$refs.image.click()
    },
    sendDataToTagsCreate(){
      if(this.$refs.formCreateTag.validate()){
        let fm = new FormData();
        fm.append('id', this.select_in_tags.id);
        fm.append('name', this.name_tag);
        fm.append('module', this.model_switch_in_tags ? 'SUBPROJECT' : 'PROJECT');
        this.$axios.post('/tags', fm)
          .then((response) =>{
            alert(response.data.message);
          })
        this.modalTagsCreate = false
      }
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
          console.log(this.imageFile)
        })
      } else {
        this.imageName = ''
        this.imageFile = ''
        this.imageUrl = ''
      }
    },
    createSubproject(){
      if(this.$refs.formSubjectCreate.validate()){
        let fm = new FormData();
        fm.append('name', this.form_create_sub_project.name );
        fm.append('description', this.form_create_sub_project.description );
        fm.append('project_id', this.form_create_sub_project.select_project.id );
        this.$axios.post('/subprojects', fm)
          .then((response) =>{
            alert(response.data.message);
            this.modalCreateSubproject = false;
            location.reload()
          })
      }
    },
    getSwitchInTagsChanged(){
      this.getProjects();
      this.getSubprojects();
      this.list_general = this.model_switch_in_tags ? this.list_subprojects_for_select : this.list_projects_for_select ;
    },
    createCompany(){
      if (this.$refs.formCreateCompany.validate()){
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
      }
    },
    getProjects(){
      this.$axios.get('/projects')
        .then((response) =>{
          let temp = response.data.data;
          this.list_projects_for_select = temp.map((item) =>{
            return{
              id : item.id,
              name: item.name
            }
          })
        })
    },
    getSubprojects(){
      this.$axios.get('/subprojects')
        .then((response) =>{
          let temp = response.data.data;
          this.list_subprojects_for_select = temp.map((item) =>{
            return{
              id : item.id,
              name: item.name
            }
          })
        })
    },
    createProject(){
      if (this.$refs.formCreateProject.validate()){
        let fm = new FormData();
        let company_id = 0;
        fm.append('name', this.form_create_project.name );
        fm.append('description', this.form_create_project.description );
        if(this.form_create_project.select_company != null){
          company_id = this.form_create_project.select_company.id
        }
        fm.append('company_id', company_id );
        this.$axios.post('/projects', fm)
          .then((response) =>{
            alert(response.data.message);
            this.modalProjectCreate = false;
            location.reload()
          })
      }
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
          this.list_companies_for_select = this.list_companies.map((item) =>{
            return {
              id: item.id,
              name: item.name,
              description: item.description,
            }
          })

          console.log(this.list_companies_for_select)
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
          this.getUsers()
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
      if(this.$refs.formCreateUser.validate()){
        this.$axios.post('/register', this.form_create_user)
          .then(response =>{
            alert('Пользователь успешно добавлен')
            this.modalCreateUserForm = false
          })
          .catch(errors =>{
            alert(errors);
          })
      }

    },
    addAcessList(){
      let fm = new FormData();
      fm.append('address', this.ip_text);
      this.$axios.post('/access-list', fm)
        .then(response =>{
          alert(response.data.message)
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

    deleteUser(id){
      this.$axios.get('user/delete/' + id)
        .then(response =>{
          alert('Удален успешно');
          this.modalUserUpdate = false;
          this.getUsers();
        })
    },
    deleteBackup(id){
      this.$axios.get('backup/delete/' + id)
        .then(response =>{
          alert('Удален успешно');
          this.modalUserUpdate = false;
          this.getBackups();
        })
    },

    deleteCompany(id){
      this.$axios.delete('company/' + id)
        .then(response =>{
          alert('Удалена успешно');
          this.getCompany();
        })
    },

    getModalAccessList(){
      this.modalAccessList = !this.modalAccessList;
    }
  }
}
</script>
