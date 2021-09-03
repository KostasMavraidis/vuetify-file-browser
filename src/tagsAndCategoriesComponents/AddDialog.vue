<template>
  <v-overlay :absolute="true">
    <v-card flat light :loading="loading" class="pa-5" color="grey lighten-5">

      <v-row class="ma-0 position-relative" >
        <button type="button" class="btn-close position-absolute top-10 end-0" aria-label="Close" @click="closeWindow"></button>
        <v-card-text class="text-h5 d-flex justify-center customHeader"> Add {{ nameOfInstance }} </v-card-text>
      </v-row>
      <v-row class="ma-0 container" >
        <div class="row">
          <v-text-field
            v-model="itemName"
            :rules="nameRules"
            label="Item name"
            required
          ></v-text-field>
        </div>
      </v-row>

      <v-row class="ma-0">
        <v-toolbar dense flat color="transparent" class="d-flex flex-row-reverse">
          <v-btn depressed color="success" @click="Submit"> 
            Add 
          </v-btn>
        </v-toolbar>
      </v-row>
    </v-card>
  </v-overlay>
</template>

<script>
  export default {
    props: {
      icons: Object,
      ownedArray: Array,
      notOwnedArray: Array,
      nameOfInstance: String
    },
    emits: ["closeAddDialog", "addItemFromDialog"],
    data() {
      return {
        itemName: '',
        nameRules: [
          v => !!v || 'Name must contain at least one character',
          v => this.isNotExistent(v) || `There is already a ${this.nameOfInstance} with same or relevant name.`
        ]
      };
    },
    methods: {
      closeWindow() {
        this.itemName = "";
        this.$emit("closeAddDialog", false);
      },
      isNotExistent(aString){
        if (this.itemName == ""){
          return false;
        }
        for (let i=0; i<this.ownedArray.length; i++){
          if (this.ownedArray[i].Name.toLowerCase() == aString.toLowerCase()) {
            return false;
          }
        }
        for (let i=0; i<this.notOwnedArray.length; i++){
          if (this.notOwnedArray[i].Name.toLowerCase() == aString.toLowerCase()){
            return false;
          }
        }
        return true;
      },
      Submit(){
        if (this.isNotExistent(this.itemName)){
          this.$emit("addItemFromDialog", this.itemName);
        this.closeWindow();
        }
      }
    }
  }
</script>