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
          <v-text-field v-model="form.name"
                        label="Наименование"
          >
          </v-text-field>
          <p>Описание подпроекта</p>
          <tiptap-vuetify
            v-model="form.description"
            :extensions="extensions"
          ></tiptap-vuetify>
          <v-btn @click="updateProject">Обновить информацию</v-btn>

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
              <v-btn @click="deleteTag"></v-btn>
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


          <template v-if="is_admin">
            <v-btn @click="modal_update = false">Закончить редактирование</v-btn>
          </template>
      </v-card>
    </template>


  </v-container>
</template>
<script>
import {axios} from 'axios';
import { TiptapVuetify, Heading, Bold, Italic, Strike, Underline, Code, Paragraph, BulletList, OrderedList, ListItem, Link, Blockquote, HardBreak, HorizontalRule, History } from 'tiptap-vuetify';
export default {
  props: ['id'],
  components: { TiptapVuetify },
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
      this.images_file = this.current_subproject.files.filter(obj => {
        return image_types.includes(obj.type)
      })
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
      for (var i = 0; i < this.project_files.length; i++) {
        fm.append("files_add[]", this.project_files[i]);
      }
      this.$axios.post('/add-files-to-subproject/' + this.project.id, fm)
        .then((response) =>{
          alert(response.data.message)
          this.project_files = null
          location.reload()
        })

    },
    getCurrent(){
      this.$axios.get('/subprojects/' + this.id)
        .then((response) => {
          this.current_subproject = response.data.data;
        })
    },
    updateProject(){
      this.$axios.put('/subprojects/' + this.id, this.form)
        .then((response) => {
          alert('Данные обновлены');
          location.reload();
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
