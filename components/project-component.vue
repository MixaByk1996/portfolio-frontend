<template>
  <v-container>

    <v-card>
      <v-card-title>Проект: {{project.name}}</v-card-title>
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
        <v-btn>Обновить данные</v-btn>
        <v-btn @click="deleteProject">Удалить</v-btn>
      </template>
    </v-card>

  </v-container>
</template>
<script>
import {axios} from 'axios';
export default {
  props: ['project'],
  beforeMount() {
    console.log(this.project)
    this.getUser()
  },
  data() {
    return {
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
