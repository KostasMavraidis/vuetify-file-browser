<template>
  <v-container class="py-0">
    <v-card color="grey lighten-2" v-if="item.Owner">
      <v-card-text class="pa-2 font-weight-bold d-flex align-center"
        ><v-icon class="pr-2"> mdi-account-circle </v-icon> Owner</v-card-text
      >
      <v-divider />
      <v-sheet>
        <v-row>
          <v-col align-self="center">
            Username:
            {{ item.Owner.UserName ? item.Owner.UserName : "n/a" }}
          </v-col>
        </v-row>
      </v-sheet>
    </v-card>
    <v-card color="grey lighten-2">
      <v-card-text class="pa-2 font-weight-bold d-flex align-center"
        ><v-icon class="pr-2"> mdi-file-eye-outline </v-icon> File
        Details</v-card-text
      >
      <v-divider />
      <v-sheet>
        <v-row v-if="item.ExtensionType">
          <v-col class="py-0"
            ><v-text-field :rules="rules" label="Name" v-model="fileName"
          /></v-col>
          <v-divider vertical />
          <v-col align-self="center">
            Size: {{ formatBytes(item.FileSize) }}
          </v-col>
          <v-divider vertical />
          <v-col align-self="center">
            Type: {{ item.ExtensionType.FileType }}
          </v-col>
        </v-row>
        <v-row class="py-0">
          <v-col align-self="center">
            Created:
            {{ item.DateCreated ? getDate(item.DateCreated) : "n/a" }}
          </v-col>
          <v-divider vertical />
          <v-col align-self="center">
            Updated:
            {{ item.LastUpdated ? getDate(item.LastUpdated) : "n/a" }}
          </v-col>
          <v-divider vertical />
          <v-col class="py-0">
            <v-switch
              v-model="item.IsPublic"
              label="Is Public"
              color="blue"
              dense
            ></v-switch>
          </v-col>
        </v-row>
      </v-sheet>
    </v-card>
    <v-card v-if="categories || tags" color="grey lighten-2">
      <v-card-text class="pa-2 font-weight-bold">
        <v-row>
          <v-col class="pa-0"  align-self="center">
        <v-icon class="pr-2"> mdi-tag </v-icon>
        Tags and Categories
          </v-col>
          <v-col class="d-flex justify-end pa-0"  align-self="center">
        <v-tooltip top>
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              v-bind="attrs"
              v-on="on" 
              small
              @click="dialog = true"
            >
              Add
            </v-btn>
          </template>
          <span> Select Tags and Categories</span>
        </v-tooltip>
          </v-col>
        </v-row>
      </v-card-text>
      <v-divider />
      <v-sheet>
        <v-card-text class="text-decoration-underline py-2">
          File Categories</v-card-text
        >
        <v-row class="py-1">
          <v-chip
            class="ma-2"
            v-for="category in selectedCategories"
            :key="category.Id"
            small
          >
            {{ category.Name }}
          </v-chip>
        </v-row>
        <v-divider />
        <v-card-text class="text-decoration-underline py-2">
          File Tags</v-card-text
        >
        <v-row class="py-1">
          <v-chip class="ma-2" v-for="tag in selectedTags" :key="tag.Id" small>
            {{ tag.Name }}
          </v-chip>
        </v-row>
      </v-sheet>
    </v-card>
    <v-card
      v-if="selectedGroups && selectedGroups.length > 0"
      color="grey lighten-2"
    >
      <v-card-text class="pa-2 font-weight-bold"
        ><v-icon class="pr-2"> mdi-account-group </v-icon> Groups</v-card-text
      >
      <v-divider />
      <v-sheet>
        <v-row class="py-1">
          <v-chip
            class="ma-2"
            v-for="group in selectedGroups"
            :key="group.Id"
            small
          >
            {{ group.Name }}
          </v-chip>
        </v-row>
      </v-sheet>
    </v-card>

    <v-dialog v-model="dialog" persistent no-click-animation>
      <tags-and-categories
        :fileDetails="true"
        @closeDialogTagsAndCategories="(e) => (dialog = e)"
        :axios="axios"
        :file.sync="item"
        :endpoints="endpoints"
        :ownedTags="selectedTags"
        :notOwnedTags="filteredTags"
        :ownedCategories="selectedCategories"
        :notOwnedCategories="filteredCategories"
      ></tags-and-categories>
    </v-dialog>
    <v-btn class="float-right" @click="putFile()">Update File</v-btn>
  </v-container>
</template>

<script>
import TagsAndCategories from "./TagsAndCategories.vue";
import { formatBytes } from "./util";

export default {
  components: {
    TagsAndCategories,
  },
  model: {},
  props: {
    file: Object,
    endpoints: Object,
    axios: Function,
  },
  data() {
    return {
      item: [],
      dialog: false,
      categories: null,
      tags: null,
      groups: null,
      selectedCategories: null,
      selectedTags: null,
      selectedGroups: null,
      fileName: "",
      rules: [(v) => !!v || "This field is required"],
    };
  },
  computed: {
    filteredTags() {
      if (this.tags && this.selectedTags) {
        const ids = this.selectedTags.map((x) => x.Id);
        return this.tags.filter((x) => !ids.includes(x.Id));
      } else {
        return [];
      }
    },
    filteredCategories() {
      if (this.categories && this.selectedCategories) {
        const ids = this.selectedCategories.map((x) => x.Id);
        return this.categories.filter((x) => !ids.includes(x.Id));
      } else {
        return [];
      }
    },
  },
  watch: {
    file() {
      if (this.file && this.file.Id !== this.item.Id) {
        this.getFile();
        console.log(this.item);
      }
    },
    selectedGroups() {
      if (this.groups) {
        this.item.Groups = this.groups.filter((x) =>
          this.selectedGroups.includes(x.Id)
        );
      }
    },
    fileName() {
      if (this.fileName) {
        this.item.Name =
          this.fileName + "." + this.item.ExtensionType.FileExtension;
      }
    },
  },
  methods: {
    show() {
      const ids = this.selectedTags.map((x) => x.Id);
      console.log(ids);
      const arr = this.tags.filter((x) => !ids.includes(x.Id));
      console.log(arr);
    },
    formatBytes,
    getDate(date) {
      return date.substring(0, date.lastIndexOf("T"));
    },
    async getFile() {
      let url = this.endpoints.file.url
        .replace(new RegExp("{storage}", "g"), "main")
        .replace(new RegExp("{id}", "g"), this.file.Id);

      let config = {
        url,
        method: this.endpoints.file.method || "get",
      };

      this.item = (await this.axios.request(config)).data;
      this.selectedCategories = this.item.Categories;
      this.selectedTags = this.item.Tags;
      this.selectedGroups = this.item.Groups.map((x) => x.Id);
      this.fileName = this.item.Name.substring(
        0,
        this.item.Name.lastIndexOf(".")
      );
    },
    async putFile() {
      let url = this.endpoints.file.url
        .replace(new RegExp("{storage}", "g"), "main")
        .replace(new RegExp("{id}", "g"), this.file.Id);

      let config = {
        url,
        method: "put",
        data: this.item,
      };

      let response = await this.axios.request(config);
      this.item = response.data;

      this.initialize();
    },
    async getCategories() {
      this.categories = (await this.axios.get("categories")).data;
    },
    async getTags() {
      this.tags = (await this.axios.get("tags")).data;
    },
    async getGroups() {
      this.groups = (await this.axios.get("groups")).data;
    },
    async initialize() {
      await this.getCategories();
      await this.getTags();
      await this.getGroups();
    },
  },
  mounted() {
    this.initialize();
    this.getFile();
  },
};
</script>

<style scoped>
.row {
  margin: 0px;
}

.v-card {
  border-radius: 0px;
  margin-bottom: 5px;
}

.col {
  padding-left: 16px;
}
</style>