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
  beforeMount() {
    this.getIPs()
  },
  data(){
    return{
      modalAccessList : false,
      ip_text: "",
      search: "",
      access_list_array: null,
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
      ]
    }
  },
  methods : {
    getIPs(){
      this.$axios.get('/access-list')
        .then(response =>{
          this.access_list_array = response.data.data;
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
      console.log(id);
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
