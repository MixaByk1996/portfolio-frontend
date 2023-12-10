<template>
  <v-container>
    <template v-if="modal_update == false">
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
          <v-btn>Начать редактирование</v-btn>
          <v-btn @click="deleteSubProject">Удалить</v-btn>
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
                        label="Описание подпроекта"
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
  props: ['id'],
  // beforeMount() {
  //   this.getCurrent()
  // },
  data() {
    return {
      is_admin: false,
      current_subproject: null,
      form:{
        name: "",
        description: ""
      },
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
    updateProject(){
      this.$axios.put('/subprojects/' + this.id, this.form)
        .then((response) => {
          alert('Данные обновлены');
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
