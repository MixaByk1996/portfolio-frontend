<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="drawer" app>
      <template v-if="projects">
        <v-list>
            <v-list-item-content>
                <v-list-item-title>User: {{this.currentUser.login}}</v-list-item-title>
              <v-list-item-subtitle>{{this.isAdminUser ? 'Администратор' : 'Пользователь'}}</v-list-item-subtitle>
            </v-list-item-content>
          <v-divider></v-divider>
          <template v-if="isAdminUser">
            <v-list-item link @click="isMenuAdmin = true; isMenuSearch=false; is_temp_subproject = false; is_temp_project = false;">
              <v-list-item-action>
                <v-icon >mdi-wrench</v-icon>
              </v-list-item-action>
              <v-list-item-content>
                <v-list-item-title>
                  Меню админа
                </v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-divider></v-divider>

          </template>
          <v-list-item link @click="isMenuSearch = true; isMenuAdmin = false;is_temp_subproject = false; is_temp_project = false;">
            <v-list-item-action>
              <v-icon>
                mdi-magnify
              </v-icon>
            </v-list-item-action>
            <v-list-item-content>
              <v-list-item-title>
                Меню поиска
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-divider></v-divider>
            <v-list-group v-for="item in projects" :key="item.id" :value="item.name">
                  <template v-slot:activator>
                    <v-list-item @click="clickProject(item)">
                      <v-icon>
                        mdi-folder-open
                      </v-icon>
                      <v-list-item-content>
                        <v-list-item-title  v-text="item.name"></v-list-item-title>
                      </v-list-item-content>
                    </v-list-item>
                  </template>

              <v-list-item style="background-color: beige"  v-for="project in item.subproject" :value="project.id" :key="project.id" @click="clickSubProject(project.id)">
                <v-icon>
                  mdi-playlist-edit
                </v-icon>
                <v-list-item-content>
                  <v-list-item-title v-text="project.name"></v-list-item-title>
                </v-list-item-content>
              </v-list-item>

              <v-list-item style="background-color: beige"  v-for="file in item.files" :value="file.id" :key="file.id" @click="downloadFile(file.file_url, file.name)">
                <v-icon>
                  mdi-file-send
                </v-icon>
                <v-list-item-content>
                  <v-list-item-title v-text="file.name"></v-list-item-title>
                </v-list-item-content>
              </v-list-item>

              <v-list-item style="background-color: beige"  v-for="tag in item.tags" :value="tag.id" :key="tag.id">
                <v-icon>
                  mdi-tag
                </v-icon>
                <v-list-item-content>
                  <v-list-item-title v-text="tag.name"></v-list-item-title>
                </v-list-item-content>
              </v-list-item>





<!--              <v-list-group v-for="project in item.subproject" :key="project.id" :value="project.name" class="pl-10">-->
<!--                <template v-slot:activator >-->
<!--              mdi-file-send    -->

<!--                </template>-->

<!--              </v-list-group>-->
            </v-list-group>
          <v-divider></v-divider>
          <v-list-item link @click="logout">
            <v-list-item-action>
              <v-icon color="red">mdi-power</v-icon>
            </v-list-item-action>
            <v-list-item-content>
              <v-list-item-title class="red--text">
                Выйти
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </template>
    </v-navigation-drawer>

    <v-app-bar app>

      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-app-bar-title>Приложение</v-app-bar-title>
    </v-app-bar>

    <v-main>
      <template v-if="isMenuAdmin">
        <MenuAdmin></MenuAdmin>
      </template>
      <template v-if="isMenuSearch">
        <SearchComponent></SearchComponent>
      </template>
      <template v-if="is_temp_project">
        <ProjectComponent v-bind:project="temp_project"></ProjectComponent>
      </template>
      <template v-if="is_temp_subproject">
        <SubprojectComponent v-bind:id="temp_subproject"></SubprojectComponent>
      </template>
    </v-main>
  </v-app>
</template>

<script>
import {nextTick, ref} from 'vue'
import MenuAdmin from "~/components/menu-admin.vue";
import SearchComponent from "~/components/search-component.vue";
import ProjectComponent from "~/components/project-component.vue";
import SubprojectComponent from "~/components/subproject-component.vue";
const drawer = ref(null)
export default {
  components: {SubprojectComponent, ProjectComponent, MenuAdmin,SearchComponent},
  beforeMount() {
    if (process.client) {
      this.token = localStorage.getItem('token');
    }
    this.$axios.defaults.headers.common['Authorization'] = `Bearer ${this.token}`;
    this.getUser();
    this.$axios.get('/projects')
      .then(response => {
        this.projects = Array.from(response.data.data);
        console.log(this.projects);
      })
  },
  data() {
    return {
      showMainMenu: false,
      drawer: null,
      projects: null,
      token : null,
      currentUser: {},
      isAdminUser: false,
      isMenuAdmin:false,
      isMenuSearch:false,
      temp_project: null,
      is_temp_subproject: false,
      is_temp_project: false,
      temp_subproject: null,
      baseURL: 'https://8657437b.com',
      itemsMain: [
        { title: 'Click Me222' },
        { title: 'Click Me2222' },
        { title: 'Click Me2' },
        { title: 'Click Me 2' },
      ],
    }
  },
  methods:{
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
    async clickProject(item){
      this.temp_project = item;
      this.is_temp_project = false;
      await this.$nextTick();
      this.is_temp_project = true;
      this.is_temp_subproject = false;
      this.isMenuAdmin = false;
      this.isMenuSearch = false;

    },
    async clickSubProject(id){
      this.temp_subproject = id;
      this.is_temp_project = false;
      this.is_temp_subproject = false;
      await this.$nextTick();
      this.is_temp_subproject = true;
      this.isMenuAdmin = false;
      this.isMenuSearch = false;
    },
    getUser(){
      this.$axios.get('/user')
        .then((response) =>{
          this.currentUser = response.data;
          this.isAdminUser = this.currentUser.rules === 'ADMIN';
          console.log(response.data);

        })
        .catch((errors)=>{
          console.log(errors)
        })
    },
    logout(){
      this.$axios.post("/logout")
        .then(response =>{
          localStorage.removeItem('token');
          this.$router.push('/');
        })
        .catch(errors => {
          console.log(errors.response.data);
        })
    },
    getAlert(title){
      alert(title);
    }
  },
  created() {

  },
}
</script>
