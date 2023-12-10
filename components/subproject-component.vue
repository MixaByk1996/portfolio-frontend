<template>
  <v-container>
    <v-card>
      <v-card-title>Подпроект: {{current_subproject.name}}</v-card-title>
      Описание : {{current_subproject.description}} <br>
      <template v-if="current_subproject.tags.length > 0">
        <v-list>
          <v-subheader>Теги</v-subheader>
          <v-list-item-group v-model="selectedTag">
            <v-list-item
              v-for="item in current_subproject.tags"
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

      <template v-if="current_subproject.files.length > 0">
        <v-list >
          <v-subheader>Файлы</v-subheader>
          <v-list-item-group v-model="selectedFile">
            <v-list-item
              v-for="item in current_subproject.files"
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
  props: ['id'],
  // beforeMount() {
  //   this.getCurrent()
  // },
  data() {
    return {
      is_admin: false,
      current_subproject: null,
      selectedTag: null,
      selectedFile: null,
    }
  },
  methods: {
    getUser(){
      this.$axios.get('/user')
        .then((response) =>{
          this.is_admin = response.data.rules === 'ADMIN';
        })
        .catch((errors)=>{
          console.log(errors)
        })
    },
    getCurrent(){
      this.$axios.get('/subprojects/' + this.id)
        .then((response) => {
          console.log(response.data.data)
          this.current_subproject = response.data.data;
        })
    },
    deleteSubProject(){
      this.$axios.delete('/subprojects/' + this.id)
        .then((response) => {
          alert('Подпроект удален');
          location.reload();
        })
    }
  },
  mounted (){
    this.getCurrent();
  }
}
</script>
