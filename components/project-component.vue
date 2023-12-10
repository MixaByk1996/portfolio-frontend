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
      <v-btn @click="updateProject">Обновить информацию</v-btn>
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
    console.log(this.project)
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
