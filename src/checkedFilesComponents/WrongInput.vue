<template>
  <v-overlay :absolute="true">
    <v-card flat light :loading="loading" class="pa-5" color="grey lighten-5">

      <v-row class="ma-0 position-relative" >
        <button type="button" class="btn-close position-absolute top-10 end-0" aria-label="Close" @click="confirmExit"></button>
        <v-card-text class="text-h5 d-flex justify-center"> Notification </v-card-text>
      </v-row>
      <v-row class="ma-0 container" >
        <div class="row">
         <p>{{text}}</p>
        </div>
      </v-row>

      <v-card-actions class="pt-0" color="transparent">
        <v-spacer></v-spacer>
        <v-btn depressed color="success" @click="confirmExit">OK</v-btn>
      </v-card-actions>

    </v-card>
  </v-overlay>
</template>

<script>
  export default {
    props: {
      icons: Object,
      text: String
    },
    emits: ["confirm"],
    methods: {
      confirmExit(){
        this.$emit("confirm", false);
      }
    }
  }
</script>

<style scoped>
  
</style>