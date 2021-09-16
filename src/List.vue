<template>
  <v-card flat tile height="100%" min-height="380" class="d-flex flex-column">
    <confirm ref="confirm"></confirm>
    <v-card-text
      v-if="!path"
      class="grow d-flex justify-center align-center grey--text"
      >Select a folder or a file</v-card-text
    >
    <v-card-text
      v-else-if="isFile"
      class="grow d-flex justify-center align-center"
    >
      <file-details
        :file.sync="selectedItem"
        :endpoints="endpoints"
        :axios="axios"
      />
    </v-card-text>
    <v-card-text v-else-if="dirs.length || files.length" class="grow">
      <v-list subheader v-if="dirs.length">
        <v-subheader>Folders</v-subheader>
        <v-list-item
          v-for="item in dirs"
          :key="item.Id"
          @click="changePath(item)"
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
        <v-card
          class="d-flex justify-space-between mb-6"
          color="transparent"
          flat
          tile
        >
          <v-subheader>Files</v-subheader>
          <v-list-item-action>
            <div
              class="row q-col-gutter-sm items-center"
              style="margin-right: 5px"
            >
              <v-btn
                icon
                @click.stop="deleteItems"
                title="delete items"
                :disabled="noFilesSelected"
              >
                <v-icon color="grey darken-1">mdi-delete-outline</v-icon>
              </v-btn>
              <v-btn
                icon
                @click.stop="moveItems"
                title="move items"
                :disabled="noFilesSelected"
              >
                <v-icon color="grey darken-1">mdi-file-move</v-icon>
              </v-btn>
              <v-btn
                icon
                @click.stop="copyItems"
                title="copy items"
                :disabled="noFilesSelected"
              >
                <v-icon color="grey darken-1">mdi-content-copy</v-icon>
              </v-btn>
            </div>
          </v-list-item-action>
        </v-card>

        <v-list-item
          v-for="item in files"
          :key="item.Id"
          @click.stop="changePath(item)"
          class="pl-0"
        >
          <v-list-item-action
            @mouseenter="isCheckBox = true"
            @mouseleave="isCheckBox = false"
          >
            <v-checkbox
              v-model="checkedFiles"
              color="grey darken-1"
              hide-details
              :value="item"
              class="pl-3"
            >
            </v-checkbox>
          </v-list-item-action>

          <v-list-item-avatar class="ma-0">
            <v-icon>
              {{ icons[item.extension.toLowerCase()] || icons["other"] }}
            </v-icon>
          </v-list-item-avatar>

          <v-list-item-content class="py-2">
            <v-list-item-title v-text="item.basename"></v-list-item-title>
            <v-list-item-subtitle>
              {{ formatBytes(item.size) }}
            </v-list-item-subtitle>
          </v-list-item-content>

          <v-list-item-action>
            <div class="row q-col-gutter-sm items-center">
              <v-btn icon @click.stop="downloadItem(item)">
                <v-icon color="grey darken-1">mdi-download-outline</v-icon>
              </v-btn>
              <v-btn icon @click.stop="deleteItem(item)">
                <v-icon color="grey darken-1">mdi-delete-outline</v-icon>
              </v-btn>
              <v-btn icon @click.stop="shareItem(item)">
                <v-icon color="grey darken-1">mdi-share-outline</v-icon>
              </v-btn>

              <v-btn
                icon
                @click.stop="tagsAndCategoriesItem(item)"
                title="Tags and Categories"
              >
                <v-icon color="grey darken-1">mdi-tag-outline</v-icon>
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
    <v-dialog v-model="dialog">
      <email-finder
        @closeDialog="(e) => (dialog = e)"
        :axios="axios"
        :path="path"
        :file="selectedItem"
        :icons="icons"
        :endpoints="endpoints"
      ></email-finder>
    </v-dialog>

    <v-dialog v-model="dialogTagsAndcategories" persistent no-click-animation>
      <tags-and-categories
        @closeDialogTagsAndCategories="(e) => (dialogTagsAndcategories = e)"
        :axios="axios"
        :path="path"
        :file="selectedItem"
        :icons="icons"
        :endpoints="endpoints"
        :ownedTags="ownedTags"
        :notOwnedTags="notOwnedTags"
        :ownedCategories="ownedCategories"
        :notOwnedCategories="notOwnedCategories"
        :startingCategories="startingCategories"
        :startingTags="startingTags"
      ></tags-and-categories>
    </v-dialog>

    <v-dialog v-model="deleteItemsConfirmDialog" max-width="300">
      <delete-items
        @deleteItemsConfirmDialog="(e) => (deleteItemsConfirmDialog = e)"
        @clearCheckedFiles="ckeckedFilesDeleted"
        :axios="axios"
        :path="path"
        :icons="icons"
        :endpoints="endpoints"
        :files="checkedFiles"
      ></delete-items>
    </v-dialog>

    <v-dialog v-model="moveItemsConfirmDialog">
      <move-items
        @clearCheckedFiles="checkedFilesMoved"
        @moveItemsConfirmDialog="(e) => (moveItemsConfirmDialog = e)"
        :axios="axios"
        :icons="icons"
        :files="checkedFiles"
        :endpoints="endpoints"
        :foldersAsList="allFoldersAsList"
        :foldersAsTree="allFoldersAsTree"
      ></move-items>
    </v-dialog>

    <v-dialog v-model="copyItemsConfirmDialog">
      <copy-items
        @clearCheckedFiles="checkedFilesCopied"
        @copyItemsConfirmDialog="(e) => (copyItemsConfirmDialog = e)"
        :axios="axios"
        :icons="icons"
        :files="checkedFiles"
        :endpoints="endpoints"
        :foldersAsList="allFoldersAsList"
        :foldersAsTree="allFoldersAsTree"
      ></copy-items>
    </v-dialog>
  </v-card>
</template>


<script>
import { formatBytes } from "./util";
import Confirm from "./Confirm.vue";
import EmailFinder from "./EmailFinder.vue";
import FileDetails from "./FileDetails.vue";
import TagsAndCategories from "./TagsAndCategories.vue";
import DeleteItems from "./checkedFilesComponents/DeleteItems.vue";
import MoveItems from "./checkedFilesComponents/MoveItems.vue";
import CopyItems from "./checkedFilesComponents/CopyItems.vue";

export default {
  props: {
    icons: Object,
    storage: String,
    path: String,
    endpoints: Object,
    axios: Function,
    refreshPending: Boolean,
    item: Object,
    showTemporary: Boolean,
    startingFolderPath: String
  },
  components: {
    Confirm,
    EmailFinder,
    FileDetails,
    TagsAndCategories,
    DeleteItems,
    MoveItems,
    CopyItems,
  },
  data() {
    return {
      selectedItem: null,
      dialog: false,
      dialogTagsAndcategories: false,
      items: [],
      filter: "",
      startingCategories: [],
      startingTags: [],
      checkedFiles: [],
      deleteItemsConfirmDialog: false,
      moveItemsConfirmDialog: false,
      copyItemsConfirmDialog: false,
      allFoldersAsTree: [],
      allFoldersAsList: [],
      isCheckBox: false,
      ownedTags: null,
      notOwnedTags: null,
      ownedCategories: null,
      notOwnedCategories: null,
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
    noFilesSelected() {
      return this.checkedFiles.length < 1;
    },
  },
  methods: {
    formatBytes,
    changePath(item) {
      if (this.isCheckBox) {
        return;
      }
      this.selectedItem = item;
      this.$emit("path-changed", item.path);
    },
    async load() {
      this.$emit("loading", true);

      if (this.item) {
        this.selectedItem = this.item;
      }
      if (this.isDir) {
        const tempIsShown  = this.showTemporary === true;
        let url = this.endpoints.getTemporaryFolders.url
          .replace(new RegExp("{path}", "g"), this.path)
          .replace(new RegExp("{showTemporary}", "g"), tempIsShown);
        let config = {
          url,
          method: this.endpoints.getTemporaryFolders.method || "get"
        };
        let response = await this.axios.request(config);
        this.items = response.data;
      } else {
        // TODO: load file
      }
      this.$emit("loading", false);
      this.checkedFiles = [];
    },
    async deleteItem(item) {
      let confirmed = await this.$refs.confirm.open(
        "Delete",
        `Are you sure<br>you want to delete this ${
          item.type === "dir" ? "folder" : "file"
        }?<br><em>${item.basename}</em>`
      );
      if (confirmed) {
        this.$emit("loading", true);
        if (item.type !== "dir") {
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
        } else {
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

      let config = { url, method: this.endpoints.download.method || "get" };
      var items = await this.axios.request(config);

      const url2 = window.URL.createObjectURL(
        new Blob([this.base64ToArrayBuffer(items.data)])
      );
      const link = document.createElement("a");
      link.href = url2;
      link.setAttribute("download", item.name); //or any other extension
      link.setAttribute("id", "link1"); //or any other extension
      document.body.appendChild(link);
      link.click();
      var element = document.getElementById("link1");
      element.parentNode.removeChild(element);
    },
    async shareItem(item) {
      this.dialog = true;
      this.selectedItem = item;
    },
    async tagsAndCategoriesItem(item) {
      var response = await this.axios.request({
        url: this.endpoints.getcategoriesAndTags.url.replace(
          new RegExp("Id", "g"),
          item.Id
        ),
        method: this.endpoints.getcategoriesAndTags.method || "get",
      });
      this.startingCategories = [];
      this.startingTags = [];
      var items = response.data;
      this.ownedTags = items.Item1.Item1;
      this.notOwnedTags = items.Item1.Item2;
      this.ownedCategories = items.Item2.Item1;
      this.notOwnedCategories = items.Item2.Item2;
      this.selectedItem = item;
      for (const tag of this.ownedTags) {
        this.startingTags.push(tag);
      }
      for (const category of this.ownedCategories) {
        this.startingCategories.push(category);
      }
      this.dialogTagsAndcategories = true;
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
    },
    deleteItems() {
      this.deleteItemsConfirmDialog = true;
    },
    async ckeckedFilesDeleted(deletedArray) {
      this.$emit("loading", true);
      this.axios({
        method: this.endpoints.deleteFiles.method || "post",
        url: this.endpoints.deleteFiles.url,
        data: deletedArray.map((x) => x.Id),
      }).then(() => {
        this.$emit("file-deleted");
        this.$emit("loading", false);
      });
    },

    async checkedFilesMoved(moveArray, folderId) {
      this.$emit("loading", true);
      this.axios({
        method: this.endpoints.moveFiles.method || "put",
        url: this.endpoints.moveFiles.url,
        data: {
          FileIds: moveArray.map((x) => x.Id),
          FolderId: folderId,
        },
      }).then((response)=>{
        this.$emit("file-deleted");
        this.$emit("loading", false);
        if (response.data !== false){
          let newPath = "/";
          if (response.data != "") {
            newPath += response.data + "/";
          }
          for (const file of moveArray) {
            file.path = newPath + file.name;
          }
        }
      });
    },
    async checkedFilesCopied(copiedArray, folderId) {
      this.$emit("loading", true);
      this.axios({
        method: this.endpoints.copyFiles.method || "post",
        url: this.endpoints.copyFiles.url,
        data: {
          FileIds: copiedArray.map((x) => x.Id),
          FolderId: folderId,
        },
      }).then((response)=>{
        this.$emit("file-deleted");
        this.$emit("loading", false);
        if (response.data !== false){
          let newPath = "/";
          if (response.data != "") {
            newPath += response.data + "/";
          }
          for (const file of copiedArray) {
            file.path = newPath + file.name;
          }
        }
      });
    },
    async moveItems() {
      console.log("helllooooooo");
      await this.getAllFolders();
      this.moveItemsConfirmDialog = true;
    },
    async copyItems() {
      await this.getAllFolders();
      this.copyItemsConfirmDialog = true;
    },
    folderReloadDone() {
      this.foldersNeedUpdate = false;
    },
    async getAllFolders() {
      console.log("starting folder path: ", this.startingFolderPath);
      let response  = await this.axios.post(
        this.endpoints.getAllFoldersAsTree.url,
        {
          Path: this.startingFolderPath
        }
      );
      this.allFoldersAsTree = response.data.tree;
      this.allFoldersAsList = response.data.list;
      
      // let url = this.endpoints.getAllFoldersAsTree.url;
      // let config = {
      //   url,
      //   method: this.endpoints.getAllFoldersAsTree.method || "get",
      // };
      // let response = await this.axios.request(config);
      // this.allFoldersAsTree = response.data.tree;
      // this.allFoldersAsList = response.data.list;
    },
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