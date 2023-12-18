<template>
  <v-container >
    <v-card :loading="loading">
      <v-card-title>Форма поиска</v-card-title>
      <v-text-field
        v-model="search_text"
        append-icon="mdi-magnify"
        label="Введите слово"
        single-line
        hide-details
      ></v-text-field>
      <p></p>
      <p></p>
      <v-btn @click="getSearch">
        Поиск
      </v-btn>
      <template v-if="items_projects !== null">
        <v-data-table
          :items="items_projects"
          :headers="headers_for_project"
          no-data-text="Данные отсувствуют"
          no-results-text="По запросу ничего не найдено"
        >
        </v-data-table>
      </template>


      <v-card-actions>
<!--        <v-btn>-->
<!--         Загрузить в WORD-->
<!--        </v-btn>-->
<!--        <v-btn @click="getPDF">-->
<!--          Загрузить в PDF-->
<!--        </v-btn>-->
      </v-card-actions>
    </v-card>

  </v-container>



</template>
<script>
import {axios} from 'axios';
import { Blob } from 'buffer';
import ProjectComponent from "~/components/project-component.vue";
export default {
  components: {ProjectComponent},
  beforeMount() {
    // this.getProjects();
  },
  data(){
    return{
      modal: false,
      loading: false,
      headers_for_project: [
        { text: 'Название листа',
          sortable: false,
          value: 'name',
        }
        // {
        //   text: "",
        //   value: "delete",
        //   sortable: false,
        // },
      ],
      search_text: "",
      text_search_result: "",
      current_project: null,
      items_projects: [],
    }
  },
  methods:{
    getSearch(){
      this.loading = true;
      var fm = new FormData;
      fm.append('search', this.search_text);
      this.$axios.post('/get-search', fm)
        .then((response) =>{
          this.items_projects = response.data.data;
          this.loading = false
        })
    },
    getProjects(){
      this.$axios.get('/projects')
        .then((response) =>{
          console.log(response.data.data);
          this.items_projects = response.data.data
        })
    },
    getPDF() {
      let fm = new FormData();
      console.log(this.items_projects.length);
        fm.append('data', JSON.stringify(this.items_projects));


      this.$axios.post('/download-pdf', fm, {
        responseType: "blob"
      })
        .then((response) => {
          const blob = response.data
          const objectUrl = window.URL.createObjectURL(blob)
          window.open(objectUrl)
          // const url = window.URL.createObjectURL(new Blob([response.data]));
          // const link = document.createElement('a');
          // link.href = url;
          // link.setAttribute('download', 'file.pdf');
          // document.body.appendChild(link);
          // link.click();
        })
    },
  }
}
</script>
