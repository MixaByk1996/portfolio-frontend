<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="drawer" app>
      <template v-if="companies">
        <v-list>
<!--          <v-subheader>Компании</v-subheader>-->

            <v-list-group v-for="item in companies" :key="item.id" :value="item.name" >
              <template v-slot:activator>
                <v-list-item-content>
                  <v-list-item-title v-text="item.name"></v-list-item-title>
                </v-list-item-content>
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

        </v-list>
      </template>
    </v-navigation-drawer>

    <v-app-bar app>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-app-bar-title>{{this.currentUser.login}}</v-app-bar-title>
    </v-app-bar>
    <v-footer color="indigo" app></v-footer>
    <v-main>
<!--      <v-btn>блаблабла</v-btn>-->
    </v-main>
  </v-app>
</template>

<script>
import { ref } from 'vue'

const drawer = ref(null)
export default {
  beforeMount() {
    this.$axios.get('/company')
      .then(response => {

        this.companies = Array.from(response.data.data);
        console.log(this.companies);
      })
  },
  data() {
    return {
      drawer: null,
      companies: null,
      token : localStorage.getItem('token'),
      currentUser: {}
    }
  },
  methods:{
    getUser(){
      this.$axios.get('/user')
        .then((response) =>{
          this.currentUser = response.data;
          console.log(response.data);
        })
        .catch((errors)=>{
          console.log(errors)
        })
    }
  },
  created() {
    this.$axios.defaults.headers.common['Authorization'] = `Bearer ${this.token}`;
    this.getUser();
  }
}
</script>
