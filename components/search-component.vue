<template>
  <v-container>
    <v-text-field
      v-model="search_text"
      append-icon="mdi-magnify"
      label="Поиск"
      single-line
      hide-details
    ></v-text-field>
    <v-data-table
      :items="items_projects"
      :headers="headers_for_project"
      :search="search_text"
      no-data-text="Данные отсувствуют"
      no-results-text="По запросу ничего не найдено"
    >
    </v-data-table>
  </v-container>
</template>
<script>
import {axios} from 'axios';
export default {
  beforeMount() {
    this.getProjects();
  },
  data(){
    return{
      headers_for_project: [
        { text: 'Id', value: 'id' },

        { text: 'Название проекта',
          sortable: false,
          value: 'name',
        },
        { text: 'Компания',
          sortable: false,
          value: 'company.name',
        }
        // {
        //   text: "",
        //   value: "delete",
        //   sortable: false,
        // },
      ],
      search_text: "",
      items_projects: [],
    }
  },
  methods:{
    getProjects(){
      this.$axios.get('/projects')
        .then((response) =>{
          console.log(response.data.data);
          this.items_projects = response.data.data
        })
    }
  }
}
</script>
