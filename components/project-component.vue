<template>
  <v-container style="margin: 5px 5px 5px 5px">
  <template v-if="modal_update == false">
    <v-card >
      Проект: {{project.name}} <br>
      Описание : <br>
      <div v-html="project.description"></div>
<!--      <tiptap-vuetify-->
<!--        v-model="project.description"-->
<!--        :extensions="extensions"-->
<!--        disabled="disabled"-->
<!--      ></tiptap-vuetify>-->
      <br>
      Компания: {{project.company.name}}
      <template v-if="this.project.tags.length > 0">
        <v-list>
          <v-subheader>Теги</v-subheader>
          <v-list-item-group v-model="selectedTag">
            <v-list-item
              v-for="item in project.tags"
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
          <template v-if="this.project.files.length > 0">
            <v-list >
              <v-subheader>Файлы</v-subheader>
              <v-list-item-group v-model="selectedFile">
                <v-list-item
                  v-for="item in project.files"
                  :key="item.id"
                  :value="item.id"
                  @click.prevent="downloadFile(item.file_url, item.name)"
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
          <template v-if="images_file.length > 0">
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


      <template v-if="this.project.subproject.length > 0">
        <v-list >
          <v-subheader>Список подпроектов</v-subheader>
          <v-list-item-group v-model="selectedSubproject">
            <v-list-item
              v-for="item in project.subproject"
              :key="item.id"
              :value="item.name"
            >
              <v-list-item-title v-text="item.name">
              </v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </template>
      <template v-else>
        <br> Подпроекты отсувствуют
      </template>
      <br>
      <template v-if="is_admin">
        <v-row>
          <v-col>
            <v-btn @click="modal_update = true">Редактировать</v-btn>
            <v-btn @click="deleteProject">Удалить проект</v-btn>
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
      <p>Описание проекта</p>
      <tiptap-vuetify
        v-model="form.description"
        :extensions="extensions"
      ></tiptap-vuetify>
      <v-btn @click="updateProject">Обновить информацию</v-btn><br>

      <template v-if="this.project.tags.length > 0">
        <v-list>
          <v-subheader>Теги</v-subheader>
          <v-list-item-group v-model="selectedTag">
            <v-list-item @click="id_tag = item.id"
              v-for="item in project.tags"
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


      <template v-if="this.project.files.length > 0">
        <v-list >
          <v-subheader>Файлы</v-subheader>
          <v-list-item-group v-model="selectedFile">
            <v-list-item @click="id_file = item.id"
              v-for="item in project.files"
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
        v-model="project_files"
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
        <v-row>
          <v-col>
            <v-btn @click="modal_update = false">Закончить редактирование</v-btn>
          </v-col>
        </v-row>
      </template>
    </v-card>
    </template>

  </v-container>
</template>
<script>
import {axios} from 'axios';
import { TiptapVuetify, Heading, Bold, Italic, Strike, Underline, Code, Paragraph, BulletList, OrderedList, ListItem, Link, Blockquote, HardBreak, HorizontalRule, History } from 'tiptap-vuetify'
export default {
  props: ['project'],
  components: { TiptapVuetify },
  beforeMount() {
    console.log(this.project.files)
    this.form.name = this.project.name
    this.form.description = this.project.description
    this.getImagesFiles()
    this.getUser()
    console.log(this.images_file)
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
      form:{
        name: "",
        description: ""
      },
      baseURL: 'https://8657437b.com',
      id_file: 0,
      id_tag: 0,
      project_files: null,
      images_file: null,
      is_admin: false,
      selectedTag: null,
      modal_update: false,
      selectedFile: null,
      selectedSubproject: null,
    }
  },
  methods:{
    getImagesFiles(){
      const image_types = 'png , jpeg, jpg';
      this.images_file = this.project.files.filter(obj => {
        return image_types.includes(obj.type)
      })
    },
    downloadFile(url, name){
      console.log(url);
      var fullurl = this.baseURL + url;
          const link = document.createElement('a')
          link.href = fullurl
          link.target = '_blank'
          link.setAttribute('download', name)
          document.body.appendChild(link)
          link.click()
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
      this.$axios.post('/add-files-to-project/' + this.project.id, fm)
        .then((response) =>{
          alert(response.data.message)
          this.project_files = null
          location.reload()
        })

    },
    updateProject(){
      this.$axios.put('/projects/' + this.project.id, this.form)
        .then((response) => {
          alert('Данные обновлены');
          location.reload();
        })
    },
    deleteProject(){
      this.$axios.delete('/projects/' + this.project.id)
        .then((response) => {
          alert('Проект удален');
          location.reload();
        })
    }
  }
}
</script>
