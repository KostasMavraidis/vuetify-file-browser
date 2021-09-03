<template>
  <v-overlay :absolute="true">
    <v-card flat light class="pa-5" color="grey lighten-5">

      <v-row class="ma-0 position-relative" >
        <button type="button" class="btn-close position-absolute top-10 end-0" aria-label="Close" @click="cancelAndNotClose"></button>
        <v-card-text class="text-h5 d-flex justify-center"> Exit </v-card-text>
      </v-row>
      <v-row class="ma-0 container" >
        <div class="row">
         <p>You have unsaved changes. Do you want to save them before exiting?</p>
        </div>
      </v-row>

      <v-card-actions class="pt-0" color="transparent">
        <v-spacer></v-spacer>
        <v-btn depressed color="grey lighten-2" @click="cancelAndNotClose">Cancel</v-btn>
        <v-btn depressed color="deep-orange accent-2" @click="noAndClose">No</v-btn>
        <v-btn depressed color="success" @click="yesAndClose">Yes</v-btn>
      </v-card-actions>

    </v-card>
  </v-overlay>
</template>

<script>
  export default {
    props: {
      icons: Object,
      file: Object, 
      ownedTags: Array,
      ownedCategories: Array,
      path: String,
      endpoints: Object,
      axios: Function,
    },
    emits: ["returnBack", "confirmedExit"],
    data() {
      return {
      };
    },
    methods: {
      cancelAndNotClose(){
        this.$emit("returnBack", false);
      },
      yesAndClose(){
        this.axios({
          method: "post",
          url: this.endpoints.postCategoriesAndTags.url,
          data: {
            FileId: this.file.Id,
            Tags: this.ownedTags,
            Categories: this.ownedCategories
          }
        })
        .then(()=>{this.noAndClose()});
      },
      noAndClose(){
        this.$emit("confirmedExit", true);
        this.$emit("returnBack", false);
      },
    }
  }
</script>

<style scoped>
  
</style>