<template >
  <v-card >
      <v-toolbar dark color="error" dense flat>
          <v-toolbar-title class="white--text">Delete files</v-toolbar-title>
          <button type="button" class="btn-close btn-close-white position-relative top-10 end-0" style="left:148px" @click="closeDialog"></button>
      </v-toolbar>
      <v-card-text class="pa-4 text-center" >
        You are about to delete {{files.length}} files. Do you wish to proceed?
      </v-card-text>
      <v-card-actions class="pt-0">
          <v-spacer></v-spacer>
          <v-btn depressed color="grey lighten-2" @click="closeDialog">Cancel</v-btn>
          <v-btn depressed color="red" @click="confirmAction">Yes</v-btn>
      </v-card-actions>
  </v-card>
</template>
<script>
  export default {
    props: {
      icons: Object,
      files: Array,
      path: String,
      endpoints: Object,
      axios: Function,
    },
    emits: ["deleteItemsConfirmDialog", "clearCheckedFiles"],
    data() {
      return {
      };
    },
    methods: {
      closeDialog(){
        this.$emit("deleteItemsConfirmDialog", false);
      },
      selectIds(){
        return this.files.map(function(f){
            return f.Id;
        });
      },
      confirmAction(){
        this.$emit("clearCheckedFiles", this.files);
        this.closeDialog();
      }
    }
  }
</script>

<style scoped>
  
</style>