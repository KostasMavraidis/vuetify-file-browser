<template>
  <v-card flat tile min-height="380" class="d-flex flex-column">
    <confirm ref="confirm"></confirm>
    <v-card-text
      v-if="!path"
      class="grow d-flex justify-center align-center grey--text"
      >Select a folder or a file</v-card-text
    >
    <v-card-text
      v-else-if="isFile"
      class="grow d-flex justify-center align-center"
      >File: {{ path }}</v-card-text
    >
    <v-card-text v-else-if="dirs.length || files.length" class="grow">
      <v-list subheader v-if="dirs.length">
        <v-subheader>Folders</v-subheader>
        <v-list-item
          v-for="item in dirs"
          :key="item.basename"
          @click="changePath(item.path)"
          class="pl-0"
        >
          <v-list-item-avatar class="ma-0">
            <v-icon>mdi-folder-outline</v-icon>
          </v-list-item-avatar>
          <v-list-item-content class="py-2">
            <v-list-item-title v-text="item.basename"></v-list-item-title>
          </v-list-item-content>
          <v-list-item-action>
                 <div class="row q-col-gutter-sm items-center">
            <v-btn icon @click.stop="deleteItem(item)">
              <v-icon color="grey lighten-1">mdi-delete-outline</v-icon>
            </v-btn>
            <v-btn icon v-if="false">
              <v-icon color="grey lighten-1">mdi-information</v-icon>
            </v-btn>
              </div>
          </v-list-item-action>
        </v-list-item>
      </v-list>
      <v-divider v-if="dirs.length && files.length"></v-divider>
      <v-list subheader v-if="files.length">
        <v-subheader>Files</v-subheader>
        <v-list-item
          v-for="item in files"
          :key="item.basename"
          @click="changePath(item.path)"
          class="pl-0"
        >
          <v-list-item-avatar class="ma-0">
            <v-icon>{{
              icons[item.extension.toLowerCase()] || icons["other"]
            }}</v-icon>
          </v-list-item-avatar>

          <v-list-item-content class="py-2">
            <v-list-item-title v-text="item.basename"></v-list-item-title>
            <v-list-item-subtitle>{{
              formatBytes(item.size)
            }}</v-list-item-subtitle>
          </v-list-item-content>

          <v-list-item-action>
                     <div class="row q-col-gutter-sm items-center">       
                   <v-btn icon @click.stop="downloadItem(item)">
              <v-icon color="grey lighten-1">mdi-download-outline</v-icon>
            </v-btn>
            <v-btn icon v-if="false">
              <v-icon color="grey lighten-1">mdi-information</v-icon>
            </v-btn>
              <v-btn icon @click.stop="deleteItem(item)">
              <v-icon color="grey lighten-1">mdi-delete-outline</v-icon>
            </v-btn>
            <v-btn icon v-if="false">
              <v-icon color="grey lighten-1">mdi-information</v-icon>
            </v-btn>
              <v-btn icon @click.stop="shareItem(item)">
              <v-icon color="grey lighten-1">mdi-share-variant-outline</v-icon>
            </v-btn>
            <v-btn icon v-if="false">
              <v-icon color="grey lighten-1">mdi-information</v-icon>
            </v-btn>
            </div>
          </v-list-item-action>
        </v-list-item>
      </v-list>
    </v-card-text>
    <v-card-text
      v-else-if="filter"
      class="grow d-flex justify-center align-center grey--text py-5"
      >No files or folders found</v-card-text
    >
    <v-card-text
      v-else
      class="grow d-flex justify-center align-center grey--text py-5"
      >The folder is empty</v-card-text
    >
    <v-divider v-if="path"></v-divider>
    <v-toolbar v-if="false && path && isFile" dense flat class="shrink">
      <v-btn icon>
        <v-icon>mdi-download</v-icon>
      </v-btn>
    </v-toolbar>
    <v-toolbar v-if="path && isDir" dense flat class="shrink">
      <v-text-field
        solo
        flat
        hide-details
        label="Filter"
        v-model="filter"
        prepend-inner-icon="mdi-filter-outline"
        class="ml-n3"
      ></v-text-field>
      <v-btn icon v-if="false">
        <v-icon>mdi-eye-settings-outline</v-icon>
      </v-btn>
      <v-btn icon @click="load()">
        <v-icon>mdi-refresh</v-icon>
      </v-btn>
    </v-toolbar>
    <v-dialog v-model="dialog"  >

       <email-finder
          @closeDialog="(e)=>dialog=e"
            :axios="axios"
            :path="path"
            :file="selectedItem"
            :icons="icons"
            :endpoints="endpoints"
        ></email-finder>
         </v-dialog>
  </v-card>
  
</template>

<script>
import { formatBytes } from "./util";
import Confirm from "./Confirm.vue";
import EmailFinder from "./EmailFinder.vue"

export default {
  props: {
    icons: Object,
    storage: String,
    path: String,
    endpoints: Object,
    axios: Function,
    refreshPending: Boolean,
  },
  components: {
    Confirm,
    EmailFinder
  },
  data() {
    return {
      selectedItem: null,
      dialog: false,
      items: [],
      filter: "",
    };
  },
  computed: {
    dirs() {
      return this.items.filter(
        (item) => item.type === "dir" && item.basename.includes(this.filter)
      );
    },
    files() {
      return this.items.filter(
        (item) => item.type === "file" && item.basename.includes(this.filter)
      );
    },
    isDir() {
      return this.path[this.path.length - 1] === "/";
    },
    isFile() {
      return !this.isDir;
    },
  },
  methods: {
    formatBytes,
    changePath(path) {
      this.$emit("path-changed", path);
    },
    async load() {
      this.$emit("loading", true);
      console.log(this)
      if (this.isDir) {
        let url = this.endpoints.list.url
          .replace(new RegExp("{storage}", "g"), "main")
          .replace(new RegExp("{path}", "g"), this.path);

        let config = {
          url,
          method: this.endpoints.list.method || "get",
        };

        let response = await this.axios.request(config);
        this.items = response.data;
      } else {
        // TODO: load file
      }
      this.$emit("loading", false);
    },
    async deleteItem(item) {
      let confirmed = await this.$refs.confirm.open(
        "Delete",
        `Are you sure<br>you want to delete this ${
          item.type === "dir" ? "folder" : "file"
        }?<br><em>${item.basename}</em>`
      );

      if (confirmed) {
        console.log(item.type)
        
        this.$emit("loading", true);
        if(item.type !==  "dir"){
        let url = this.endpoints.delete.url
          .replace(new RegExp("{storage}", "g"), "main")
          .replace(new RegExp("{Id}", "g"), item.Id);

        let config = {
          url,
          method: this.endpoints.delete.method || "delete",
        };
          await this.axios.request(config);
        this.$emit("file-deleted");
        this.$emit("loading", false);
        }
        else{
          console.log(item.path)
            let url = this.endpoints.dirdelete.url
          .replace(new RegExp("{storage}", "g"), "main")
          .replace(new RegExp("{path}", "g"), item.path);

        let config = {
          url,
          method: this.endpoints.dirdelete.method || "delete",
        };
          await this.axios.request(config);
        this.$emit("file-deleted");
        this.$emit("loading", false);
        }
      
      
      }
    },
    async downloadItem(item) {
      let url = this.endpoints.download.url
        .replace(new RegExp("{storage}", "g"), "main")
           .replace(new RegExp("Id", "g"), item.Id);
        
        
      let config = {
        url,
        method: this.endpoints.download.method || "get",
      };
      var items = await this.axios.request(config);
    console.log(items.data)
  
   // console.log(item.extension)
    const url2 = window.URL.createObjectURL(new Blob( [this.base64ToArrayBuffer(items.data)]));
        const link = document.createElement("a");
        link.href = url2;
        link.setAttribute("download", item.name); //or any other extension
        link.setAttribute("id", "link1"); //or any other extension
        document.body.appendChild(link);
        link.click();
        var element = document.getElementById("link1");
           element.parentNode.removeChild(element);

    },
    async shareItem(item){ 
      this.dialog=true;
      this.selectedItem=item;
          console.log(item);
    },
    base64ToArrayBuffer(base64) {
    var binaryString = window.atob(base64);
    var binaryLen = binaryString.length;
    var bytes = new Uint8Array(binaryLen);
    for (var i = 0; i < binaryLen; i++) {
       var ascii = binaryString.charCodeAt(i);
       bytes[i] = ascii;
    }
    return bytes;
 }
  },
  watch: {
    async path() {
      this.items = [];
      await this.load();
    },
    async refreshPending() {
      if (this.refreshPending) {
        await this.load();
        this.$emit("refreshed");
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.v-card {
  height: 100%;
}
</style>