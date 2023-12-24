
<template>
<v-container>
  <v-dialog max-width="1000" v-model="show">
    <v-card >
      <v-form >
        <v-text-field
          v-model="form_show.name"
          disabled
          label="Наименование">
        </v-text-field>
        <p>Описание листа</p>

        <vue-editor  disabled="disabled" :editor-options="editorSettings" v-model="form_create.text"></vue-editor>
      </v-form>
    </v-card>

  </v-dialog>
  <template v-if="is_created">
    <v-card max-width="1000">
      <v-btn @click="is_created = false">
        Выйти из добавления
      </v-btn>
      <v-card-title>Добавить шаблон</v-card-title>

<!--          <v-btn-->
<!--          >-->
<!--            Информация-->
<!--            <v-tooltip-->
<!--              activator="parent"-->
<!--              location="end"-->
<!--            >Tooltip</v-tooltip>-->
<!--          </v-btn>-->
      <v-form ref="form_create" @submit.prevent="addTemplate">
        <v-text-field
          v-model="form_create.name"
          label="Наименование">
        </v-text-field>
        <p>Описание листа</p>
        <vue-editor id="editor" :editor-options="editorSettings" v-model="form_create.text"></vue-editor>
        <v-btn type="submit">Добавить</v-btn>
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
          @click:row="showTemplate"
          no-data-text="Отсувствуют данные"
        >
          <template v-slot:[`item.delete`]="{ item }">
            <v-btn small color="error" @click="deleteTemplate(item.id)" >Удалить </v-btn>
          </template>
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
import VueFroala from 'vue-froala-wysiwyg';
export default {
  components:{
    VueEditor
  },
  beforeMount() {
    this.getTemplates();
  },
  data(){
    return{
      config: {
        events: {
          'froalaEditor.initialized': function () {
            console.log('initialized')
          }
        }
      },
      expanded: [],
      templates:null,
      show: false,
      is_created:false,
      form_create:{
        name: '',
        text: ''
      },
      form_show:{
        name: '',
        text: ''
      },
      editorSettings: {
        modules: {
          imageDrop: true,
          imageResize: {},
        }
      },
      headers:[
        {
          text: "Название шаблона",
          align: 'start',
          value: "name",
          sortable: true,
        },
        {
          text: "Action",
          value: "delete",
          align: 'end',
          sortable: false,
        }
      ]
    }
  },
  methods:{
    showTemplate(value){
      console.log(value)
      this.show = true
      this.form_show.text = value.text
      this.form_show.name = value.name
    },
    deleteTemplate(id){
      this.show = false;
      this.$axios.delete('/templates/' + id)
        .then((response) =>{
          alert('Шаблон удален успешно')
          this.is_created = false
          this.getTemplates()
        })
    },
    getTemplates(){
      this.$axios.get('/templates')
        .then((response) =>{
          this.templates = response.data.data
        })
    },
    addTemplate(){
      this.$axios.post('/templates', this.form_create)
        .then((response) =>{
          alert(response.data.message)
          this.is_created = false
          this.getTemplates()
        })
    }
  }
}
</script>
<style scoped>
#editor {
  width: 100%;
  height: 100%;
}
</style>
