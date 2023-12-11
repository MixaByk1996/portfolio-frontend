<template>
  <v-container>
  <template v-if="modal_update == false">
    <v-card>
      Проект: {{project.name}} <br>
      Описание : {{project.description}} <br>
      Компания: {{project.company.name}}
      <template v-if="this.project.tags.length > 0">
        <v-list>
          <v-subheader>Теги</v-subheader>
          <v-list-item-group v-model="selectedTag">
            <v-list-item
              v-for="item in project.tags"
              :key="item.id"
              :value="item.name"
            >
              <v-list-item-title v-text="item.name"></v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </template>
      <template v-else>
        <br> Теги отсувствуют
      </template>

      <template v-if="this.project.files.length > 0">
        <v-list >
          <v-subheader>Файлы</v-subheader>
          <v-list-item-group v-model="selectedFile">
            <v-list-item
              v-for="item in project.files"
              :key="item.id"
              :value="item.name"
            >
              <v-list-item-title v-text="item.name"></v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </template>
      <template v-else>
        <br> Файлы отсувствуют
      </template>

      <template v-if="this.project.subproject.length > 0">
        <v-list >
          <v-subheader>Список подпроектов</v-subheader>
          <v-list-item-group v-model="selectedSubproject">
            <v-list-item
              v-for="item in project.subproject"
              :key="item.id"
              :value="item.name"
            >
              <v-list-item-title v-text="item.name"></v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </template>
      <template v-else>
        <br> Подпроекты отсувствуют
      </template>
      <br>



      <template v-if="is_admin">
          <v-btn @click="modal_update = true">Начать редактирование</v-btn>
          <v-btn @click="deleteProject">Удалить</v-btn>


      </template>
    </v-card>
  </template>
    <template v-else>
    <v-card>
      <v-text-field v-model="form.name"
        label="Наименование"
      >
      </v-text-field>
      <v-text-field v-model="form.description"
                    label="Описание проекта"
      >
      </v-text-field>
      <v-btn @click="updateProject">Обновить информацию</v-btn><br>
      <template v-if="this.project.files.length > 0">
        <v-list >
          <v-subheader>Файлы</v-subheader>
          <v-list-item-group v-model="selectedFile">
            <v-list-item @click="id_file = item.id"
              v-for="item in project.files"
              :key="item.id"
              :value="item.id"
            >
              <v-list-item-title v-text="item.name"></v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </template>

      <template v-else>
        <br> Файлы отсувствуют
      </template>
      <p></p>
      <v-file-input
        v-model="project_files"
        small-chips
        show-size
        multiple
        clearable
        placeholder="Выберите файлы для добавления"
      ></v-file-input>
      <v-btn @click="addFilesToProject">Добавить файлы</v-btn><p></p>
      <template v-if="selectedFile">
        <v-btn @click="deleteFile">Удалить файл</v-btn>
      </template>
      <template v-if="is_admin">
        <v-btn @click="modal_update = false">Закончить редактирование</v-btn>
      </template>
    </v-card>
    </template>

  </v-container>
</template>
<script>
import {axios} from 'axios';
export default {
  props: ['project'],
  beforeMount() {
    console.log(this.project.files)
    this.form.name = this.project.name
    this.form.description = this.project.description
    this.getUser()
  },
  data() {
    return {
      form:{
        name: "",
        description: ""
      },
      id_file: 0,
      project_files: null,
      is_admin: false,
      selectedTag: null,
      modal_update: false,
      selectedFile: null,
      selectedSubproject: null,
    }
  },
  methods:{
    getUser(){
      this.$axios.get('/user')
        .then((response) =>{
          this.is_admin = response.data.rules === 'ADMIN';
        })
        .catch((errors)=>{
          console.log(errors)
        })
    },
    deleteFile(){
      console.log(this.selectedFile)
      this.$axios.delete('/files/' + this.id_file)
        .then((response) => {
          alert(response.data.message)
          location.reload()
        })
    },
    addFilesToProject(){
      let fm = new FormData();
      for (var i = 0; i < this.project_files.length; i++) {
        fm.append("files_add[]", this.project_files[i]);
      }
      this.$axios.post('/add-files-to-project/' + this.project.id, fm)
        .then((response) =>{
          alert(response.data.message)
          this.project_files = null
          location.reload()
        })

    },
    updateProject(){
      this.$axios.put('/projects/' + this.project.id, this.form)
        .then((response) => {
          alert('Данные обновлены');
        })
    },
    deleteProject(){
      this.$axios.delete('/projects/' + this.project.id)
        .then((response) => {
          alert('Проект удален');
          location.reload();
        })
    }
  }
}
</script>
