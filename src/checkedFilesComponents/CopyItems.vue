<template>
  <v-overlay :absolute="true">
    <v-card flat light :loading="loading" class="pa-5" color="grey lighten-5">

      <v-row class="ma-0 position-relative" >
        <v-col>
          <v-card-text class="text-h5"> Copy Files </v-card-text>
        </v-col>
        <v-col>
          <v-btn icon fab absolute right class="rounded-0" small @click="closeDialog" >
            <v-icon>mdi-close-thick</v-icon>
          </v-btn>
        </v-col>
      </v-row>
      <v-row class="ma-0 container" >
        <v-col>
          Choose a new location for these {{files.length}} files and press Confirm to move them.
        </v-col>
      </v-row>

      <v-card flat tile width="550" color="grey lighten-5" min-height="300" max-height="300" class="d-flex flex-column" style="height:100%; left:5px; margin-bottom:18px; overflow-x:auto">
        <v-treeview
          :items="foldersAsTree"
          item-key="Id"
          item-text="Name"
          v-on:update:active="activeChanged"
          :active="active"
          transition
          dense
          color="grey darken-3"
          hoverable
          activatable
          style="cursor:pointer"
        ></v-treeview>
      </v-card>

      <v-card-actions class="pt-0" color="transparent">
        <v-spacer></v-spacer>
        <v-btn depressed color="grey lighten-2" @click="closeDialog">Cancel</v-btn>
        <v-btn depressed color="success" @click="confirmAction">Confirm</v-btn>
      </v-card-actions>

      <v-dialog v-model="argumentException">
        <wrong-input
          :text="argumentText"
          :icons="icons"
          @confirm="(e)=>argumentException=e"
        ></wrong-input>
      </v-dialog>

    </v-card>
    
  </v-overlay>
</template>

<script>
import WrongInput from "./WrongInput.vue";

  export default {
    props: {
      icons: Object,
      files: Array,
      endpoints: Object,
      axios: Function,
      foldersAsTree: Array,
      foldersAsList: Array
    },
    components: {
      WrongInput
    },
    emits: ["copyItemsConfirmDialog", "clearCheckedFiles"],
    data() {
      return {
        active: [],
        argumentException: false,
        argumentText: ""
      };
    },
    methods: {
      closeDialog(){
        this.$emit("copyItemsConfirmDialog", false);
        this.init();
      },
      async confirmAction(){
        if (this.files.length > 0 && this.active.length > 0){
          this.$emit("clearCheckedFiles", this.files, this.active[0]);
          this.closeDialog();
        }
        else {
          this.argumentText = this.active.length < 1 ? 
            "No folder selected. Select one and then press Confirm, or cancel to return back." : 
            "Unexpected outcome. Please refresh.";
          this.argumentException = true;
        }
      },
      init() {
        this.active = [];
        this.items = [];
        this.argumentException = false;
        this.argumentText = "";
      },
      activeChanged(active){
        this.active = active;
      },
      getFolderFromId(id){
        return this.foldersAsList.find(x => x.Id == id);
      }
    }
  }
</script>

<style scoped>
  
</style>