<template>
  <v-container>
    <v-card>
      <v-card-title>Форма поиска</v-card-title>
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
        @click:row="getTabsByProject"
        no-data-text="Данные отсувствуют"
        no-results-text="По запросу ничего не найдено"
      >
      </v-data-table>
      <v-dialog v-model="modal" max-width="700">
        <ProjectComponent v-bind:project="current_project"></ProjectComponent>
      </v-dialog>
      <v-card-actions>
<!--        <v-btn>-->
<!--         Загрузить в WORD-->
<!--        </v-btn>-->
        <v-btn @click="getPDF">
          Загрузить в PDF
        </v-btn>
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
    this.getProjects();
  },
  data(){
    return{
      modal: false,
      headers_for_project: [
        { text: 'Id', value: 'id' },

        { text: 'Название проекта',
          sortable: false,
          value: 'name',
        },
        { text: 'Компания',
          sortable: false,
          value: 'company.name',
        },
        { text: 'Описание',
          sortable: false,
          value: 'description',
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
    getTabsByProject(item) {
      this.modal = true;
      this.current_project = item;
    }
  }
}
</script>
