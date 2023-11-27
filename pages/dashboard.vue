<template>
  <v-app id="inspire">

    <v-navigation-drawer v-model="drawer" app>
      <template v-if="companies">
        <v-list>
<!--          <v-subheader>Компании</v-subheader>-->
            <v-list-item-content>
              <v-list-item-title>User: {{this.currentUser.login}}</v-list-item-title>
              <v-list-item-subtitle>{{this.isAdminUser ? 'Администратор' : 'Пользователь'}}</v-list-item-subtitle>
            </v-list-item-content>
          <v-divider></v-divider>
          <template v-if="isAdminUser">
            <v-list-item link @click="isMenuAdmin = !isMenuAdmin">
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

            <v-list-group v-for="item in companies" :key="item.id" :value="item.name"  >

              <template v-slot:activator>
                <v-list-item-content>

                  <v-list-item-title v-text="item.name"></v-list-item-title>
                </v-list-item-content>
                <template v-if="isAdminUser">
                  <v-list-item-action>
                    <v-menu
                      bottom
                      left
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-btn
                          icon
                          v-bind="attrs"
                          v-on="on"
                        >
                          <v-btn icon>
                            <v-icon>mdi-plus</v-icon>
                          </v-btn>
                        </v-btn>
                      </template>

                      <v-list>
                        <v-list-item
                          v-for="(item, i) in items"
                          :key="i" link @click="getAlert(item.title)"
                        >
                          <v-list-item-title>{{ item.title }}</v-list-item-title>
                        </v-list-item>
                      </v-list>
                    </v-menu>

                  </v-list-item-action>
                </template>

              </template>
<!--              <v-subheader>Проекты компани {{item.name}}</v-subheader>-->
              <v-list-group v-for="project in item.projects" :key="project.id" class="pl-10">
                <template v-slot:activator>
                  <v-list-item-content>
                    <v-list-item-title v-text="project.name"></v-list-item-title>
                  </v-list-item-content>
                </template>
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

      <v-app-bar-title>{{this.currentUser.login}}</v-app-bar-title>
    </v-app-bar>
    <v-footer color="indigo" app></v-footer>
    <v-main>
      <template v-if="isMenuAdmin">
        <MenuAdmin></MenuAdmin>
      </template>
    </v-main>
  </v-app>
</template>

<script>
import { ref } from 'vue'
import MenuAdmin from "~/pages/menu-admin.vue";
const drawer = ref(null)
export default {
  components: {MenuAdmin},
  beforeMount() {

    this.$axios.get('/company')
      .then(response => {

        this.companies = Array.from(response.data.data);
        console.log(this.companies);
      })
  },
  data() {
    return {
      showMainMenu: false,
      drawer: null,
      companies: null,
      token : localStorage.getItem('token'),
      currentUser: {},
      isAdminUser: false,
      isMenuAdmin:false,
      items: [
        { title: 'Click Me' },
        { title: 'Click Me' },
        { title: 'Click Me' },
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
    this.$axios.defaults.headers.common['Authorization'] = `Bearer ${this.token}`;
    this.getUser();
  },
}
</script>
