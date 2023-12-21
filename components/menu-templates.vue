
<template>
<v-container>

  <template v-if="is_created">
    <v-card>
      <v-btn @click="is_created = false">
        Выйти из добавления
      </v-btn>
      <v-card-title>Добавить шаблон</v-card-title>
      <v-form ref="form_create">
        <v-text-field
          v-model="form_create.name"
          label="Наименование">
        </v-text-field>
      </v-form>

    </v-card>
  </template>
  <template v-else>
    <v-card>
      <v-btn @click="is_created = true">Добавить шаблон</v-btn>
      <v-card-title>Меню шаблонов</v-card-title>
      <v-divider></v-divider>
      <template v-if="templates">
        <v-data-table
          :headers="headers"
          :items="templates"
          no-data-text="Отсувствуют данные"
        >
        </v-data-table>
      </template>

    </v-card>
  </template>

</v-container>
</template>
<script>
import {axios} from 'axios';
import {Quill, VueEditor} from "vue2-editor";
import {ImageDrop} from "quill-image-drop-module";
import ImageResize from "quill-image-resize-vue";
Quill.register("modules/imageDrop", ImageDrop);
Quill.register("modules/imageResize", ImageResize);
export default {
  components:{
    VueEditor
  },
  beforeMount() {
    this.getTemplates();
  },
  data(){
    return{
      templates:null,
      is_created:false,
      form_create:{
        name: '',
        text: ''
      },
      headers:[
        {
          text: "Название шаблона",
          align: 'start',
          value: "name",
          sortable: true,
        },
      ]
    }
  },
  methods:{
    getTemplates(){
      this.$axios.get('/templates')
        .then((response) =>{
          this.templates = response.data.data
        })
    }
  }
}
</script>
<style scoped>

</style>
