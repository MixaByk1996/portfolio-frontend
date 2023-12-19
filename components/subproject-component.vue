<template>
  <v-container style="margin: 5px 5px 5px 5px">
    <template v-if="modal_update === false">
      <v-card>
        <v-card-title>Подпроект: {{current_subproject.name}}</v-card-title>
        Описание : <br>
        <div v-html="current_subproject.description"></div>

        <template v-if="current_subproject.tags !== null">
          <v-list>
            <v-subheader>Теги</v-subheader>
            <v-list-item-group v-model="selectedTag">
              <v-list-item
                v-for="item in current_subproject.tags"
                :key="item.id"
                :value="item.id"
              >
                <v-list-item-title v-text="item.name"></v-list-item-title>
              </v-list-item>
            </v-list-item-group>
          </v-list>
        </template>
        <template v-else>
          <br> Теги отсувствуют
        </template>

        <v-row>
          <v-col>
            <template v-if="current_subproject.files !== null">
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
          </v-col>
          <v-col>
            <template v-if="images_file !== null">
              <v-card max-width="700">
                <v-carousel>
                  <v-carousel-item v-for="item in images_file"
                                   :key="item.id"
                                   :src="baseURL + item.file_url"
                                   cover
                  >

                  </v-carousel-item>
                </v-carousel>
              </v-card>

            </template>
          </v-col>
        </v-row>

        <br>
        <template v-if="is_admin">
          <v-row>
            <v-col>
              <v-btn @click="modal_update = true">Редактировать</v-btn>
              <v-btn @click="deleteSubProject">Удалить</v-btn>
            </v-col>

          </v-row>

        </template>


      </v-card>
    </template>



    <template v-else>
        <v-card>
          <v-text-field v-model="current_subproject.name"
                        label="Наименование"
          >
          </v-text-field>
          <p>Описание листа</p>
          <vue-editor :editor-options="editorSettings" v-model="current_subproject.description"></vue-editor>
<!--          <tiptap-vuetify-->
<!--            v-model="form.description"-->
<!--            :extensions="extensions"-->
<!--          ></tiptap-vuetify>-->
<!--          <v-btn @click="updateProject">Обновить информацию</v-btn>-->

          <template v-if="current_subproject.tags !== null">
            <v-list>
              <v-subheader>Теги</v-subheader>
              <v-list-item-group v-model="selectedTag">
                <v-list-item @click="id_tag = item.id"
                  v-for="item in current_subproject.tags"
                  :key="item.id"
                  :value="item.id"
                >
                  <v-list-item-title v-text="item.name"></v-list-item-title>
                </v-list-item>
              </v-list-item-group>
            </v-list>
            <template v-if="selectedTag">
              <v-btn @click="deleteTag">Удалить тег</v-btn>
            </template>
          </template>

          <template v-if="current_subproject.files !== null">
            <v-list >
              <v-subheader>Файлы</v-subheader>
              <v-list-item-group v-model="selectedFile">
                <v-list-item @click="id_file = item.id"
                             v-for="item in current_subproject.files"
                             :key="item.id"
                             :value="item.id"
                >
                  <v-list-item-title v-text="item.name"></v-list-item-title>
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </template>

          <template v-else>
            <br> Файлы отсувствуют
          </template>
          <p></p>
          <v-file-input
            v-model="subprojectFile"
            small-chips
            show-size
            multiple
            clearable
            placeholder="Выберите файлы для добавления"
          ></v-file-input>
          <v-row>
            <v-col>
              <v-btn @click="addFilesToProject">Добавить файлы</v-btn>
              <template v-if="selectedFile">
                <v-btn @click="deleteFile">Удалить файл</v-btn>
              </template>
            </v-col>
          </v-row>
          <v-row>
            <v-col>

            </v-col>
          </v-row> <v-row>
          <v-col>

          </v-col>
        </v-row>

          <template v-if="is_admin">
            <v-row>
              <v-col>
                <v-btn @click="confirm_dialog = true">Закончить редактирование</v-btn>
              </v-col>
              <v-col align="center">
                <v-btn @click="deleteSubProject">Удалить лист</v-btn>
              </v-col>
              <v-col>

              </v-col>
            </v-row>
          </template>
      </v-card>
    </template>


    <v-dialog
      v-model="confirm_dialog"
      persistent
      width="auto"
    >
      <v-card>
        <v-card-title class="text-h5">
          Внимание!!!
        </v-card-title>
        <v-card-text>Сохранить ли все изменения</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="green-darken-1"
            variant="text"
            @click=" confirm_dialog= false; modal_update = false"
          >
            ОТМЕНА
          </v-btn>
          <v-btn
            color="green-darken-1"
            variant="text"
            @click="updateProject(); confirm_dialog = false;modal_update = false"
          >
            ОК
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>


  </v-container>
</template>
<script>
import {axios} from 'axios';
import {Quill, VueEditor} from "vue2-editor";
import { TiptapVuetify, Heading, Bold, Italic, Strike, Underline, Code, Paragraph, BulletList, OrderedList, ListItem, Link, Blockquote, HardBreak, HorizontalRule, History } from 'tiptap-vuetify';
import {ImageDrop} from "quill-image-drop-module";
import ImageResize from "quill-image-resize-vue";
Quill.register("modules/imageDrop", ImageDrop);
Quill.register("modules/imageResize", ImageResize);
export default {
  props: ['id'],
  components: { TiptapVuetify ,VueEditor },
  beforeMount() {
    this.getCurrent()
    this.getUser()
    this.getImagesFiles()
  },
  data() {
    return {
      extensions: [
        History,
        Blockquote,
        Link,
        Underline,
        Strike,
        Italic,
        ListItem,
        BulletList,
        OrderedList,
        [Heading, {
          options: {
            levels: [1, 2, 3]
          }
        }],
        Bold,
        Code,
        HorizontalRule,
        Paragraph,
        HardBreak
      ],
      is_admin: false,
      current_subproject: null,
      form:{
        name: "",
        description: ""
      },
      baseURL: 'https://8657437b.com',
      id_file: 0,
      id_tag: 0,
      editorSettings: {
        modules: {
          imageDrop: true,
          imageResize: {},
        }
      },
      confirm_dialog: false,
      modal_update: false,
      selectedTag: null,
      images_file: null,
      selectedFile: null,
      subprojectFile: null,
    }
  },
  methods: {
    getImagesFiles(){
      const image_types = 'png , jpeg, jpg';
      if(this.current_subproject !== null){
        if(this.current_subproject.files !== null){
          this.images_file = this.current_subproject.files.filter(obj => {
            return image_types.includes(obj.type)
          })
        }
        else{
          this.images_file = null;
        }
      }
      else{
        this.images_file = null;
      }


    },
    getUser(){
      this.$axios.get('/user')
        .then((response) =>{
          this.is_admin = response.data.rules === 'ADMIN';
        })
        .catch((errors)=>{
          console.log(errors)
        })
    },
    deleteFile(){
      console.log(this.selectedFile)
      this.$axios.delete('/files/' + this.id_file)
        .then((response) => {
          alert(response.data.message)
          location.reload()
        })
    },
    deleteTag(){
      this.$axios.delete('/tags/' + this.id_tag)
        .then((response) => {
          alert(response.data.message)
          location.reload()
        })
    },
    addFilesToProject(){
      let fm = new FormData();
      for (var i = 0; i < this.subprojectFile.length; i++) {
        fm.append("files_add[]", this.subprojectFile[i]);
      }
      this.$axios.post('/add-files-to-subproject/' + this.current_subproject.id, fm)
        .then((response) =>{
          alert(response.data.message)
          this.project_files = null
          location.reload()
        })

    },
    getCurrent(){
      this.$axios.get('/subprojects/' + this.id)
        .then((response) => {
          console.log(this.id);
          this.current_subproject = response.data.data;
        })
    },
    updateProject(){
      this.form.name = this.current_subproject.name;
      this.form.description = this.current_subproject.description;
      this.$axios.put('/subprojects/' + this.id, this.form)
        .then((response) => {
          alert('Данные обновлены');

          location.reload();
        })
    },
    deleteSubProject(){
      this.$axios.delete('/subprojects/' + this.id)
        .then((response) => {
          alert('Лист удален');
          location.reload();
        })
    }
  },
  mounted (){
    this.getCurrent();
  },
  computed(){
    this.getCurrent();
  }
}
</script>
