<template>
  <v-container>
    <v-card>
      <v-card-title>
        Проект: {{project.name}}
      </v-card-title>
    </v-card>
  </v-container>
</template>
<script>
import {axios} from 'axios';
export default {
  props:{
    project: Object
  },
}
</script>
