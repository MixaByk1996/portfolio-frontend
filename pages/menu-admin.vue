<template>
  <v-container>
    <v-dialog v-model="modalAccessList">
<v-card>
  <v-card>
    <v-card-title>Добавить ip</v-card-title>
    <v-text-field
      v-model="ip_text"
      placeholder="Введите ip адрес"
    ></v-text-field>
    <v-btn @click="addAcessList">Добавить адрес</v-btn>
  </v-card>
</v-card>
    </v-dialog>
    <v-row>
      <v-col>
          <v-btn width="240" height="100">Редактировать шаблон</v-btn>
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
        <v-btn width="240"  height="100">Список пользователей</v-btn>
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
  data(){
    return{
      modalAccessList : false,
      ip_text: "",
      access_list_array: null,
    }
  },
  methods : {
    addAcessList(){
      let fm = new FormData();
      fm.append('address', this.ip_text);
      this.$axios.post('/access-list', fm)
        .then(response =>{
          alert(response.data.message)
        })

    },
    getModalAccessList(){
      this.modalAccessList = !this.modalAccessList;
      this.$axios.get('/access-list')
        .then(response =>{
          this.access_list_array = response.data.data;
          console.log(response.data.data)
        })
    }
  }
}
</script>
