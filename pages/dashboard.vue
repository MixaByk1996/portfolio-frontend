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
            <v-list-item link @click="isMenuAdmin = !isMenuAdmin; isMenuSearch=false">
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
          <v-list-item link @click="isMenuSearch = !isMenuSearch; isMenuAdmin = false">
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
            <v-list-group v-for="item in projects" :key="item.id" :value="item.name"  >

              <template v-slot:activator>
                <v-icon>
                  mdi-folder-open
                </v-icon>
                <v-list-item-content>
                  <v-list-item-title  v-text="item.name"></v-list-item-title>
                </v-list-item-content>
              </template>

              <v-list-group v-for="project in item.subproject" :key="project.id" class="pl-10">
                <template v-slot:activator>
                  <v-icon>
                    mdi-folder-open
                  </v-icon>
                  <v-list-item-content>
                    <v-list-item-title v-text="project.name"></v-list-item-title>
                  </v-list-item-content>


                </template>

<!--                <v-list-group v-for="object in project.subproject" :key="object.id" class="pl-10">-->
<!--&lt;!&ndash;                  <template v-slot:activator>&ndash;&gt;-->
<!--                    <v-list-item-content>-->
<!--                      <v-list-item-title v-text="object.name"></v-list-item-title>-->
<!--                    </v-list-item-content>-->
<!--&lt;!&ndash;                  </template>&ndash;&gt;-->
<!--                </v-list-group>-->

              </v-list-group>
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
    </v-main>
  </v-app>
</template>

<script>
import { ref } from 'vue'
import MenuAdmin from "~/components/menu-admin.vue";
import SearchComponent from "~/components/search-component.vue";
const drawer = ref(null)
export default {
  components: {MenuAdmin,SearchComponent},
  beforeMount() {
    if (process.client) {
      this.token = localStorage.getItem('token');
    }
    this.$axios.defaults.headers.common['Authorization'] = `Bearer ${this.token}`;
    this.getUser();
    this.$axios.get('/projects')
      .then(response => {
        this.projects = Array.from(response.data.data);
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
      itemsMain: [
        { title: 'Click Me222' },
        { title: 'Click Me2222' },
        { title: 'Click Me2' },
        { title: 'Click Me 2' },
      ],
    }
  },
  methods:{
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
